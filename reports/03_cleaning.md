# Day 3: Data Cleaning & Preparation Notes

## Overview
This document records all data cleaning decisions applied to the NovaPay transaction dataset as part of Day 3 of the project.  
The objective of this stage was to produce a clean, analysis-ready dataset while preserving fraud label integrity and ensuring that all transformations are transparent, reproducible, and defensible.

Raw data stored in `data/raw/` was left untouched. All cleaning was performed on a working copy of the dataset.

---

## Initial Dataset
- Rows: 11,400  
- Columns: 26  

---

## Cleaning Actions and Justifications

### 1. Timestamp Parsing and Validation
- The `timestamp` column was converted to a datetime format using safe parsing.
- Rows with invalid or missing timestamps were removed.

**Justification:**  
Time-based features (e.g. velocity, hour-of-day patterns) require valid timestamps. Transactions without valid timestamps cannot be reliably analysed.

- Rows removed: **61**

---

### 2. Duplicate Transaction IDs
- Duplicate values were detected in the `transaction_id` column.
- A total of **199 duplicated transaction IDs** were identified after timestamp validation.
- The dataset was sorted by timestamp, and only the **latest record per transaction ID** was retained.

**Justification:**  
Transaction IDs are expected to uniquely identify a single transaction. Retaining the latest record reflects the final transaction state in payment systems.

- Rows removed: **199**

---

### 3. Amount Field Standardisation
- Two numeric amount fields were required for analysis:
  - `amount_src` (source currency amount)
  - `amount_usd` (USD-normalised amount)
- New numeric columns were created:
  - `amount_src_num`
  - `amount_usd_num`
- Conversion was performed using safe coercion to handle non-numeric values.

**Findings:**
- 4 records contained non-numeric `amount_src` values.
- 300 records contained missing `amount_usd` values.

**Justification:**  
Numeric validation and modelling require consistent numeric types. Original columns were preserved for traceability.

---

### 4. Missing Value Handling (Amounts)
- Missing values in `amount_src_num` and `amount_usd_num` were imputed using the median of each column.

**Justification:**  
`amount_usd` is a derived field, and median imputation provides a transparent, low-bias baseline while preserving data volume and fraud labels.

---

### 5. Invalid Transaction Amounts
- Transactions with non-positive amounts (≤ 0) in either `amount_src_num` or `amount_usd_num` were identified.
- A total of **100 transactions** violated this rule and were removed.

**Justification:**  
Valid payment transactions must have strictly positive amounts. Non-positive values represent logically invalid transactions in this context.

- Rows removed: **100**

---

## Final Dataset Summary
- Final rows: **11,040**
- Final columns: **28**
- New columns added:
  - `amount_src_num`
  - `amount_usd_num`

The cleaned dataset was saved as: cleaned_transactions.csv

---

## Conclusion
The dataset is now clean, internally consistent, and suitable for feature engineering and exploratory data analysis.  
All cleaning decisions were driven by domain logic, explicitly documented, and validated through exploratory checks. This ensures transparency, reproducibility, and defensibility for downstream modelling and evaluation.

The dataset is ready for **Day 4: Feature Engineering & Exploratory Data Analysis**.