# **ATIVIDADE INTEGRADORA - RELATÓRIO OPERACIONAL DE PRÉ-DECOLAGEM - FASE 01**
This repository is from the 1st phase of my bacharel degree course on Computer Science from FIAP University.

# Projeto de Telemetria Sintética para Tomada de Decisão de Lançamento na Missão Aurora Siger

## Visão geral

Este projeto tem como objetivo simular um sistema de **telemetria aeroespacial** com dados sintéticos em formato CSV e, a partir desses dados, realizar análises exploratórias, energéticas e de apoio à tomada de decisão.

O conjunto de dados foi construído para representar leituras sequenciais de sensores e estados críticos do sistema, permitindo classificar cada instante operacional em duas saídas possíveis:

- **READY**: sistema apto para lançamento;
- **ABORT**: sistema não apto para lançamento devido a falhas críticas ou condições inseguras.

O projeto foi utilizado para desenvolver competências em:

- leitura e manipulação de dados com **Pandas**;
- visualização de dados com **Matplotlib**;
- construção de regras de decisão com `if`, `elif` e `else`;
- interpretação de indicadores energéticos e operacionais;
- reflexão crítica sobre ética, impacto social da exploração espacial e sustentabilidade tecnológica.

---

## Objetivos do projeto

1. Gerar um dataset sintético de telemetria com múltiplas variáveis operacionais;
2. Classificar os registros em **READY** e **ABORT**;
3. Identificar anomalias em temperatura, pressão, energia e comunicação;
4. Estimar a autonomia inicial do sistema com base na energia disponível e no consumo;
5. Visualizar os dados em gráficos para facilitar a análise comparativa;
6. Apoiar a tomada de decisão de lançamento com uma lógica baseada em condições seguras.

---

## Estrutura dos dados

O dataset contém, entre outras, as seguintes colunas:

- `timestamp`
- `internal_temp_c`
- `external_temp_c`
- `battery_voltage_v`
- `battery_current_a`
- `battery_soc_percent`
- `battery_capacity_ah`
- `energy_available_kwh`
- `power_load_kw`
- `energy_loss_percent`
- `tank_pressure_bar`
- `structural_integrity`
- `critical_modules_status`
- `telemetry_link_status`
- `estimated_autonomy_min`
- `launch_decision`

Além disso, durante a análise, foram criadas colunas derivadas, como:

- `full_capacity_kwh`
- `current_energy_kwh`
- `useful_energy_kwh`
- `initial_autonomy_h`
- `initial_autonomy_m`
- `new_launch_decision`

---

## Lógica de decisão

A lógica de decisão utiliza regras simples para verificar se o sistema está em condição segura de operação. O algoritmo marca o registro como **ABORT** quando encontra qualquer uma das seguintes situações:

- falha na integridade estrutural;
- falha nos módulos críticos;
- falha no link de telemetria;
- nível de bateria abaixo do limite mínimo;
- pressão do tanque fora da faixa segura;
- temperatura interna fora da faixa segura;
- temperatura externa fora da faixa segura.

Caso nenhuma dessas condições seja identificada, o sistema é classificado como **READY TO LAUNCH**.

---

## Análise energética

A análise energética foi realizada para estimar a autonomia inicial do sistema considerando:

- capacidade total da bateria;
- percentual de carga disponível;
- perdas energéticas;
- potência consumida pelo sistema.

As fórmulas aplicadas foram:

```python
# Capacidade total em kWh
full_capacity_kwh = (battery_voltage_v * battery_capacity_ah) / 1000

# Energia atual considerando o estado de carga da bateria (SOC)
current_energy_kwh = full_capacity_kwh * (battery_soc_percent / 100)

# Energia útil após considerar perdas
useful_energy_kwh = current_energy_kwh * (1 - energy_loss_percent / 100)

# Autonomia inicial
initial_autonomy_h = useful_energy_kwh / power_load_kw
initial_autonomy_m = initial_autonomy_h * 60
```

Essa análise permitiu verificar a relação entre **carga da bateria**, **energia útil**, **consumo** e **autonomia**.

---

## Prints da execução

### 1. Comparação geral entre READY e ABORT

