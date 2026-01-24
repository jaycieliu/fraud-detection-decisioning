# Model Card — IEEE-CIS Fraud Detection (Baseline + Decisioning)

## 1) Intended use
This model produces a **fraud risk score** to **rank transactions** and support operational actions under capacity constraints:
**decline / review / step-up / approve**.

**Not intended for:** making final fraud decisions without human/secondary checks, or deployment without calibration + monitoring.

---

## 2) Data
- Source: IEEE-CIS Fraud Detection (Kaggle)
- Join: `transaction` + `identity` on `TransactionID` (left join)
- Label: `isFraud` (train only)
- Split: **chronological 60/20/20** using `TransactionDT` (prevents leakage, matches production drift)

> Raw Kaggle data is not included in this repository.

---

## 3) Model (baseline)
- Algorithm: `HistGradientBoostingClassifier`
- Features: **numeric-only baseline (400 numeric columns)**
- Missing values: `SimpleImputer(strategy="median")` (fit on train only; applied to valid/test)
- Output: probability-like score used for ranking + decisioning

Why this baseline:
- Establishes a strong reference point before adding categorical encoding and calibration.

---

## 4) Evaluation setup
Primary goal is **ranking quality** under class imbalance (~3–4% fraud).
Metrics reported on validation + test:
- **PR-AUC (Average Precision)**: overall ranking performance for rare fraud
- **Precision@K / Recall@K**: quality of top-K alerts given limited capacity

---

## 5) Results (Validation → Test)

**Base rate**
- Validation: **3.90%**
- Test: **3.44%**

**PR-AUC**
- Validation: **0.572**
- Test: **0.462**

**Precision@K / Recall@K**
- **K=1000** (review-like capacity)
  - Valid: precision **0.915**, recall **0.198**
  - Test:  precision **0.900**, recall **0.221**
- **K=10000** (review + step-up total capacity)
  - Valid: precision **0.317**, recall **0.688**
  - Test:  precision **0.243**, recall **0.598**

## Key takeaways
- **Drift exists:** PR-AUC decreases on the future test slice (0.572 → 0.462), but top-K precision remains strong, indicating fraud is still concentrated near the top ranks.
- **Capacity tradeoff:** K controls precision vs recall (K=1000: high precision, lower recall; K=10000: higher recall, lower precision). Choose K based on review/step-up capacity and cost.
- **Stable operations:** Rank-based decisioning is more robust than fixed thresholds under score drift because it keeps queue volume stable over time.
---

## 6) Decision policy (capacity-locked ranking)
Because risk score distribution shifts over time, fixed probability thresholds can cause queue volume drift; rank-based (capacity-locked) policies keep workloads stable.

Example:
- Top **K_decline** → decline
- Next **K_review** → review
- Next **K_step_up** → step-up
- Rest → approve

Artifacts:
- `reports/sample_decisions_test.csv`
- `reports/sample_decisions_test_rank_policy.csv`
- `reports/metrics_summary.json`

---

## 7) Monitoring
The following monitoring artifacts are produced to detect drift:
- Score drift over time bins (mean score)
- Fraud rate drift over time bins (label drift)
- PSI vs first time bin (distribution shift)

### Score drift (mean model score over time bins)
<p align="center">
  <img src="figures/score_drift_test.png" width="650">
</p>

### Fraud rate drift (label drift over time bins)
<p align="center">
  <img src="figures/fraud_rate_over_time_test.png" width="650">
</p>

### PSI drift (distribution shift vs first time bin)
<p align="center">
  <img src="figures/psi_score_drift_test.png" width="650">
</p>

Conclusion:
- PSI indicates small score-distribution drift (max 0.067), while fraud rate and mean score vary across time—suggesting non-stationary prevalence rather than a complete model breakdown.
---

## 8) Limitations & risks
- Baseline is **numeric-only**; categorical variables are not yet encoded.
- Predicted probabilities are **not calibrated**; thresholds may not transfer across time.
- Dataset features are anonymized; interpretation is limited.
- Real systems require fairness / policy review depending on use case.

---

## 9) Next improvements
- Add categorical handling (frequency / target encoding)
- Add calibration (Platt / isotonic) and evaluate calibration curves
- Cost-sensitive thresholding (expected profit / loss by action)
- Automated monitoring reports and alerting rules
