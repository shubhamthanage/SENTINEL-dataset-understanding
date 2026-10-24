# Project SENTINEL: Dataset Understanding Report

**Pod task:** Dataset Understanding, Attribute Analysis and Data Exploration
**Client:** PayVista Payments Pvt. Ltd. (CRD PV-RISK-CRD-2026-014)
**Compiled by:** Shubham Thanage
**Pod discussion:** Saturday 17 October, 8 PM IST

### 📄 Read the full consolidated report

| How to open it | Link |
|---|---|
| Live page (GitHub Pages) | [shubhamthanage.github.io/SENTINEL-dataset-understanding](https://shubhamthanage.github.io/SENTINEL-dataset-understanding/) |
| Rendered preview of the HTML file | [Open formatted report](https://htmlpreview.github.io/?https://github.com/shubhamthanage/SENTINEL-dataset-understanding/blob/main/SENTINEL_Pod_Report.html) |
| GitHub repository | [github.com/shubhamthanage/SENTINEL-dataset-understanding](https://github.com/shubhamthanage/SENTINEL-dataset-understanding) |
| Source file in this repo | [SENTINEL_Pod_Report.html](SENTINEL_Pod_Report.html) (same content as [index.html](index.html)) |

> GitHub shows `.html` files as source code. Use one of the first two links to see the formatted report. The [individual contributions](#individual-contributions) and [consolidation process](#how-the-report-was-consolidated) are also reproduced below, so they can be reviewed from this page directly.

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

## Individual contributions

Before the discussion, the pod split the 18 tables and all 171 attributes into five work areas, so that **every attribute had an owner**. On Saturday 17 October, each member explained their attributes to the whole pod, covering type, business meaning, ranges, relationships and their role in SENTINEL. The others then questioned them until everyone could explain every column.

| Member | Tables owned | Attributes | What they explored | What they explained to the pod | Report sections |
|---|---|---|---|---|---|
| **Shubham Thanage** (consolidator) | Cross-table quality review of all 18 tables | All 171 (quality pass) | Profiled every row with DuckDB: nulls, duplicate keys, outliers, foreign keys, the 10 integrity checks, leakage columns, PII scan of all 39,006 documents | Missing-value patterns (UPI-only device nulls), template repetition, the 05:00 IST gap, reviewer-verdict leakage, the 26 mature validation frauds, DPDP obligations | §4 Data quality; consolidation and editing of §1–§7 |
| **[Member 2 name]** | transactions, customers, merchants, devices | 32 | The core payment record and the 14 CRD §7.1 fields; channel mix; amount distribution; paise vs rupee fields; timezone handling | What one record means in the real business; why amount alone is weak; how customer, merchant and device link to each transaction | §1 Overview; §2 Data dictionary (core tables) |
| **[Member 3 name]** | features, labels, payment_outcomes | 34 | FE-01…FE-10 definitions; the 60-day label delay; time-based splits; fraud rate per signal | Point-in-time features; why night, failures and shared devices raise fraud about 10×; why outcome status leaks | §2 (features and labels); §3 Insights; §5 Inputs and targets |
| **[Member 4 name]** | decision_audit, threshold_sets, hard_rules, list_entries, config_changes, reviewer_feedback | 55 | How a probability becomes ALLOW / REVIEW / BLOCK; hard-rule overrides; the audit trail; the reviewer feedback loop | How the rules layer and audit log meet CRD objectives, RBI and auditor needs; real-time vs batch flow | §2 (policy and audit tables); §3 Requirement mapping; §5 Real-time vs batch |
| **[Member 5 name]** | documents, document_chunks, legacy_cases, gold_explanations, preference_pairs | 50 | The LLM corpus: document kinds, CPT eligibility, gold and DPO targets, deduplication status | How CPT, SFT, DPO and retrieval use the text; why repetitive targets are a risk | §2 (text and training tables); §5 ML approaches |

**Total: 32 + 34 + 55 + 50 = 171 attributes**, every one assigned and explained.

## How the report was consolidated

1. **Split the work:** each member took one work area from the table above and profiled their tables.
2. **Pod discussion (17 October, 8 PM IST):** each member presented their attributes. The pod agreed on decisions, assumptions and open questions, which are recorded in §6 of the report.
3. **Consolidation:** Shubham Thanage merged the five write-ups into one report. He used a single data dictionary format, re-checked every figure against the full dataset, and added the cross-table quality review.
4. **Review:** the pod read the consolidated draft and agreed on the final version.
5. **Submission:** every member submits this same report link.

## Files

| File | Description |
|---|---|
| `SENTINEL_Pod_Report.html` | The full consolidated pod report |
| `index.html` | Same report, served as the GitHub Pages home page |
| `README.md` | This summary, including individual contributions |

## Method

All figures were computed from the complete `sentinel_data` tables (every row, using DuckDB). Client facts come from the SENTINEL Client Requirement Document.

The dataset is fully synthetic. The scores, decisions and latencies inside it are simulated and do not represent model performance.
