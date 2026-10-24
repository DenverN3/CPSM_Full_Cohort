# TCGA-ACC RPPA CPSM/MTLR run review
Goal: identify RPPA antibody features prognostic for PFI in TCGA-ACC and assess MTLR model performance.

## Run facts (from qc.json)
Platform: rppa. Endpoint: PFI, selected by the auto rule after no candidate met thresholds; recorded candidates were OS 14/46 events and PFI 26/46 events. Analysis n=45 with 25 events; train n=36 with 21 events and test n=9 with 4 events. Underpowered: true. Covariates used: Age, gender, stage_group; none dropped. Stage I was merged into Other because it was rare. CPSM 1.4.0. Patches: mtlr_model3_median_assignment already_fixed; univariate_median_cutoff already_fixed; univariate_empty_result_guard applied(1,1). Seed 123456; train fraction 0.8.

## Checklist
- FLAG: status is partial because qc.json records one error-level issue, although Models 1 to 4 were produced.
- PASS: all three required patches are applied or already fixed.
- FLAG: auto endpoint selected PFI after no candidate met thresholds; the log records PFI with 26 events and rate 0.565.
- PASS: n_train + n_test = 36 + 9 = 45 samples; 21 train events and 4 test events are reported.
- FLAG: underpowered with only 4 held-out events; performance and replication estimates are unstable.
- PASS: no covariates were dropped; rare Stage I was merged into Other and is disclosed.
- PASS: CPSM median-split minority-fraction median is 0.50.
- FLAG: 54 antibody features violate proportional hazards; their hazard-ratio interpretation requires caution.
- PASS: no model has Test C-index below 0.55 and no train minus test gap exceeds 0.15.
- PASS: held-out direction agreement among FDR-significant antibody features is 36/45 = 80.0%.
- PASS: qc.json platform is rppa and the run is stored under results/ACC/rppa/.
- PASS: 458 features were modelled, consistent with an RPPA panel and not a 5,000-gene RNA run.
- FLAG: 29/487 antibodies were dropped for greater than 30% missingness, and 480 values were median-imputed per antibody using whole-cohort medians. This imputation is a minor information leak.
- PASS: n_genes_removed_by_cpsm_filter is 0, as expected for proteomics.
- PASS: feature rows identify antibodies and leave Ensembl_ID empty (NA) by design.
- FAIL: qc.json and the log report all 458 feature columns as missing at combined_input and not explained by the CPSM near-constant filter. The log line is: "ERROR [genes] combined_input: 458 gene columns missing NOT explained by the CPSM near-constant filter". The Test_Norm_data feature set also differs from train. Per AGENTS.md, do not rerun or scale until this trace failure is diagnosed.

## Models
| Model | Train C-index | Test C-index | Train IBS | Test IBS |
|---|---:|---:|---:|---:|
| Model1 | 0.71 | 0.86 | 0.210 | 0.287 |
| Model2 | 0.80 | 0.90 | 0.183 | 0.172 |
| Model3 | 0.78 | 0.95 | 0.165 | 0.176 |
| Model4 | 0.80 | 0.81 | 0.136 | 0.185 |

Best model: Model3, test C-index 0.95. This estimate is flagged because the test set contains only 4 events and the run has an unresolved feature-column trace error.

## Prognostic antibody features
Continuous per-feature Cox results: 141 at p<0.05; 45 at FDR<0.05; 36/45 FDR-significant features agree in held-out direction. PH violations: 54.

Top 15 antibody features by training p:
| Antibody target | HR per SD (95% CI) | P_adj | Test HR | Direction agrees | LASSO selected |
|---|---|---:|---:|---|---|
| GCLM | 0.295 (0.161-0.538) | 0.0234 | 0.135 | true | true |
| ERK2 | 2.490 (1.558-3.979) | 0.0234 | 18.789 | true | true |
| ARID1A | 2.678 (1.608-4.461) | 0.0234 | 3.585 | true | false |
| BCL2A1 | 2.105 (1.356-3.269) | 0.0234 | 1.571 | true | false |
| BCLXL | 0.426 (0.273-0.665) | 0.0234 | 0.593 | true | false |
| SDHA | 0.389 (0.247-0.613) | 0.0234 | 0.117 | true | false |
| SOD2 | 0.375 (0.235-0.598) | 0.0234 | 0.152 | true | false |
| TSC1 | 0.422 (0.266-0.670) | 0.0234 | 0.104 | true | false |
| UQCRC2 | 0.418 (0.259-0.673) | 0.0234 | 0.117 | true | false |
| CD171 | 2.610 (1.512-4.506) | 0.0315 | 2.618 | true | false |
| MSI2 | 2.343 (1.408-3.898) | 0.0347 | 2.226 | true | false |
| P70S6K_pT389 | 0.376 (0.210-0.672) | 0.0347 | 0.043 | true | false |
| DNA-Ligase-IV | 1.898 (1.293-2.786) | 0.0347 | 1.348 | true | false |
| DNMT1 | 2.146 (1.353-3.404) | 0.0347 | 1.496 | true | false |
| CDK1_pT14 | 1.858 (1.276-2.706) | 0.0347 | 0.597 | false | false |

Antibody-level hazard ratios are protein-array measurements. They are not gene-level RNA hazard ratios and are not merged with RNA results.

## Issues and diagnoses
1. Feature-column trace FAIL. Root cause is not yet established. Evidence is the explicit error that 458 combined-input feature columns are missing and are not explained by the near-constant filter, plus the train/test feature-set mismatch. Required action: diagnose the feature naming and trace logic before any rerun or scale-out; do not change statistical settings speculatively.
2. Underpowered held-out evaluation. Evidence: 21 train events and 4 test events. Action: retain as a pilot caveat and do not interpret the 0.95 C-index as stable.
3. Missingness handling. Evidence: 29/487 antibodies dropped and 480 values whole-cohort median-imputed. Action: disclose the minor leak; no ad hoc reprocessing.
4. Endpoint warning. Evidence: no auto candidate met thresholds; PFI was chosen with 26/46 candidate events. Action: report the selection and event counts.
