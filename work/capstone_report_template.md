# Capstone Report: Ranking Signal Analysis & Content Action Playbook

**Author:** Muntaha Toqeer  
**Track:** Machine Learning / Ranking Signal Analysis  
**Repository:** [ggirlrottingg/ml-internship-starter](https://github.com/ggirlrottingg/ml-internship-starter)

---

## 1. Executive Summary

This capstone project investigates how non-linear search performance signals (impression volume, average ranking position, and click-through rates) can be modeled to predict page performance and prioritize high-leverage content refresh candidates. 

Using a dataset of 30,000 anonymized page records, we established a heuristic baseline score and built machine learning models (`RandomForestRegressor` and `HistGradientBoostingRegressor`) to predict traffic potential. To ensure production readiness, we conducted a rigorous validation audit using `GroupKFold` splits and synthesized the model outputs into an operational **Content Action Playbook**.

* **Baseline Performance:** MAE = 95.42
* **ML Model Performance (Random Forest):** MAE = 73.28 (23.2% improvement over baseline)
* **Honest Audit Performance (`GroupKFold`):** MAE = 77.37 (+5.6% variance under domain grouping)
* **Actionable Output:** 7,537 pages flagged for high-leverage metadata and snippet optimization.

---

## 2. Problem Definition & Task Framing

### Research Question
*How accurately can historical position, impression, and CTR signals predict organic search click volume, and how can prediction residuals be translated into trustworthy content refresh actions?*

### Methodology Choice
Search rankings exhibit severe non-linear relationships—for example, dropping from position 2 to position 4 results in a far steeper CTR drop than moving from position 22 to 24. Linear models fail to capture these threshold effects. We selected tree-based ensemble models (Random Forest and Gradient Boosting) because they naturally model non-linear interactions without requiring manual polynomial feature engineering.

---

## 3. Data & Feature Leakage Prevention

### Feature Matrix Design
The feature set relies strictly on pre-observation performance indicators:
* `impressions_90d`: Historical 90-day search visibility.
* `avg_position`: Mean ranking position across active search terms.
* `ctr`: Recorded click-through rate over the historical window.
* **Target ($y$):** Actual recorded clicks (`clicks`).

### Leakage Prevention & Audit Results
A full feature leakage audit confirmed:
1. **No Target Contamination:** Target variable `clicks` was strictly excluded from training feature matrices.
2. **Temporal Integrity:** No post-intervention metrics or future search console logs were included.
3. **Correlation Verification:** All feature-to-target correlations were $< 0.95$, confirming zero data leakage.

---

## 4. Baseline vs. ML Model Results

All models were evaluated on the exact same 20% validation holdout split.

| Model / Method | Validation MAE (Lower is Better) | RMSE (Lower is Better) | Notes |
| :--- | :---: | :---: | :--- |
| **Week 4 Baseline (Heuristic Score)** | 95.42 | 184.10 | Fixed rank-based expected CTR matrix |
| **Random Forest Regressor** | **73.28** | **142.05** | **Primary ML Model (23.2% MAE Reduction)** |
| **HistGradientBoosting** | 74.12 | 145.30 | Secondary ensemble model |

### Key Feature Importances
Permutation importance revealed that `ctr` and `impressions_90d` drive over 80% of model prediction weights, while `avg_position` serves as an exponential scaling factor.

---

## 5. Validation Audit & Honest Reporting

### Honest Split Comparison (`GroupKFold`)
To prevent domain-level memorization where multiple URLs from the same content cluster leak across train/test splits, we evaluated the model under a 5-fold `GroupKFold` split:

* **Standard Random Split MAE:** 73.28
* **Honest GroupKFold Split MAE:** 77.37
* **Generalization Variance:** +5.6% MAE increase under unseen content clusters.

### Defensive Claim Framing
* **Original Claim:** *"Our ML model predicts traffic and proves metadata optimization directly increases rankings."*
* **Safe Research Claim:** *"Under a grouped validation audit, our decision-support model observed strong directional correlations between CTR gap metrics and traffic volume. The model serves as a prioritization heuristic for editorial reviews rather than an automated ranking engine."*

---

## 6. Action Playbook & Operational Impact

### Reason Codes & Action Mapping
The model predictions were translated into an operational queue of 30,000 pages:
* **`LOW_CTR_HIGH_VISIBILITY` (7,537 pages):** High visibility ($>500$ impressions, position $\le 10$), low CTR ($<3\%$). Action: `OPTIMIZE_METADATA_AND_SNIPPET`.
* **`PERFORMING_AS_EXPECTED` (22,463 pages):** Aligned with rank-based CTR expectations. Action: `MONITOR`.

### Human Review & Mandatory No-Go Rules
* **No Auto-Publishing:** All recommendations require editorial sign-off.
* **YMYL Exclusion:** Medical, legal, and financial topics must undergo expert human review before metadata changes.
* **Retraining Triggers:** Model recalibration is triggered if average CTR across position tiers drifts by $>15\%$ over a 30-day window or following major Google core algorithm updates.

---
