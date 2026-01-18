# NovaPay Fraud Environment & Success Criteria (Day 1)

## Business context
NovaPay is a digital-first cross-border money transfer company headquartered in Toronto with operations across the United Kingdom, Canada, and the United States. It enables customers to send, receive, and hold multiple currencies with a strong focus on affordability, speed, and a seamless digital experience. Secure transaction processing is central to NovaPay’s customer trust, operational performance, and regulatory standing, making early and accurate fraud detection a strategic priority.

## Fraud environment: key risks
NovaPay’s fraud environment reflects common digital payments threats, amplified by cross-border complexity and real-time expectations:

- **Identity theft & account takeover (ATO):** attackers compromise accounts (phishing, credential stuffing, SIM-swap) and initiate unauthorized transfers.
- **Unauthorized payments:** fraudulent transactions initiated without legitimate customer intent.
- **Transaction laundering / mule networks:** illicit funds routed through chains of accounts and corridors to obscure origin and destination.
- **Device/IP abuse:** suspicious IPs, new devices, and mismatches between declared location and observed IP geography.
- **Velocity anomalies:** bursts of activity over short windows (e.g., 1h/24h transaction velocity) suggesting automation or laundering.
- **Cross-border corridor risk:** certain routes/currency corridors may have elevated risk due to fraud concentration, weaker verification, or known mule flows.

## Operational and regulatory constraints
Fraud controls must operate under tight constraints:

- **Real-time processing:** transactions may need instant approval/decline decisions, limiting manual review.
- **Rules-based limits:** static rules can become outdated and are vulnerable to adaptive fraud tactics.
- **False positives:** blocking legitimate users harms experience and can increase churn.
- **Compliance expectations:** AML and KYC requirements mean decisions should be **auditable and explainable**, especially for high-impact declines and account restrictions.

## Data and modelling implications (early risks)
Fraud detection performance depends heavily on data and labels:

- **Label reliability:** fraud labels may come from investigations, chargebacks, or disputes, which can be delayed or incomplete.
- **Class imbalance:** fraud is typically rare in production. Models must handle imbalance so they do not default to predicting “non-fraud.”
- **Bias/fairness risk:** features tied to geography, KYC tiers, IP risk, and internal scores can produce uneven outcomes across segments. We must monitor error rates (false positives/false negatives) across meaningful groups (e.g., corridor, home country, KYC tier) and ensure legitimate users are not disproportionately impacted.

## Success criteria (what “good” looks like)

### Business success
A successful solution should:
1. **Reduce fraud losses** by catching more fraud earlier.
2. **Lower operational burden** by prioritizing the highest-risk cases for review.
3. **Protect customer experience** by keeping false positives manageable.
4. **Support compliance** via transparent, auditable, explainable decisions.

### Model success (technical KPIs)
The model should:
- Improve **fraud recall** versus a baseline (rules/manual process), while controlling alert volume.
- Balance **precision vs recall** to prevent analyst overload.
- Report **Recall, Precision, F1, PR-AUC** (especially important for imbalanced fraud labels).
- Provide **transaction-level explainability** (e.g., SHAP) to show why a transaction was flagged.
- Be designed for deployment (FastAPI scoring service) as a step toward real-time production scoring.

## Week 1 deliverables mapping
- **Day 1:** Fraud environment + success criteria (this document)
- **Day 2:** Data assessment report (schema, imbalance, bias risks, label reliability)
- **Day 3:** Cleaning pipeline + cleaned dataset + cleaning notes
- **Day 4:** Engineered features + EDA visuals and comparisons fraud vs non-fraud
- **Day 5:** Feature set finalized and ready for modelling (documentation included)