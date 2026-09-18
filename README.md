# Segmentação de Clientes: RFM + K-Means

Projeto de segmentação de clientes desenvolvido a partir do **Brazilian E-Commerce Public Dataset by Olist**, utilizando a metodologia **RFM (Recência, Frequência e Valor Monetário)** combinada ao algoritmo **K-Means**.

O projeto parte de dados de pedidos, clientes e itens de pedidos para construir indicadores de comportamento de compra, identificar grupos de clientes com características semelhantes e analisar a composição de cada segmento.

## Objetivo

O objetivo é utilizar dados reais de comércio eletrônico para identificar diferentes perfis de clientes a partir do comportamento de compra.

A análise combina duas abordagens:

* **RFM**, para representar o comportamento de cada cliente;
* **K-Means**, para agrupar clientes com características semelhantes.

Além da segmentação, o projeto avalia os resultados encontrados e verifica se os grupos formados pelo algoritmo realmente correspondem aos perfis esperados.

## Dataset

Foi utilizado o **Brazilian E-Commerce Public Dataset by Olist**, disponibilizado publicamente no Kaggle.

A base contém diferentes tabelas relacionadas às operações de um marketplace brasileiro, incluindo informações de:

* pedidos;
* clientes;
* itens dos pedidos;
* produtos;
* pagamentos;
* avaliações.

Neste projeto, foram utilizadas principalmente as tabelas relacionadas a **pedidos, clientes e itens dos pedidos**.

A escolha de trabalhar com as tabelas separadas permite reproduzir uma situação mais próxima de um cenário real de análise de dados, no qual as informações necessárias para uma análise normalmente estão distribuídas em diferentes estruturas.

## Tratamento dos dados

Antes da construção dos indicadores RFM, os dados passaram por etapas de preparação para evitar que registros inválidos interferissem na análise.

Entre os tratamentos realizados estão:

* remoção de pedidos cancelados ou não entregues;
* tratamento de datas ausentes;
* remoção de valores de preço zerados ou negativos;
* relacionamento entre as tabelas de pedidos, clientes e itens;
* identificação dos clientes por `customer_unique_id`.

### `customer_id` x `customer_unique_id`

Um ponto importante encontrado durante a análise foi a diferença entre `customer_id` e `customer_unique_id`.

O `customer_id` está relacionado ao registro do cliente dentro de um pedido, enquanto o `customer_unique_id` permite identificar o mesmo cliente ao longo de diferentes pedidos.

Para a análise de frequência, utilizar o identificador incorreto poderia fazer com que diferentes compras de uma mesma pessoa fossem tratadas como clientes distintos.

Por isso, o `customer_unique_id` foi utilizado como identificador do cliente na construção do RFM.

## Análise RFM

A metodologia RFM representa cada cliente através de três indicadores:

| Métrica         | Representa                                     |
| --------------- | ---------------------------------------------- |
| Recência        | Quantos dias se passaram desde a última compra |
| Frequência      | Quantidade de pedidos realizados               |
| Valor Monetário | Valor total gasto pelo cliente                 |

Essas três dimensões permitem analisar não apenas quanto um cliente gastou, mas também sua frequência de compra e o tempo desde sua última interação.

### RFM Score

Também foi calculado um score RFM utilizando quartis para atribuir notas às métricas.

Cada indicador recebe uma classificação de **1 a 4**, permitindo comparar os clientes de acordo com seu comportamento.

A distribuição da frequência exigiu um tratamento específico, pois a maior parte da base possui apenas uma compra. Nesse cenário, uma divisão automática em quartis não representa adequadamente a distribuição da variável.

Por esse motivo, a classificação da frequência foi ajustada para refletir melhor a distribuição observada nos dados.

## Segmentação com K-Means

Após o cálculo das variáveis RFM, foi aplicado o algoritmo **K-Means** para identificar grupos de clientes com características semelhantes.

Antes do agrupamento, as variáveis passaram por:

