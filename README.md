# 🚲 Análise e Previsão de Aluguel de Bicicletas

Este projeto aplica conceitos de Ciência de Dados e Séries Temporais para analisar, limpar, tratar e modelar o comportamento de aluguel de bicicletas com base em variáveis temporais e climáticas. O objetivo principal é entender os padrões de demanda e prever o volume de aluguéis futuros para auxiliar na tomada de decisões estratégicas.

---

## 📌 Sumário
* [1. Identificação do Problema](#1-identificação-do-problema)
* [2. Preparação e Processamento dos Dados](#2-preparação-e-processamento-dos-dados)
* [3. Análise Exploratória de Dados (EDA)](#3-análise-exploratória-de-dados-eda)
* [4. Otimização do Modelo e Testes Estatísticos](#4-otimização-do-modelo-e-testes-estatísticos)
* [5. Modelagem Preditiva Avançada (Longo Prazo)](#5-modelagem-preditiva-avançada-longo-prazo)
* [6. Compartilhamento e Próximos Passos](#6-compartilhamento-e-próximos-passos)

---

## 1. Identificação do Problema

### O Objetivo Final
Identificar os fatores que mais influenciam o aluguel de bicicletas (como clima, sazonalidade e horários) e validar hipóteses estatísticas, construindo um modelo preditivo robusto capaz de antecipar a demanda futura.

### Dados Disponíveis
O conjunto de dados conta com informações temporais (anos de 2015 e 2016), registros de contagem de aluguéis, além de variáveis climáticas como temperatura, sensação térmica, umidade e condições do tempo.

---

## 2. Preparação e Processamento dos Dados

Para garantir a qualidade das informações antes da análise, o dataset passou por etapas de tratamento:

* **Dados Nulos:** Tratados de forma inteligente utilizando a técnica de interpolação linear com o método `.interpolate()`.
* **Dados Duplicados:** Verificados via `df.duplicated().sum()` e devidamente removidos com `drop_duplicates()` para evitar distorções nas métricas.

---

## 3. Análise Exploratória

Investigamos como as variáveis se relacionam e como o ambiente afeta o comportamento dos usuários.

###  Correlação entre Variáveis
Utilizando o mapa de calor (`sns.heatmap`), identificamos relações importantes:
* **Temperatura vs. Sensação Térmica:** Alta correlação positiva de **0.99** (variáveis praticamente colineares).
* **Umidade:** Apresenta uma correlação negativa de **-0.46**. Constatou-se que **quanto menor a umidade, menor é a contagem de bicicletas alugadas**.

### 🌤️ Impacto do Clima na Demanda
* **Céu Limpo e Parcialmente Nublado:** São as condições com o maior volume de bicicletas alugadas.
* **Tempo Nublado/Chuvoso:** Há uma queda severa e visível no número de aluguéis de bicicletas.

###  Comparação entre Estações do Ano
A análise visual por meio de gráficos de Boxplot revelou padrões marcantes na distribuição da demanda. Ao analisar a mediana de aluguéis por estação (`df_limpo.groupby('estacao')['contagem'].median()`), obtivemos:

| Estação | Mediana de Aluguéis |
| :--- | :--- |
|  Inverno | 632 |
|  Outono | 898 |
|  Primavera | 823 |
|  Verão | **1.214** (Maior demanda) |

*Nota: Primavera e outono apresentam comportamentos bem semelhantes entre si.*

### 📈 Comportamento Temporal (Sazonalidade)
* **Análise por Ano (`sns.lineplot`):** Em ambos os anos (2015 e 2016), o comportamento se repete. Há um aumento gradativo no início do ano que atinge o pico entre **junho e julho**, seguido por uma queda constante até o final do ano.
* **Análise por Mês:** Os meses de **janeiro e fevereiro** concentram os números mais baixos. O volume sobe gradativamente até **julho** (pico absoluto) e **agosto**, decrescendo a partir de setembro.

---

## 4. Otimização do Modelo e Testes Estatísticos

### Teste de Hipótese (Mann-Whitney U)
Para validar se as distribuições de aluguéis mudam significativamente entre cenários específicos, aplicamos o teste não-paramétrico de Mann-Whitney U.
* **P-Value obtido:** `0.00047`
* **Conclusão:** Como o $p\text{-value} < 0.05$, **rejeitamos a hipótese nula ($H_0$)**. Isso comprova estatisticamente que as distribuições analisadas são significativamente diferentes, confirmando o impacto das variáveis externas na demanda.

### 📈 Ajuste de Sazonalidade e Redução do Erro (Prophet)
Inicialmente, o modelo baseline apresentava um **RMSE (Erro Quadrático Médio)** na casa dos **21.000**. Após definir explicitamente que a sazonalidade dos dados era anual (`yearly_seasonality=True`), o Prophet conseguiu se adequar muito melhor ao comportamento histórico, reduzindo o RMSE de forma drástica para a casa dos **6.000**.

###  Tratamento  de Outliers
Para buscar uma assertividade ainda maior, foi implementada uma estratégia experimental de filtragem de discrepâncias (*outliers*):
1. **Identificação:** Foi gerado um modelo baseline com `periods=0` apenas para extrair os intervalos de confiança inferior (`yhat_lower`) e superior (`yhat_upper`).
2. **Remoção:** Filtramos o dataset original mantendo apenas os registros contidos rigidamente dentro dessa faixa de confiança.

```python
# Semente aleatória para reprodutibilidade
np.random.seed(4587)

# Filtro dinâmico baseado no intervalo de confiança do Prophet
sem_outliers = df_prophet[(df_prophet['y'] > previsao['yhat_lower']) & (df_prophet['y'] < previsao['yhat_upper'])]




