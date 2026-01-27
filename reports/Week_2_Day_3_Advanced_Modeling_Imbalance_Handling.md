# Week 2 · Day 3 Report — Advanced Modeling & Imbalance Handling :contentReference[oaicite:0]{index=0}

## 1) Overview
Fraud detection is an **imbalanced classification problem**: fraud cases are rare relative to legitimate transactions, and the business cost of missing fraud (false negatives) is typically high.  
The goal of Week 2 Day 3 was to move beyond baseline models by introducing **advanced models** and **imbalance-handling strategies** that improve recall and robustness while managing false positives.

This work builds directly on Week 2 Day 1 baselines by:
- keeping a **time-based split** (to mimic real deployment),
- testing imbalance strategies (sampling + weighting),
- and comparing models using metrics that matter for fraud operations.

---

## 2) Objectives (Met)
- Train and evaluate **advanced models** (XGBoost, LightGBM).
- Apply imbalance-handling strategies:
  - **Random undersampling**
  - **SMOTE oversampling**
  - **Class-weighting**
- Compare results using a structured experiment log with:
  - **Precision, Recall, F1, ROC-AUC**
- Reflect on how **fraud prevalence changes over time** impact performance stability.

---

## 3) Business Framing (Why Imbalance Handling Matters)
In fraud detection:
- **False negatives (missed fraud)** can cause direct financial loss and reputational damage.
- **False positives (legitimate transactions flagged)** create customer friction, failed payments, and increased review workload.

Because fraud is uncommon, a model can appear “accurate” while still performing poorly on fraud detection.  
Therefore, evaluation focused on:
- **Recall**: How much fraud we catch (fraud-team priority)
- **Precision**: How many flagged cases are truly fraud (customer/ops priority)
- **F1**: Balanced measure combining precision and recall
- **ROC-AUC**: How well the model ranks fraud higher than legit across thresholds

---

## 4) Data Splitting Approach (Time-Based Split)
A time-based split was used to prevent leakage and to match real-world deployment conditions:

- **Train fraud rate:** ~7.7%  
- **Test fraud rate:** ~14.1%

This indicates fraud prevalence increased over time, which is realistic in fraud operations and directly affects threshold behaviour and stability.

---

## 5) Experiments Run (Steps, Rationale, and Outcomes)

### 5.1 Experiment Logging (Structured Comparison)
A consistent evaluation function was used to measure performance at a set probability threshold and store outcomes in an experiment log.  
This ensured every model/strategy was compared fairly under the same test window.

---

### 5.2 Random Forest + RandomUnderSampler (Undersampling)
**Why:** Undersampling reduces the number of majority-class (legit) examples so the model learns more strongly from minority-class fraud patterns.  
**Trade-off:** Can increase recall but may reduce precision because the model sees fewer legit examples.

**Result (threshold = 0.5):**
- Precision: **0.9034**
- Recall: **0.9325** *(highest recall among tested approaches)*
- F1: **0.9177**
- ROC-AUC: **0.9780**

**Interpretation:**  
This approach is useful if the business goal is **maximum fraud capture**, but it increases false positives (more legitimate users flagged).

---

### 5.3 Random Forest + SMOTE (Oversampling)
**Why:** Instead of discarding legit transactions, SMOTE creates synthetic fraud-like samples to balance training data.  
**Trade-off:** Can improve learning of fraud patterns without losing majority-class signal, but synthetic samples may introduce noise.

**Result (threshold = 0.5):**
- Precision: **1.0000**
- Recall: **0.9164**
- F1: **0.9564**
- ROC-AUC: **0.9713**

**Interpretation:**  
Very high precision means fewer unnecessary reviews and fewer customer disruptions.  
Recall is strong, but slightly lower than undersampling.

---

### 5.4 Random Forest + Class Weights (No Sampling)
**Why:** Sampling changes the dataset distribution. Class weighting keeps the dataset intact but penalizes fraud errors more heavily during training.  
This is often preferred operationally because it avoids synthetic data and avoids discarding real transactions.

**Result (threshold = 0.5):**
- Precision: **1.0000**
- Recall: **0.9164**
- F1: **0.9564**
- ROC-AUC: **0.9741**