1. transformação logarítmica;
2. padronização;
3. aplicação do algoritmo K-Means.

A transformação foi utilizada para reduzir o efeito da assimetria presente nas variáveis de valor e frequência, enquanto a padronização colocou as métricas em escalas comparáveis.

### Definição do número de clusters

Foram utilizados:

* **Método do Cotovelo (Elbow Method)**;
* **Coeficiente de Silhueta (Silhouette Score)**.

A análise indicou a utilização de **4 clusters** para a segmentação.

Após o agrupamento, os clusters foram analisados considerando as médias de recência, frequência e valor monetário para interpretar o comportamento de cada grupo.

## Resultados

A segmentação produziu quatro grupos:

| Segmento                   | Clientes | % da base |         Receita | % da receita | Recência média |
| -------------------------- | -------: | --------: | --------------: | -----------: | -------------: |
| Risco de Churn / Inativos  |   37.625 |     40,3% | R$ 1.756.950,69 |        13,3% |       290 dias |
| Novos / Em desenvolvimento |   36.394 |     39,0% | R$ 8.860.586,96 |        67,0% |       274 dias |
| Fiéis / Regulares          |   16.538 |     17,7% | R$ 1.875.551,71 |        14,2% |        43 dias |
| VIP / Campeões             |    2.801 |      3,0% |   R$ 728.408,75 |         5,5% |       220 dias |

Os resultados mostram uma concentração significativa da receita em um dos grupos, enquanto outros segmentos possuem maior quantidade de clientes, mas menor participação no faturamento.

## Análise dos clusters

A interpretação dos clusters revelou uma limitação importante da segmentação.

Mais de **90% dos clientes da base realizaram apenas uma compra**, fazendo com que a variável Frequência apresentasse pouca capacidade de diferenciar os clientes.

Como consequência, o K-Means acabou utilizando principalmente as diferenças de **valor gasto** e, em menor grau, de **recência** para separar os grupos.

Isso produz uma diferença entre o nome inicialmente atribuído aos segmentos e o comportamento efetivamente observado.

### Risco de Churn / Inativos

É o grupo com menor valor histórico e maior recência média, apresentando aproximadamente **290 dias desde a última compra**.

O comportamento indica clientes com baixa atividade recente e menor contribuição histórica de receita.

### Novos / Em desenvolvimento

Apesar do nome inicial, os dados mostram um comportamento diferente.

Esse grupo concentra aproximadamente **67% da receita da base**, mas apresenta recência média de aproximadamente **274 dias**.

A análise indica que se trata principalmente de clientes que realizaram uma compra de valor elevado, mas que não voltaram a comprar durante o período analisado.

Por isso, o nome "Novos / Em desenvolvimento" não representa completamente o comportamento observado.

### Fiéis / Regulares

É o grupo com a menor recência média, aproximadamente **43 dias**.

Apesar de representar 17,7% da base, concentra clientes com atividade de compra mais recente em comparação aos demais segmentos.

Esse foi o grupo que apresentou o comportamento mais próximo do conceito esperado de clientes ativos e recorrentes.

### VIP / Campeões

É o menor grupo da base, representando aproximadamente **3% dos clientes**.

Apesar do nome atribuído inicialmente, a recência média é de aproximadamente **220 dias**.

Nesse contexto, o segmento representa principalmente clientes com maior valor histórico de compra, e não necessariamente clientes recentes ou recorrentes.

## Principal aprendizado da análise

O resultado demonstra uma limitação importante da aplicação direta do RFM + K-Means em uma base com baixa recorrência de compra.

A metodologia consegue separar os clientes de acordo com seus padrões observados, mas isso não significa que os clusters automaticamente correspondam a categorias de negócio como "VIP", "fiel" ou "novo".

A interpretação dos segmentos precisa considerar a distribuição dos dados e as características do conjunto analisado.

Neste caso, a baixa frequência de compras reduziu a capacidade da variável Frequência de diferenciar os clientes, fazendo com que o valor gasto tivesse maior influência na formação dos grupos.

