MVP – Engenharia de Dados
Análise do Desempenho Operacional de Entregas de Alimentos
Rafaela Neves da Silva
Pós-graduação em Ciência de Dados
Disciplina: Engenharia de Dados

Descrição do Projeto
Este projeto apresenta o desenvolvimento de um MVP (Minimum Viable Product) de Engenharia de Dados com foco na análise do desempenho operacional de entregas de alimentos. O trabalho contempla a construção de um pipeline de dados, desde a coleta e tratamento até a análise exploratória, utilizando dados públicos e tecnologias em nuvem.
O objetivo principal é demonstrar, de forma prática, a aplicação de conceitos fundamentais de Engenharia de Dados, como ETL, qualidade de dados, modelagem analítica e análise orientada a perguntas de negócio.

Objetivo
Construir um pipeline de dados que permita analisar como fatores contextuais e operacionais — como contexto urbano, volume de entregas, condições de tráfego, clima, tipo de veículo e períodos festivos — influenciam o tempo de entrega e a avaliação do serviço de delivery.

Observação Importante – Delimitação do Escopo
O conjunto de dados utilizado não contém informações no nível de restaurante, mas sim dados relacionados aos entregadores e ao contexto operacional das entregas. Dessa forma, para manter a coerência técnica e metodológica da análise, as perguntas de negócio foram ajustadas para considerar a avaliação média por contexto urbano (cidade), que representa o nível máximo de agregação possível com consistência a partir dos dados disponíveis.

Perguntas de Negócio
1.	Como a avaliação média das entregas varia entre diferentes contextos urbanos?
2.	Existe relação entre o volume de entregas e a avaliação média do serviço nos diferentes contextos urbanos?
3.	O tempo médio de entrega varia de acordo com o contexto urbano?
4.	As condições de tráfego influenciam o tempo médio de entrega?
5.	As condições climáticas impactam o tempo de entrega?
6.	Existe diferença no tempo de entrega de acordo com o tipo de veículo utilizado?
7.	Entregas realizadas durante períodos festivos apresentam diferença de desempenho em relação a períodos regulares?

Fonte dos Dados
•	Dataset público disponibilizado no Kaggle
•	Coleta realizada via API, utilizando a biblioteca kagglehub

Tecnologias Utilizadas
•	Python
•	Google Colab
•	Pandas
•	NumPy
•	Kaggle API
•	GitHub

Pipeline de Dados (ETL)
O pipeline desenvolvido contempla as seguintes etapas:
1.	Coleta dos dados via API do Kaggle
2.	Exploração inicial e entendimento do schema
3.	Análise de qualidade dos dados, identificando:
o	Tipagem incorreta
o	Valores inválidos
o	Categorias mal padronizadas
4.	Tratamento e saneamento (ETL):
o	Conversão de tipos
o	Padronização de categorias
o	Tratamento de valores ausentes
5.	Carga dos dados tratados em estrutura analítica
6.	Análise exploratória orientada às perguntas de negócio

Análises Realizadas
As análises foram conduzidas por meio de agregações e estatísticas descritivas, utilizando métricas como:
•	Avaliação média
•	Tempo médio de entrega
•	Volume de entregas
Os resultados são discutidos de forma contextualizada, conectando os dados aos desafios operacionais do serviço de delivery.

Principais Aprendizados
•	Importância da análise de qualidade de dados antes de qualquer análise
•	Necessidade de adequar o problema de negócio aos dados disponíveis
•	Aplicação prática de processos de ETL
•	Construção de análises consistentes e metodologicamente corretas

Trabalhos Futuros
•	Enriquecimento do dataset com identificadores de restaurantes
•	Inclusão de dados geográficos mais detalhados
•	Criação de visualizações interativas
•	Implementação do pipeline em uma plataforma de dados em nuvem dedicada (ex.: Databricks)
