# TCGA-CHOL CPSM/MTLR run review
Goal: identify genes prognostic for PFI in TCGA-CHOL and assess MTLR model performance.

## Run facts (from qc.json)

- Endpoint: PFI. Rule: auto. No auto candidate met the configured thresholds, so the engine selected PFI, which had 20 events among 35 endpoint-eligible samples (rate 0.571); OS had 18 among 35.
- Analysis set: n = 34, events = 20. Train: n = 27, events = 18. Test: n = 7, events = 2.
- Underpowered: true.
- Covariates used: Age, gender, stage_group. Dropped covariates: none.
- CPSM version: 1.4.0.
- Patches: mtlr_model3_median_assignment = already_fixed; univariate_median_cutoff = already_fixed; univariate_empty_result_guard = applied(1,1).
- CPSM near-constant filter removed 1 gene, ENSG00000211930.1, from Train_Norm_data because its training modal-value fraction was 0.852.

## Checklist

- PASS, run status: qc.json records status `success`, four MTLR models, and zero error-level issues.
- PASS, patches: all three patches are `already_fixed` or `applied(1,1)`.
- FLAG, endpoint: registry rule is `auto`; PFI was selected with recorded candidate counts, but no candidate met thresholds. Evidence: PFI 20/35 events versus OS 18/35.
- FLAG, split and power: 27 train + 7 test = 34 samples, with 18 train and only 2 test events; `underpowered` is true.
- FLAG, clinical information: no configured covariate was dropped, but Stage III occurred in the test set and was unseen in training, so it was converted to NA.
- PASS, median-cutoff patch behavior: median minority fraction is 0.4815, within the expected approximately 0.45 to 0.50 range.
- FLAG, proportional hazards: 364 genes violated the PH assumption; their hazard ratios require caution.
- FLAG, model discrimination: all test C-indices are below 0.55. Model2 has a train-test gap of 0.27, Model3 0.32, and Model4 0.36, each above 0.15 and suggestive of overfit. Model1 has a 0.14 gap.
- FLAG, held-out FDR replication: 0 genes passed FDR < 0.05, so the direction replication rate is not estimable rather than evidence of failed replication (0/0).

## Models

| Model | Train C-index | Train Mean MAE | Train Median MAE | Train IBS | Test C-index | Test Mean MAE | Test Median MAE | Test IBS |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Model1 | 0.58 | 8.04 | 5.29 | 0.203 | 0.44 | 16.19 | 12.22 | 0.373 |
| Model2 | 0.72 | 8.02 | 5.41 | 0.098 | 0.45 | 27.76 | 13.53 | 0.450 |
| Model3 | 0.76 | 6.91 | 4.29 | 0.141 | 0.44 | 14.80 | 10.25 | 0.376 |
| Model4 | 0.80 | 7.99 | 7.61 | 0.161 | 0.44 | 16.68 | 10.97 | 0.439 |

The engine identifies Model2 as best by test C-index, at 0.45. Its discrimination is below the 0.55 flag threshold, and its train-test gap is 0.27.

## Prognostic genes

The continuous per-gene Cox screen found 107 genes at unadjusted p < 0.05 and 0 at FDR < 0.05. Held-out FDR direction replication is therefore not estimable (0 agreements among 0 FDR-significant genes). There were 364 PH violations. LASSO selected 2 genes.

Top 15 genes by training p-value:

| Symbol | HR per SD (95% CI) | P_adj | Test HR | Direction agrees | LASSO selected |
|---|---|---:|---:|---|---|
| KYNU | 3.135 (1.695 to 5.799) | 0.9966 | 1.802 | Yes | No |
| ADGRG7 | 2.828 (1.549 to 5.163) | 0.9966 | 0.648 | No | No |
| MIR4477B | 2.107 (1.324 to 3.354) | 0.9966 | 0.771 | No | No |
| GDNF | 2.113 (1.312 to 3.404) | 0.9966 | 90.415 | Yes | No |
| ACE2 | 2.267 (1.337 to 3.844) | 0.9966 | 0.00194 | No | No |
| GIPC2 | 0.472 (0.282 to 0.790) | 0.9966 | 1.159 | No | Yes |
| AGTR1 | 2.224 (1.263 to 3.914) | 0.9966 | 1.883 | Yes | No |
| SIRPG | 2.037 (1.220 to 3.401) | 0.9966 | 0.180 | No | No |
| KIF20A | 2.129 (1.233 to 3.677) | 0.9966 | 1.069 | Yes | No |
| UGT2B11 | 1.826 (1.180 to 2.826) | 0.9966 | 1.728 | Yes | No |
| MLLT11 | 2.000 (1.203 to 3.325) | 0.9966 | 1.148 | Yes | No |
| MTRNR2L1 | 1.760 (1.156 to 2.679) | 0.9966 | 0.987 | No | No |
| LINC01615 | 1.803 (1.162 to 2.795) | 0.9966 | 0.861 | No | No |
| PBX1 | 0.503 (0.299 to 0.847) | 0.9966 | 1.768 | No | No |
| IFITM3P2 | 2.173 (1.205 to 3.917) | 0.9966 | 0.512 | No | No |

## Issues and diagnoses

- Underpowered split: the run has 18 training events and 2 test events. This is expected for this small cohort and makes test C-indices and test hazard ratios unstable. No settings were changed.
- Endpoint threshold warning: neither auto candidate met thresholds. The deterministic rule selected PFI because it had the larger event count, 20 versus 18 for OS. No endpoint change was made.
- Weak and potentially overfit model performance: every test C-index is 0.44 to 0.45, and Models 2 to 4 have train-test gaps from 0.27 to 0.36. No learner was substituted and no tuning settings were changed.
- Stage III was unseen in training and present in test, so the engine mapped it to NA. No covariate configuration was changed.
- One gene was removed by the expected CPSM 1.4.0 near-constant filter. The qc.json gene trace explains the missing column, so this is not an unexplained gene-column failure.
- The analysis log reports 50 or more R warnings without listing them individually. qc.json records zero error-level issues, but the warnings should be retained as a caveat if CHOL is revisited.
