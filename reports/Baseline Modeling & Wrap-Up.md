# Week 2 · Day 1 Report — Baseline Modeling & Wrap-Up

## 1) Overview
With the dataset cleaned and engineered features prepared, the focus of Week 2 Day 1 was to establish **baseline fraud-classification performance**. The purpose of these baselines is not to reach “final” accuracy, but to create a **measurable benchmark** that we can improve in Week 2 Day 3 (imbalance handling + advanced models).

---

## 2) Objectives (Met)
- Build and evaluate **initial fraud classification models**
- Create a **reliable benchmark** using proper evaluation metrics for imbalanced data
- Add **interpretability** using SHAP (global feature importance)

---

## 3) Tasks Completed

### 3.1 Time-based Train/Test Split
A **time-based split** was used to reflect real-world deployment conditions:
- Earlier transactions → **Train**
- Later transactions → **Test**

This avoids *data leakage* that can happen if we randomly shuffle time-dependent transactions.

---

### 3.2 Baseline Models Trained
Two baseline classifiers were trained:

1. **Logistic Regression (LogReg)**
   - Used as a strong linear baseline.
   - Provides stable probabilities and easy interpretability.

2. **Random Forest (RF)**
   - Captures non-linear patterns and feature interactions.
   - Often improves precision in fraud detection.

Both models were trained using the same time-based split for fair comparison.

---

### 3.3 Model Evaluation Metrics
Because fraud detection is typically **highly imbalanced**, model evaluation focused on:
- **Precision** (how many flagged frauds are truly fraud)
- **Recall** (how many true frauds we successfully catch)
- **F1 score** (harmonic mean of precision and recall)
- **ROC-AUC** (ranking quality across thresholds)

---

## 4) Results Achieved (Baseline Performance)

### 4.1 Threshold Sweep Results
Thresholds were tested to understand the precision–recall trade-off.

#### Logistic Regression (selected thresholds)
| Threshold | Precision | Recall | F1 | ROC-AUC |
|---:|---:|---:|---:|---:|
| 0.1 | 0.2329 | 0.9871 | 0.3769 | 0.9802 |
| 0.3 | 0.4942 | 0.9646 | 0.6540 | 0.9802 |
| **0.5** | **0.7787** | **0.9398** | **0.8513** | **0.9802** |

#### Random Forest (selected thresholds)
| Threshold | Precision | Recall | F1 | ROC-AUC |
|---:|---:|---:|---:|---:|
| 0.1 | 0.5122 | 0.9486 | 0.6652 | 0.9741 |
| 0.3 | 0.9258 | 0.9228 | 0.9243 | 0.9741 |
| **0.5** | **0.9965** | **0.9164** | **0.9548** | **0.9741** |

---

## 5) Explainability (SHAP) — What Was Done
To interpret model behavior and feature influence:
- Global SHAP explanations were generated for:
  - **Logistic Regression**
  - **Random Forest**
- SHAP output handling was corrected so the Random Forest explanation used **fraud class (class 1)** where applicable.
- Both **beeswarm (dot)** and **bar** plots were produced:
  - Beeswarm shows distribution + direction of feature effects
  - Bar shows mean absolute SHAP (global importance ranking)

**Outcome:** We can now identify which features contribute most strongly to fraud predictions and use that insight to guide Week 2 Day 3 improvements (imbalance handling + tuning).

---

## 6) Reflection — Threshold Choice
**Question:** *Which probability threshold provided the best balance between recall and precision?*

**Approach Used:** **Method B (Highest F1 Score)**  
- F1 is the best single metric here because it combines precision and recall into one value.

✅ **Selected threshold: 0.5 (Random Forest)**
- Precision = **0.9965**
- Recall = **0.9164**
- F1 = **0.9548** *(highest observed for RF)*

This threshold produced the strongest overall trade-off (highest F1) while maintaining extremely high precision.

---

## 7) Assessment — Short Comparison Note (LogReg vs RF)

At the chosen comparison threshold (**0.5**):

### Logistic Regression @ 0.5
- Precision: **0.7787**
- Recall: **0.9398**
- Interpretation: catches slightly more fraud (higher recall), but produces more false positives (lower precision).

### Random Forest @ 0.5
- Precision: **0.9965**
- Recall: **0.9164**
- Interpretation: flags far fewer legitimate transactions as fraud (very high precision), with slightly lower recall.

**Key Difference**
- **Random Forest prioritizes precision strongly** (fewer false positives).
- **Logistic Regression provides higher recall** (catches slightly more fraud).
- Overall, **Random Forest is the stronger baseline** by combined performance (**F1: 0.9548 vs 0.8513**).

---

## 8) Expected Output Delivered
✅ `05_baseline_models.ipynb` (baseline notebook) contains:
- Time-based split
- Logistic Regression & Random Forest training
- Precision/Recall/F1/ROC-AUC evaluation
- Threshold analysis
- SHAP global interpretability plots (beeswarm + bar)

---

## 9) Next Steps (Week 2 Day 3 Readiness)
With solid baselines established, the next improvements will focus on:
- Imbalance handling (SMOTE / class weights / undersampling)
- Advanced models (XGBoost / LightGBM)
- Experiment logging and stability analysis across time