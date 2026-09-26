# Causal Inference: Smoking Cessation and Weight Gain

Estimating the causal effect of quitting smoking on 10-year weight change in the NHEFS cohort, with a full causal ML pipeline benchmarked against the published Hernán and Robins reference value.

`Python` `Causal Inference` `causallib` `EconML` `statsmodels` `IPW` `Observational Data`

> **Quitting smoking causes an average weight gain of 3.38 kg (95% CI 2.46 to 4.30) over 10 years.** The IPW estimate reproduces the Hernán and Robins reference value exactly (3.44 kg), and the effect survives 5 of 5 robustness checks.

---

## Results

| Metric | Value |
|---|---|
| Average treatment effect (ATE) | **3.38 kg** (95% CI 2.46 to 4.30) |
| IPW replication vs. Hernán and Robins | **3.44 kg**, exact match |
| Robustness checks passed | **5 of 5** |

<!-- TODO: name the estimator behind the 3.38 kg headline and add one row per estimator from the estimator ladder. -->

*Treatment: quitting smoking between 1971 and 1982. Outcome: weight change in kg over the same period.*

## Visualisations

Estimator ladder: point estimates and 95% confidence intervals across all fitted estimators, compared against the Hernán and Robins reference value for comparison.

<img src="figures/estimator_ladder.png" alt="Estimator ladder" width="900">

Love plot: standardized mean differences of all confounders before and after weighting.

<img src="figures/love_plot.png" alt="Love plot of covariate balance" width="900">

## Why NHEFS

Four candidate datasets were screened against five criteria: a real treatment decision, temporal precedence of treatment before outcome, a published reference value, documented provenance, and not being saturated on Kaggle.

| Dataset | Decision | Reason |
|---|---|---|
| Stroke Prediction (Kaggle) | Rejected | Documented authenticity problems (Gibson et al. 2026, BMC Medicine) |
| CERN Electron Collision | Rejected | No treatment structure, so no causal question to ask |
| Lalonde (NSW job training) | Rejected | Identical to an exercise already covered in the course |
| **NHEFS** | **Selected** | Meets all five criteria |

NHEFS (NHANES I Epidemiologic Follow-up Study) records a real individual decision (quitting smoking), measures confounders at baseline in 1971 before the outcome in 1982, has a published reference estimate in Hernán and Robins, *Causal Inference: What If*, comes from a documented CDC and NCHS cohort, and is rarely used in Kaggle-style projects.

## Methodology

The notebook follows an eight-step causal workflow. Details, including the DAG and the reasoning behind each assumption, are in the [background document](docs/NHEFS_background_document.docx).

| Step | What was done |
|---|---|
| **1. Research question** | Does quitting smoking cause weight gain, and by how much? |
| **2. Causal structure** | DAG encoding baseline confounders (age, sex, race, education, smoking intensity and duration, exercise, activity, baseline weight) |
| **3. Estimand** | Average treatment effect (ATE) of quitting on 10-year weight change |
| **4. Assumptions** | Conditional exchangeability, positivity, consistency |
| **5. Fit** | Estimator ladder from naive comparison through outcome regression and IPW to ML-based estimators (causallib, EconML) |
| **6. Evaluate** | Covariate balance (love plot), propensity overlap, agreement with the published reference value |
| **7. Robustness** | Five checks, all passed |
| **8. Interpretation** | Effect size in context, limits of the causal claim |

<!-- TODO: verify steps 2, 5 and 7 against the notebook, name the five robustness checks. -->

## Reproduce

```bash
git clone https://github.com/JananthanU/causal-inference-smoking-weight
cd causal-inference-smoking-weight
pip install -r requirements.txt
jupyter notebook notebooks/nhefs_causal_ml_pipeline.ipynb
```

In Jupyter, run **Kernel > Restart Kernel and Run All Cells**. A full run takes about 4 minutes. Tested with econml 0.17 and scikit-learn 1.9.

<!-- TODO: state where the NHEFS data comes from (bundled CSV or causallib.datasets.load_nhefs). -->

## Limitations

<!-- TODO: replace with the limitations from docs/NHEFS_background_document.docx. -->

- Exchangeability is untestable: the estimate is causal only if the measured baseline covariates capture all confounding.
- Participants lost to follow-up have no 1982 weight, so the analysis relies on how censoring is handled.
- The cohort is a US population from 1971 to 1982, so transfer to other populations or periods requires re-validation.
