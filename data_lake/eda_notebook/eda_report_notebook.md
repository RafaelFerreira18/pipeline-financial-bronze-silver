# Relatório EDA — Pipeline Bronze → Silver
- Gerado em: 2026-05-27T00:15:10
- Linhas: 8500
- Colunas: 56
- Datasets unificados: incidents_master + financial_impact + market_impact

## Gráficos Gerados
- 01_histograma_numerica.png
- 02_distribuicao_label.png
- 03_heatmap_correlacao.png
- 04_boxplot_numerica.png
- 05_nulos_por_coluna.png

## Observações e Próximos Passos
- Validar possíveis outliers nos boxplots/histogramas.
- Revisar distribuição do label para equilíbrio de classes.
- Priorizar variáveis com correlação consistente para baseline de ML.
- Definir estratégia formal de imputação e tratamento de outliers.
- Testar modelos baseline adicionais (árvore, random forest, XGBoost).