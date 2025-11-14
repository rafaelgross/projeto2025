# Projeto Avaliativo — Data Warehouse Completo

**Disciplina:** Banco e Armazém de Dados em Ciências de Dados  
**Modalidade:** Grupos de 4 pessoas  
**Formato:** Entrega online (repositório + documentação + apresentação)  
**Peso:** Prova prática substitutiva  
**Prazo:** entrega no site rafaelgross.pro.br 28/11/25

---

## 1. Contexto e Objetivo

Você já aprendeu a construir um Data Warehouse com os dados do Olist (e-commerce). Agora é hora de criar seu próprio projeto completo!

**O que você vai fazer:**
- Escolher um tema de negócio (vendas, saúde, educação, etc.)
- Baixar dados públicos (vamos dar os links!)
- Construir um DW do zero (igual ao que fizemos na aula)
- Criar gráficos para mostrar insights
- Documentar tudo em um relatório

**Por que isso é importante:**
Este projeto simula um trabalho real de Analista/Engenheiro de Dados. Você vai mostrar que sabe transformar dados brutos em informação útil para decisões de negócio.

---

## 2. Escolha do Domínio e Dataset

Cada grupo deve escolher **UM** dos seguintes domínios (ou propor outro com aprovação prévia):

### Opção 1: Vendas e Logística
- **Dataset:** Brazilian E-Commerce Olist (vocês já conhecem!)
- **Onde baixar:** 
  - Kaggle: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
  - OU usar os arquivos que já temos em `data/olist/`
- **Tamanho:** ~45 MB (9 arquivos CSV)
- **O que tem:** 99 mil pedidos, produtos, vendedores, avaliações, frete
- **Perguntas que você pode responder:**
  - Qual categoria vende mais em cada mês?
  - Quais estados têm maior ticket médio?
  - Qual vendedor tem melhor avaliação?
  - Como está a evolução de vendas ao longo do tempo?

### Opção 2: Saúde Pública (COVID-19)
- **Dataset:** COVID-19 Data Repository
- **Onde baixar:**
  - Our World in Data: https://github.com/owid/covid-19-data/tree/master/public/data
  - Arquivo principal: `owid-covid-data.csv` (~30 MB)
- **O que tem:** Casos, mortes, vacinação, testes por país e data
- **Perguntas que você pode responder:**
  - Como evoluiu a vacinação no Brasil vs outros países?
  - Qual mês teve mais casos?
  - Estados com maior taxa de mortalidade?
  - Relação entre vacinação e queda de casos?

### Opção 3: Mobilidade Urbana (Bicicletas Compartilhadas)
- **Dataset:** Citibike NYC Trip Data
- **Onde baixar:**
  - Site oficial: https://citibikenyc.com/system-data
  - Escolha 1 mês (ex: Janeiro 2024, ~50 MB)
- **O que tem:** Viagens de bike, estações, horários, duração
- **Perguntas que você pode responder:**
  - Quais estações são mais usadas?
  - Qual horário tem pico de viagens?
  - Qual a duração média por dia da semana?
  - Diferença entre dias úteis e fins de semana?

### Opção 4: Educação (ENADE)
- **Dataset:** Microdados do ENADE
- **Onde baixar:**
  - Portal INEP: https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/enade
  - Escolha 1 ano (ex: 2021, ~100 MB)
- **O que tem:** Notas de alunos, cursos, instituições, questionários
- **Perguntas que você pode responder:**
  - Qual área tem melhores notas?
  - Desempenho por região do Brasil?
  - Perfil socioeconômico dos alunos?
  - Comparação entre universidades públicas e privadas?

### Opção 5: Energia e Sustentabilidade
- **Dataset:** World Energy Statistics
- **Onde baixar:**
  - Our World in Data: https://github.com/owid/energy-data
  - Arquivo: `owid-energy-data.csv` (~5 MB)
- **O que tem:** Consumo e geração de energia por país, fontes renováveis
- **Perguntas que você pode responder:**
  - Como cresceu a energia solar no Brasil?
  - Quais países usam mais energia renovável?
  - Evolução do consumo per capita?
  - Comparação fontes limpas vs fósseis?

### Opção 6: Proposta Própria
Quer usar outro tema? Pode! Mas envie antes:
- Nome do tema (ex: Netflix, Spotify, Games, Esportes)
- Link do dataset (onde baixar)
- 3 perguntas que quer responder
- Tamanho dos dados (mínimo 10 mil linhas)

