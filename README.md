# ClearCycle: Revenue Cycle Analytics &amp; Risk Intelligence Platform

A fully operational healthcare revenue cycle management system built in Monday CRM, simulating how collections teams manage patient accounts receivable and client churn risk at scale.

---

## Project Overview

This project replicates the operational data infrastructure of a real-world healthcare revenue cycle company, including patient account tracking, client churn scoring, automated escalation triggers, and a cross-board command center dashboard.

Built to demonstrate understanding of how CRM data pipelines feed human workflows in a revenue cycle operations environment.

---

## What's Built

### Board 1: Patient Accounts Tracker

Tracks individual patient claims moving through the collections pipeline.

**Structure:**
- 20 patient accounts across 4 aging groups
- 7 columns per account: Account ID, Claim Value ($), Days Open, Payer/Insurer, Assigned To, Last Contact Date, Status

**Aging Groups:**
| Group | Purpose |
|---|---|
| Current Month | Newly entered claims, Days Open < 30 |
| Aging 30-60 Days | Claims approaching escalation threshold |
| Aging 60+ Days (Critical) | At-risk claims requiring immediate action |
| Resolved This Month | Closed accounts for audit trail |

**Status Pipeline:**
```
New → Working → Escalated → Resolved → Written Off
```

**Payers tracked:** Medicaid · Blue Cross · Aetna · UnitedHealth · Medicare

---

### Board 2: Churn Risk Accounts

Tracks client relationships (hospitals, clinics) at the macro level, above individual claims.

**Structure:**
- 8 client accounts across 3 risk groups: High Risk · Moderate Risk · Low Risk
- Subitems per account showing next action steps
- Contract Renewal timeline view
- Key Issues Identified per client (Billing / Pricing / Support / Product Features)

**Formula Columns (auto-calculated, no manual input):**

```
Days Since Contact = ROUND(DAYS(TODAY(), {Last Contact Date}), 0)
```
Updates automatically every day.

```
Churn Score (%) = ROUND(MIN(100, (({Open Escalations} * 10) + ({Days Since Contact} * 1)) * (1 + ({Revenue Value ($)} / 1000000))), 0)
```

**Score interpretation:**
| Score | Risk Level | Example |
|---|---|---|
| 80-100 | Critical - immediate intervention | Riverside Medical: 100 |
| 60-79 | High - urgent follow-up | Lakewood Hospital: 86 |
| 40-59 | Moderate - monitor closely | Northside Clinic: 60 |
| 0-39 | Low - routine check-in | Pinecrest Health: 5 |

---

### Dashboard: Revenue Operations Command Center

Cross-board command center pulling live data from both boards simultaneously.

**5 Widgets:**

| Widget | Source | What it shows |
|---|---|---|
| Total $ At Risk | Patient Accounts | Sum of all open claim values = $116,468 |
| Accounts by Status | Patient Accounts | Pie chart: 35% Escalated, 25% Working, 25% Resolved, 15% New |
| Priority Work Queue | Patient Accounts | Aging accounts sorted by claim value descending |
| Churn Risk by Revenue | Churn Risk | Bar chart: revenue at risk per risk tier |
| Critical Accounts | Churn Risk | High + Moderate risk clients with renewal dates and churn scores |

---

### Automations (6 Total - All Active)

**Patient Accounts Tracker (4 automations):**

```
Every day at 8:00 AM
→ Notify analyst: "Daily digest: Review all Escalated accounts and accounts with Days Open 45+"
```

```
When Status changes
AND Status is Escalated
→ Move item to group: Aging 60+ Days (Critical)
```

```
When Status changes
AND Status is Resolved
→ Move item to group: Resolved This Month
```

```
When Status changes
AND Status is Escalated
→ Notify analyst: "Account escalated, immediate follow up required"
```

**Churn Risk Accounts (2 automations):**

```
When Open Escalations changes
AND Open Escalations > 3
→ Notify analyst: "Critical churn alert, account has 4+ escalations, immediate review required"
```

```
Every Monday at 9:00 AM
→ Notify analyst: "Weekly churn review, check all High Risk and Moderate Risk accounts"
```

---

## Data Model

The two boards follow a **star schema** pattern:

```
Board 2 (Churn Risk Accounts) - Dimension table
        One row per client (hospital / clinic)
                |
                | One client → many claims
                |
Board 1 (Patient Accounts Tracker) — Fact table
        One row per patient claim / transaction
```

**The operational link:**
When too many claims in Board 1 go **Escalated** for the same hospital → that hospital's Churn Score in Board 2 rises automatically. The two boards tell a connected story about operational performance and client health.

---

## Churn Scoring - Methodology

The Churn Score formula uses three drivers:

| Driver | Weight | Rationale |
|---|---|---|
| Open Escalations | ×10 | Strongest signal — unresolved claims directly correlate with client dissatisfaction |
| Days Since Contact | ×1 | Recency signal — longer silence = higher churn risk |
| Revenue Value | Multiplier (1 + Revenue/1M) | Higher-value accounts amplified, $284K at risk deserves more urgency than $64K |

**Important caveat:** These weights are heuristic approximations, a defensible baseline before historical data is available. In a production environment, weights would be derived from logistic regression or Random Forest feature importance trained on historical churn outcomes. The formula produces the same output shape as an ML model; the derivation method differs.

**Production architecture this would plug into:**
```
Python ML Model (Random Forest)
        ↓
Azure Data Factory (nightly pipeline)
        ↓
CRM displays score + triggers automations
        ↓
Analyst sees prioritized work queue → acts → revenue recovered
```

---

## Key Metrics Surfaced

| Metric | Value |
|---|---|
| Total revenue at risk (open claims) | $116,468 |
| Accounts in Critical aging bucket | 5 accounts · $44,603 |
| High Risk client revenue at stake | $479,000 |
| Highest churn score | 100 (contract renewing Jun 15) |
| Total automations running | 6 |
| Dashboard widgets | 5 (cross-board) |

---

## Tech Stack

| Tool | Usage |
|---|---|
| Monday CRM | Primary operational platform |
| Formula columns | Live churn scoring + days calculation |
| Dashboard widgets | Cross-board analytics |
| AI Workflows | Automation templates |
| Timeline view | Contract renewal visualization |


---

## Files

```
/
├── README.md                          
└── screenshots/
    ├── Patient_Accounts_Tracker.png
    ├── Churn_Risk_Accounts.png
    ├── Revenue_Operations_Command_Center.png
    ├── Churn_Risk_Timeline.png
    └── Automations_List.png
```

---

## Key Insight

> Data errors in a revenue cycle operation don't just affect reports, they affect what collectors work each morning, which claims get followed up, and ultimately how much revenue gets recovered. Pipeline reliability and data integrity are not technical concerns; they are business outcomes.

---

*Built as a hands-on simulation of healthcare revenue cycle CRM operations. All account names and data are fictional.*
