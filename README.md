# Segmentação de Clientes (RFM) — Olist E-Commerce

**Stack:** Python • Pandas • scikit-learn • Matplotlib • Seaborn

Modelo de segmentação de clientes utilizando a metodologia **RFM (Recência, Frequência e Valor Monetário)** combinada com **K-Means clustering**, aplicado à base pública de e-commerce da Olist (marketplace brasileiro). O projeto identifica perfis de clientes com base em comportamento de compra real e gera insights acionáveis para campanhas de marketing.

## Base de dados

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — dataset público disponível no Kaggle, com pedidos reais e anonimizados de um marketplace brasileiro entre 2016 e 2018.

Tabelas utilizadas: `olist_orders_dataset.csv`, `olist_order_items_dataset.csv` e `olist_customers_dataset.csv`, unidas por `order_id` e `customer_id`. A identificação do cliente usa `customer_unique_id`, já que `customer_id` é gerado por pedido (não por pessoa) nesse dataset.

## Metodologia

1. **Limpeza:** remoção de pedidos não entregues (`order_status != 'delivered'`), preços inválidos e datas nulas.
2. **Cálculo RFM:** Recência (dias desde a última compra), Frequência (nº de pedidos únicos) e Valor Monetário (soma gasta) por cliente.
3. **RFM Score:** pontuação por quartis (1–4) para cada dimensão, com regra manual adaptada para Frequência (a maioria dos clientes tem apenas 1 compra, o que inviabiliza divisão em quartis).
4. **Clustering:** aplicação de log(1+x) e padronização (`StandardScaler`) nas três variáveis, seguido de **K-Means** (k=4, escolhido via método do cotovelo e coeficiente de silhueta).
5. **Nomeação dos segmentos:** clusters traduzidos em rótulos de negócio com base no perfil médio de cada grupo.

## Resultados

| Segmento | Clientes | % da base | Receita | % da receita | Recência média |
|---|---|---|---|---|---|
| Risco de Churn / Inativos | 37.625 | 40,3% | R$ 1.756.950,69 | 13,3% | 290 dias |
| Novos / Em desenvolvimento | 36.394 | 39,0% | R$ 8.860.586,96 | **67,0%** | 274 dias |
| Fiéis / Regulares | 16.538 | 17,7% | R$ 1.875.551,71 | 14,2% | **43 dias** |
| VIP / Campeões | 2.801 | 3,0% | R$ 728.408,75 | 5,5% | 220 dias |

## Observação crítica sobre os clusters

Um achado importante desta análise: como **mais de 90% dos clientes da Olist compraram apenas uma vez**, a dimensão Frequência tem baixo poder de separação entre grupos. Como consequência, o K-Means acabou segmentando os clientes majoritariamente pelo **Valor Monetário**, não pela recência de relacionamento — por isso o segmento "VIP / Campeões" apresenta recência média (220 dias) pior que o "Fiéis / Regulares" (43 dias).

Isso expõe uma limitação real do modelo nesse contexto de negócio: em um marketplace onde a recompra é rara, "quem gasta mais numa única compra" não é o mesmo que "quem tem relacionamento contínuo com a marca". Uma leitura mais precisa dos segmentos seria:

- **Fiéis / Regulares** → clientes com atividade recente (43 dias) — o grupo mais próximo de um relacionamento contínuo real.
- **Novos / Em desenvolvimento** → na prática, clientes de **compra única de alto valor**, concentrando 67% da receita mas inativos há quase 9 meses — carecem de uma campanha de reativação, não de "boas-vindas".
- **VIP / Campeões** → clientes de alto valor histórico, também já inativos há bastante tempo.
- **Risco de Churn / Inativos** → menor valor histórico e maior tempo de inatividade — grupo de menor prioridade de investimento.

## Recomendações de marketing

| Segmento | Ação recomendada |
|---|---|
| Novos / Em desenvolvimento (compra única de alto valor) | Prioridade máxima: campanha de win-back — concentram 67% da receita e estão inativos há ~9 meses |
| VIP / Campeões | Programa de fidelidade e oferta exclusiva de retorno, dado o alto ticket histórico |
| Fiéis / Regulares | Cross-sell/up-sell — único grupo com atividade recente, potencial de aumento de ticket médio |
| Risco de Churn / Inativos | Reativação de baixo custo (cupom simples) — menor prioridade orçamentária dado o menor valor histórico |

## Estrutura do repositório

```
segmentacao-rfm/
├── data/
│   ├── olist_*.csv                       (dados brutos do Kaggle — não versionados)
│   ├── clientes_segmentados_rfm.csv      (resultado: cliente + segmento)
│   └── resumo_clusters.csv               (perfil médio de cada cluster)
├── notebooks/
│   └── segmentacao_rfm_olist.ipynb
├── requirements.txt
└── README.md
```

## Como reproduzir

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Baixe o dataset em [kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), extraia os CSVs em `data/` e execute `notebooks/segmentacao_rfm_olist.ipynb`.

## Próximos passos

- Incorporar dados de categoria de produto e avaliação (`olist_order_reviews_dataset.csv`) para enriquecer os perfis.
- Testar segmentação separada para clientes de compra única vs. recorrente, já que misturá-los distorce a leitura de Frequência.
- Automatizar a atualização periódica do modelo em pipeline de dados.
