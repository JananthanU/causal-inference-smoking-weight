# Causal Inference: Smoking Cessation and Weight Gain

Causal machine learning analysis of NHEFS that estimates how much weight smokers gain because they quit, validated against the published Hernán and Robins reference value.

`Python` `Causal Inference` `Causal Forest` `EconML` `causallib` `AIPW` `IPW` `Sensitivity Analysis`

> **Quitting smoking causes 3.38 kg of additional weight gain over 11 years (95% CI 2.46 to 4.30).** IPW reproduces Hernán and Robins exactly (3.44 kg), and the effect passes 5 of 5 robustness checks. No reliable difference in the effect between types of smokers.

---

## Results

| Result | Value |
|---|---|
| Average treatment effect (causal forest, doubly robust) | **3.38 kg** (95% CI 2.46 to 4.30) |
| IPW replication of Hernán and Robins | **3.44 kg** (2.41 to 4.47) vs. published 3.4 kg (2.4 to 4.5) |
| Robustness checks passed | **5 of 5** |
| Effect heterogeneity (CATE) | Not detectable (calibration test β = 0.54, one-sided p = 0.14) |

Estimator ladder, n = 1,566 smokers, effect on weight change 1971 to 1982 in kg:

| Estimator | Estimate | SE | 95% CI |
|---|---|---|---|
| Difference in means (naive) | 2.54 | 0.49 | 1.59 to 3.50 |
| Direct: outcome model (standardization) | 3.52 | 0.49 | 2.57 to 4.47 |
| IPW: treatment model (stabilized weights) | 3.44 | 0.53 | 2.41 to 4.47 |
| AIPW: parametric nuisances, cross-fit | 3.26 | 0.51 | 2.27 to 4.26 |
| **AIPW: causal forest** | **3.38** | **0.47** | **2.46 to 4.30** |

*The naive comparison understates the effect by about a quarter: quitters were older and heavier at baseline, characteristics linked to smaller weight gain. About 0.3 kg per year, small compared with the health benefits of quitting.*

## Why NHEFS

Candidate datasets were screened against five criteria: a real treatment decision, temporal precedence of treatment over outcome, a published reference value, documented provenance, and not saturated on Kaggle.

| Dataset | Verdict | Reason |
|---|---|---|
| Stroke Prediction (Kaggle) | Rejected | Authenticity and provenance cannot be verified ([Gibson et al. 2026, BMC Medicine](https://doi.org/10.1186/s12916-026-04981-y)) |
| CERN Electron Collision | Rejected | No treatment structure, so no causal question |
| Lalonde (NSW job training) | Rejected | Identical to an exercise already covered in the course |
| **NHEFS** | **Selected** | Meets all five criteria |

NHEFS (NHANES I Epidemiologic Follow-up Study, NCHS and CDC) fits all five: participants decided themselves whether to quit, all nine confounders were measured in 1971 before anyone quit while the outcome was measured in 1982, Hernán and Robins publish a reference estimate in *Causal Inference: What If*, the cohort has documented federal provenance, and it is rarely used in Kaggle-style projects.

## Methodology

The notebook follows the eight-step causal ML pipeline. Full reasoning, tables and appendix figures are in the [background document](docs/NHEFS_background_document.docx).

| Step | What was done |
|---|---|
| **1. Research question** | ATE of quitting (1971 to 1982) on weight change, CATE as secondary question. 1,629 smokers, 63 with missing outcome, 1,566 complete cases (403 quitters, 1,163 continuing) |
| **2. Causal structure** | DAG with nine baseline confounders (age, sex, race, education, cigarettes per day, years smoked, exercise, activity, weight 1971). Backdoor criterion checked with networkx; mediators such as appetite are not adjusted (total effect) |
| **3. Estimand** | ATE primary, CATE secondary, ATT as fallback if overlap failed |
| **4. Assumptions** | SUTVA argued. Positivity tested (propensities 0.05 to 0.78). Unconfoundedness supported by balance: largest ASMD 0.20 before, below 0.03 after weighting |
| **5. Fit** | Honest regression forests for nuisances, causal forest (econml.grf, 2,000 trees, leaf size chosen by R-loss), estimator ladder from naive to AIPW |
| **6. Evaluate** | Calibration test, predicted vs. confirmed effects by quartile, RATE with split-and-score: heterogeneity not confirmed |
| **7. Robustness** | Placebo treatment, random common cause, semi-synthetic outcome with known effect, sensitivity to unmeasured confounding (Cinelli and Hazlett), IP weighting for censoring |
| **8. Interpretation** | Best linear projection, partial dependence and illustrative profiles, reported as exploratory because Step 6 found no confirmed heterogeneity |

## Visualisations

Estimator ladder. Every adjusted point estimate lands inside the published 95% CI of Hernán and Robins (grey band); the naive difference in means is biased downward.

<img src="figures/fig05b_ate_ladder.png" alt="Estimator ladder with 95% confidence intervals" width="900">

Love plot. Before weighting, quitters differ most in age and cigarettes per day; after stabilized IP weighting, every standardized mean difference is below 0.03.

<img src="figures/fig04b_love_plot.png" alt="Love plot of covariate balance before and after weighting" width="900">

Robustness scorecard. An unmeasured confounder would need about 9 times the strength of age to erase the effect.

<img src="figures/fig07d_robustness_scorecard.png" alt="Robustness scorecard, 5 of 5 checks passed" width="900">

All 17 figures are in [`figures/`](figures), key numbers as JSON and CSV in [`results/`](results).

## Reproduce

```bash
git clone https://github.com/JananthanU/causal-inference-smoking-weight
cd causal-inference-smoking-weight
pip install -r requirements.txt
jupyter notebook notebooks/nhefs_causal_ml_pipeline.ipynb
```

Run **Kernel > Restart Kernel and Run All Cells**. A full run takes about 4 minutes. Tested with econml 0.17 and scikit-learn 1.9.

- **Data:** loaded through `causallib.datasets.load_nhefs`, no separate download needed.
- **Randomness:** one seed (42) for all forests, splits, permutations and bootstraps. Parametric estimates are deterministic; forest-based secondary values (quartile effects, RATE) can shift by up to about 0.15 kg across platforms. The headline numbers above are stable.
- **Outputs:** figures are written to `figures/`, key numbers and an executive summary to `results/`.

## Limitations

- **Unconfoundedness is untestable.** Supported by the DAG and bounded by the sensitivity analysis (robustness value 18%), but not proven.
- **Compound treatment.** Timing, method and relapses of quitting are unknown, so consistency holds only approximately.
- **Missing outcomes.** Censoring was informative (5.8% of quitters vs. 3.2% of continuing smokers), but IP weighting for censoring barely changes the result (3.50 kg).
- **Self-reported smoking.** Treatment and smoking-related confounders are subject to measurement error.
- **Transportability.** US smokers from 1971 to 1982; products, diets and populations differ today.
- **Limited power for heterogeneity.** With 1,566 people and a noisy outcome, a null CATE result means "not detectable", not "identical for everyone".
- **Total effect.** The estimate includes mediated pathways such as appetite and metabolism and says nothing about the mechanism.
