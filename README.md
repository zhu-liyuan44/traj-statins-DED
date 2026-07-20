# Statin Initiation Timing and Diabetic Eye Disease

Code for a longitudinal prediction and causal analysis of **statin initiation timing** and the risk of **diabetic eye disease (DED)** in people with diabetes and multiple long-term conditions (MLTCs), using **UK Biobank** and **CPRD**.

> **Status:** Research code accompanying a manuscript in preparation. This repository contains analysis code only. **No data are included** (see [Data](#data)).



## Background

People with diabetes frequently develop multiple long-term conditions, with cardiovascular disease among the most common. Statins are widely prescribed to reduce cardiovascular risk, but their longitudinal relationship with diabetes complications such as retinopathy is less well understood, and in particular, whether the *timing* of statin initiation relative to diabetes onset is associated with the risk of sight-threatening eye disease.

This project asks a question of treatment timing rather than treatment presence: **is later statin initiation associated with a higher risk of diabetic eye disease, and is there a window during disease progression in which earlier intervention may be beneficial?** It also examines how sex, ethnicity and socioeconomic status relate to the burden and progression of MLTCs.

The causal estimates here are derived from observational data and are **association-level** - see [Limitations](#limitations).



## Data

**No data are included in this repository.** All analyses use access-controlled datasets that cannot be shared:

- **UK Biobank** - primary cohort. Access is application-based via the UK Biobank Access Management System. Main analytic cohort: **13,391** participants with diabetes.
- **CPRD** (Clinical Practice Research Datalink) - replication cohort using linked primary care records. Access is licensed and governed by CPRD approvals. Replication cohort: **131,310** patients (including deaths) / **108,139** patients (excluding deaths).

To reproduce these analyses you must hold your own approved access to the relevant dataset(s).

**Codelists / phenotype definitions** used in this work are published openly in separate repositories:
- Long-term conditions (MULTIPLY Initiative): https://github.com/zhu-liyuan44/MULTIPLY-Initiative
- CPRD Aurum codelists (Exeter Diabetes team): https://github.com/zhu-liyuan44/CPRD-Codelists

Statin exposure and drug features were defined using **BNF** codes; CPRD phenotyping additionally uses **Read** and **SNOMED** codes.



## Methods

**Cohort construction and phenotyping.** Diabetes cases, statin exposure, comorbidities and outcomes were derived from longitudinal electronic health records (baseline demographics plus medical history up to the study end).

**Exposure - timing of statin initiation.** The key exposure is the interval between diabetes diagnosis and statin initiation (`days_from_diabetes_to_statins`). For grouped analyses, **early initiation** is defined as starting a statin **within 180 days** of diabetes diagnosis, and **later initiation** as more than 180 days.

**Outcome.** The outcome is **diabetic eye disease (DED)**, modelled as a binary endpoint.

**Trajectory encoding.** MLTC histories and age of onset were encoded as temporal embeddings, capturing the order and timing of condition onset rather than treating comorbidities as a static count.

**Prediction.** Several machine-learning models were trained to predict DED from the trajectory and demographic features. A gradient boosting model achieved **AUC 0.87 / F1 0.80**, compared with **AUC 0.74 / F1 0.67** for a static-feature baseline. **SHAP** values were used for interpretability. (Prediction models use the full feature set to maximise predictive performance.)

**Association between timing and DED (adjusted).** A logistic regression modelled DED as a function of statin timing, adjusting for **sex, smoking status, alcohol intake, diabetes type, Townsend Deprivation Index and BMI**. Later statin initiation was associated with **higher odds of DED**: after adjustment, each additional year of delay corresponded to roughly an **8.3% increase in the odds** of DED (OR per 100 days ≈ 1.022), and the association remained statistically significant. Covariate directions were consistent with clinical expectation (e.g. higher odds for men than women; higher odds for former smokers; lower odds for type 2 vs type 1 diabetes). This is consistent with a potential benefit of earlier statin initiation.

**Causal effect estimation (exploratory).** As a complementary estimate, the average treatment effect (ATE) of earlier vs later initiation on DED was explored using **propensity score matching (PSM)** and **inverse probability of treatment weighting (IPTW)**, pointing in the same direction (lower DED risk with earlier initiation). These estimates are treated as exploratory and are interpreted alongside the adjusted regression above.

**Health-equity analysis.** Variation in MLTC burden, onset and outcomes was examined by **sex, ethnicity, and socioeconomic status** (Townsend Deprivation Index and Index of Multiple Deprivation).



## Replication across two data sources

The full pipeline was first developed in **UK Biobank** and then **replicated in a structurally different CPRD population** (131,310 / 108,139 patients). Replicating an entire cohort-to-estimate pipeline across two data sources with different coding systems, structures and follow-up required re-mapping phenotype definitions while holding the analytic logic fixed, and provides a check on the robustness of the findings beyond a single cohort. Keeping the pipeline modular (see [Repository structure](#repository-structure)) is what made this practical.



## Repository structure

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── code/
│   ├── data_engineering_demographics.ipynb          # Baseline demographic variables
│   ├── data_engineering_prescriptions.ipynb         # Prescription records -> statin exposure
│   ├── data_engineering_death.ipynb                 # Mortality censoring
│   ├── data_engineering_MLTCs.ipynb                 # Long-term condition identification and visualisation
│   ├── feature_engineering_drugs.ipynb              # Drug-level features (BNF)
│   ├── feature_engineering_ltcs_and_onset_age.ipynb # LTC features and age of onset
│   ├── feature_engineering_trajectories.ipynb       # Statin-embedded MLTCs trajectory
│   ├── feature_engineering_final_sync.ipynb         # Assemble analysis-ready dataset
│   ├── modelling_prediction.ipynb                   # ML models + SHAP for DED prediction
│   ├── analysis_causal.ipynb                        # Timing–DED association + PSM / IPTW
│   └── analysis_ethnicity.ipynb                     # Health-equity analysis by ethnicity
└── examples/
    └── synthetic_demo.ipynb    # Runnable demo on fully synthetic data (no real records)
```

**Intended execution order:** the `data_engineering_*` notebooks run first (in any order among themselves), then the `feature_engineering_*` notebooks (with `feature_engineering_final_sync` last), then `modelling_prediction`, then the `analysis_*` notebooks.



## How to run

These notebooks assume you have your own approved access to UK Biobank and/or CPRD. Any UK Biobank data should be run through the [UKB-RAP](https://www.ukbiobank.ac.uk/use-our-data/research-analysis-platform/). CPRD should be extracted to a local secure environment. They will **not** run without that data.

```bash
# Python 3.x
pip install -r requirements.txt
```

Open the notebooks in `code/` and run in the order described above. Local data paths are set at the top of each notebook.

If you do not have data access, see `examples/synthetic_demo.ipynb`, which runs the core timing–DED analysis end-to-end on **fully synthetic data**, so the method can be inspected without any real records.



## Limitations

These estimates are derived from observational data and should be read as **association-level evidence**, not as the result of a randomised comparison. They remain vulnerable to residual confounding and to time-related biases common in pharmacoepidemiology, for example, immortal-time bias and prevalent-user bias. The adjusted regression accounts for measured confounders only, and the propensity score estimates are exploratory. A more rigorous approach to the timing question, for example emulating a target trial, with careful time-zero alignment and handling of time-varying confounding would strengthen the causal interpretation, and is a natural next direction for this work.



## Citation

If you refer to this work, please cite the accompanying manuscript (in preparation). Contact details below.

## Contact

Liyuan Zhu, NIHR Newcastle Patient Safety Research Collaboration (PSRC), Newcastle University
l.zhu20@newcastle.ac.uk

## Licence

Released under the MIT Licence (see `LICENSE`).
