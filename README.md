# Pipeline Bronze → Prata — Dados de Cibersegurança

Projeto de Engenharia de Dados com construção das camadas **Bronze** e **Prata**, análise de qualidade e EDA, seguindo boas práticas de governança de dados.

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

## Como Executar

### Execução completa via notebook principal

Abra `pipeline_principal.ipynb` no VS Code ou Jupyter e execute as células na ordem. O notebook realiza automaticamente:

1. **Parte 1 — Camada Bronze**: lê os 3 CSVs de `input/`, normaliza colunas e tipos, salva cada tabela em Parquet individual e registra metadados de ingestão (nome, linhas, hash SHA-256, timestamp).

2. **Parte 2 — Análise de Qualidade da Bronze**: verifica nulos (> 5%), duplicados, categorias inconsistentes (cardinalidade > 90%) e datas fora do padrão, gerando relatório automático por tabela.

3. **Parte 3 — Camada Silver**: faz **LEFT JOIN** das 3 tabelas por `incident_id` (base: `incidents_master`, 850 linhas), garantindo que nenhum incidente é perdido. Aplica limpeza, padronização, deduplicação, cria `label_final` e remove colunas de leakage. Gera logs de transformação, lineage de colunas e diagrama Mermaid.

4. **Parte 4 — EDA**: 5 visualizações gráficas (histograma, barras, heatmap, boxplot, nulos), testes estatísticos (Pearson, t-test) e baseline de classificação (Regressão Logística). Exporta artefatos (CSV, JSON, relatório Markdown).

Saída gerada em `data_lake/`:
- `bronze/tabela=<nome>/data_carga=<data>/<arquivo>.parquet`
- `silver/tabela=<nome>/data_processamento=<data>/<arquivo>_silver.parquet`
- `metadata/` (logs, lineage, relatório de qualidade)
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
                                            └─► Dataset final para ML (850 linhas)
```

---

## Sobre o Dataset

Três tabelas de cibersegurança conectadas por `incident_id`:

- **`incidents_master.csv`** — Tabela principal com dados do incidente: empresa, vetor de ataque, datas, dados comprometidos, score de qualidade.
- **`financial_impact.csv`** — Impacto financeiro: perdas diretas, custos de recuperação, multas, ransomware.
- **`market_impact.csv`** — Impacto no mercado: retornos anormais, CAR, volume, volatilidade pré/pós incidente.

Consulte `anti_leakage_checklist.md` para o mapeamento completo de risco de vazamento de dados.
