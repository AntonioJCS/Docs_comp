## 📘 Visão Técnica da Solução

Esta documentação descreve a implementação técnica da análise de transbordo EX3 → ONE, detalhando:

* estrutura de dados esperada
* regras de transformação
* exemplos de consultas SQL (Athena/Trino)
* modelo da tabela de elegibilidade
* lógica de classificação e enriquecimento

O objetivo é permitir construção, manutenção e evolução da pipeline com rastreabilidade e clareza.

---

## 🧱 Modelo de Dados (Visão Lógica)

A solução parte de três fontes principais:

### 1. EX3 (origem do transbordo)

| campo            | descrição                           |
| ---------------- | ----------------------------------- |
| pasta            | identificador transversal           |
| dt_transbordo    | timestamp do envio para ONE         |
| flag_tratado_ex3 | indica se foi tratado integralmente |

---

### 2. ONE (eventos operacionais)

| campo                    | descrição                 |
| ------------------------ | ------------------------- |
| pasta                    | identificador transversal |
| dt_evento                | timestamp do evento       |
| tipo_solicitacao         | categoria da ação         |
| tipo_baixa               | resultado/baixa           |
| status / fase (opcional) | etapa do processo         |

---

### 3. Tabela de Elegibilidade

| campo            | descrição              |
| ---------------- | ---------------------- |
| tipo_solicitacao | chave de análise       |
| tipo_baixa       | chave de análise       |
| elegivel_ex3     | TRUE / FALSE / NULL    |
| regra_negocio    | identificador da regra |
| descricao_regra  | explicação             |

---

## 🗂️ Exemplo — Tabela de Elegibilidade

```sql
CREATE TABLE mapa_elegibilidade_ex3 (
    tipo_solicitacao VARCHAR,
    tipo_baixa VARCHAR,
    elegivel_ex3 BOOLEAN,
    regra_negocio VARCHAR,
    descricao_regra VARCHAR
);
```

### Exemplo de carga

```sql
INSERT INTO mapa_elegibilidade_ex3 VALUES
('SOLIC_PADRAO', 'BAIXA_OK', TRUE, 'R1', 'Tratamento padrão coberto pelo EX3'),
('DOC_INSUFICIENTE', 'BAIXA_PENDENTE', FALSE, 'R2', 'Necessita análise manual fora do EX3'),
('EXCECAO_PROCESSUAL', 'BAIXA_EXCECAO', FALSE, 'R3', 'Fluxo fora do escopo do EX3');
```

Observação: itens não mapeados permanecem como NULL após join.

---

## ⏱️ Definição das Janelas

### t₀ — início (transbordo)

```sql
WITH transbordo AS (
    SELECT
        pasta,
        dt_transbordo AS t0,
        flag_tratado_ex3
    FROM ex3_transbordo
)
```

---

### t₁ — fim (encerramento do subsídio)

Exemplo baseado em tipo de baixa:

```sql
, fim_subsidio AS (
    SELECT
        pasta,
        MIN(dt_evento) AS t1
    FROM one_eventos
    WHERE tipo_baixa IN ('BAIXA_SUBSIDIO_CONCLUIDO', 'BAIXA_ENVIO_PROX_ETAPA')
    GROUP BY pasta
)
```

---

## 🔍 Seleção da Janela de Eventos

```sql
, eventos_filtrados AS (
    SELECT
        e.pasta,
        e.dt_evento,
        e.tipo_solicitacao,
        e.tipo_baixa
    FROM one_eventos e
    JOIN transbordo t ON e.pasta = t.pasta
    JOIN fim_subsidio f ON e.pasta = f.pasta
    WHERE e.dt_evento BETWEEN t.t0 AND f.t1
)
```

---

## 🔗 Enriquecimento com Elegibilidade

```sql
, eventos_enriquecidos AS (
    SELECT
        e.*,
        m.elegivel_ex3,
        m.regra_negocio
    FROM eventos_filtrados e
    LEFT JOIN mapa_elegibilidade_ex3 m
        ON e.tipo_solicitacao = m.tipo_solicitacao
       AND e.tipo_baixa = m.tipo_baixa
)
```

---

## 📊 Agregação por Pasta

```sql
, agregados AS (
    SELECT
        pasta,

        COUNT(*) AS qtd_eventos,

        COUNT_IF(elegivel_ex3 = TRUE) AS qtd_elegiveis,
        COUNT_IF(elegivel_ex3 = FALSE) AS qtd_nao_elegiveis,
        COUNT_IF(elegivel_ex3 IS NULL) AS qtd_nao_mapeados,

        ARRAY_AGG(
            CONCAT(tipo_solicitacao, '|', tipo_baixa, '|',
                   COALESCE(CAST(elegivel_ex3 AS VARCHAR), 'NAO_MAPEADO'))
        ) AS evidencias

    FROM eventos_enriquecidos
    GROUP BY pasta
)
```

