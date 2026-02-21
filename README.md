# Fraud Detection & Decisioning Pipeline (IEEE-CIS)

An end-to-end fraud risk workflow built on the IEEE-CIS dataset with a focus on **real fraud operations**:  
**time-aware evaluation**, **ranking-based performance**, **capacity-aware decisioning**, and **monitoring artifacts** (drift / PSI).

This project treats fraud modeling as a **triage problem** (who to decline / review / step-up / approve under limited capacity), not just a binary classification exercise.

---

## Why this project matters

Fraud is rare, so standard accuracy can look good while still failing in practice.

What matters operationally is:

- **How well the model ranks risky transactions**
- **How much fraud is captured in the top-K queue**
- **How decisions stay stable when score distributions shift over time**

That is why this project emphasizes:
- **PR-AUC**
- **Precision@K / Recall@K**
- **Capacity-locked rank policy**
- **Drift / PSI monitoring**
  
## Quick links：
- [Model card](reports/model_card.md)
- [Metrics summary](reports/metrics_summary.json)
- [Notebook](notebooks/01_eda.ipynb)

---

## Dataset

- **Source:** IEEE-CIS Fraud Detection (Kaggle)
- **Core tables:** transaction + identity
- **Repo note:** raw Kaggle files are kept **local only** (not tracked)

> The repo includes preparation code and outputs/artifacts, but not redistributed Kaggle source files.

---

## What this repo does

### 1) Data preparation
- Merges IEEE transaction + identity tables
- Exports analysis-ready parquet files
- Keeps raw files local for reproducibility and clean version control

### 2) Time-aware EDA
Using `notebooks/01_eda.ipynb`, the analysis is framed around **future performance**, not random-split performance:
- fraud base rate / class imbalance
- distribution behavior over time bins
- train/validation/test differences (drift-aware framing)

### 3) Ranking-focused evaluation
The model is evaluated as a **ranking system** under review constraints:
- **PR-AUC** for imbalanced discrimination quality
- **Precision@K / Recall@K** for top-of-queue usefulness
- validation vs future test performance comparison

### 4) Capacity-aware decisioning
Model scores are converted into operational actions, such as:
- **Decline**
- **Manual review**
- **Step-up / challenge**
- **Approve**

The repo includes a **rank-based, capacity-locked policy**, which is more stable than fixed score thresholds when score distributions drift.

### 5) Monitoring artifacts
Post-model outputs support deployment-style thinking:
- **PSI by time bin**
- drift summaries / plots
- sample decision outputs
- model card documentation

---

## Key outputs in this repo

- **Model metrics summary** (ranking performance, top-K quality)
- **Sample decision tables** (score-based and rank-policy variants)
- **PSI by time bin** (distribution shift monitoring)
- **Model card** (assumptions, limitations, intended usage)

These artifacts are designed to show not only model performance, but also **decisioning behavior and monitoring readiness**.

---

## Repo structure

```text
fraud-detection-decisioning/
├── README.md
├── .gitignore
├── data/                                # local-only (Kaggle IEEE-CIS files; not tracked)
│   └── raw/
│       └── ieee/
├── notebooks/
│   └── 01_eda.ipynb                     # EDA, baseline exploration, sanity checks
├── src/
│   └── fraud/
│       └── prepare_ieee.py              # merges transaction + identity tables, exports parquet
├── reports/
│   ├── figures/                         # PR curve, score drift, fraud-rate drift, PSI plots
│   ├── metrics_summary.json             # PR-AUC, Precision@K / Recall@K, base rates
│   ├── psi_by_timebin_test.csv          # PSI by test time bin (distribution shift monitoring)
│   ├── sample_decisions_test.csv        # example decisions from score-based policy
│   ├── sample_decisions_test_rank_policy.csv  # capacity-locked rank-based decisions
│   └── model_card.md                    # model assumptions, metrics, limitations, usage notes

