# Visão Geral do Projeto
Este repositório documenta um benchmarking técnico com foco em Performance e Escalabilidade de pipelines de dados (ETL/ELT). 

O estudo compara as bibliotecas Pandas e Polars em tarefas críticas de Engenharia de Features (Window Functions, Joins e Agregações) sob alto volume de dados sintéticos.

O objetivo é fornecer evidências quantificáveis sobre qual framework é mais adequado para ambientes de Ciência de Dados de Alta Demanda (HDL), mitigando riscos de memória e latência.


# O Desafio (Challenge Based Learning)

O problema com o crescimento exponencial dos Data Lakes, pipelines baseados em arquiteturas eager e single-thread (como Pandas) enfrentam falhas de memória (OOM) 
e longos tempos de execução, impactando a agilidade do retreino de modelos de ML.


## Fundamentos Arquitetônicos

**Pandas** é a biblioteca fundamental de análise de dados em Python, utilizando arrays NumPy e uma arquitetura *eager* (executa cada operação imediatamente) 
que armazena e processa o *DataFrame* inteiro, que é *mutável* na memória de forma sequencial. 
Sua principal **vantagem** é a maturidade e o vasto ecossistema de integração (com ML, visualização, etc.), 
enquanto sua maior **desvantagem** é a limitação do *Global Interpreter Lock* (GIL) e o alto risco de falha por falta de memória (**OOM**) em grandes volumes. 

**Polars** é um *framework* de processamento de dados moderno, escrito em Rust, otimizado para paralelismo multi-thread e eficiência de memória via **Apache Arrow**, 
utilizando o modelo de **avaliação *Lazy*** (lenta). 
Sua **vantagem** é ser drasticamente mais rápido e escalável para grandes volumes de dados em um único nó, 
sendo a **desvantagem** a curva de aprendizado (devido à sintaxe de *Expressions*) e um ecossistema de integração ML em desenvolvimento. 
Seu *Dataframe* suporta tanto o modo **eager** (para compatibilidade) quanto o modo **lazy** (o padrão para performance), 
onde ele constrói um **Plano de Consulta** antes de executar a tarefa, permitindo otimizações de I/O como *Projection Pushdown*. 


# Metodologia de Benchmarking

## Dados Sintéticos

Fonte: Dados gerados via Faker e numpy para simular transações com 10 milhões de linhas (~1.5 GB) e 15+ colunas de lixo (para testar o Projection Pushdown).

Formato de Teste: Todos os testes são executados em arquivos Parquet, otimizados para leitura colunar.


## Tarefas Críticas (T5 - Foco do Gargalo)

O benchmarking concentrou-se na tarefa de maior complexidade computacional e de I/O:

Tarefa T5: Cálculo da Média Móvel (Rolling Mean) por Cliente. 

Esta operação exige ordenação (sort), agrupamento (groupby) e função de janela (rolling), expondo as limitações de paralelismo do Pandas.


## Métricas

Tempo de Execução: Medido em segundos (usando time) em $N=3$ repetições.

Uso de Memória: Pico de RAM (em GB) registrado pelo memory-profiler.


## Análise Gráfica

Gráfico de Escalabilidade (Log-Log): Demonstra a diferença na inclinação das curvas, provando que o Polars mantém a performance quase linear, enquanto o Pandas sofre com a escala.

Gráfico de Robustez de Memória: Ilustra o pico de RAM, justificando a resiliência do Polars em ambientes limitados por memória.


## Implicações em Produção e Qualidade Total

A escolha do framework não é apenas uma questão de velocidade, mas de Qualidade Total (TQM) e custo.


## Qualidade do Processo (Resiliência)

Vantagem Polars: O modelo Lazy garante Resiliência contra OOM. 

As otimizações de Projection Pushdown e Predicate Pushdown minimizam o I/O, aumentando a rastreabilidade (Plano de Consulta) e reduzindo a chance de falhas.






Execute o 02_benchmark_t5.py (ou use o notebook Colab) para obter os resultados de tempo e memória

