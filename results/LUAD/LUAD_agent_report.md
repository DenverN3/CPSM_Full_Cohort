# TCGA-LUAD CPSM/MTLR run review
Goal: identify genes prognostic for OS in TCGA-LUAD and assess MTLR model performance.

## Run facts (from qc.json)

No run facts are available because `results/LUAD/qc.json` was not produced. LUAD is a locked cohort configured for OS.

## Checklist

- FAIL, run status: no `qc.json` exists, so the required `success` or explained `partial` status is unavailable.
- FAIL, patches: patch application cannot be verified at cohort run start because no LUAD run log or `qc.json` was produced. Environment preflight did pass all three patch checks.
- FAIL, endpoint: the registry specifies OS for locked LUAD, but no run output records the applied endpoint.
- FAIL, split and power: sample counts, event counts, split arithmetic, and underpowered status are unavailable.
- FAIL, covariates: used and dropped covariates and clinical information issues are unavailable.
- FAIL, median-cutoff patch behavior: `cpsm_median_split_minority_fraction_median` is unavailable.
- FAIL, proportional hazards: `n_genes_ph_violation` is unavailable.
- FAIL, model performance: no Model_Performance output is available.
- FAIL, held-out FDR replication: `n_genes_fdr05` and `n_genes_fdr05_test_direction_agrees` are unavailable.

## Models

No model table was produced. No best model or test C-index can be reported.

## Prognostic genes

No LUAD gene survival table or `qc.json` was produced, so counts, held-out direction replication, PH violations, and the top 15 genes cannot be reported.

## Issues and diagnoses

- The launcher reported `LUAD launched (all) pid 54640`, but at poll 1 LUAD was no longer running and had status `not_run`.
- Evidence on disk is limited to an empty `results/LUAD/logs/launcher_last.out` file of 0 bytes. There is no stage log and no `qc.json`.
- Root cause cannot be established from a proving log line because the background process exited before emitting any diagnostic output. Per the operating contract, no speculative retry or configuration change was attempted.
- Recommended action: inspect the launch mechanism or execution environment with user approval, then rerun LUAD unchanged. LUAD is locked, so its endpoint and covariates must not be altered.
