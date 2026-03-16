# Delivery Analytics

Este repositório contém um projeto de Análise e Ciência de Dados focado na operação logística de entregas de "last-mile" (Cidade 11). O objetivo principal é diagnosticar as causas raízes do aumento de cancelamentos ocorrido e propor soluções baseadas em dados.

## Problema
A operação enfrentou um aumento relativo de 4,6% na Taxa de Cancelamento e uma estagnação no On-Time Delivery (OTD). A análise investiga os ofensores operacionais (distância, turnos, modalidades) e financeiros (repasses de dinâmica, margem unitária) para entender o que está quebrando o SLA (Service Level Agreement) das entregas.

## Estrutura do Projeto

A análise foi construída em um pipeline modular, separando a Engenharia de Dados, o Diagnóstico Descritivo, a Modelagem Estatística e o Machine Learning Preditivo:

* **`notebooks/01_etl_auditoria.ipynb`**: Pipeline de ETL (Extract, Transform, Load). Responsável por limpar inconsistências, remover duplicatas com base em regras de negócio de prioridade, criar dezenas de *features* calculadas (atraso, custo por km, margem) e exportar a **Camada Gold** (`base_tratada.csv`).
* **`notebooks/02_eda_diagnostico.ipynb`**: Análise Exploratória de Dados (EDA). Mapeamento de distribuições, detecção de ofensores de tempo (ex: tempo de deslocamento consome 60% da operação) e impacto da suspensão de incentivos financeiros (dinâmica).
* **`notebooks/03_modelagem_hipoteses.ipynb`**: Validação estatística e visual de 8 hipóteses de negócio, conectando ineficiência logística à perda de margem financeira.
* **`notebooks/04_modelagem_estatistica_ols.ipynb`**: Inferência estatística utilizando Regressão Linear Múltipla (Ordinary Least Squares) para quantificar o peso exato de cada variável no tempo de atraso.
* **`notebooks/05_machine_learning.ipynb`**: Modelagem preditiva comparando algoritmos (Regressão Linear, KNN, Random Forest e XGBoost) através de Validação Cruzada Aninhada (Nested CV) e otimização de hiperparâmetros para estimar o tempo exato de atraso.

## Principais Descobertas (Diagnóstico)

1. **O Ofensor da Distância (SLA vs. Realidade):** A distância é o maior detrator da operação. Entregas acima de 12 km derrubam o OTD para 46,6% e o tempo P90 chega a 119 minutos. 
2. **Modalidade Expressa:** O baixo OTD (85%) da modalidade Expressa não é causado por lentidão da frota, mas por um SLA irrealista (40 min) que deixa a operação sem margem de segurança frente a imprevistos.
3. **Colapso de Margem Unitária:** Entregas longas (12+ km) e operações no Turno da Noite apresentam os maiores Custos Operacionais. Como a receita não acompanha esse custo (economia de escala reversa no custo absoluto), a margem unitária da empresa é severamente esmagada nesses cenários.
4. **Resiliência de Canais:** Em momentos de crise (suspensão de dinâmicas em Jan/26), o canal de entrada via **API** mostrou-se o mais resiliente, aumentando seu OTD para 90%, enquanto o App estagnou.

## A Equação Matemática do Atraso (Inferência)
Através de um modelo de Regressão Linear Múltipla (`statsmodels`), isolamos as variáveis e descobrimos a fórmula que rege o atraso da operação (com 95% de confiança):

$Atraso = -49.19 + 4.26(km) + 2.01(evento) + 1.57(Noite)$

**Interpretação:**
* Cada 1 km adicional na rota adiciona, matematicamente, **4,26 minutos** de atraso.
* Operar à noite adiciona **1,57 minutos** de atraso em relação à manhã.
* O Turno da Tarde não apresentou significância estatística, provando que seus atrasos derivam do volume e distância, e não do horário em si.

## Machine Learning e o "Teto da Informação"
A transição da inferência para a predição revelou o real limite dos dados logísticos atuais:
* O algoritmo **XGBoost** entregou a melhor performance preditiva, com um Erro Médio Absoluto (MAE) de 10,42 minutos e um $R^2$ de ~34%.
* A paridade de resultados entre modelos simples e complexos (mesmo após intenso *tuning* de hiperparâmetros) provou estatisticamente que os **66% restantes da variância não conseguem ser captados pelos dados atuais**. 
* **Viabilidade de Negócio:** Em vez de fornecer estimativas cravadas, o modelo provou ser viável para a construção de **"Janelas de Expectativa" dinâmicas** no aplicativo, gerenciando a ansiedade do cliente com margens de SLA personalizadas pela complexidade da rota.

## Tecnologias Utilizadas
* **Linguagem:** Python
* **Manipulação de Dados:** Pandas, NumPy
* **Visualização:** Matplotlib, Seaborn
* **Estatística Clássica:** Statsmodels
* **Machine Learning:** Scikit-Learn, XGBoost
* **Versionamento:** Git / GitHub

## Próximos Passos 
Explorar o conjunto de dados para além da cidade 11 e entender se os padrões encontrados se repetem em outras cidades.