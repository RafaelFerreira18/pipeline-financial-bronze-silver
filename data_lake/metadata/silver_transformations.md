## Execução 2026-04-16T00:17:43.728580
- Silver run id: d5d910df-e7b5-423a-96a8-2df086338a22
- Tabelas Bronze: financial_impact, incidents_master, market_impact
- Join: LEFT JOIN por 'incident_id' (base: incidents_master)
- Linhas entrada (pós-join): 850
- Linhas saída: 850
- Duplicidades removidas: 0 (chaves=['incident_id'])
- Nulls normalizados: 9857
- Colunas limpas: industry_secondary, stock_ticker, attack_vector_secondary, attack_chain, attributed_group, attribution_confidence, data_type, data_source_secondary, review_flag, notes, direct_loss_method, ransom_source, total_loss_method, cpi_index_used, notes_fin, created_at_fin, updated_at_fin, stock_ticker_mar, sector_index, earnings_announcement_within_7d, notes_mar, created_at_mar, updated_at_mar
- Regras de data: nenhuma
- Label: label criado a partir de 'abnormal_return_1d' (valor < 0.0 → 1 = impacto negativo)
- Colunas removidas por leakage: 26
- Saída Silver: data_lake\silver\tabela=joined\data_processamento=2026-04-16\joined_silver_20260416_001743.parquet

