# Week 2 · Day 4 — Explainability & Stakeholder‑Ready Interpretation  
**Project:** NovaPay — Fraudulent Transaction Detection for Digital Money Transfer  
**Deliverables:** `07_explainability.ipynb` + review template Markdown

---

## 1) Why this day matters (business + regulator view)

Fraud detection is not only about accuracy — it must also be **explainable**, **auditable**, and **actionable**:

- **Analysts** need clear *reason codes* to review cases quickly and consistently.  
- **Stakeholders (Ops / Compliance / Risk)** need confidence that the model flags transactions for sensible, defensible reasons.  
- **Regulators / auditors** expect transparent logic, repeatable explanations, and evidence that the model is not “memorising” identifiers or introducing unfair bias.

**Goal for Day 4:** Build transaction‑level explanations using SHAP and produce stakeholder‑ready interpretation artefacts.

---

## 2) What we implemented in `07_explainability.ipynb`

### 2.1 Time‑based split (realistic deployment simulation)
We kept the time ordering and used an **80/20 time split**:

- **Train:** earlier transactions  
- **Test:** later transactions  

This reduces leakage and better matches production behaviour (fraud patterns evolve over time).

### 2.2 Feature hygiene (critical for explainability)
During local explanations, we observed that **identifier‑like fields** (e.g., `transaction_id`, `customer_id`, `device_id`, `ip_address`) dominated explanations with extremely large SHAP values. This is a red flag for:

- **Non‑actionable explanations** (IDs aren’t meaningful “reasons”)
- **Potential leakage / memorisation** (high‑cardinality features can let the model “remember” specific IDs)

✅ **Fix applied:** These identifier columns were removed from the categorical feature set before preprocessing and modelling.

**Impact:** Explanations became meaningful and operational (transaction velocity, risk scores, KYC tier, etc.).

### 2.3 Model selected for explainability (stakeholder‑friendly)
We trained and explained a model that is easier to justify operationally:

- **Random Forest** with `class_weight="balanced_subsample"`  
- No synthetic oversampling was used for Day 4 explainability output (more regulator‑friendly).

### 2.4 SHAP local explanations (transaction‑level “reason codes”)
We implemented a robust SHAP workflow for tree models:

- Transform features through the **same preprocessing pipeline** used for training.
- Use SHAP **TreeExplainer** with:
  - `feature_perturbation="interventional"`
  - a **background dataset** sampled from training data
  - `check_additivity=False` to avoid the known additivity‑check crash in some RF settings
  - `model_output="probability"` so explanations align to probability space (more intuitive for stakeholders)

---

## 3) What the outputs mean for stakeholders

### 3.1 Transaction‑level explanation example (local SHAP)
For a **high‑confidence flagged transaction** (fraud probability ≈ **1.0**), the top drivers (reason codes) were:

1) **txn_velocity_24h**  
2) **txn_velocity_1h**  
3) **risk_score_internal**  
4) **ip_risk_score**  
5) **account_age_days**  
(plus supporting signals such as device trust, chargeback history, location mismatch, and low KYC tier)

**Stakeholder interpretation (plain English):**
- The account is behaving like a “burst” pattern (many transactions in a short time).  
- Internal risk indicators and risky network signals are high.  
- The account is relatively new, which increases risk.  
- Supporting features suggest weaker verification or unusual behaviour patterns.

**Why this is useful operationally:**
- These are **actionable** reasons an analyst can understand and validate.
- The logic aligns with typical fraud typologies (ATO, mule accounts, automated bursts).

> Note: RF probability can reach 1.0 if most trees vote strongly for fraud. Stakeholders should interpret “1.0” as **very high confidence**, not absolute certainty.

---

## 4) Reflection support — False positive drivers

### 4.1 At threshold 0.5
At the default threshold (**0.5**) the model produced:

- **False positives (FP): 0**

**Meaning:** At this operating point, the model is extremely conservative on flagging legitimate transactions (very high precision).

### 4.2 Sensitivity test (lower threshold to observe FP patterns)
To answer the reflection question (“Which features most often lead to false positives and why?”), we reduced the threshold to simulate a more aggressive flagging policy.

- At **threshold 0.4 → FP = 0**  
- At **threshold 0.3 → FP = 2**

**Top FP drivers at threshold 0.3 (mean |SHAP| across FP cases):**
- `account_age_days` (~0.0477)  
- `amount_src_num` (~0.0361)  
- `amount_usd_num` (~0.0359)  
- `fee` (~0.0323)  
- `amount_usd` (~0.0311)

**Stakeholder interpretation (why these create false positives):**
- **New accounts** and **high‑value transfers** are real fraud risk signals — but they also occur in legitimate scenarios:
  - New customers making first transfers  
  - Tuition/rent/family support payments  
  - Urgent transfers that generate higher fees  
- Lowering the threshold increases sensitivity and begins to classify “legit but risky‑looking” behaviour as fraud.

---

## 5) Analyst review template (stakeholder‑ready artefact)

We generated a Markdown template that can be used directly in:
- fraud ops playbooks,
- case review tickets,
- audit packs.

The template includes:
- fraud probability (“confidence”),
- threshold used,
- top reason codes (positive SHAP),
- analyst outcome + notes + escalation fields,
- audit metadata (reviewer, date, case link).

✅ Saved by the notebook to:  
`../reports/Week_2_Day_4_Fraud_Review_Template.md`

---

## 6) What this means for decision‑makers

### Operational takeaways
- At threshold **0.5**, the model is highly precise (no false positives observed in this run).  
  - Pros: fewer unnecessary investigations, good customer experience  
  - Cons: may miss some borderline fraud (depends on FN rate)

- If the business wants **higher recall**, lowering the threshold increases flagged volume but introduces FP risk mainly driven by:
  - **new accounts**
  - **high transfer amounts**
  - **fees correlated with urgent/high-value transfers**

### Governance takeaways
- Removing identifier‑like fields improved interpretability and reduced memorisation risk.
- Explanations are now aligned with auditable “reason code” logic.

---

## 7) Recommended next steps
1) **Agree an operating threshold policy** (Ops + Risk):
   - choose threshold based on investigation capacity + fraud loss tolerance
2) **Standardise reason codes**:
   - map SHAP top features into a fixed “reason catalogue” (e.g., Velocity, KYC, Network risk, Account age)
3) **Fairness & monitoring**:
   - check if some reason codes disproportionately affect certain user segments (only where legally/ethically appropriate)
4) **Deployable artefacts**:
   - export templates to PDF and create a short SOP for case reviewers

---

## Appendix — What was delivered
- ✅ `07_explainability.ipynb` (local explainability + FP driver analysis + template generation)
- ✅ `Week_2_Day_4_Fraud_Review_Template.md` (analyst/stakeholder case review template)
