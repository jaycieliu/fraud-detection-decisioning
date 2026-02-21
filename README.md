# Fraud Detection & Decisioning Pipeline

An end-to-end fraud analytics project that combines **fraud risk modeling** with **business decisioning logic** to support operational actions such as **approve / manual review / decline**.

---

## Project Overview

Fraud detection is not only a machine learning classification problem — it is also a **business decision problem**.  
A useful fraud system must balance:

- **Fraud capture** (catching risky transactions)
- **Customer experience** (avoiding unnecessary false alarms)
- **Operational capacity** (prioritizing limited manual review resources)

This project builds a fraud detection pipeline and extends it into a **decisioning framework** that translates model scores into practical actions.

## Quick links：
- [Model card](reports/model_card.md)
- [Metrics summary](reports/metrics_summary.json)
- [Notebook](notebooks/01_eda.ipynb)

## Objectives

- Build a fraud risk classification workflow using transaction data
- Handle **class imbalance** and evaluate beyond accuracy
- Compare model performance and threshold tradeoffs
- Design a simple decision policy:
  - **Approve** (low risk)
  - **Review** (medium risk)
  - **Decline** (high risk)

## Dataset

- **Source:** IEEE-CIS Fraud Detection (Kaggle) *(update if different)*
- **Granularity:** Transaction-level observations
- **Target:** Fraud vs non-fraud
- **Challenge:** Highly imbalanced classes (fraud is rare)

> Some fields in the dataset are anonymized, so feature interpretation may rely on proxy reasoning.

## Methodology

### 1. Data Preparation
- Loaded and cleaned transaction data
- Checked missing values and variable types
- Prepared train/validation splits
- Removed or flagged leakage-prone variables (if applicable)

### 2. Exploratory Data Analysis (EDA)
- Examined fraud class distribution
- Reviewed feature patterns, outliers, and missingness
- Explored variables with stronger fraud signal

### 3. Feature Engineering
- Encoded categorical variables
- Built model-ready features *(replace with your specific feature engineering steps)*
- Standardized/scaled variables where needed

### 4. Modeling
- Built a baseline model
- Trained and compared classification models *(keep only those you used, e.g., Logistic Regression / Random Forest / XGBoost)*
- Generated fraud risk probabilities for threshold-based decisioning

### 5. Evaluation
Because of class imbalance, the project emphasizes:
- **Precision**
- **Recall**
- **F1-score**
- **ROC-AUC / PR-AUC** *(use what you computed)*
- **Confusion matrix**
- **Threshold sensitivity analysis**

### 6. Decisioning Framework
Converted model outputs into business actions:
- **Low score → Approve**
- **Mid score → Manual Review**
- **High score → Decline / Block**

This makes the project more realistic than a binary “fraud / not fraud” prediction output.

## Key Results

> Replace the bullets below with your actual metrics and findings.

- Improved fraud detection performance over baseline under class imbalance.
- Threshold tuning helped balance **fraud capture** and **false positive control**.
- A three-tier decisioning policy provided a more practical operational workflow than a single cutoff.
- Confusion-matrix analysis clarified tradeoffs between recall and customer friction.

## Business Interpretation

This project demonstrates how fraud analytics supports **operational decision-making**, not just model accuracy.

### Recommendations
- Use **risk-score thresholds** instead of a default 0.50 cutoff
- Align review thresholds with **manual review capacity**
- Track performance by action bucket (approve/review/decline)
- Monitor model drift as fraud behavior changes over time

## Tech Stack

- **Languages:** Python
- **Libraries:** pandas, NumPy, scikit-learn, matplotlib *(add seaborn/xgboost if used)*
- **Environment:** Jupyter Notebook 

## Repository Structure
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
