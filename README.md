# Segmentação de Clientes (RFM) — Olist E-Commerce

**Stack:** Python • Pandas • scikit-learn • Matplotlib • Seaborn

Projeto de segmentação de clientes usando a metodologia **RFM (Recência, Frequência e Valor Monetário)** combinada com **K-Means**, aplicado aos dados reais e públicos da Olist, um marketplace brasileiro. A ideia foi sair do dataset genérico de tutorial e trabalhar com uma base real, com toda a bagunça que isso traz (tabelas separadas, dados sujos, comportamento de compra que não segue o "livro-texto").

## Por que esse dataset

Usei o [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle) em vez de um CSV único já pronto, de propósito: dados reais quase nunca vêm numa tabela só. Precisei unir três tabelas (`orders`, `order_items`, `customers`) pelo `order_id` e `customer_id`, o que já é, na prática, boa parte do trabalho real de quem mexe com dados no dia a dia.

Um detalhe que só descobri rodando o código: o Olist tem duas colunas de cliente diferentes, `customer_id` e `customer_unique_id`. A primeira muda a cada pedido do mesmo cliente (!), e usar ela por engano faria todo mundo parecer que comprou uma única vez. Tive que usar `customer_unique_id` para identificar a pessoa de verdade.

## O que eu fiz

1. **Limpeza:** tirei pedidos cancelados/não entregues, preços zerados ou negativos, datas vazias.
2. **Cálculo do RFM:** para cada cliente, calculei há quantos dias ele não compra (Recência), quantos pedidos diferentes fez (Frequência) e quanto gastou no total (Monetário).
3. **RFM Score clássico:** dividi cada métrica em quartis (nota 1 a 4). Precisei adaptar a nota de Frequência na mão, porque quase todo mundo comprou só uma vez, dividir isso em quartis "iguais" simplesmente não funciona.
4. **K-Means:** apliquei log + padronização nas três variáveis e rodei o clustering. Usei o método do cotovelo e o coeficiente de silhueta para escolher k=4.
5. **Nomeei os clusters** olhando o perfil médio de cada grupo (recência, frequência e valor médios).

## Resultado

| Segmento | Clientes | % da base | Receita | % da receita | Recência média |
|---|---|---|---|---|---|
| Risco de Churn / Inativos | 37.625 | 40,3% | R$ 1.756.950,69 | 13,3% | 290 dias |
| Novos / Em desenvolvimento | 36.394 | 39,0% | R$ 8.860.586,96 | **67,0%** | 274 dias |
| Fiéis / Regulares | 16.538 | 17,7% | R$ 1.875.551,71 | 14,2% | **43 dias** |
| VIP / Campeões | 2.801 | 3,0% | R$ 728.408,75 | 5,5% | 220 dias |

## O que me chamou atenção (e por que os nomes dos clusters estão "errados" de propósito)

Reparei numa coisa estranha olhando a tabela acima: o cluster "VIP / Campeões" tem recência média de 220 dias, **pior** que o "Fiéis / Regulares", que tem só 43 dias. Isso não fazia sentido para um "VIP".

Fui investigar e a explicação é simples: como mais de 90% dos clientes da base compraram uma única vez, a Frequência quase não ajuda a separar os grupos, na prática, o K-Means acabou agrupando os clientes principalmente pelo **valor gasto**, não pela proximidade da última compra. Ou seja, "VIP" aqui significa "gastou bastante numa única compra", não "cliente fiel e recente" como o nome sugere.

Isso muda a leitura de negócio:

- **Fiéis / Regulares** — na real, é o único grupo com atividade recente de verdade (43 dias). É o mais próximo de um "cliente fiel" de verdade.
- **Novos / Em desenvolvimento** — nome enganoso: são clientes de **compra única de alto valor**, que concentram 67% de toda a receita mas estão sumidos há quase 9 meses. Não são "novos", são clientes valiosos que sumiram.
- **VIP / Campeões** — alto valor histórico, mas também já bem inativos.
- **Risco de Churn / Inativos** — o grupo de menor valor e maior inatividade.

Deixei os nomes originais na tabela de propósito para mostrar esse processo, acho mais honesto do que já entregar os nomes "corrigidos" sem explicar como cheguei lá.

## O que eu faria de campanha para cada grupo

| Segmento | Ação |
|---|---|
| Novos / Em desenvolvimento (compra única de alto valor) | Prioridade #1: campanha de reativação — é quem mais gerou receita e está mais sumido |
| VIP / Campeões | Programa de fidelidade / oferta exclusiva de retorno |
| Fiéis / Regulares | Cross-sell / up-sell, já que é o único grupo ativo |
| Risco de Churn / Inativos | Reativação mais barata (cupom simples), prioridade menor dado o valor histórico baixo |

## Estrutura

```
segmentacao-rfm/
├── data/
│   ├── olist_*.csv                       (dados brutos — baixar do Kaggle, ver abaixo)
│   ├── clientes_segmentados_rfm.csv      (resultado: cliente + segmento)
│   └── resumo_clusters.csv
├── notebooks/
│   └── segmentacao_rfm_olist.ipynb
├── requirements.txt
└── README.md
```

## Como rodar

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Baixa o dataset em [kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), joga os CSVs em `data/` e roda o notebook.

## O que ainda quero mexer

- Separar a análise de quem comprou uma vez só de quem é recorrente, misturar os dois distorce a Frequência.
- Puxar dados de categoria de produto e avaliação para enriquecer os perfis.
- Automatizar a atualização do modelo periodicamente.

---

*Projeto desenvolvido com apoio de IA para estruturação do passo a passo e revisão de código, todo o código foi executado, testado e depurado por mim, célula por célula.*