**Prazo para aprovação:** [definir - sugestão: 1 semana antes do Checkpoint 1]

**Sites para procurar dados:**
- Kaggle: https://www.kaggle.com/datasets (maior repositório)
- Google Dataset Search: https://datasetsearch.research.google.com
- Data.gov: https://data.gov (dados dos EUA)
- IBGE: https://www.ibge.gov.br (dados do Brasil)

---

## 3. Requisitos Funcionais

Seu DW deve implementar **TODOS** os seguintes componentes:

### 3.1. Modelagem Dimensional (30 pts)

**O que você precisa criar:**

```
[Dimensões]              [Fato]                [O que significa]

dim_date          -----> fact_sales         Data da venda
dim_customer      -----> fact_sales         Quem comprou
dim_product       -----> fact_sales         O que foi comprado
dim_seller        -----> (opcional)         Quem vendeu
```

**Requisitos mínimos:**
-  **3 dimensões** (pelo menos 1 com histórico de mudanças - SCD Type 2)
- **1 tabela fato** (onde ficam as vendas/eventos principais)
- **Dimensão de data** (dim_date com ano, mês, dia, etc.)
- **Chaves geradas automaticamente** (não usar CPF/ID original como chave primária)
- **Diagrama visual** (desenho do modelo - pode usar draw.io)

**Exemplo de SCD Type 2 (rastreamento de mudança):**
Se um cliente mudar de cidade:
```
customer_key | customer_id | name    | city       | start_date | end_date   | is_current
1            | C001        | João    | São Paulo  | 2020-01-01 | 2023-05-10 | FALSE
2            | C001        | João    | Campinas   | 2023-05-11 | NULL       | TRUE
```
Assim você sabe que João morava em SP e depois mudou para Campinas!

**Como será avaliado:**
-  Modelo correto e fácil de entender
-  SCD2 funcionando (detecta mudanças)
-  Documentação: o que cada tabela representa
-  Sem erros de relacionamento (todas as vendas têm cliente, produto, data)

### 3.2. Pipeline ETL/ELT (25 pts)

**O que é ETL?** Extract (extrair), Transform (transformar), Load (carregar)

**Seu pipeline deve seguir estes passos:**

```
1. STAGING (00_staging.sql)
   CSVs ---> Views temporárias
    Lê os arquivos CSV
    Não transforma nada ainda

2. OLTP (01_oltp.sql)
   Views ---> Tabelas normalizadas
    Cria clientes, produtos, pedidos separados
    Remove duplicatas
    Trata dados nulos

3. DW ESTRUTURA (02_dw_model.sql)
   Cria dimensões e fato (vazias ainda)
    dim_customer, dim_product, dim_date, fact_sales

4. ETL CARGA (03_etl_load.sql)
   OLTP ---> DW (popula tudo)
   Preenche dim_date (todos os dias do período)
   Carrega dimensões com SCD2
   Carrega fact_sales

5. VALIDAÇÕES
   Conta linhas: OLTP tem 1000 pedidos? DW tem 1000 na fato?
   Sem NULLs nas chaves estrangeiras
   Todas as vendas têm cliente/produto válido
```

**Importante:**
- Scripts devem rodar **várias vezes sem erro** (idempotência)
- Se rodar 2x, não duplica dados
- Comentar o código explicando cada passo

**Como será avaliado:**
-  Scripts organizados (00, 01, 02, 03...)
-  Pode reexecutar sem quebrar
-  Trata erros (dados faltando, duplicados)
-  Documentação: explica o que cada script faz

### 3.3. Consultas Analíticas (20 pts)

**Você precisa criar 5 consultas SQL respondendo perguntas de negócio.**

**Tipos obrigatórios de consulta:**

**1. Análise Temporal (linha do tempo)**
```sql
-- Exemplo: Vendas por mês
SELECT 
  d.year, d.month,
  SUM(f.revenue) as total_vendas
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY d.year, d.month
ORDER BY d.year, d.month;
```
 Mostra como as vendas evoluem ao longo do tempo

**2. Ranking / TOP N (os melhores/piores)**
```sql
-- Exemplo: Top 10 produtos mais vendidos
SELECT 
  p.product_name,
  SUM(f.revenue) as total
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY p.product_name
ORDER BY total DESC
LIMIT 10;
```
 Identifica os produtos campeões de venda