O gráfico abaixo mostra a comparação entre variáveis operacionais e estados binários, permitindo identificar quais fatores se aproximam mais dos casos de `READY` e `ABORT`.

![Visão geral da telemetria](./Screenshot%202026-03-30%20at%2018.16.38.png)

### 2. Dashboard de análise energética

O painel abaixo resume a relação entre energia útil, consumo, perdas energéticas, autonomia e a comparação entre `READY` e `ABORT`.

![Dashboard de análise energética](./Screenshot%202026-03-31%20at%2011.46.24.png)

---

## Instruções de execução do código

### 1. Pré-requisitos

Certifique-se de ter instalado:

- Python 3.x
- Pandas
- Matplotlib
- Jupyter Notebook ou JupyterLab (opcional, mas recomendado)

Instalação das bibliotecas:

```bash
pip install pandas matplotlib
```

---

### 2. Arquivos necessários

Mantenha os seguintes arquivos na mesma pasta do projeto:

- `telemetria_sintetica_500_linhas_abort.csv`
- `README.md`

---

### 3. Leitura do arquivo CSV

```python
import pandas as pd

# Ler o dataset com casos de ABORT
df = pd.read_csv("telemetria_sintetica_500_linhas_abort.csv")

print(df.head())
```

---

### 4. Aplicação da função de decisão

```python
def launch_decision(row):
    if row["structural_integrity"] == 0:
        return "ABORT - Structural Failure"
    elif row["critical_modules_status"] == 0:
        return "ABORT - Critical Modules Failure"
    elif row["telemetry_link_status"] == 0:
        return "ABORT - Telemetry Link Failure"
    elif row["battery_soc_percent"] < 60:
        return "ABORT - Low Battery"
    elif row["tank_pressure_bar"] < 95 or row["tank_pressure_bar"] > 145:
        return "ABORT - Unsafe Tank Pressure"
    elif row["internal_temp_c"] < 18 or row["internal_temp_c"] > 35:
        return "ABORT - Unsafe Internal Temperature"
    elif row["external_temp_c"] < -5 or row["external_temp_c"] > 30:
        return "ABORT - Unsafe External Temperature"
    else:
        return "READY TO LAUNCH"

df["new_launch_decision"] = df.apply(launch_decision, axis=1)
```

---

### 5. Análise energética

```python
df["full_capacity_kwh"] = (df["battery_voltage_v"] * df["battery_capacity_ah"]) / 1000
df["current_energy_kwh"] = df["full_capacity_kwh"] * (df["battery_soc_percent"] / 100)
df["useful_energy_kwh"] = df["current_energy_kwh"] * (1 - df["energy_loss_percent"] / 100)
df["initial_autonomy_h"] = df["useful_energy_kwh"] / df["power_load_kw"]
df["initial_autonomy_m"] = df["initial_autonomy_h"] * 60

print(df[[
    "timestamp",
    "full_capacity_kwh",
    "battery_soc_percent",
    "power_load_kw",
    "energy_loss_percent",
    "useful_energy_kwh",
    "initial_autonomy_m"
]].head())
```

---

### 6. Exemplo de gráfico simples

```python
import matplotlib.pyplot as plt

counts = df["launch_decision"].value_counts().reindex(["READY", "ABORT"])
counts.plot(kind="bar")
plt.title("READY vs ABORT")
plt.xlabel("Launch Decision")
plt.ylabel("Count")
plt.xticks(rotation=0)
plt.show()
```

---

## Resultados esperados

Ao executar o projeto, espera-se:

- visualizar a diferença entre casos `READY` e `ABORT`;
- identificar falhas críticas que levam ao aborto da missão;
- compreender como a energia útil e o consumo afetam a autonomia;
- interpretar graficamente as relações entre variáveis operacionais e energéticas.

---

## Considerações finais

Este projeto demonstra como dados sintéticos podem ser utilizados para simular cenários de telemetria e apoiar decisões operacionais. Além do desenvolvimento técnico em Python, Pandas e Matplotlib, o trabalho também contribui para discussões mais amplas sobre confiabilidade, segurança, responsabilidade tecnológica e análise crítica em contextos ligados à exploração espacial.