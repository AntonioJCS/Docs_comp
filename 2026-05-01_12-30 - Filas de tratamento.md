## Contexto da operação

Existe uma operação terceirizada responsável por tratar filas de um determinado processo. Essas filas são segmentadas por natureza, como criminal, trabalhista, cível e ofícios, entre outras. Para cada uma delas, há um SLA definido com o fornecedor.

Do ponto de vista de prazo, a regra é simples: toda solicitação deve ser tratada até D+1 (D1), ou seja, considerando o dia de entrada na fila como D0, o tratamento deve ocorrer até o dia seguinte para estar dentro do prazo. Qualquer item tratado após esse limite é considerado fora do SLA.

## Modelagem do tempo de tratamento

Para analisar o desempenho da operação, foi necessário estruturar os dados considerando três elementos principais:

* Data de início da tratativa
* Data de conclusão
* Tempo total de tratamento (lead time), calculado como a diferença entre início e fim

Esse lead time apresenta grande variabilidade. Existem casos resolvidos no mesmo dia (D0), enquanto outros permanecem na fila por períodos mais longos, ultrapassando diversos dias.

## Estratégia de análise

Dado esse comportamento heterogêneo, foi aplicada uma estratégia de agrupamento por faixas de tempo (ranges) para facilitar a leitura e interpretação dos dados. Os agrupamentos foram definidos da seguinte forma:

* D0 até D20 (faixas diárias)
* Acima de D20 (consolidado)

Com esses agrupamentos, é possível construir algumas visualizações de dados:
- Pareto (distribuição estática)
- Gráficos temporais (linha + barra empilhada)
- Tabela acumulada (pivot estilo Pareto contínuo)

---
## Regras de Negócio

### Metas de desempenho

| Esteira     | Segregações | Agrupamento | Meta |
| ----------- | ----------- | ----------: | ---: |
| Cível       | -           |          D1 |  90% |
| Cível       | -           |          D2 | 100% |
| Trabalhista | Com segredo |          D2 |  20% |
| Trabalhista | Com segredo |          D5 |  50% |
| Trabalhista | Com segredo |         D10 | 100% |
| Trabalhista | Sem segredo |          D1 |  90% |
| Trabalhista | Sem segredo |          D2 | 100% |

### Regras de agrupamento

| regra | agrupamento | range_min | range_max | ordem_visual |
| ----- | ----------- | --------: | --------: | -----------: |
| X     | D1          |         0 |         1 |            1 |
| X     | D2          |         2 |         2 |            2 |
| X     | D6-D10      |         6 |        10 |            6 |
| X     | D11-D15     |        11 |        15 |            7 |
| X     | D16-D20     |        16 |        20 |            8 |
| X     | >D20        |        21 |      NULL |            9 |
| Y     | D1          |         0 |         1 |            1 |
| Y     | D2          |         2 |         2 |            2 |
| Y     | D10         |        10 |        10 |           10 |
| Y     | >D10        |        11 |      NULL |           11 |

---

## Exemplos Principais

1. Curva de sobrevivência do lead time
2. Breakdown percentual do SLA
3. Aging operacional da fila atual
4. Driver chart de atraso
5. Heatmap acumulado
6. Tendência individual por bucket
7. Scatter lead time médio vs volume
8. Matriz de priorização operacional
9. Funil operacional
10. Waterfall de variação do SLA


