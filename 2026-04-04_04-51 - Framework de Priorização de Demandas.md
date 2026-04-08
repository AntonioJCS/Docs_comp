
## 🎯 Dores principais vs soluções

### ⚠️ Falta de critério claro de priorização

Demandas disputam atenção sem regra objetiva, gerando conflito com stakeholders.  
**Solução:** score de prioridade (Valor + Urgência + Criticidade / Esforço)

---

### ⚠️ Sensação de “injustiça” dos stakeholders

Cada solicitante acha sua demanda mais importante.  
**Solução:** transparência com ranking do backlog + explicação baseada no score

---

### ⚠️ Capacidade invisível

Você assume mais trabalho do que consegue entregar.  
**Solução:** explicitar capacidade (ex: horas semanais) e planejar dentro desse limite

---

### ⚠️ Backlog desorganizado

Demandas sem padrão dificultam comparação e decisão.  
**Solução:** estrutura única com campos obrigatórios (esforço, valor, urgência, status)

---

### ⚠️ Falta de previsibilidade

Não consegue dizer quando algo será entregue.  
**Solução:** sequenciamento por prioridade + capacidade → previsão de início/fim

---

### ⚠️ Interrupções constantes (urgências)

Demandas novas quebram o planejamento.  
**Solução:** reservar capacidade fixa para urgências (ex: 20–30%)

---

### ⚠️ Acompanhamento fraco de progresso

Stakeholder não sabe em que pé está a demanda.  
**Solução:** progresso simples (% ou horas consumidas vs planejadas)

---

### ⚠️ Comunicação reativa

Você só responde quando cobrado.  
**Solução:** ritual semanal com status, próximos passos e backlog

---

### ⚠️ Muitas demandas em paralelo

Baixa eficiência e alto tempo de entrega.  
**Solução:** limitar WIP (trabalhos simultâneos)

---

### ⚠️ Discussões subjetivas e desgastantes

Decisões viram opinião, não dado.  
**Solução:** transformar tudo em regra objetiva (score + capacidade)

---

## 📌 Síntese

- Organizar → backlog estruturado
- Priorizar → score objetivo
- Limitar → capacidade explícita
- Comunicar → rotina + transparência

---


Essas são algumas das ferramentas de priorização que achei interessante:
- Matriz / score GUT
- Matriz / score de risco
- Método Eisenhower
- Método Rice
- Matriz de Esforço x impacto

Para escopo de projetos:
- Metodo Moscow


---

# Framework Unificado de Priorização: Problemas, Riscos e Oportunidades

**Framework de Priorização de Demandas**, unificando a gestão de problemas reais e riscos potenciais em um sistema quantitativo e objetivo.

Este documento define o sistema de diagnóstico e o cálculo do **Normalized Aggressive Score (NAS)** para a governança de demandas técnicas e institucionais.

## 1. As 3 Categorias de Demanda (A Natureza)

Para cada entrada no sistema, deve-se identificar a origem do gatilho:

1.  **Problema Real (Incidente/Crise):** Fatos que prejudicam a operação atual. 
    * *Exemplo:* Migração de tecnologia legado que será descontinuada (o prazo de suporte tem data final fixa).
2.  **Problema Potencial (Risco):** Hipóteses fundamentadas de falhas futuras.
    * *Exemplo:* Adoção de uma nova arquitetura alvo para evitar que a instituição se torne obsoleta tecnologicamente no próximo ano.
3.  **Oportunidade (Melhoria/Evolução):** Sistemas saudáveis, mas que possuem potencial de otimização.
    * *Exemplo:* Refatoração de código para reduzir custos de nuvem e aumentar a velocidade de resposta, sem que haja uma falha atual.

---

## 2. Dimensões de Análise (Atributos de Severidade)

Independentemente da natureza, toda demanda é avaliada por quatro vetores:

* **Impacto (I):** Extensão do benefício ou do dano (financeiro, operacional, reputacional, segurança).
* **Gravidade / Valor (G):** No caso de problemas, é a intensidade da dor. No caso de melhorias, é o valor agregado pela mudança.
* **Tendência (T):** Se não agirmos, a situação piora (problema) ou a oportunidade se perde (melhoria)?
* **Tempestividade (U):** O quão urgente é a resposta para garantir o resultado ou evitar o caos.