**3. Agregação Multidimensional (várias perspectivas)**
```sql
-- Exemplo: Vendas por categoria E por estado
SELECT 
  p.category,
  c.customer_state,
  COUNT(*) as qtd_vendas,
  AVG(f.revenue) as ticket_medio
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY p.category, c.customer_state;
```
 Cruza informações: qual categoria vende mais em qual estado?

**4. Análise de Cohort / Retenção (comportamento ao longo do tempo)**
```sql
-- Exemplo: Quantos clientes voltaram a comprar?
SELECT 
  first_purchase_month,
  COUNT(DISTINCT customer_id) as total_clientes
FROM (
  SELECT customer_id, MIN(purchase_date) as first_purchase_month
  FROM orders
  GROUP BY customer_id
)
GROUP BY first_purchase_month;
```
 Entende fidelização de clientes

**5. KPI (indicador-chave de negócio)**
```sql
-- Exemplo: Ticket médio por estado
SELECT 
  c.customer_state,
  AVG(f.revenue) as ticket_medio,
  COUNT(*) as qtd_pedidos
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.customer_state
ORDER BY ticket_medio DESC;
```
 Métrica importante para o negócio

**Como será avaliado:**
-  5 consultas funcionando corretamente
-  Usa JOINs, GROUP BY, funções de agregação
-  Resultados fazem sentido para o negócio
-  Comentários explicando o que cada query faz
-  Executa em menos de 30 segundos

### 3.4. Visualizações (15 pts)

**Criar 4 gráficos para mostrar os resultados das suas análises.**

**Gráficos obrigatórios:**

**1. Gráfico de Linha (evolução no tempo)**
```python
import plotly.express as px

# Exemplo: vendas por mês
fig = px.line(df, x='mes', y='vendas', title='Evolução de Vendas')
fig.write_image('grafico_1_evolucao.png')
```
 Mostra tendência: subindo, descendo ou estável

**2. Gráfico de Barras (comparação)**
```python
# Exemplo: top 10 produtos
fig = px.bar(df, x='produto', y='receita', title='Top 10 Produtos')
fig.write_image('grafico_2_top_produtos.png')
```
Compara: qual é maior/menor

**3. Mapa de Calor OU Dispersão (correlação)**
```python
# Exemplo: vendas por estado
import seaborn as sns
sns.heatmap(df.pivot_table(values='vendas', index='estado', columns='mes'))
```
 Mostra padrões: onde/quando tem mais atividade

**4. Dashboard / Composição**
```python
# Exemplo: vários gráficos em uma imagem
from plotly.subplots import make_subplots

fig = make_subplots(rows=2, cols=2)
fig.add_trace(grafico1, row=1, col=1)
fig.add_trace(grafico2, row=1, col=2)
# ...
fig.write_html('dashboard.html')
```
 Painel completo com vários gráficos

**Ferramentas que você pode usar:**
-  Python + Plotly (igual fizemos na aula - RECOMENDADO)
-  Python + Matplotlib/Seaborn
-  Power BI (se souber)
-  Google Data Studio
-  Tableau Public

**Requisitos dos gráficos:**
-  Título claro
-  Eixos com nomes (não deixar "x" e "y")
-  Cores profissionais (evitar cores muito berrantes)
-  Legenda quando necessário
-  Salvar como PNG ou HTML

**Importante:** Para cada gráfico, escreva **1 parágrafo** explicando o insight:
> "Este gráfico mostra que as vendas cresceram 40% entre janeiro e dezembro, com pico em novembro devido à Black Friday."

**Como será avaliado:**
-  4 gráficos diferentes
-  Visual profissional e legível
-  Insights escritos para cada gráfico
-  Arquivos salvos (PNG/HTML)

### 3.5. Performance e Otimização (10 pts - bônus)
- Criar **1 tabela agregada** ou materialized view
- Comparar EXPLAIN da query original vs. query otimizada
- Documentar ganho de performance (tempo de execução)
- (Opcional) Implementar índices, particionamento ou compressão

---

## 4. Requisitos Não-Funcionais

