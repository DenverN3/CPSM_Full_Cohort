# TCGA-LUAD RPPA CPSM/MTLR run review
Goal: identify RPPA antibody features prognostic for OS in TCGA-LUAD and assess MTLR model performance.

## Run facts (from qc.json)
Platform: rppa. Endpoint: configured OS for this locked cohort; candidate record 141/354 events. Analysis n=349 with 140 events; train n=279 with 113 events and test n=70 with 27 events. Underpowered: false. Covariates used: Age, gender, stage_group; none dropped. CPSM 1.4.0. Patches: mtlr_model3_median_assignment already_fixed; univariate_median_cutoff already_fixed; univariate_empty_result_guard applied(1,1). Seed 123456; train fraction 0.8.

## Checklist
- FLAG: status is partial because qc.json records two error-level issues; Models 1 and 4 were produced, while Models 2 and 3 were skipped.
- PASS: all three required patches are applied or already fixed.
- PASS: endpoint is configured OS, matching the locked LUAD registry setting.
- PASS: n_train + n_test = 279 + 70 = 349 samples; 113 train events and 27 test events are reported.
- PASS: underpowered is false.
- PASS: no covariates were dropped.
- PASS: CPSM median-split minority-fraction median is 0.498.
- FLAG: 34 antibody features violate proportional hazards; their hazard-ratio interpretation requires caution.
- FLAG: Model4 Test C-index is 0.53, below 0.55, and its train-test C-index gap is 0.19, above 0.15 and consistent with possible overfit.
- PASS: Model1 has Train/Test C-indices 0.70/0.62 with gap 0.08.
- FLAG: no FDR-significant antibody features were found, so held-out direction agreement is 0/0 and not estimable.
- PASS: qc.json platform is rppa and the run is stored under results/LUAD/rppa/.
- PASS: 455 features were modelled, consistent with an RPPA panel and not a 5,000-gene RNA run.
- FLAG: 32/487 antibodies were dropped for greater than 30% missingness, and 2,629 values were median-imputed per antibody using whole-cohort medians. This imputation is a minor information leak.
- PASS: n_genes_removed_by_cpsm_filter is 0, as expected for proteomics.
- PASS: feature rows identify antibodies and leave Ensembl_ID empty by design.
- FAIL: qc.json and the log report all 455 feature columns as missing at combined_input and not explained by the CPSM near-constant filter. The log line is: "ERROR [genes] combined_input: 455 gene columns missing NOT explained by the CPSM near-constant filter". The Test_Norm_data feature set also differs from train. Per AGENTS.md, do not rerun or scale until this trace failure is diagnosed.
- FLAG: LASSO failed with "x should be a matrix with 2 or more columns" because no stable panel was available. Models 2 and 3 were legitimately skipped. Do not change n_repeats, alpha, or frequency threshold without approval.

## Models
| Model | Train C-index | Test C-index | Train IBS | Test IBS |
|---|---:|---:|---:|---:|
| Model1 | 0.70 | 0.62 | 0.177 | 0.237 |
| Model4 | 0.72 | 0.53 | 0.174 | 0.252 |

Best model: Model1, test C-index 0.62. Model4 is flagged for low test discrimination and a 0.19 train-test gap. Models 2 and 3 are absent because LASSO was unavailable.

## Prognostic antibody features
Continuous per-feature Cox results: 31 at p<0.05; 0 at FDR<0.05; held-out agreement among FDR-significant features is not estimable. PH violations: 34.

Top 15 antibody features by training p:
| Antibody target | HR per SD (95% CI) | P_adj | Test HR | Direction agrees | LASSO selected |
|---|---|---:|---:|---|---|
| CD49B | 1.320 (1.094-1.593) | 0.572 | 1.278 | true | false |
| CYCLINB1 | 1.273 (1.081-1.499) | 0.572 | 1.106 | true | false |
| TUBERIN_pT1462 | 0.758 (0.622-0.923) | 0.572 | 1.150 | false | false |
| GSK3_pS9 | 0.769 (0.638-0.929) | 0.572 | 1.438 | false | false |
| PAI1 | 1.287 (1.073-1.544) | 0.572 | 1.109 | true | false |
| GSK3ALPHABETA_pS21S9 | 0.780 (0.647-0.939) | 0.572 | 1.319 | false | false |
| ADAR1 | 1.296 (1.066-1.576) | 0.572 | 0.966 | false | false |
| FOXM1 | 1.252 (1.055-1.486) | 0.572 | 1.015 | true | false |
| VASP | 1.265 (1.051-1.522) | 0.643 | 0.965 | false | false |
| PDCD4 | 0.777 (0.636-0.951) | 0.643 | 0.850 | true | false |
| PKCPANBETAII_pS660 | 0.791 (0.650-0.962) | 0.643 | 1.258 | false | false |
| MTOR_pS2448 | 0.807 (0.672-0.968) | 0.643 | 1.343 | false | false |
| CYCLINE1 | 1.207 (1.026-1.421) | 0.643 | 0.998 | false | false |
| MIG6 | 1.233 (1.025-1.483) | 0.643 | 0.831 | false | false |
| PDL1 | 1.205 (1.022-1.421) | 0.643 | 0.852 | false | false |

Antibody-level hazard ratios are protein-array measurements. They are not gene-level RNA hazard ratios and are not merged with RNA results.

## Issues and diagnoses
1. Feature-column trace FAIL. Root cause is not yet established. Evidence is the explicit error that 455 combined-input feature columns are missing and are not explained by the near-constant filter, plus the train/test feature-set mismatch. Required action: diagnose the feature naming and trace logic before any rerun or scale-out; do not change statistical settings speculatively.
2. LASSO unavailable. Evidence: "x should be a matrix with 2 or more columns", lasso_selected=0, and Models 2 and 3 skipped. The playbook treats this as no stable features; no tuning changes are allowed without approval.
3. Model4 performance warning. Evidence: Train/Test C-index 0.72/0.53, gap 0.19. Action: flag possible overfit and retain Model1 as the best model.
4. Missingness handling. Evidence: 32/487 antibodies dropped and 2,629 values whole-cohort median-imputed. Action: disclose the minor leak; no ad hoc reprocessing.
