# NovaPay — Fraudulent Transaction Detection for Digital Money Transfer

NovaPay is a **digital-first cross‑border money transfer** platform operating in the **United Kingdom, Canada, and the United States**, enabling users to send/receive/hold multiple currencies across borders. NovaPay processes **millions of monthly transactions** and competes on **affordability, speed, and digital experience** — which makes **secure, accurate fraud detection** a strategic priority. fileciteturn2file0

This repository contains an **end‑to‑end machine learning fraud detection project**, including:
- time‑based evaluation (deployment‑realistic),
- baseline and advanced models,
- class‑imbalance handling,
- **stakeholder‑ready explainability** (local SHAP reason codes),
- reporting artefacts for review and audit.

---

## 1) Business problem

Digital payment platforms face three connected challenges: fileciteturn2file0

### Operational challenges
- Fraud must be detected quickly, but **manual review capacity is limited**.
- Static rules + manual workflows struggle to **scale** and **adapt** to evolving fraud tactics (account takeover, identity theft, laundering, unauthorized payments).

### Financial challenges
- Fraud drives **refunds, chargebacks, and compliance penalties**.
- **False positives** (blocking legitimate users) damage customer experience and can increase churn.
- **False negatives** (missed fraud) cause direct loss and higher operational cost later.

### Regulatory challenges
- AML/KYC frameworks and financial regulation demand **transparent, auditable, explainable decisions**.
- Weak fraud controls can lead to penalties and greater regulatory scrutiny.

**Goal:** build a model that helps NovaPay flag high‑risk transactions early while providing explanations that analysts and stakeholders can trust.

---

## 2) Project objectives

This project aims to: fileciteturn2file0
1. Build supervised classifiers to distinguish fraudulent vs legitimate transactions across diverse patterns.
2. Handle severe class imbalance (fraud is rare) using re‑sampling and class‑weighted learning.
3. Provide **transaction‑level explainability** (SHAP) for analysts and regulators.
4. Improve recall compared to a rules-based baseline while maintaining acceptable precision.
5. Package outputs for production deployment (planned FastAPI scoring service) and monitoring.

---

## 3) Dataset (high-level schema)

The dataset includes three main components: fileciteturn2file0

### Transaction data
- amounts, currency pairs, timestamps
- channel indicators (e.g., web/mobile)
- device fingerprints, IP, geo country codes
- behavioural signals (e.g., velocity features)

### Customer data
- account age, KYC tier
- historical behaviour summaries, risk scores

### Labels
- binary target (`is_fraud`), derived from investigations/chargebacks/disputes

**Data model:** many transactions per customer; one fraud label per transaction. fileciteturn2file0

---

## 4) Workflow (what we did)

The project follows a structured workflow: fileciteturn2file0

1. **Kickoff & success criteria** — define the business goal and evaluation approach.
2. **Data profiling** — missingness, distributions, baseline fraud prevalence.
3. **Data preparation** — cleaning, standardisation, feature engineering (velocity, mismatch indicators, risk signals).
4. **EDA** — patterns by channel, geography and time; anomaly discovery.
5. **Model development** — baselines → advanced models; compare precision/recall/F1/ROC‑AUC.
6. **Explainability** — transaction‑level SHAP explanations and stakeholder templates.
7. **(Planned) Deployment & monitoring** — FastAPI scoring endpoint + drift monitoring and retraining playbook.

---

## 5) Key notebooks (project deliverables)

> The Week 2 tasks below are based on the internship specification. fileciteturn2file1

### Week 1
- `notebooks/01_kickoff.ipynb` — problem framing, success criteria
- `notebooks/02_data_assessment.ipynb` — profiling and quality checks
- `notebooks/03_cleaning.ipynb` — cleaning & preparation
- `notebooks/04_features_eda.ipynb` — feature engineering & EDA

### Week 2
- `notebooks/05_baseline_models.ipynb` — **baseline modelling** (LogReg, Random Forest), time‑based split, metrics, SHAP global plots fileciteturn2file1  
- `notebooks/06_advanced_models.ipynb` — **advanced modelling & imbalance handling** (SMOTE / undersampling / class weights + XGBoost + LightGBM) fileciteturn2file1  
- `notebooks/07_explainability.ipynb` — **local explainability + stakeholder review template** (reason codes, confidence, false‑positive driver analysis) fileciteturn2file1  

### Final compiled notebook
- `Final_NovaPay_Fraud_Detection.ipynb` — curated single narrative notebook for final submission.

---

## 6) Modelling approach (summary)

### Time‑based split (deployment realism)
We use a **time‑based train/test split** (train on earlier transactions, test on later ones) to simulate how the model would behave in production and reduce leakage.

### Metrics
We evaluate with:
- **Precision** (how many flagged transactions are truly fraud)
- **Recall** (how much fraud we catch)
- **F1-score** (balance of precision and recall)
- **ROC‑AUC** (ranking quality)

### Class imbalance handling
Fraud is rare, so we tested:
- **Undersampling**
- **SMOTE oversampling**
- **Class weighting**
- Gradient boosted models with imbalance settings (XGBoost `scale_pos_weight`, LightGBM `class_weight`)

---

## 7) Explainability & stakeholder readiness

Fraud models must be explainable for analysts and regulators. fileciteturn2file1

In `07_explainability.ipynb`, we provide **transaction‑level explanations** using **SHAP**:
- For a flagged transaction, we generate “**reason codes**” (top positive drivers pushing risk up).
- We also analyse **false positive drivers** by lowering the threshold and seeing which features commonly push legitimate transactions over the decision boundary.

**Practical output:** a stakeholder‑ready review template (Markdown) that captures:
- fraud probability (confidence),
- threshold used,
- top features / reason codes,
- analyst outcome and notes (audit trail).

---

## 8) Reports

- `reports/Baseline Modeling & Wrap-Up.md`
- `reports/Week_2_Day_3_Advanced_Modeling_Imbalance_Handling.md`
- `reports/Week_2_Day_4_Explainability_Stakeholder_Interpretation.md`

---

## 9) How to run locally

### 9.1 Create environment (recommended)
```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate
```

### 9.2 Install dependencies
```bash
pip install -r requirements.txt
```

### 9.3 Run notebooks
Open Jupyter and run notebooks in order:
1. `notebooks/01_kickoff.ipynb` → `notebooks/07_explainability.ipynb`  
or open the curated notebook:
- `Final_NovaPay_Fraud_Detection.ipynb`

---

## 10) (Planned) Deployment & monitoring

Following the project roadmap: fileciteturn2file0
- Deploy as a **FastAPI** microservice with a `/score` endpoint for real‑time scoring.
- Containerise with **Docker** for portability.
- Monitor drift (e.g., with Evidently) and maintain a retraining cadence.

---

## 11) License / usage
This repository is for educational and demonstration purposes. If adapted for production, ensure proper governance, privacy controls, security review, and model risk management.