```python
import numpy as np  
import pandas as pd  
import matplotlib.pyplot as plt  
from IPython.display import display  
  
# Dados exemplo coerentes com o contexto  
meses = ["Jan/26", "Fev/26", "Mar/26", "Abr/26", "Mai/26", "Jun/26"]  
df = pd.DataFrame({  
    "Mês": meses,  
    "D0": [600, 580, 620, 610, 590, 605],  
    "D1": [150, 160, 140, 155, 165, 150],  
    "D2": [120, 130, 110, 115, 125, 120],  
    "D3": [80, 90, 85, 80, 85, 90],  
    "D4+": [50, 60, 55, 40, 35, 15],  
})  
df["Total"] = df[["D0","D1","D2","D3","D4+"]].sum(axis=1)  
df["Dentro SLA"] = df["D0"] + df["D1"]  
df["Fora SLA"] = df["D2"] + df["D3"] + df["D4+"]  
df["% SLA"] = df["Dentro SLA"] / df["Total"] * 100  
df["% Fora SLA"] = df["Fora SLA"] / df["Total"] * 100  
  
# 1. Curva de sobrevivência do lead time  
buckets = ["D0", "D1", "D2", "D3", "D4+"]  
vol_jun = df.loc[df["Mês"] == "Jun/26", buckets].iloc[0]  
total_jun = vol_jun.sum()  
resolvido_acum = vol_jun.cumsum() / total_jun * 100  
sobrevivencia = 100 - resolvido_acum  
sobrevivencia = np.insert(sobrevivencia.values, 0, 100)  
x_surv = ["Entrada", "Após D0", "Após D1", "Após D2", "Após D3", "Após D4+"]  
  
plt.figure(figsize=(9, 5))  
plt.plot(x_surv, sobrevivencia, marker="o")  
plt.axvline(x=2, linestyle="--")  
plt.title("Curva de Sobrevivência do Lead Time — Jun/26")  
plt.ylabel("% ainda não tratado")  
plt.xlabel("Marco temporal")  
plt.ylim(0, 105)  
for i, v in enumerate(sobrevivencia):  
    plt.text(i, v + 2, f"{v:.1f}%", ha="center")  
plt.grid(axis="y", alpha=0.3)  
plt.show()  
  
# 2. Breakdown do SLA por mês  
plot_df = df[["Mês", "D0", "D1", "D2", "D3", "D4+"]].copy()  
for col in buckets:  
    plot_df[col] = plot_df[col] / df["Total"] * 100  
  
bottom = np.zeros(len(plot_df))  
plt.figure(figsize=(10, 5))  
for col in buckets:  
    plt.bar(plot_df["Mês"], plot_df[col], bottom=bottom, label=col)  
    bottom += plot_df[col].values  
plt.axhline(100, linewidth=1)  
plt.axhline(75, linestyle="--")  
plt.title("Breakdown do SLA — Composição Percentual por Mês")  
plt.ylabel("% do volume")  
plt.xlabel("Mês")  
plt.legend(title="Bucket")  
plt.ylim(0, 105)  
plt.show()  
  
# 3. Aging operacional da fila atual  
aging_atual = pd.Series({  
    "D0": 240,  
    "D1": 180,  
    "D2": 90,  
    "D3": 55,  
    "D4-D7": 70,  
    "D8-D20": 35,  
    "D20+": 15  
})  
plt.figure(figsize=(9, 5))  
plt.bar(aging_atual.index, aging_atual.values)  
plt.axvline(x=1.5, linestyle="--")  
plt.title("Aging Operacional — Itens Atualmente em Aberto")  
plt.ylabel("Volume em aberto")  
plt.xlabel("Aging atual")  
for i, v in enumerate(aging_atual.values):  
    plt.text(i, v + 5, str(v), ha="center")  
plt.show()  
  
# 4. Driver chart: dentro x fora SLA com abertura do fora SLA  
driver_df = df[["Mês", "D0", "D1", "D2", "D3", "D4+"]].copy()  
driver_df["Dentro SLA"] = (driver_df["D0"] + driver_df["D1"]) / df["Total"] * 100  
driver_df["D2"] = driver_df["D2"] / df["Total"] * 100  
driver_df["D3"] = driver_df["D3"] / df["Total"] * 100  
driver_df["D4+"] = driver_df["D4+"] / df["Total"] * 100  
  
plt.figure(figsize=(10, 5))  
bottom = np.zeros(len(driver_df))  
for col in ["Dentro SLA", "D2", "D3", "D4+"]:  
    plt.bar(driver_df["Mês"], driver_df[col], bottom=bottom, label=col)  
    bottom += driver_df[col].values  
plt.title("Driver Chart — SLA vs Causadores de Atraso")  
plt.ylabel("% do volume")  
plt.xlabel("Mês")  
plt.ylim(0, 105)  
plt.legend()  
plt.show()  
  
# 5. Heatmap simples de percentual acumulado  
cum_df = df[buckets].div(df["Total"], axis=0).cumsum(axis=1).T * 100  
plt.figure(figsize=(9, 5))  
im = plt.imshow(cum_df.values, aspect="auto")  
plt.title("Heatmap Acumulado — Percentual Tratado até Cada Bucket")  
plt.xticks(range(len(meses)), meses)  
plt.yticks(range(len(buckets)), buckets)  
plt.xlabel("Mês")  
plt.ylabel("Bucket acumulado")  
plt.colorbar(im, label="% acumulado")  
for i in range(cum_df.shape[0]):  
    for j in range(cum_df.shape[1]):  
        plt.text(j, i, f"{cum_df.iloc[i, j]:.1f}%", ha="center", va="center")  
plt.show()  
  
# 6. Small multiples: tendência individual por bucket  
for col in buckets:  
    plt.figure(figsize=(8, 4))  
    pct = df[col] / df["Total"] * 100  
    plt.plot(df["Mês"], pct, marker="o")  
    plt.title(f"Tendência do Bucket {col} — % do Volume Mensal")  
    plt.ylabel("% do volume")  
    plt.xlabel("Mês")  
    plt.ylim(0, max(pct) + 10)  
    for i, v in enumerate(pct):  
        plt.text(i, v + 0.8, f"{v:.1f}%", ha="center")  
    plt.grid(axis="y", alpha=0.3)  
    plt.show()  
  
# 7. Scatter lead time médio x volume por natureza  
np.random.seed(42)  
naturezas = ["Cível", "Trabalhista", "Criminal", "Ofícios", "Canais críticos", "Bancário", "Consumidor"]  
volume = np.array([3600, 2100, 900, 1400, 700, 2800, 1600])  
lead_medio = np.array([1.4, 2.1, 1.1, 2.8, 3.6, 1.7, 2.4])  
sla_pct = np.array([78, 69, 84, 62, 55, 74, 66])  
  
plt.figure(figsize=(9, 5))  
sizes = volume / 8  
plt.scatter(lead_medio, volume, s=sizes, alpha=0.6)  
plt.axvline(1, linestyle="--")  
plt.title("Lead Time Médio vs Volume por Natureza")  
plt.xlabel("Lead time médio (dias)")  
plt.ylabel("Volume mensal")  
for i, n in enumerate(naturezas):  
    plt.text(lead_medio[i] + 0.03, volume[i] + 40, n)  
plt.grid(alpha=0.3)  
plt.show()  
  
# 8. Waterfall de variação de SLA  
start = df.loc[0, "% SLA"]  
end = df.loc[1, "% SLA"]  
impactos = pd.Series({  
    "Ganho D0": (df.loc[1, "D0"]/df.loc[1, "Total"] - df.loc[0, "D0"]/df.loc[0, "Total"]) * 100,  
    "Ganho D1": (df.loc[1, "D1"]/df.loc[1, "Total"] - df.loc[0, "D1"]/df.loc[0, "Total"]) * 100,  
    "Perda D2": -(df.loc[1, "D2"]/df.loc[1, "Total"] - df.loc[0, "D2"]/df.loc[0, "Total"]) * 100,  
    "Perda D3": -(df.loc[1, "D3"]/df.loc[1, "Total"] - df.loc[0, "D3"]/df.loc[0, "Total"]) * 100,  
    "Perda D4+": -(df.loc[1, "D4+"]/df.loc[1, "Total"] - df.loc[0, "D4+"]/df.loc[0, "Total"]) * 100,  
})  
labels = ["SLA Jan"] + list(impactos.index) + ["SLA Fev"]  
values = [start] + list(impactos.values) + [end]  
running = [0]  
for v in values[:-1]:  
    running.append(running[-1] + v)  
  
plt.figure(figsize=(11, 5))  
base = [0]  
cum = start  
bases = [0]  
heights = [start]  
for v in impactos.values:  
    bases.append(cum if v >= 0 else cum + v)  
    heights.append(abs(v))  
    cum += v  
bases.append(0)  
heights.append(end)  
  
plt.bar(labels, heights, bottom=bases)  
plt.title("Waterfall — Variação do SLA de Jan/26 para Fev/26")  
plt.ylabel("% SLA / impacto em p.p.")  
plt.xticks(rotation=30, ha="right")  
plt.axhline(start, linestyle="--", linewidth=1)  
plt.axhline(end, linestyle="--", linewidth=1)  
for i, (b, h, lab) in enumerate(zip(bases, heights, labels)):  
    txt = f"{values[i]:.1f}%" if i in [0, len(labels)-1] else f"{values[i]:+.1f} p.p."    plt.text(i, b + h + 0.5, txt, ha="center")  
plt.tight_layout()  
plt.show()  
  
# 9. Matriz de prioridade operacional: volume fora SLA x severidade média  
naturezas2 = ["Cível", "Trabalhista", "Criminal", "Ofícios", "Canais críticos", "Bancário", "Consumidor"]  
fora_sla = np.array([780, 650, 140, 530, 310, 720, 480])  
severidade = np.array([2.2, 2.9, 1.8, 3.4, 4.1, 2.5, 3.0])  
  
plt.figure(figsize=(9, 5))  
plt.scatter(fora_sla, severidade, s=fora_sla/2, alpha=0.6)  
plt.axvline(fora_sla.mean(), linestyle="--")  
plt.axhline(severidade.mean(), linestyle="--")  
plt.title("Matriz de Priorização — Volume Fora SLA vs Severidade")  
plt.xlabel("Volume fora do SLA")  
plt.ylabel("Severidade média do atraso")  
for i, n in enumerate(naturezas2):  
    plt.text(fora_sla[i] + 10, severidade[i] + 0.03, n)  
plt.grid(alpha=0.3)  
plt.show()  
  
# 10. Funnel operacional D0 -> D1 -> D2+  
funnel = pd.Series({  
    "Entradas": 980,  
    "Tratadas D0": 605,  
    "Tratadas D1": 150,  
    "Viraram atraso D2+": 225,  
    "Backlog crítico D4+": 15  
})  
plt.figure(figsize=(9, 5))  
plt.barh(funnel.index[::-1], funnel.values[::-1])  
plt.title("Funil Operacional do Prazo — Jun/26")  
plt.xlabel("Volume")  
for i, v in enumerate(funnel.values[::-1]):  
    plt.text(v + 10, i, str(v), va="center")  
plt.show()
```