### 4.1. Tecnologia
- **Banco de dados:** DuckDB (recomendado) ou PostgreSQL
- **Linguagem ETL:** SQL (obrigatório) + Python (opcional para pré-processamento)
- **Visualização:** Python (plotly/matplotlib) ou ferramenta BI
- **Versionamento:** Git (repositório privado ou público)

### 4.2. Organização do Código
```
projeto-dw-grupo-X/
├── README.md                     # Visão geral do projeto
├── data/                         # CSVs (ou link para download)
├── scripts/
│   ├── 00_staging.sql
│   ├── 01_oltp.sql
│   ├── 02_dw_model.sql
│   ├── 03_etl_load.sql
│   ├── 04_analytics.sql
│   └── 05_performance.sql
├── visualizacoes/
│   ├── gerar_graficos.py (ou .ipynb)
│   └── *.png / *.html
├── docs/
│   ├── relatorio_tecnico.pdf
│   ├── diagrama_modelo_estrela.png
│   └── dicionario_dados.md
└── run_all.sh / run_all.ps1
```

### 4.3. Documentação Obrigatória
1. **README.md** — Como executar o projeto (pré-requisitos, comandos)
2. **Relatório Técnico** (ver template) — 8-12 páginas
3. **Diagrama do modelo estrela** — Visual claro (draw.io, Lucidchart, ERD)
4. **Dicionário de dados** — Tabela com nome/tipo/descrição de cada coluna

---

## 5. Entregas e Prazos

| Entrega | Data | Peso | Descrição |
|---------|------|------|-----------|
| **Checkpoint 1** | Semana 2 | 10% | Escolha do domínio, fonte de dados, modelo conceitual |
| **Checkpoint 2** | Semana 3 | 15% | Pipeline ETL funcional (staging → oltp → dw) |
| **Entrega Final** | Semana 4 | 65% | Código completo + documentação + visualizações |
| **Apresentação** | Semana 5 | 10% | Vídeo gravado (10-15 min) ou apresentação síncrona |

### Checkpoint 1 (Planejamento)
Enviar via AVA:
- Nome do grupo e integrantes
- Domínio escolhido e justificativa (1 parágrafo)
- Link do dataset
- 5 perguntas analíticas que pretendem responder
- Esboço do modelo estrela (pode ser à mão)

### Checkpoint 2 (Validação Técnica)
Enviar via Git (link do repositório):
- Scripts 00, 01, 02 funcionando
- README com instruções de execução
- Pelo menos 1 dimensão com dados carregados

### Entrega Final
Enviar via AVA + Git:
- Link do repositório (público ou com acesso ao professor)
- Relatório técnico em PDF
- Prints dos gráficos (ou link para dashboard online)
- Arquivo `.duckdb` ou dump SQL (se PostgreSQL)

### Apresentação
- Vídeo de 10-15 minutos explicando:
  - Problema de negócio e dataset
  - Modelo dimensional
  - 2-3 insights mais relevantes das análises
  - Desafios técnicos e soluções


---

## 6. Critérios de Avaliação (Rubrica Detalhada)

| Critério | Excelente (100%) | Bom (75%) | Satisfatório (50%) | Insuficiente (<50%) |
|----------|------------------|-----------|-------------------|---------------------|
| **Modelagem** | Modelo estrela completo, SCD2 correto, grain claro, diagrama profissional | Modelo correto, pequenas inconsistências, diagrama básico | Modelo funcional, mas sem SCD2 ou grain mal definido | Modelo incompleto ou incorreto |
| **ETL** | Pipeline robusto, idempotente, com validações e logs | Pipeline funcional, algumas validações | Pipeline básico, sem tratamento de erros | Pipeline quebrado ou incompleto |
| **Consultas** | 5+ queries complexas, corretas, bem documentadas | 5 queries funcionais, pouca documentação | 3-4 queries básicas | <3 queries ou incorretas |
| **Visualizações** | 4+ gráficos profissionais, insights claros | 4 gráficos básicos, insights razoáveis | 2-3 gráficos simples | <2 gráficos ou sem insights |
| **Documentação** | Relatório completo, README claro, diagrama detalhado | Relatório básico, README funcional | Documentação mínima | Documentação ausente ou confusa |
| **Código** | Organizado, comentado, versionado, reprodutível | Funcional, organização razoável | Funciona mas desorganizado | Código quebrado ou difícil de executar |
| **Apresentação** | Clara, objetiva, demonstra domínio | Adequada, cobre pontos principais | Básica, falta clareza | Confusa ou incompleta |

