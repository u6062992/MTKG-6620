# Results — Case 1 churn analysis

This document organizes the saved results and factual notes about the supplied script. Boosted trees was selected before final evaluation; the student's original explanation is preserved in [analysis.md](analysis.md). 

## 1. Data checks, cleaning, and preprocessing

| Check | Recorded treatment or result |
|---|---|
| Assigned extract | 7,043 rows and 21 columns; 1,869 Yes and 5,174 No churn outcomes. |
| Customer identifiers | Present and unique; excluded from prediction. |
| Missing values | Converted `TotalCharges` to numeric and filled exactly 11 blank entries with zero after confirming that all 11 had zero tenure; no rows removed. No missing values remained in the seven analysis inputs. |
| Numeric validity and outliers | Numeric inputs passed the finite-value check. The supplied script did not perform an outlier screening procedure, remove outliers, cap values, or apply log transformations. |
| Standardization | Both logistic regression and boosted trees used `StandardScaler` for `tenure`, `MonthlyCharges`, and `TotalCharges`, fitted on training rows only. The contract rule did not use numeric inputs. |
| Categorical inputs | Both models used one-hot encoding for `Contract`, `InternetService`, `PaperlessBilling`, and `PaymentMethod`, learned from training rows with unknown categories ignored. |
| Outcome and class balance | `Churn` was converted from Yes/No to 1/0 and excluded from predictors; no class rebalancing was applied. |
| Separation | 4,225 training rows, 1,409 validation rows, and 1,409 final-test rows; stratified splits used seed 0. Saved assignments contained 7,043 unique source rows across 7,043 assignments. |

Filling blank charges with zero was the supplied classroom assumption for customers with zero tenure; passing the numeric checks does not establish that there are no outliers. Contract rates, model parameters, scaling, and category encoding were learned from training rows only.

## 2. Final-test AUC and 95% bootstrap intervals

| Method | Final-test AUC | 95% bootstrap interval for AUC |
|---|---:|---:|
| Contract rule | 0.737265 | [0.716259, 0.755731] |
| Logistic regression | 0.847143 | [0.825415, 0.869946] |
| Boosted trees | 0.849665 | [0.828267, 0.873164] |

All three methods were evaluated on the same 1,409 final-test customers, using the supplied settings and the same training partition. AUC measures ranking, with half credit for ties; it is not the percentage of customers classified correctly.

## 3. Uncertainty: paired AUC differences against the contract baseline

| Comparison | AUC difference | 95% paired bootstrap interval | Does the interval cross or include zero? |
|---|---:|---:|---|
| Logistic regression minus contract rule | 0.109878 | [0.093136, 0.128450] | No — both endpoints are positive. |
| Boosted trees minus contract rule | 0.112400 | [0.095891, 0.131117] | No — both endpoints are positive. |

The script used 1,000 valid bootstrap samples of test rows with replacement, scored every method on the same sampled rows, and reported the 2.5th and 97.5th percentiles for AUCs and paired differences. Both baseline-comparison intervals exclude zero; these exploratory intervals describe test-sample uncertainty for fixed models under row independence, without adjustment for multiple comparisons or allowance for retraining, future changes, or offer effectiveness.

## 4. Scenario calculations

Each model’s final-test top-20% list contains **281 customers**. Each list is held fixed across all three assumed save rates.

Using $66 per additional retention ($81 − $15) and $6.20 per contact:

\[
\text{Net value per 1,000 contacts}=1{,}000(r\times s\times66-6.20)
\]

| Model | Observed list churn rate \(r\) | 10% save rate | 15% save rate | 20% save rate | Break-even save rate |
|---|---:|---:|---:|---:|---:|
| Contract rule | 39.8577% | −$3,569.40 | −$2,254.09 | −$938.79 | 23.5687% |
| Logistic regression | 69.3950% | −$1,619.93 | $670.11 | $2,960.14 | 13.5369% |
| Boosted trees (chosen model) | 69.3950% | −$1,619.93 | $670.11 | $2,960.14 | 13.5369% |

Dollar amounts are **per 1,000 contacts**, calculated using unrounded churn rates.

### Independent arithmetic check: boosted trees at 15%

\[
1{,}000\left(\frac{195}{281}\times0.15\times66-6.20\right)
=\boxed{\$670.11}
\]

### Break-even calculation: chosen boosted-trees list

\[
s_{\text{break-even}}
=\frac{6.20}{(195/281)\times66}
=0.135369
=\boxed{13.5369\%}
\]

These are hypothetical scenario values under the supplied fictional assumptions, not measured campaign profit. Save rates are assumed, not estimated from the data; offer costs exclude payments to customers who would have stayed anyway.

Sources: [Final-test metrics](outputs/test_metrics.csv) and [saved scenario calculations](outputs/scenarios.csv).

## Sources and reproducibility

- Cleaning, preprocessing, and bootstrap procedure: [VD1_analysis.py](VD1_analysis.py).
- Final-test AUCs: [outputs/test_metrics.csv](outputs/test_metrics.csv).
- Bootstrap intervals and paired differences: [outputs/intervals.csv](outputs/intervals.csv).
- Run settings, partition sizes, and valid bootstrap count: [outputs/evaluate_run.json](outputs/evaluate_run.json).
- Row assignments: [outputs/split_rows.csv](outputs/split_rows.csv).
- AUC results are displayed to six decimal places; scenario dollars are rounded to cents and scenario percentages to four decimal places. The CSV files retain full precision.
- The recorded runtime was Python 3.13.15, pandas 3.0.5, NumPy 2.5.2, and scikit-learn 1.9.0, which differs from the supplied reference environment.
- AI assistance: organized existing outputs and factual code descriptions into this document; no models were rerun and no decision rationale was added.