## Curva de desempenho

```python
import matplotlib.pyplot as plt
import numpy as np

# eixo de dias
dias = np.arange(0, 11)

# curva esperada (alvo)
esperado = np.array([60, 90, 100] + [100]*8)[:len(dias)]

# curva real (exemplo com atraso)
real = np.array([55, 75, 92, 96, 98, 99, 100, 100, 100, 100, 100])

plt.figure(figsize=(9,5))
plt.plot(dias, esperado, marker='o', linestyle='--', label='Curva Esperada (Alvo)')
plt.plot(dias, real, marker='o', label='Curva Real')

plt.title('Curva Alvo de Desempenho (Lead Time Acumulado)')
plt.xlabel('Dias (D+X)')
plt.ylabel('% acumulado concluído')
plt.ylim(0,105)

# linhas de referência
plt.axvline(1, linestyle='--')  # D1
plt.axvline(2, linestyle='--')  # D2

# anotações
plt.text(1, 92, 'Meta: 90% até D1')
plt.text(2, 102, 'Meta: 100% até D2')

plt.legend()
plt.grid(alpha=0.3)

plt.show()
```




---

## Problema com agrupamentos

O problema com o agrupamento de lead time reside na **inconsistência semântica e estrutural** entre diferentes regras de negócio

| regra | agrupamento | range_min | range_max | ordem_visual |
| ----- | ----------- | --------: | --------: | -----------: |
| X     | D1          |         0 |         1 |            1 |
| X     | D2          |         2 |         2 |            2 |
| X     | D6-D10      |         6 |        10 |            6 |
| X     | D11-D15     |        11 |        15 |            7 |
| X     | D16-D20     |        16 |        20 |            8 |
| X     | >D20        |        21 |      NULL |            9 |
| Y     | D1          |         0 |         1 |            1 |
| Y     | D2          |         2 |         2 |            2 |
| Y     | D10         |        10 |        10 |           10 |
| Y     | >D10        |        11 |      NULL |           11 |


  ```
  D10 ≠ >D10
  D20 ≠ >D20
  D6 ≠ D6-D10
  ```
  
