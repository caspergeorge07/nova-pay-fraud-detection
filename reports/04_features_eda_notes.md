# Day 4: Feature Engineering & Exploratory Data Analysis (EDA)

## Overview
Day 4 focused on engineering predictive features and performing exploratory data analysis to understand how fraudulent and non-fraudulent transactions differ across behavioural, temporal, amount-based, and corridor dimensions. The objective was to identify meaningful patterns that can inform downstream modelling decisions.

The analysis combines descriptive statistics, correlations, and visualisations to ensure insights are both interpretable and operationally relevant.

---

## 1. Temporal Features and Time-of-Day Patterns

### Hour-of-Day Effects
Transaction hour was extracted from the transaction timestamp and analysed against the fraud label.

Visual analysis of fraud rate by hour shows a clear temporal pattern:
- Fraud rates are significantly higher during **early morning hours (approximately 03:00–08:00)**.
- The highest fraud rates occur around **05:00–06:00**, exceeding 20%.
- During typical daytime and evening hours, fraud rates are lower and more stable.

**Interpretation:**  
Fraud activity is more likely to occur during off-peak hours, potentially reflecting reduced human monitoring, automated attack behaviour, or exploitation of time-zone differences.

**Implication:**  
Time-based features such as `tx_hour`, `tx_weekday`, and `is_weekend` provide meaningful predictive signal.

---

## 2. Transaction Amount Behaviour

### Distribution and Central Tendency
Transaction amounts were analysed using both visualisations and summary statistics.

Key findings:
- The **median fraudulent transaction amount (~$550)** is substantially higher than the **median non-fraudulent amount (~$150)**.
- Fraudulent transactions exhibit a more right-skewed distribution with higher-value outliers.

Both standard and log-scaled visualisations were used. Log scaling was applied to accommodate the highly skewed nature of transaction values, where amounts range from small everyday payments to large transfers.

**Interpretation:**  
Higher transaction values are associated with increased fraud likelihood, making `amount_usd_num` a strong candidate feature.

---

## 3. Behavioural Features: Frequency and Velocity

### Transaction Frequency per Customer
Customer-level transaction frequency was computed and analysed.

Key observations:
- Customer activity varies widely, with some customers conducting hundreds or thousands of transactions.
- Fraud rates are highest among **low-activity customers** and decline as customer transaction frequency increases.

**Interpretation:**  
Low-activity accounts may represent newly compromised or synthetic identities, while high-activity customers tend to show more stable behaviour.

### Velocity Features
Velocity-based features proved to be among the strongest indicators of fraud.

Correlation analysis shows:
- `txn_velocity_24h` has a correlation of approximately **0.76** with the fraud label.
- `txn_velocity_1h` has a correlation of approximately **0.69**.

Boxplot visualisations further confirm that fraudulent transactions typically occur during periods of elevated recent activity.

**Implication:**  
Rapid bursts of transactions are a defining characteristic of fraudulent behaviour.

---

## 4. Recency-Based Behaviour (Time Since Previous Transaction)

A recency feature was engineered to capture the time (in hours) since a customer’s previous transaction.

### Recency Feature Summary
- Median time since previous transaction: **~22.4 hours**
- Distribution is highly skewed, with values ranging from minutes to several weeks.

### Fraud vs Non-Fraud Recency
Visual comparison shows:
- Fraudulent transactions tend to occur **much sooner after a previous transaction**.
- Legitimate transactions generally have longer gaps between transactions.

### Fraud Rate by Recency Bucket
When transactions were grouped into recency buckets, the following pattern emerged:

- **Highest fraud rate occurs in the 6–24 hour window (~18.9%)**
- Fraud rates are substantially lower for:
  - Transactions occurring after several days
  - Long inactivity periods (>7 days)

**Interpretation:**  
Fraud is most likely during short to moderate gaps between transactions, consistent with burst-like behaviour seen in account takeover and mule activity.

**Implication:**  
The recency feature complements frequency and velocity, completing the behavioural risk profile.

---

## 5. Corridor-Based Risk (Cross-Border Features)

A transfer corridor feature was created by combining source and destination currencies.

### Corridor Fraud Rates
While some corridors account for high transaction volume, fraud risk is unevenly distributed. The highest fraud rates were observed in corridors such as:
- **CAD → INR**
- **GBP → NGN**
- **USD → MXN**

Some corridors exhibit fraud rates exceeding **30–60%**, significantly above the dataset average.

**Interpretation:**  
Fraud risk is strongly dependent on transaction corridors, reflecting geographic, regulatory, and operational differences.

**Implication:**  
Corridor-based features capture important cross-border risk and should be retained for modelling.

---

## 6. Correlation Analysis Summary

Correlation analysis highlights the most influential numeric features:

- Transaction velocity features (`txn_velocity_24h`, `txn_velocity_1h`)
- Internal risk scores and chargeback history
- Recency and transaction amount features
- Corridor-related risk indicators

These features demonstrate strong alignment with known fraud patterns.

---

## Conclusion
Day 4 analysis confirms that fraud within the NovaPay dataset is structured rather than random. Fraudulent behaviour is characterised by:

- Elevated activity during off-peak hours
- Higher transaction amounts
- Rapid transaction velocity
- Short to moderate recency gaps
- Elevated risk in specific cross-border corridors

The engineered features collectively provide strong predictive signal and form a robust foundation for feature selection and model development in the next stage of the project.