---

## 3. O Fator de Ocorrência e Viabilidade

Para equilibrar o score, aplicamos uma métrica quantitativa baseada no tipo de demanda:

* **Problema Real:** Mede-se a **Frequência** (Ex: Quantas vezes o sistema legado falha por semana?).
* **Problema Potencial:** Mede-se a **Probabilidade** (Ex: Qual a chance da nova arquitetura ser exigida pelo mercado no próximo semestre?).
* **Melhoria:** Mede-se a **Eficiência/Ganho** (Ex: Qual o percentual de redução de custo ou ganho de milissegundos esperado?).

---

## 4. Normalized Aggressive Score (Cálculo)

A fórmula prioriza demandas com alto impacto sistêmico e evolução rápida (Tendência), utilizando uma escala logarítmica para o tempo.

$$Score = \frac{(G \times U \times T) \times (P \times I)}{1 + \log_{10}(\text{Horas até o Prazo} + 1)}$$

*Onde **P** representa Probabilidade, Frequência ou Ganho Estimado, conforme a categoria.*

---

## 5. Tabela de Ancoragem Quantitativa (Refatorada)

| Nível | Problema Real | Problema Potencial | Melhoria (Oportunidade) |
| :--- | :--- | :--- | :--- |
| **1 (Baixo)** | Erro estético; processo manual atende bem. | Risco improvável ou de longo prazo (> 2 anos). | Ganho marginal de performance (< 2%). |
| **3 (Médio)** | Falhas intermitentes; afeta produtividade interna. | Risco moderado; tecnologia atual perderá suporte em 1 ano. | Redução de custos entre 10% e 20% ou ganho perceptível de UX. |
| **5 (Alto)** | Operação parada; perda financeira direta e contínua. | Risco iminente; nova arquitetura é mandatória para sobrevivência. | Redução drástica de custos (> 40%) ou vantagem competitiva crítica. |

---

## 6. Exemplos de Aplicação do Framework

| **Categoria**          | **Cenário Exemplo**                                                                                | **GUT (G×U×T)** | **Risco/Ganho (P×I)** | **Prazo (Horas)** | **Score NAS (Final)** | **Comportamento do Score**                                                                                           |
| ---------------------- | -------------------------------------------------------------------------------------------------- | --------------- | --------------------- | ----------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Problema Real**      | **Migração de Tecnologia Legado:** Fornecedor encerrará suporte em 30 dias. Risco de parada total. | **125** (5x5x5) | **25** (5x5)          | 720h              | **810,0**             | **Crítico:** A combinação de gravidade máxima e risco total empurra a tarefa para o topo imediato.                   |
| **Problema Potencial** | **Nova Arquitetura Alvo:** Estudo de tecnologia para o próximo ano para evitar obsolescência.      | **27** (3x3x3)  | **16** (4x4)          | 8.760h            | **87,3**              | **Estratégico:** O impacto é alto, mas o prazo longo (denominador) mantém a demanda no radar sem sufocar o presente. |
| **Melhoria**           | **Otimização de Performance:** App estável, mas refatoração reduz custos de nuvem em 30%.          | **27** (3x3x3)  | **15** (5x3)          | 2.160h            | **93,4**              | **Eficiência:** O ganho garantido permite que esta melhoria supere problemas reais de baixa gravidade.               |
| **Problema Real**      | **Ajuste Estético:** Erro de digitação em relatório interno pouco utilizado.                       | **1** (1x1x1)   | **1** (1x1)           | 48h               | **0,37**              | **Irrelevante:** Mesmo com prazo curto (48h), o baixo impacto resulta em um score desprezível.                       |


---

## 7. Governança com IA
O formulário de entrada deve ser processado por um agente de IA que:
1.  **Classifica automaticamente** a demanda em uma das 3 categorias baseada na descrição.
2.  **Valida as notas** Avaliando o que foi descrito.
3.  **Sugere o Prazo Final (Deadline)** com base na criticidade da tecnologia citada (ex: detecta que "Tecnologia X" expira em Dezembro e calcula as horas restantes).