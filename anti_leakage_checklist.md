# Checklist Anti-Leakage — Dataset Unificado de Cibersegurança

Avaliação de risco de *data leakage* para as colunas do dataset unificado (left join de `incidents_master`, `financial_impact` e `market_impact` por `incident_id`).

> **Leakage** ocorre quando uma feature usa informações que só estariam disponíveis **após** o evento-alvo, tornando o modelo inutilizável em produção.

## Legenda

| Status | Significado |
|--------|-------------|
| ✅ Segura | Disponível **antes** ou **no momento** do evento. |
| ⚠️ Atenção | Requer validação de temporalidade. |
| ❌ Leakage | Gerada **após** o evento — removida no pipeline. |

---

## incidents_master

| Coluna | Status | Justificativa |
|--------|--------|---------------|
| `incident_id` | ⚠️ | Identificador — não é feature preditiva. |
| `company_name` | ✅ | Informação pré-existente. |
| `company_revenue_usd` | ✅ | Receita antes do incidente. |
| `country_hq` | ✅ | País da sede. |
| `industry_primary` | ✅ | Setor primário. |
| `industry_secondary` | ✅ | Setor secundário. |
| `employee_count` | ✅ | Tamanho da empresa. |
| `is_public_company` | ✅ | Status público. |
| `stock_ticker` | ✅ | Ticker. |
| `incident_date` | ✅ | Data do incidente. |
| `incident_date_estimated` | ⚠️ | Indica se a data é estimada. |
| `discovery_date` | ⚠️ | Pode ser posterior ao incidente. |
| `disclosure_date` | ⚠️ | Data de divulgação — posterior ao incidente. |
| `attack_vector_primary` | ✅ | Vetor de ataque. |
| `attack_vector_secondary` | ✅ | Vetor secundário. |
| `attack_chain` | ✅ | Cadeia de ataque. |
| `attributed_group` | ✅ | Grupo atribuído. |
| `attribution_confidence` | ✅ | Confiança da atribuição. |
| `data_compromised_records` | ⚠️ | Contagem pode ser posterior. |
| `data_type` | ✅ | Tipo de dado comprometido. |
| `systems_affected` | ✅ | Sistemas afetados. |
| `downtime_hours` | ❌ | Tempo de inatividade — só conhecido após. |
| `data_source_primary` | ✅ | Fonte de dados. |
| `data_source_secondary` | ✅ | Fonte secundária. |
| `data_source_type` | ✅ | Tipo de fonte. |
| `confidence_tier` | ✅ | Nível de confiança. |
| `quality_score` | ✅ | Score de qualidade do registro. |
| `quality_grade` | ✅ | Grade de qualidade. |
| `review_flag` | ⚠️ | Flag de revisão. |
| `notes` | ❌ | Pode conter informações pós-evento. |
| `created_at` | ❌ | Metadado temporal do registro. |
| `updated_at` | ❌ | Metadado temporal. |

## financial_impact

| Coluna | Status | Justificativa |
|--------|--------|---------------|
| `direct_loss_usd` | ❌ | Perda direta — só conhecida após. |
| `direct_loss_method` | ⚠️ | Método de cálculo. |
| `ransom_demanded_usd` | ⚠️ | Resgate demandado — pode ser conhecido cedo. |
| `ransom_paid_usd` | ❌ | Resgate pago — decisão posterior. |
| `ransom_source` | ⚠️ | Fonte da informação. |
| `recovery_cost_usd` | ❌ | Custo de recuperação — posterior. |
| `legal_fees_usd` | ❌ | Custos legais — posterior. |
| `regulatory_fine_usd` | ❌ | Multa regulatória — posterior. |
| `insurance_payout_usd` | ❌ | Pagamento de seguro — posterior. |
| `total_loss_usd` | ❌ | Perda total — posterior. |
| `total_loss_method` | ❌ | Método de cálculo. |
| `total_loss_lower_bound` | ❌ | Limite inferior — posterior. |
| `total_loss_upper_bound` | ❌ | Limite superior — posterior. |
| `inflation_adjusted_usd` | ❌ | Ajuste inflacionário — posterior. |
| `cpi_index_used` | ❌ | Índice CPI — posterior. |
| `notes` | ❌ | Pode conter informações pós-evento. |
| `created_at` | ❌ | Metadado temporal. |
| `updated_at` | ❌ | Metadado temporal. |

## market_impact

| Coluna | Status | Justificativa |
|--------|--------|---------------|
| `stock_ticker` | ✅ | Identificação do ativo. |
| `price_7d_before` | ✅ | Preço 7 dias antes — pré-evento. |
| `price_disclosure_day` | ⚠️ | Preço no dia da divulgação. |
| `price_1d_after` | ❌ | Preço 1 dia após. |
| `price_7d_after` | ❌ | Preço 7 dias após. |
| `price_30d_after` | ❌ | Preço 30 dias após. |
| `volume_avg_30d_baseline` | ✅ | Volume médio pré-evento. |
| `volume_disclosure_day` | ⚠️ | Volume no dia da divulgação. |
| `sector_index` | ✅ | Índice do setor. |
| `sector_return_same_period` | ❌ | Retorno do setor no período. |
| `abnormal_return_1d` | ❌ | **Usado para criar `label_final`, depois removido.** |
| `abnormal_return_7d` | ❌ | Retorno anormal 7 dias. |
| `abnormal_return_30d` | ❌ | Retorno anormal 30 dias. |
| `car_neg1_to_pos1` | ❌ | CAR -1 a +1 dia. |
| `car_0_to_7` | ❌ | CAR 0-7 dias. |
| `car_0_to_30` | ❌ | CAR 0-30 dias. |
| `car_0_to_90` | ❌ | CAR 0-90 dias. |
| `t_statistic_1d` | ❌ | Estatística t 1 dia. |
| `p_value_1d` | ❌ | P-valor 1 dia. |
| `t_statistic_30d` | ❌ | Estatística t 30 dias. |
| `p_value_30d` | ❌ | P-valor 30 dias. |
| `earnings_announcement_within_7d` | ⚠️ | Parcialmente pós-evento. |
| `market_cap_at_disclosure` | ✅ | Cap. de mercado na divulgação. |
| `volume_ratio_disclosure` | ⚠️ | Razão de volume na divulgação. |
| `pre_incident_volatility_30d` | ✅ | Volatilidade pré-evento. |
| `post_incident_volatility_30d` | ❌ | Volatilidade pós-evento. |
| `days_to_price_recovery` | ❌ | Dias até recuperação. |
| `notes` | ❌ | Campo livre. |
| `created_at` | ❌ | Metadado temporal. |
| `updated_at` | ❌ | Metadado temporal. |

---

## Resumo

| Categoria | Quantidade |
|-----------|-----------|
| ✅ Seguras | ~25 |
| ⚠️ Atenção | ~12 |
| ❌ Leakage (removidas no pipeline) | ~35 |

**Label:** `label_final = (abnormal_return_1d < 0)` → 1 = impacto negativo, 0 = sem impacto. A coluna-fonte é removida como leakage após criação do label.

**Nota:** Colunas `notes`, `created_at`, `updated_at` existem nas 3 tabelas — após o join com sufixo, todas as variantes são removidas.