Essa etapa de validação foi importante para evitar uma interpretação incorreta dos resultados do algoritmo.

## Possíveis ações por segmento

A segmentação também pode ser utilizada como ponto de partida para ações direcionadas:

| Segmento observado                      | Possível ação                              |
| --------------------------------------- | ------------------------------------------ |
| Clientes de alto valor e baixa recência | Campanhas de reativação                    |
| Clientes ativos e recentes              | Cross-sell e up-sell                       |
| Clientes de baixo valor e alta recência | Campanhas de reativação de menor custo     |
| Clientes com alto valor histórico       | Ofertas específicas e incentivo à recompra |

Essas ações são hipóteses baseadas no comportamento observado na base e não fazem parte de uma avaliação real de campanhas comerciais.

## Tecnologias

| Tecnologia       | Utilização                                     |
| ---------------- | ---------------------------------------------- |
| Python           | Desenvolvimento da análise                     |
| Pandas           | Manipulação e preparação dos dados             |
| NumPy            | Operações numéricas e transformação dos dados  |
| Scikit-learn     | K-Means, padronização e avaliação dos clusters |
| Matplotlib       | Visualização dos resultados                    |
| Seaborn          | Visualização e análise exploratória            |
| Jupyter Notebook | Desenvolvimento e documentação da análise      |

## Estrutura do projeto

```text
segmentacao-rfm/
│
├── data/
│   ├── olist_*.csv
│   ├── clientes_segmentados_rfm.csv
│   └── resumo_clusters.csv
│
├── notebooks/
│   └── segmentacao_rfm_olist.ipynb
│
├── requirements.txt
└── README.md
```

Os arquivos originais do dataset podem ser obtidos diretamente no Kaggle e armazenados na pasta `data/`.

## Como executar

### 1. Criar o ambiente virtual

```bash
python -m venv venv
```

No Windows:

```bash
venv\Scripts\activate
```

### 2. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 3. Baixar o dataset

Baixe o **Brazilian E-Commerce Public Dataset by Olist** no Kaggle e coloque os arquivos CSV necessários dentro da pasta:

```text
data/
```

### 4. Executar o notebook

Abra o Jupyter Notebook:

```bash
jupyter notebook
```

Depois execute:

```text
notebooks/segmentacao_rfm_olist.ipynb
```

As etapas de preparação, cálculo do RFM, definição dos clusters e análise dos resultados estão documentadas no notebook.

## Limitações

O projeto possui algumas limitações que devem ser consideradas na interpretação dos resultados:

* a base possui forte concentração de clientes com apenas uma compra;
* a variável Frequência possui baixa capacidade de diferenciação;
* os clusters são determinados pelo comportamento estatístico observado na base;
* os nomes dos segmentos são interpretações e não categorias originalmente presentes no dataset;
* os resultados representam o período disponível no dataset da Olist;
* a segmentação não foi validada com dados de campanhas ou comportamento posterior dos clientes.

## Próximas etapas

Algumas possibilidades de evolução do projeto seriam:

* testar outras técnicas de clustering;
* comparar diferentes configurações de variáveis;
* avaliar outros critérios de definição dos clusters;
* incorporar informações de categorias de produtos;
* analisar ticket médio por cliente;
* acompanhar a evolução dos segmentos ao longo do tempo;
* criar um dashboard para exploração dos segmentos;
* avaliar a estabilidade dos clusters em diferentes períodos.

## Conclusão

O projeto demonstra a aplicação de uma metodologia de segmentação sobre uma base real de comércio eletrônico, combinando preparação de dados, análise exploratória, construção de métricas RFM e aprendizado não supervisionado com K-Means.

O principal resultado foi identificar que a baixa recorrência de compras da base limita a utilização da Frequência como variável de segmentação, fazendo com que o valor gasto tenha maior influência na formação dos grupos.

Essa análise reforça a importância de validar e interpretar os resultados de modelos de segmentação antes de utilizá-los para conclusões de negócio.
