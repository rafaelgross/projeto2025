
# Template de Relatório Técnico — Projeto DW

**Nome do Grupo:** [Ex: Grupo 4 - Data Wizards]  
**Integrantes:**
- Nome Completo 1 — RA: ________
- Nome Completo 2 — RA: ________
- Nome Completo 3 — RA: ________
- Nome Completo 4 — RA: ________

**Domínio:** [Ex: E-commerce, Saúde, Educação]  
**Data de Entrega:** [DD/MM/AAAA]  
**Repositório:** [link do GitHub/GitLab]

---

## 1. Resumo Executivo (1 página)

Apresente em 1-2 parágrafos:
- Problema de negócio que o DW resolve
- Dataset utilizado (fonte, período, volume)
- Principais resultados/insights obtidos
- Tecnologias empregadas

**Exemplo:**
> "Este projeto implementa um Data Warehouse para análise de vendas online da plataforma Olist (99.441 pedidos entre 2016-2018). O objetivo é identificar padrões de consumo, performance de vendedores e oportunidades de otimização logística. Utilizamos DuckDB para modelagem dimensional (esquema estrela com 4 dimensões SCD2) e Python/Plotly para visualizações. Os principais achados incluem: 40% dos produtos concentram 80% da receita (curva ABC), categoria 'cama_mesa_banho' tem maior ticket médio (R$ 150), e região Sudeste responde por 60% das vendas."

---

## 2. Introdução

### 2.1. Contexto e Motivação
- Por que este domínio foi escolhido?
- Qual problema de negócio o DW resolve?
- Quem são os stakeholders/usuários das análises?

### 2.2. Objetivos do Projeto
Liste 3-5 objetivos específicos. Exemplo:
1. Modelar um DW dimensional para vendas online
2. Implementar pipeline ETL automatizado e idempotente
3. Analisar padrões temporais de consumo (sazonalidade)
4. Identificar produtos e categorias de maior lucratividade
5. Gerar dashboards para tomada de decisão gerencial

### 2.3. Escopo
- **Incluído:** Análise de vendas, logística, satisfação
- **Excluído:** Previsão de demanda (ML), integração com ERP real

---

## 3. Fonte de Dados

### 3.1. Dataset Utilizado
- **Nome:** [Ex: Brazilian E-Commerce Public Dataset by Olist]
- **Fonte:** [Link do Kaggle/Data.gov/etc]
- **Período:** [Ex: Set/2016 a Set/2018]
- **Volume:**
  - Clientes: 99.441
  - Pedidos: 98.666
  - Itens: 112.650
  - Produtos: 32.951
  - Vendedores: 3.095
- **Formato:** CSV (9 arquivos, ~45 MB compactado)

### 3.2. Qualidade dos Dados
Descreva problemas encontrados e tratamentos aplicados:
- **Valores nulos:** [Ex: 15% dos produtos sem categoria → atribuído "Outros"]
- **Duplicatas:** [Ex: 23 clientes duplicados por inconsistência de CPF → removidos]
- **Outliers:** [Ex: 3 pedidos com preço negativo → excluídos]
- **Normalização:** [Ex: UF padronizado para maiúsculas, datas convertidas para ISO8601]

---

## 4. Modelagem Dimensional

### 4.1. Arquitetura Geral
Descreva o fluxo: **Fonte → Staging → OLTP → DW**

Inclua diagrama simples (pode ser textual):
```
[CSVs] → [Staging Views] → [OLTP Normalizado] → [ETL] → [DW Estrela]
```

### 4.2. Modelo OLTP (Normalizado)
Liste as tabelas principais e relacionamentos:
- `customers` (PK: customer_id)
- `orders` (PK: order_id, FK: customer_id)
- `order_items` (PK: order_id + item_seq, FK: product_id, seller_id)
- `products` (PK: product_id)
- `sellers` (PK: seller_id)

**Objetivo:** Garantir integridade referencial antes de carregar o DW.

### 4.3. Modelo Dimensional (Estrela)
**Inserir diagrama visual aqui** (PNG/JPG do draw.io ou similar).