Os principais pontos de falha são:
*   **Incompatibilidade Semântica**: A mistura de valores pontuais (ex: D10), intervalos fechados (ex: D6-D10) e caudas abertas (ex: >D10) impede a comparação direta entre diferentes esteiras
*   **Índices Dependentes**: Como cada regra define sua própria ordem visual, o mesmo índice pode representar grupos diferentes, quebrando a ordenação lógica em visões consolidadas
*   **Distorção Analítica**: Essa fragmentação gera acúmulos incorretos e perda de significado em heatmaps ou curvas de distribuição quando se tenta consolidar os dados
  

Como solução, podemos adotar a estratégia de aplicar a lógica de negócio na camada de BI (como o QuickSight) em vez da fonte de dados (Athena) pode ser resumida em **preservar a granularidade para ganhar flexibilidade**

Aqui estão os pontos centrais:
*   **Dinamismo Semântico**: Você mantém o dado "atômico" (ex: lead time bruto em dias) na fonte, permitindo que as faixas de agrupamento sejam alteradas no BI sem necessidade de reprocessar o histórico de dados
*   **Consolidação por Eixo Comum**: Ao usar campos calculados para normalizar índices, você consegue comparar esteiras com metas diferentes (ex: uma com SLA de D2 e outra de D10) sob uma mesma métrica de performance
*   **Desacoplamento**: A camada de dados foca em **armazenamento e performance**, enquanto a camada de BI foca em **regras de negócio e interpretação**, facilitando a manutenção e a evolução das metas

Em suma: o dado entra como um **número** e a visualização o transforma em **informação estratégica** através de lentes dinâmicas
```sql
/* Campo Calculado: LeadTime_Bucket_Normalizado */
ifelse(
    {lead_time} <= 1, "D0-D1",
    {lead_time} <= 2, "D2",
    {lead_time} <= 5, "D3-D5",
    {lead_time} <= 10, "D6-D10",
    ">D10"
)
```