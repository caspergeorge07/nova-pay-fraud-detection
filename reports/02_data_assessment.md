# Day 2: Data Sources & Initial Assessment

## 1. Dataset Overview

The dataset contains transaction-level records representing digital money transfers processed by NovaPay. Each row corresponds to a single transaction and includes transactional attributes (amounts, currencies, timestamps), customer-related information (account age, KYC tier), device and network indicators (device ID, IP address, location mismatch), internal risk metrics, and a binary fraud label (`is_fraud`).

The dataset is intended for supervised fraud detection modelling, where the objective is to distinguish fraudulent transactions from legitimate ones.

---

## 2. Schema and Data Types

Initial inspection of the dataset schema revealed that most fields are stored in appropriate data types. However, an inconsistency was identified in the transaction amount fields:

- `amount_usd` is stored as a numeric type (`float64`)
- `amount_src` is stored as an `object` (string)

This inconsistency indicates that `amount_src` likely contains non-numeric values or mixed formatting, preventing direct numerical validation or comparison at this stage. Standardisation of this field will be addressed during the data cleaning phase.

---

## 3. Fraud Label Distribution and Class Imbalance

The fraud label (`is_fraud`) is highly imbalanced, with fraudulent transactions representing a small minority of total observations.

This level of imbalance is typical in real-world fraud detection problems but has important modelling implications:
- Accuracy alone will be a misleading performance metric
- Recall, precision, F1-score, and precision–recall AUC will be more informative
- Imbalance-aware techniques will be required during model development

---

## 4. Data Quality Issues Identified

Several data quality issues were identified during exploratory checks:

### 4.1 Duplicate Transactions
- A total of **200 duplicated `transaction_id` values** were detected.
- As transaction IDs are expected to uniquely identify individual transactions, this suggests potential ingestion, joining, or synthetic data artefacts.
- The nature of these duplicates (exact duplicates vs conflicting records) will be examined and resolved during the data cleaning phase.

### 4.2 Missing and Invalid Values
- Missing values were observed across several transactional and risk-related fields.
- Initial validity checks on transaction amounts revealed that numeric comparisons could not be applied consistently due to data type inconsistencies (see Section 2).

### 4.3 Timestamp Validity
- Timestamp fields require parsing and validation to ensure consistency and suitability for time-based analysis (e.g. velocity features and temporal patterns).

---

## 5. Bias and Fairness Risk Assessment

Several potential sources of bias were identified at the data level:

- **Geographic bias:** Certain countries or corridors may be over-represented in fraud labels.
- **KYC tier effects:** Customers with lower verification tiers may be flagged more frequently, increasing false-positive risk.
- **Device and IP-based indicators:** These features may act as proxies for regional or socioeconomic factors.
- **Label bias:** Fraud labels are derived from investigations, disputes, or chargebacks, which may reflect investigation focus rather than true fraud prevalence.

These risks will be monitored in later stages by evaluating model error rates across meaningful segments (e.g. geography, KYC tier, corridor).

---

## 6. Conclusion

The dataset is suitable for fraud detection modelling but exhibits several issues that must be addressed before model training:

- Severe class imbalance
- Duplicate transaction identifiers
- Inconsistent data types in key numeric fields
- Missing and potentially invalid values
- Potential fairness and bias risks

These findings directly inform the **Day 3 data cleaning and preparation stage**, where explicit rules will be defined, applied, and documented to produce a clean, model-ready dataset.