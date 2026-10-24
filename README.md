# Project SENTINEL: Dataset Understanding Report

**Pod task:** Dataset Understanding, Attribute Analysis and Data Exploration
**Client:** PayVista Payments Pvt. Ltd. (CRD PV-RISK-CRD-2026-014)
**Compiled by:** Shubham Thanage
**Pod discussion:** Saturday 17 October, 8 PM IST

**📄 Read the full report:** `https://<your-github-username>.github.io/<repo-name>/`

---

## About the project

PayVista is an RBI-authorised payment aggregator processing about 4.2 million transactions a day. Its 340-rule fraud system has three problems:

- Losses rose from 4.1 to 9.8 bps.
- It declines or reviews 2.3% of payments, yet only 1 in 19 of those is fraud.
- It cannot explain its decisions.

**SENTINEL-LM** is an in-house fraud LLM. For every transaction it returns a fraud probability, an **ALLOW / REVIEW / BLOCK** decision and a plain-language reason, within **250 ms at p99**.

This report studies the synthetic dataset the system will be designed and tested on.

## Dataset at a glance

| Item | Value |
|---|---|
| Transactions | 1,100,000 (1M main + 100K warmup) |
| Fraud rate | 0.60% (about 1 in 167) |
| Tables / attributes | 18 tables / 171 attributes |
| Customers · merchants · devices | 25,000 · 2,000 · 100,120 |
| Text documents | 39,006 (investigator notes, chargebacks, playbooks, regulatory exercises) |
| Time splits | Train Jan–Oct 2025 · Validation Nov–Dec 2025 · Holdout Jan–Feb 2026 |
| Payment mix | UPI 58% · Card 31% · Wallet 7% · Net banking 4% |

## Report contents

1. **Dataset overview:** purpose, structure, record meaning, splits, table relationships
2. **Data dictionary:** every attribute with its type, business meaning, values and role in SENTINEL
3. **Business mapping:** client requirements (CRD 7.1–7.3, FE-01…10, OBJ-01…12) mapped to the data
4. **Data quality observations:** nulls, duplicates, types, outliers, validation rules, leakage, privacy
5. **AI/ML opportunities:** inputs and targets, preprocessing, feature engineering, approaches, real-time vs batch
6. **Discussion notes:** observations, decisions, assumptions, open questions, learnings
7. **Individual contributions**

## Key findings

- **Context beats amount.** Fraud is about 11× more likely at night (2.72% vs 0.25%), about 11× after a failed attempt, and about 9× on shared devices. Payment channel makes no difference.
- **Missing values are structural, not random noise.**
  - 3% of rows have no customer location.
  - 1% have no device, and all of those are UPI.
  - Missingness carries no fraud signal, so we keep the nulls and add flags.
- **No duplicate keys or documents.** The training text is repetitive, though: the 8,500 gold examples have only 59 distinct targets.
- **Outliers:**
  - Amounts go up to ₹79.9 lakh.
  - 830 UPI payments are above ₹1 lakh.
  - No transactions at all occur between 05:00 and 05:59 IST, which is a generator artefact.
- **Leakage traps:** reviewer verdicts match the label 100% of the time. Simulated decisions and label metadata must also never be model inputs.
- **Label delay plus imbalance:** only 26 validation frauds have mature labels in time, which is too few for calibration.
- **Privacy:** all 39,006 documents were scanned, with zero PII pattern hits. Audit logs still need DPDP-compliant masking and access control.
- **All 10 built-in integrity checks pass** with 0 violations.

## Files

| File | Description |
|---|---|
| `index.html` | The full pod report (served by GitHub Pages) |
| `README.md` | This summary |

## Method

All figures were computed from the complete `sentinel_data` tables (every row, using DuckDB). Client facts come from the SENTINEL Client Requirement Document.

The dataset is fully synthetic. The scores, decisions and latencies inside it are simulated and do not represent model performance.