Descreva cada tabela:

#### Dimensão: dim_customer (SCD Type 2)
- **Surrogate Key:** customer_key (INT, gerado por sequência)
- **Business Key:** customer_id (VARCHAR)
- **Atributos:** customer_name, customer_city, customer_state
- **Controle SCD2:** start_date, end_date, is_current
- **Grain:** Cada versão de um cliente (por mudança de cidade/estado)

#### Dimensão: dim_product (SCD Type 2)
- **Surrogate Key:** product_key
- **Business Key:** product_id
- **Atributos:** product_name, category, price
- **Controle SCD2:** start_date, end_date, is_current
- **Grain:** Cada versão de um produto (por mudança de categoria/preço)

#### Dimensão: dim_seller (SCD Type 2)
- **Surrogate Key:** seller_key
- **Business Key:** seller_id
- **Atributos:** seller_name, seller_city, seller_state
- **Controle SCD2:** start_date, end_date, is_current
- **Grain:** Cada versão de um vendedor

#### Dimensão: dim_date
- **Primary Key:** date_key (INT, formato YYYYMMDD)
- **Atributos:** full_date, year, quarter, month, month_name, day_of_week, is_weekend
- **Grain:** Um dia

#### Fato: fact_sales
- **Grain:** Cada item de cada pedido
- **Chaves Estrangeiras:**
  - customer_key → dim_customer
  - product_key → dim_product
  - seller_key → dim_seller
  - date_key → dim_date
- **Medidas:**
  - price (DECIMAL)
  - freight_value (DECIMAL)
  - quantity (INT)
  - revenue (DECIMAL, calculado: price × quantity)
- **Atributos degenerados:** order_id, item_seq
- **Contagem esperada:** 112.650 linhas (Olist completo)

### 4.4. Justificativa do Grain
> "Escolhemos o grain de item do pedido (e não pedido completo) para permitir análises detalhadas por produto e vendedor. Um pedido pode conter múltiplos produtos de diferentes vendedores, logo a agregação no nível de item preserva essa granularidade analítica."

---

## 5. Pipeline ETL

### 5.1. Visão Geral
Descreva cada etapa:
1. **Staging (00_staging.sql):** Views lendo CSVs com `read_csv_auto`
2. **OLTP (01_oltp.sql):** Carga do modelo normalizado (DELETE+INSERT)
3. **DW Estrutura (02_dw_model.sql):** Criação de dimensões, fato e sequências
4. **ETL Carga (03_etl_load.sql):** População das dimensões (com SCD2) e fato
5. **Analytics (04_analytics.sql):** Consultas analíticas
6. **Performance (05_perf.sql):** Tabela agregada mensal

### 5.2. Implementação do SCD Type 2
Explique a lógica de detecção de mudanças. Exemplo:

```sql
-- Pseudocódigo: Detectar mudanças em dim_customer
INSERT INTO dw.dim_customer
SELECT NEXTVAL(seq), customer_id, name, city, state, CURRENT_DATE, NULL, TRUE
FROM oltp.customers c
WHERE NOT EXISTS (
  SELECT 1 FROM dw.dim_customer d
  WHERE d.customer_id = c.customer_id AND d.is_current
);

-- Fechar versões antigas quando detectar mudança
UPDATE dw.dim_customer SET end_date = CURRENT_DATE - 1, is_current = FALSE
WHERE customer_id IN (
  SELECT customer_id FROM oltp.customers
  WHERE city != customer_city OR state != customer_state
) AND is_current;

-- Inserir nova versão
INSERT INTO dw.dim_customer ...
```

### 5.3. Validações de Qualidade
Liste as validações implementadas:
- Contagem de linhas pré/pós ETL
- Verificação de NULLs em chaves estrangeiras
- Teste de integridade referencial (fact → dims)
- Soma de receita total (conferência com OLTP)