---

## ⚖️ Classificação Final

```sql
SELECT
    a.pasta,
    t.flag_tratado_ex3,

    CASE
        WHEN t.flag_tratado_ex3 = TRUE THEN 'DEVIDO'

        WHEN a.qtd_nao_elegiveis > 0 THEN 'DEVIDO'

        WHEN a.qtd_nao_mapeados > 0 THEN 'INCONCLUSIVO'

        WHEN a.qtd_elegiveis = a.qtd_eventos THEN 'INDEVIDO'

        ELSE 'INCONCLUSIVO'
    END AS classificacao,

    a.qtd_eventos,
    a.qtd_elegiveis,
    a.qtd_nao_elegiveis,
    a.qtd_nao_mapeados,

    a.evidencias

FROM agregados a
JOIN transbordo t ON a.pasta = t.pasta;
```

---

## 🧾 Justificativa Resumida (Opcional)

```sql
SELECT
    pasta,
    classificacao,

    CASE
        WHEN classificacao = 'DEVIDO' AND qtd_nao_elegiveis > 0
            THEN CONCAT('Devido: ', qtd_nao_elegiveis, ' evento(s) fora do escopo EX3')

        WHEN classificacao = 'INDEVIDO'
            THEN CONCAT('Indevido: todos os ', qtd_eventos, ' eventos são elegíveis ao EX3')

        WHEN classificacao = 'INCONCLUSIVO'
            THEN CONCAT('Inconclusivo: ', qtd_nao_mapeados, ' evento(s) não mapeados')

        ELSE 'Sem justificativa padrão'
    END AS justificativa

FROM resultado_final;
```

---

## 🧱 Estrutura Final da Tabela Analítica

| campo             | descrição                         |
| ----------------- | --------------------------------- |
| pasta             | identificador                     |
| dt_transbordo     | início da janela                  |
| dt_fim_subsidio   | fim da janela                     |
| classificacao     | devido / indevido / inconclusivo  |
| qtd_eventos       | volume analisado                  |
| qtd_elegiveis     | eventos tratáveis no EX3          |
| qtd_nao_elegiveis | eventos fora do escopo            |
| qtd_nao_mapeados  | eventos sem regra                 |
| evidencias        | array com eventos + classificação |
| justificativa     | texto resumido                    |

---

## ⚠️ Pontos Técnicos Críticos

### 1. Determinação correta do t₁

* Deve ser validada com negócio
* Evitar heurísticas sempre que possível

---

### 2. Integridade da tabela de elegibilidade

* Fonte de verdade da análise
* Deve ser versionada
* Pode evoluir incrementalmente

---

### 3. Volume e performance

* Tabela de eventos pode ser grande
* Recomendado:

  * particionamento por data (`dt_evento`)
  * filtros iniciais por período

---

### 4. Duplicidade de eventos

* Avaliar necessidade de deduplicação:

```sql
ROW_NUMBER() OVER (PARTITION BY pasta, tipo_solicitacao, tipo_baixa ORDER BY dt_evento)
```

---

### 5. Ordem dos eventos (opcional)

Para análises futuras:

```sql
ROW_NUMBER() OVER (PARTITION BY pasta ORDER BY dt_evento)
```

---

## 🔄 Evoluções Técnicas Possíveis

* Criar dimensão de eventos normalizada
* Separar regras por tipo (solicitação vs baixa)
* Introduzir pesos por evento
* Criar camada SPEC (Gold) com snapshot diário
* Expor via QuickSight com drill-down

---

## 🧪 Estratégia de Validação

Antes de produção:

* validar amostra manual com negócio
* testar casos conhecidos (devido / indevido)
* medir % de inconclusivo
* revisar top eventos não mapeados

---

## ✅ Conclusão Técnica

A solução se baseia em:

* recorte temporal preciso (t₀ → t₁)
* leitura do comportamento no ONE
* aplicação de regra de elegibilidade
* agregação por pasta
* classificação determinística

Essa arquitetura é:

* escalável
* auditável
* incremental (permite melhoria contínua)
* aderente ao modelo analítico em Athena/Trino

Se quiser, o próximo passo pode ser transformar isso em:

* modelo físico (Glue + particionamento S3)
* ou pipeline orquestrada (Step Functions + Athena)
