# Pipeline Bronze → Prata → Ouro — Dados de Cibersegurança

Projeto de Engenharia de Dados com construção das camadas **Bronze**, **Prata** e **Ouro (ML-Ready)**, análise de qualidade, EDA orientada a hipóteses, modelagem com Árvores de Decisão e refatoração com PySpark, seguindo boas práticas de governança de dados e anti-leakage.

---

## Estrutura do Projeto

```
projetinho/
├── input/                          # Arquivos brutos de entrada
│   ├── incidents_master.csv        # 850 incidentes (tabela principal)
│   ├── financial_impact.csv        # 778 registros de impacto financeiro
│   └── market_impact.csv           # 358 registros de impacto no mercado
├── data_lake/                      # Gerado automaticamente pelo notebook
│   ├── bronze/                     # Cada CSV salvo individualmente em Parquet
│   ├── silver/                     # Dataset unificado (left join) em Parquet
│   ├── metadata/
│   │   ├── ingestion_log.parquet
│   │   ├── bronze_quality_report.csv
│   │   ├── silver_transformation_log.parquet
│   │   ├── silver_validation_log.parquet
│   │   ├── silver_column_lineage.parquet
│   │   ├── silver_schema_lineage.parquet
│   │   ├── silver_transformations.md
│   │   └── pipeline_lineage.mmd
│   └── eda_notebook/
│       ├── 01_histograma_numerica.png
│       ├── 02_top_categorias.png
│       ├── 03_heatmap_correlacao.png
│       ├── 04_boxplot_numerica.png
│       ├── 05_nulos_por_coluna.png
│       ├── silver_preparado_<ts>.csv
│       ├── eda_metrics.json
│       └── eda_report_notebook.md
├── pipeline_principal.ipynb        # Notebook principal (executa tudo)
├── anti_leakage_checklist.md       # Checklist de colunas de risco
└── README.md
```

---

## Pré-requisitos

- Python 3.9+

Instale as dependências com:

```bash
pip install -r requirements.txt
```

---

## Pré-requisitos adicionais (2º Bimestre)

- **Java 11+** (necessário para PySpark) — verificar com `java -version`
- **PySpark** — instalado automaticamente: `pip install pyspark`

---

## Como Executar

### Execução completa via notebook principal

Abra `pipeline_principal.ipynb` no VS Code ou Jupyter e execute as células na ordem. O notebook realiza automaticamente:

1. **Parte 1 — Camada Bronze**: lê os 3 CSVs de `input/`, normaliza colunas e tipos, salva cada tabela em Parquet individual e registra metadados de ingestão (nome, linhas, hash SHA-256, timestamp).

2. **Parte 2 — Análise de Qualidade da Bronze**: verifica nulos (> 5%), duplicados, categorias inconsistentes (cardinalidade > 90%) e datas fora do padrão, gerando relatório automático por tabela.

3. **Parte 3 — Camada Silver**: faz **LEFT JOIN** das 3 tabelas por `incident_id` (base: `incidents_master`, 850 linhas), garantindo que nenhum incidente é perdido. Aplica limpeza, padronização, deduplicação, cria `label_final` e remove colunas de leakage. Gera logs de transformação, lineage de colunas e diagrama Mermaid.

4. **Parte 4 — EDA básica**: 5 visualizações gráficas (histograma, barras, heatmap, boxplot, nulos), testes estatísticos (Pearson, t-test) e baseline de classificação (Regressão Logística). Exporta artefatos (CSV, JSON, relatório Markdown).

5. **Parte 5 — EDA orientada a hipóteses** *(2º bimestre)*: 3 hipóteses formais com ≥ 6 gráficos — distribuição de variáveis-chave, análise de outliers (IQR), matriz de correlação, análise por vetor de ataque, tipo de empresa e setor. Cada visualização acompanhada de interpretação orientada a decisão.

6. **Parte 6 — Camada Ouro** *(2º bimestre)*: Remoção de leakage residual, 2 estratégias de imputação (mediana + moda), IQR-clip em 2 colunas, OrdinalEncoder + OneHotEncoder, RobustScaler, padrão fit/transform (sem leakage de teste no treino), salvo em Parquet, tabela de transformações documentada.

7. **Parte 7 — Modelagem ML** *(2º bimestre)*: 2 Árvores de Decisão com configurações distintas (depth=5/gini vs depth=10/entropy), divisão 75/25 estratificada, métricas: Acurácia, Precisão, Recall, F1, Matriz de Confusão, visualização da árvore, comparação Silver vs Gold.

8. **Parte 8 — PySpark** *(2º bimestre)*: Refatoração de 2 etapas do pipeline com PySpark local — LEFT JOIN, groupBy+agg, Window function (RANK por setor), escrita em Parquet, comparação de tempo vs Pandas.

Saída gerada em `data_lake/`:
- `bronze/tabela=<nome>/data_carga=<data>/<arquivo>.parquet`
- `silver/tabela=<nome>/data_processamento=<data>/<arquivo>_silver.parquet`
- `gold/tabela=ml_ready/<arquivo>.parquet` *(Camada Ouro ML-Ready)*
- `gold/tabela=pyspark_agg/<arquivo>.parquet` *(Agregação PySpark)*
- `gold/tabela=pyspark_ranked/<arquivo>.parquet` *(Ranking PySpark)*
- `metadata/` (logs, lineage, relatório de qualidade, tabela de transformações Gold)
- `eda_notebook/` (gráficos, métricas, relatório)

---

## Criação do Label

O `label_final` é construído a partir da coluna `abnormal_return_1d` (retorno anormal 1 dia após o incidente):
- **`label_final = 1`** → retorno anormal negativo (impacto no mercado)
- **`label_final = 0`** → retorno anormal não-negativo (sem impacto relevante)

A coluna `abnormal_return_1d` é usada para criar o label e depois removida como leakage.

---

## Data Lineage

O arquivo `data_lake/metadata/pipeline_lineage.mmd` contém o diagrama de lineage no formato Mermaid. Para visualizar, cole o conteúdo em [mermaid.live](https://mermaid.live) ou use a extensão Mermaid no VS Code.

Fluxo resumido:

```
Arquivos Brutos (input/)
    ├── incidents_master.csv (850 linhas)
    ├── financial_impact.csv (778 linhas)
    └── market_impact.csv (358 linhas)
          └─► Bronze (cada CSV → Parquet individual)
                  └─► Validação de qualidade por tabela
                          └─► LEFT JOIN por incident_id (base: incidents_master)
                                  └─► Prata (limpeza, datas, dedup, label, leakage)
                                            └─► EDA orientada a hipóteses (3 hipóteses, 6+ gráficos)
                                                      └─► Ouro / ML-Ready (encoding, scaling, outliers, fit/transform)
                                                                └─► Modelagem ML (2 Árvores de Decisão, Silver vs Gold)
                                                                          └─► PySpark (join, groupBy, window, Parquet)
```

---

## Sobre o Dataset

Três tabelas de cibersegurança conectadas por `incident_id`:

- **`incidents_master.csv`** — Tabela principal com dados do incidente: empresa, vetor de ataque, datas, dados comprometidos, score de qualidade.
- **`financial_impact.csv`** — Impacto financeiro: perdas diretas, custos de recuperação, multas, ransomware.
- **`market_impact.csv`** — Impacto no mercado: retornos anormais, CAR, volume, volatilidade pré/pós incidente.

Consulte `anti_leakage_checklist.md` para o mapeamento completo de risco de vazamento de dados.
