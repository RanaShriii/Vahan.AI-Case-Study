# Vahan Product Analytics Case Study

## Overview

This repository contains the complete analysis for the Vahan Product Analytics Case Study.

The project answers three business questions:

1. **Which lead-source cohorts perform best based on FT after upload conversion?**
2. **What does cohort-level funnel aggregation reveal about lead performance?**
3. **Can FT after upload be predicted, and which observable factors are associated with FT?**

---

## Repository Structure

```text
Vahan_Product_Analytics_Case_Study/
│
├── Vahan_Product_Analytics_Case_Study.ipynb
├── Vahan_Case_Study.xlsx
├── Vahan_Case_Study_Report.pdf
└── README.md
```

### Files

| File | Description |
|---|---|
| `Vahan_Product_Analytics_Case_Study.ipynb` | Complete analysis notebook containing data validation, cohort analysis, SQL aggregation, modeling and evaluation |
| `Vahan_Case_Study.xlsx` | Supporting workbook containing the key Q1, Q2 and Q3 outputs |
| `Vahan_Case_Study_Report.pdf` | Professional summary of methodology, findings, model results and recommendations |
| `README.md` | Project documentation |

---

## Dataset Overview

The dataset contains **18,198 lead-level observations** across **16 lead sources**.

### Key outcome

- Uploaded leads: **18,198**
- FT after upload: **54**
- Overall FT conversion rate: **0.297%**

The target is highly imbalanced, with FT occurring in only 54 observations.

---

# Question 1 — Best Cohorts

### Metric

The primary metric is:

```text
FT after upload conversion rate
= FT after upload / Uploaded Leads × 100
```

This metric is used instead of raw FT count because cohort sizes differ substantially.

Cohorts with fewer than **100 uploaded leads** are excluded from ranking because extremely small cohorts can produce unstable conversion rates.

### Top 3 cohorts

| Rank | Cohort | Uploaded Leads | FT | FT Conversion |
|---:|---|---:|---:|---:|
| 1 | Single Referral > 7 days- 24th Jul | 1,500 | 14 | **0.933%** |
| 2 | Khanna- 2W 26th Jul | 1,546 | 14 | **0.906%** |
| 3 | PreOb-Ob Fees Paid 29th Jul (set 1) | 1,483 | 7 | **0.472%** |

The overall FT conversion rate is **0.297%**.

The first two cohorts therefore perform at approximately **3.15×** and **3.05×** the overall FT rate.

---

# Question 2 — SQL / Funnel Aggregation

The lead-level data is aggregated by `lead_source` using DuckDB.

The analysis reports:

- Uploaded leads
- Attempted
- Connected
- Tag filled
- Interested
- OB after upload
- OB after first attempt
- FT after upload
- FT after first attempt
- Attempt rate
- Attempt → Connected rate
- Connect → Interested rate
- FT after upload conversion rate

Rates are calculated from aggregated counts rather than by averaging row-level percentages.

### Key finding

Lead volume does not necessarily indicate cohort quality.

For example:

- **OLX - Ashwin - 2W - 17 Jul:** 5,182 leads, 4 FT, **0.077% FT conversion**
- **Single Referral > 7 days- 24th Jul:** 1,500 leads, 14 FT, **0.933% FT conversion**

This shows why acquisition volume should be evaluated together with downstream FT efficiency.

---

# Question 3 — FT Prediction

## Target

```text
FT_after_upload
```

## Final predictors

The final interpretable model uses:

- `lead_source`
- `funnel_stage`

The funnel stage represents the deepest observed stage reached by a lead:

| Stage | Meaning |
|---:|---|
| 0 | Not attempted |
| 1 | Attempted |
| 2 | Connected |
| 3 | Tag filled |
| 4 | Interested |

Downstream FT/OB outcome variables are excluded from the predictors to avoid target leakage.

---

## Model Selection

Three initial models were evaluated:

| Model | Mean ROC-AUC | Mean PR-AUC |
|---|---:|---:|
| Logistic Regression | 0.7769 | 0.00965 |
| Random Forest | 0.7702 | 0.01016 |
| Extra Trees | 0.7604 | 0.00984 |

Logistic Regression was selected because it provides strong ranking performance while remaining interpretable, which is important for identifying factors associated with FT.

---

## Final Model Performance

The final Funnel-stage Logistic Regression achieved:

- **Holdout ROC-AUC: 0.7815**
- **Holdout PR-AUC: 0.0130**

At a 0.5 classification threshold:

```text
Confusion Matrix

[[2483 1146]
 [   4    7]]
```

This gives:

- FT recall: **63.64%**
- FT precision: **0.61%**
- Accuracy: **68.41%**

Accuracy is not treated as the primary metric because FT is extremely rare.

The model is therefore more appropriate for **lead ranking and prioritization** than for strict binary FT classification.

---

## Factors Associated with FT

The final Logistic Regression indicates that:

1. **Lead source** is the strongest source of predictive variation.
2. **Funnel stage** has a positive model association with FT.
3. High-performing sources such as Single Referral and Khanna-2W have strong positive model coefficients.
4. Some sources have negative model associations relative to the reference cohort.

These are **predictive associations, not causal effects**.

Because class weighting was used to handle severe class imbalance, model odds ratios should not be interpreted as direct real-world probability multipliers.

---

## Cohort-Level Validation

Random holdout performance is higher than validation where entire lead-source cohorts are kept together.

### Cohort-level results

- Mean ROC-AUC: **0.6471 ± 0.0729**
- Mean PR-AUC: **0.0064 ± 0.0035**

This indicates that some predictive signal is cohort-specific and that performance on completely new lead-source cohorts may be materially lower than random holdout performance.

---

# Business Recommendations

### 1. Prioritize high-performing cohorts

Single Referral > 7 days- 24th Jul and Khanna- 2W 26th Jul substantially outperform the overall FT conversion rate and should be investigated for the characteristics driving their performance.

### 2. Do not optimize only for lead volume

Large cohorts can have poor downstream conversion. Lead-source evaluation should include FT conversion rather than relying on volume alone.

### 3. Use funnel progression for prioritization

Deeper funnel progression is positively associated with FT in the model and can be used as a prioritization signal for follow-up.

### 4. Use the ML model as a ranking tool

Because FT is extremely rare, the model produces many false positives at a standard threshold. Ranking leads by predicted FT propensity is more appropriate than using a hard binary rule.

### 5. Monitor new cohorts

The reduction in cohort-level validation performance shows that the model should be monitored and periodically retrained as new acquisition cohorts are introduced.

### 6. Collect more FT outcomes

Only 54 FT outcomes are available. Additional positive observations would improve statistical stability, model evaluation and confidence in cohort comparisons.

---

# Methodology Notes

The analysis includes:

- Data quality and missing-value checks
- Cohort-level aggregation
- FT conversion analysis
- Funnel-stage construction
- Stratified train/test split
- Logistic Regression, Random Forest and Extra Trees comparison
- ROC-AUC and PR-AUC evaluation
- Confusion matrix and classification metrics
- Feature coefficient analysis
- Cohort-level validation

The analysis is observational. Cohort differences and model coefficients should therefore be interpreted as associations rather than causal effects.

---

## Reproducibility

The primary executable analysis is contained in:

```text
Vahan_Product_Analytics_Case_Study.ipynb
```

The notebook contains the complete analytical workflow and model evaluation.

The Excel workbook provides the key outputs in a structured format, while the PDF provides a concise presentation of the findings.

---

## Final Takeaway

The analysis shows that **cohort quality and funnel progression are more informative than lead volume alone**.

The strongest historical cohorts achieved FT conversion rates more than three times the overall rate. The ML model demonstrates useful ranking ability on a random holdout, but weaker performance on unseen cohorts and very low positive-class precision due to the extreme rarity of FT.

The recommended business use is therefore to combine **cohort performance analysis + funnel progression + model-based prioritization**, while continuously validating performance on new cohorts.