**Interpretation:**  
This matched SMOTE on F1 and recall **without sampling**, making it a strong and operationally simpler improvement over the baseline.

---

### 5.5 XGBoost + Class Weighting (scale_pos_weight)
**Why XGBoost:** Gradient boosting models are powerful for structured/tabular fraud datasets.  
**Imbalance method:** `scale_pos_weight = (#negative / #positive)` to weight fraud errors higher.

**Initial Result (threshold = 0.5):**
- Precision: **0.8994**
- Recall: **0.9196**
- F1: **0.9094**
- ROC-AUC: **0.9753**

XGBoost showed strong ranking ability (ROC-AUC), but its default threshold did not maximize F1.

#### Threshold Sweep (0.1 → 0.7) to optimize decision threshold
**Best threshold by Method B (highest F1): `0.7`**
- Precision: **0.9828**
- Recall: **0.9164**
- F1: **0.9484**
- ROC-AUC: **0.9753**

**Interpretation:**  
Higher thresholds reduced false positives while maintaining recall, improving F1 substantially.

---

### 5.6 LightGBM + Class Weights
**Why LightGBM:** Similar benefits to XGBoost, often faster and strong on tabular data.  
**Imbalance method:** `class_weight="balanced"`.

**Initial Result (threshold = 0.5):**
- Precision: **0.9727**
- Recall: **0.9164**
- F1: **0.9437**
- ROC-AUC: **0.9665**

#### Threshold Sweep (0.1 → 0.7)
**Best threshold by Method B (highest F1): `0.7`**
- Precision: **0.9896**
- Recall: **0.9164**
- F1: **0.9516**
- ROC-AUC: **0.9665**

**Interpretation:**  
Like XGBoost, LightGBM benefited from raising the threshold. It became the strongest boosting option by F1.

---

## 6) Summary of Best Results (Business Interpretation)

### Best overall by **highest F1 (Method B)**
- **Random Forest (SMOTE or class_weight) @ threshold 0.5**
  - Precision: **1.0000**
  - Recall: **0.9164**
  - F1: **0.9564**
  - ROC-AUC: **0.9741** (class_weight version)

**Why this is strong for the business:**  
- Near-perfect precision means fewer legitimate users are impacted.
- Recall remains high enough to catch the majority of fraud.
- Operationally, class-weighting avoids dataset manipulation and is easier to justify.

### Best option if the business prioritizes **maximum fraud capture**
- **Random Forest + RandomUnderSampler @ threshold 0.5**
  - Recall: **0.9325** *(highest)*
  - Precision: **0.9034**

**Why this matters:**  
If the cost of missed fraud is extremely high, a slightly higher false positive rate may be acceptable in exchange for improved fraud capture.

### Best “advanced model” by F1
- **LightGBM @ threshold 0.7**
  - F1: **0.9516**
- **XGBoost @ threshold 0.7**
  - F1: **0.9484**

**Insight:**  
Boosting models performed strongly, but required threshold tuning to reach their best operating point.

---

## 7) Reflection — Fraud Prevalence Over Time & Stability
Fraud prevalence increased from the training period (~7.7%) to the test period (~14.1%), showing the distribution of the target changes over time. When prevalence shifts, a fixed threshold can produce different precision/recall behaviour because the base rate of fraud affects how many alerts the model generates and how many are true positives. This can make model performance less stable in production, especially precision, because more (or fewer) cases fall above the decision threshold depending on the time window. As a result, models should be validated on time-based splits and monitored continuously, with periodic threshold recalibration. Using imbalance-aware approaches (class weights, SMOTE, tuned thresholds) helps maintain recall while controlling false positives under changing fraud rates.

---

## 8) Deliverables Produced
- **Notebook:** `06_advanced_models.ipynb`   
  Contains:
  - time-based split
  - imbalance experiments (undersampling, SMOTE, class weights)
  - advanced model training (XGBoost, LightGBM)
  - threshold sweeps for boosted models
  - structured experiment log comparing results

---

## 9) Next Steps (Week 2 Day 4 Readiness)
With advanced models and imbalance strategies tested, the next step is to focus on **explainability and stakeholder review**, including:
- transaction-level reason codes
- local explanations (SHAP/LIME)
- templates for analyst review and audit readiness