### Penalidades
- **-20%**: Entrega após prazo (até 2 dias)
- **-50%**: Entrega após prazo (>2 dias) ou plágio detectado
- **0**: Projeto não executa, dados não fornecidos, ou trabalho individual detectado

### Bônus (até +15%)
- **+5%**: Implementação em PostgreSQL **E** DuckDB (comparação)
- **+5%**: Dashboard interativo (Streamlit, Dash, Power BI publicado)
- **+5%**: Testes automatizados (validação de integridade, contagem de linhas)
- **+5%**: Deploy em nuvem (AWS RDS, Google BigQuery, DuckDB no S3)

---

## 7. Divisão de Tarefas Sugerida (4 pessoas)

### Pessoa 1: Arquiteto de Dados
- Modelagem dimensional (esquema estrela)
- Definição do grain e chaves substitutas
- Diagrama e dicionário de dados

### Pessoa 2: Engenheiro ETL
- Scripts de staging e OLTP
- Implementação do ETL (carga dimensional + SCD2)
- Validações de qualidade

### Pessoa 3: Analista SQL
- Consultas analíticas (5 queries)
- Análise exploratória inicial
- Documentação das queries

### Pessoa 4: Cientista de Dados / Visualização
- Gráficos e dashboards
- Relatório técnico (redação principal)
- Integração Python↔DuckDB

**Observação:** Todos devem participar de todas as etapas. Esta divisão é apenas uma sugestão de responsabilidade principal.

---

## 8. Recursos de Apoio

### Datasets Públicos Recomendados
- **Kaggle:** https://www.kaggle.com/datasets (filtrar por SQL/CSV)
- **Data.gov:** https://data.gov (dados governamentais EUA)
- **IBGE:** https://www.ibge.gov.br/estatisticas
- **Inep:** https://www.gov.br/inep (educação)
- **Our World in Data:** https://ourworldindata.org
- **UCI ML Repository:** https://archive.ics.uci.edu/ml

### Ferramentas de Diagrama
- **draw.io:** https://app.diagrams.net (gratuito)
- **Lucidchart:** https://www.lucidchart.com (free tier)
- **dbdiagram.io:** https://dbdiagram.io (SQL → diagrama)

### Templates e Exemplos
- Use o projeto Olist como referência estrutural
- Consulte `TUTORIAL.md` para comandos DuckDB da da ultima aula
- Ver `GUIA_DE_AULA_DW.md` para conceitos da ultima aula

---

## 9. Perguntas Frequentes (FAQ)

**P: Podemos usar um dataset pequeno?**  
R: Sim, mas deve ter pelo menos 10.000 linhas na fato e permitir análises temporais (dados de pelo menos 1 ano).

**P: É obrigatório usar DuckDB?**  
R: Não, mas é recomendado pela facilidade. PostgreSQL é aceito (ver pasta `postgres/` do projeto base).

**P: Podemos usar dados de API em vez de CSV?**  
R: Sim, mas documente o script de coleta e forneça os dados coletados (CSVs) no repositório.

**P: Quantas páginas deve ter o relatório?**  
R: Entre 8-12 páginas (incluindo diagramas). Foque em qualidade, não quantidade.

**P: E se o grupo tiver menos de 4 pessoas?**  
R: Grupos de 3 são aceitos (não reduz requisitos). Grupos de 2 ou individuais não são permitidos.

**P: Podemos reutilizar o código do Olist?**  
R: Sim, como base estrutural. Mas o domínio, dados e análises devem ser originais.


---

## 10. Critério de Nota Final

Nota = (Checkpoint 1 × 0,10) + (Checkpoint 2 × 0,15) + (Entrega Final × 0,65) + (Apresentação × 0,10) + Bônus

**Conversão:**
- 90-100: Excelente (A / 9-10)
- 75-89: Bom (B / 7-8,9)
- 60-74: Satisfatório (C / 6-6,9)
- <60: Insuficiente (D/F / <6)

---

## 11. Contato e Dúvidas

- **E-mail:** prof.rafaelgross@fatecjd.edu.br


Lembre-se: começar cedo é fundamental. Um bom planejamento (Checkpoint 1) garante 80% do sucesso do projeto!

Bom trabalho! 