**Exemplo de log:**
```
[2025-11-14 10:30:15] ETL iniciado
[2025-11-14 10:30:18] dim_date: 774 linhas inseridas
[2025-11-14 10:30:25] dim_customer: 99.441 versões correntes
[2025-11-14 10:30:32] fact_sales: 112.650 linhas carregadas
[2025-11-14 10:30:35] Validação: 0 NULLs em FKs
[2025-11-14 10:30:35] ETL concluído com sucesso
```

### 5.4. Idempotência
Explique como o pipeline pode ser reexecutado:
> "Todas as cargas usam DELETE antes de INSERT, garantindo que rodar o script múltiplas vezes não gera duplicatas. As dimensões com SCD2 verificam existência antes de inserir nova versão."

---

## 6. Análises e Consultas

### 6.1. Perguntas de Negócio
Liste as 5+ perguntas analíticas que o projeto responde:
1. Qual é a evolução mensal da receita por categoria?
2. Quais são os 10 produtos mais vendidos nos últimos 90 dias?
3. Qual a taxa de retenção de clientes por cohort mensal?
4. Qual estado tem maior ticket médio?
5. Qual a distribuição ABC dos produtos (80/20)?

### 6.2. Consultas SQL Implementadas

#### Q1: Receita Mensal por Categoria
```sql
SELECT 
  d.year,
  d.month,
  p.category,
  SUM(f.revenue) AS total_revenue
FROM dw.fact_sales f
JOIN dw.dim_date d ON f.date_key = d.date_key
JOIN dw.dim_product p ON f.product_key = p.product_key
GROUP BY 1, 2, 3
ORDER BY 1, 2, 4 DESC;
```

**Resultado resumido:**
| Ano | Mês | Categoria | Receita |
|-----|-----|-----------|---------|
| 2017 | 11 | cama_mesa_banho | R$ 125.430 |
| 2017 | 11 | eletronicos | R$ 98.760 |
| ... | ... | ... | ... |

**Insight:** Novembro de 2017 foi o mês de maior faturamento (Black Friday).

#### Q2: Top 10 Produtos (90 dias)
```sql
WITH recent_sales AS (
  SELECT product_key, SUM(revenue) AS total
  FROM dw.fact_sales
  WHERE date_key >= (SELECT MAX(date_key) - 90 FROM dw.fact_sales)
  GROUP BY 1
)
SELECT p.product_name, r.total
FROM recent_sales r
JOIN dw.dim_product p ON r.product_key = p.product_key
ORDER BY r.total DESC
LIMIT 10;
```

**Resultado:** [Cole tabela ou print]

**Insight:** Produto "X" domina vendas recentes com R$ 45k.

#### Q3-Q5: [Repetir estrutura para cada query]
- Cole o SQL
- Mostre resultado parcial
- Explique o insight

---

## 7. Visualizações

### 7.1. Gráfico 1: Receita Mensal por Categoria


**Descrição:** Gráfico de linhas mostrando evolução temporal da receita das 8 principais categorias.

**Insight:** Sazonalidade clara em Novembro (Black Friday) e Dezembro (Natal).

### 7.2. Gráfico 2: Top 10 Produtos


**Descrição:** Barras horizontais dos produtos mais rentáveis.

**Insight:** Concentração: top 3 produtos respondem por 40% da receita.

### 7.3. Gráfico 3: Curva ABC


**Descrição:** Curva de Pareto mostrando % acumulada de receita por produto.

**Insight:** 26% dos produtos (classe A) geram 80% da receita.

### 7.4. Gráfico 4: Ticket Médio por Estado


**Descrição:** Mapa de calor ou barras do ticket médio por UF.

**Insight:** SP tem maior ticket médio (R$ 135), seguido por RJ (R$ 128).

---

## 8. Performance e Otimização

### 8.1. Tabela Agregada Mensal
Criamos uma tabela pré-calculada para acelerar consultas de tendência mensal:

```sql
CREATE TABLE dw.fact_sales_monthly AS
SELECT 
  d.year, d.month,
  p.category,
  s.seller_state,
  COUNT(*) AS order_count,
  SUM(f.revenue) AS total_revenue
FROM dw.fact_sales f
JOIN dw.dim_date d ON f.date_key = d.date_key
JOIN dw.dim_product p ON f.product_key = p.product_key
JOIN dw.dim_seller s ON f.seller_key = s.seller_key
GROUP BY 1,2,3,4;
```

### 8.2. Comparação de Performance

| Query | Tabela Original | Tabela Agregada | Ganho |
|-------|-----------------|-----------------|-------|
| Receita mensal | 2.3s | 0.05s | 46x mais rápido |
| Top categorias | 1.8s | 0.03s | 60x mais rápido |

**EXPLAIN antes:**
```
Seq Scan on fact_sales (cost=0..15234 rows=112650)
```

**EXPLAIN depois:**
```
Seq Scan on fact_sales_monthly (cost=0..234 rows=1283)
```

**Conclusão:** Agregação reduziu varredura de 112k para 1.3k linhas.

---

## 9. Desafios e Soluções

### 9.1. Problema 1: Performance com Dataset Grande
**Desafio:** Consultas no Olist completo (112k linhas) demoravam >10s.

**Solução:** Criamos tabela agregada mensal (seção 8) e usamos filtros temporais nas queries (`WHERE date_key >= ...`).

### 9.2. Problema 2: Dados de Categoria Ausentes
**Desafio:** 15% dos produtos sem categoria.

**Solução:** Criamos categoria "Outros" e documentamos no dicionário de dados.

### 9.3. Problema 3: SCD2 Complexo
**Desafio:** Lógica de fechar versões antigas e abrir novas versões era propensa a erros.

**Solução:** Usamos CTEs para separar lógica de detecção e aplicação de mudanças. Testes unitários validaram cada cenário (nova inserção, mudança, sem mudança).

---

## 10. Conclusões

### 10.1. Resultados Alcançados
Resumo dos objetivos cumpridos:
-  DW dimensional completo com 4 dimensões SCD2
-  Pipeline ETL automatizado e validado
-  5 consultas analíticas respondendo perguntas de negócio
-  4 visualizações profissionais
-  Performance otimizada (46x ganho em queries mensais)

### 10.2. Principais Insights
1. **Concentração de receita:** 26% dos produtos geram 80% da receita (curva ABC)
2. **Sazonalidade:** Picos em Nov/Dez (Black Friday e Natal)
3. **Geografia:** Sudeste domina (60% das vendas), mas Norte tem maior crescimento (+35% YoY)
4. **Satisfação:** Categorias com prazo de entrega >20 dias têm nota média <3.5

### 10.3. Limitações e Trabalhos Futuros
- **Limitações:**
  - Dados apenas até 2018 (análise histórica)
  - Não há dados de custo (impossível calcular margem real)
  - SCD2 rastreia apenas cidade/estado (poderia incluir mais atributos)

- **Trabalhos Futuros:**
  - Implementar carga incremental diária (ETL delta)
  - Integrar com API de logística para rastreamento real-time
  - Criar modelos preditivos (ML) para forecast de demanda
  - Migrar para cloud (DuckDB no S3 ou BigQuery)

---

## 11. Referências

- Kimball, R. & Ross, M. (2013). *The Data Warehouse Toolkit*. 3rd ed.
- Documentação DuckDB: https://duckdb.org/docs/
- Dataset Olist: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- [Outras fontes consultadas]

---

## Anexos

### Anexo A: Dicionário de Dados Completo
(Ver arquivo `docs/dicionario_dados.md`)

### Anexo B: Scripts SQL
(Ver pasta `scripts/`)

### Anexo C: Código Python de Visualização
(Ver `visualizacoes/gerar_graficos.py`)

---

**Fim do Relatório**

---

## Checklist de Autoavaliação

Antes de entregar, confirme:
- [ ] Todas as seções preenchidas
- [ ] Diagramas incluídos e legíveis
- [ ] Pelo menos 1 print de cada gráfico
- [ ] SQL formatado e comentado
- [ ] Insights explícitos em cada análise
- [ ] Referências citadas
- [ ] Revisão ortográfica
- [ ] PDF gerado (8-12 páginas)
- [ ] Nome do grupo e RAs no cabeçalho
