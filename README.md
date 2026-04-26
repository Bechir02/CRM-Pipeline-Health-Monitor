# 🏢 CRM Pipeline Health Monitor via n8n Automation

> A weekly CRM health monitoring workflow that pulls data from HubSpot, runs business health checks, detects revenue risk, tracks CRM data quality, forecasts pipeline coverage, and delivers a structured Slack report, Google Sheets audit log, risk-detail register, and AI executive summary every Monday morning using free-tier tools.

![n8n](https://img.shields.io/badge/n8n-self--hosted-orange)
![HubSpot](https://img.shields.io/badge/CRM-HubSpot%20Free-red)
![Groq](https://img.shields.io/badge/LLM-Groq%20LLaMA%203.1-blue)
![Cost](https://img.shields.io/badge/Cost-%240%2Fmonth-brightgreen)
![Status](https://img.shields.io/badge/Status-Demo%20%2B%20Production%20Modes-success)

---

## ✨ Features

- 🔍 **5 automated health checks** — idle deals, overdue close dates, leads with no logged follow-up, stuck deals, missing data
- 🎯 **Revenue forecast** — probability-weighted pipeline vs monthly target with gap calculation
- 🧭 **Forecast confidence** — classifies forecast reliability based on pipeline gap and at-risk value
- 👤 **Owner-level follow-up risk** — highlights owners with idle deals, overdue deals, or leads with no logged follow-up
- 🧼 **CRM data quality score** — summarizes missing contact info, missing owners, and missing close dates
- 🏆 **Win/loss analysis** — closed won vs closed lost this month with values
- 📈 **Week-over-week comparison** — pipeline growth or decline vs last week
- 🤖 **AI executive summary** — Groq LLaMA generates a 4-sentence manager narrative
- 📋 **Risk details log** — every flagged record logged individually with issue type, severity, and recommended action
- 📊 **Weekly audit log** — aggregated metrics per week for trend tracking
- 🔧 **Demo / production mode switch** — demo mode uses 0-day thresholds to validate all branches; production mode uses realistic 14/30/7-day thresholds
- 💰 **$0/month** on free tiers

---

## 📌 The Problem It Solves

Sales managers at SMBs spend Monday mornings manually checking their CRM — deal by deal, contact by contact — to find what needs attention. Most never finish. Things slip.

This workflow answers the question every sales manager asks but rarely gets a clean answer to:

> *"Which deals are at risk, which leads have no logged follow-up, are we on track for target, and which CRM records or owner-level processes need attention right now?"*

---

## 🏗️ Architecture

```
Schedule Trigger (Monday 7AM)
│
├── Fetch Open Deals (HubSpot API)
├── Fetch Recent Contacts (HubSpot API)
├── Fetch Owners (HubSpot API)
├── Fetch Pipeline Stages (HubSpot API)
├── Fetch Closed Deals (HubSpot API)
└── Read Last Week Log (Google Sheets)
        │
        ▼
   Combine All (Merge — Append)
        │
        ▼
   Set Run Mode → Mode is Demo?
        │
        ▼
   Run Health Checks (Code node)
   ├── Check 1: Idle deals (14+ days no logged CRM activity)
   ├── Check 2: Overdue close dates
   ├── Check 3: Leads with no logged follow-up (7+ days)
   ├── Check 4: Missing critical data
   ├── Check 5: Stuck deals (30+ days same stage)
   ├── Forecast: Probability-weighted pipeline vs target
   ├── Forecast confidence classification
   ├── CRM data quality score
   ├── Owner-level follow-up risk
   ├── Win/loss analysis
   └── Week-over-week comparison
        │
        ├── Build Risk Detail Rows → Log Risk Details (Google Sheets)
        ├── Log Weekly Audit (Google Sheets)
        ├── Send Slack Report
        └── Generate AI Summary (Groq) → Send AI Summary by Email
```

---

## 🔧 Tech Stack

| Tool | Purpose | Free Tier |
|------|---------|-----------|
| **HubSpot Free CRM** | Data source — deals, contacts, owners, pipeline | Unlimited objects |
| **n8n** (self-hosted) | Workflow orchestration + all business logic | Unlimited |
| **Groq LLaMA 3.1 8B** | AI executive summary generation | Generous free tier |
| **Google Sheets** | Audit log + risk details log | Free |
| **Slack Incoming Webhook** | Monday morning report | Free |
| **Gmail** | HTML email with AI summary | Free |

**Total monthly cost: $0**

---

## 📊 Google Sheets Structure

### `audit_log` tab — one row per weekly run

| Column | Description |
|--------|-------------|
| week | ISO week label (e.g. 2026-W17) |
| run_date | Date of run |
| total_open_deals | Open deal count |
| total_pipeline_value | Total open pipeline value |
| at_risk_value | Value of idle + overdue deals |
| at_risk_percentage | % of pipeline at risk |
| idle_deals_count | Deals with no logged CRM activity 14+ days |
| overdue_deals_count | Deals past close date |
| uncontacted_leads_count | Leads with no logged follow-up in CRM |
| stuck_deals_count | Deals in same stage 30+ days |
| missing_contact_info | Contacts with no email or phone |
| deals_no_owner | Deals with no assigned owner |
| deals_no_close_date | Deals missing close date |
| weighted_pipeline | Probability-weighted pipeline |
| forecast_gap | Gap to monthly target |
| forecast_confidence | High / Medium / Low forecast reliability |
| data_quality_score | CRM hygiene score based on missing required fields |
| deals_closed_won | Won deals this month |
| deals_closed_lost | Lost deals this month |
| closed_deal_count | Total closed won + closed lost this month |
| pipeline_change_pct | % change vs last week |
| crm_mode | demo or production |
| idle_days_threshold | Threshold used for idle deal detection |
| stuck_days_threshold | Threshold used for stuck deal detection |
| uncontacted_days_threshold | Threshold used for no logged follow-up detection |

### `risk_details` tab — one row per flagged record

| Column | Description |
|--------|-------------|
| issue_id | Unique ID (week + record + issue type) |
| week | ISO week label |
| run_date | Date of run |
| record_type | deal / contact |
| record_name | Name of deal or contact |
| owner | Assigned owner |
| amount | Deal value (0 for contacts) |
| issue_type | idle_deal / overdue_close_date / no_logged_follow_up / stuck_deal / missing_contact_info / missing_close_date / missing_owner |
| severity | critical / warning |
| days | Days idle / overdue / in stage |
| recommended_action | Specific action for the manager |
| url | Direct HubSpot link |

---

## 🔍 The 5 Health Checks

| Check | Threshold | Severity | What It Catches |
|-------|-----------|----------|-----------------|
| Idle deals | 14 days with no logged CRM activity | 🔴 Critical | Revenue going cold silently |
| Overdue close dates | Past close date still open | 🔴 Critical | Pipeline forecast integrity |
| Leads with no logged follow-up | 7 days since creation and no CRM activity logged | 🔴 Critical | Follow-up discipline and lead response risk |
| Missing critical data | No email+phone / no owner / no close date | 🟡 Warning | CRM usability gaps |
| Stuck deals | 30 days same stage | 🟡 Warning | Pipeline velocity bottlenecks |

---

## 📱 Slack Report Example

```
📊 CRM Health Report — 2026-W17 — 2026-04-24
Demo mode: thresholds set to 0 days for idle/stuck/no-follow-up checks.
Production recommendation: idle 14d, stuck 30d, uncontacted 7d.

🔴 CRITICAL — Act today
→ 9 idle deals ($238,000 at risk)
→ 1 deals past close date ($18,000)
→ 17 new leads with no logged follow-up

🟡 WARNINGS
→ 9 deals stuck 30+ days
→ 3 contacts missing contact info
→ 1 deals with no owner
→ CRM data quality score: 81/100

🎯 FORECAST
→ Monthly target: $150,000
→ Weighted pipeline: $150,800
→ Gap: $-800 (-1%) ✅ On track
→ Forecast confidence: Low

📉 PIPELINE
→ Total: $238,000 ▲ 0% vs last week
→ At risk: $238,000 (100%)

🔥 TOP RISK DEALS
→ 1. Transformation Digitale Tunis — $55,000 — Stage age: 0d
→ 2. Projet ERP Tunis — $45,000 — Stage age: 0d
→ 3. Migration Cloud Sfax — $35,000 — Stage age: 0d

🏆 WIN/LOSS THIS MONTH
→ Won: 2 deals ($18,000)
→ Lost: 2 deals ($21,000)
→ Win rate: 50% based on 4 closed deals

👤 OWNER FOLLOW-UP RISK
→ Becher Zribi — Follow-up risk: High — Idle: 7 Overdue: 1 Uncontacted: 13
→ Unassigned — Follow-up risk: High — Idle: 2 Overdue: 0 Uncontacted: 4

🧭 PRIORITY ACTIONS
→ Fix overdue close dates today to protect forecast accuracy
→ Review idle deals and assign next steps
→ Contact unworked leads within 24 hours
→ Assign owners to unowned deals immediately
```

---

## 🤖 AI Summary Example

> *"The alert volume is intentionally inflated because demo mode is using 0-day thresholds to validate every detection branch. The most actionable real CRM issue is the overdue open deal and incomplete ownership/close-date data, because these affect forecast reliability. Although the weighted pipeline is slightly above target, forecast confidence remains low because a large portion of the open pipeline is flagged as at risk. The manager should first update overdue close dates, assign missing owners, and restore production thresholds before using the report operationally."*

---

## 🔧 Demo vs Production Mode

The workflow includes a **Set Run Mode** node.

| Mode | Thresholds | Purpose |
|------|------------|---------|
| `demo` | idle = 0 days, stuck = 0 days, no logged follow-up = 0 days | Forces all detection branches to fire for testing and screenshots |
| `production` | idle = 14 days, stuck = 30 days, no logged follow-up = 7 days | Realistic weekly CRM monitoring |

Demo mode intentionally inflates issue counts. This is useful for validating the logic with fresh HubSpot demo data. Production mode should be used for real operational runs.

---

## 🔐 Security Notes

Before publishing or sharing this project:

- Remove or rotate any real Slack webhook URLs
- Do not commit HubSpot private app tokens, Groq API keys, Gmail credentials, or Google Sheet IDs
- Replace personal emails and workspace-specific IDs with placeholders
- Export a sanitized workflow file for GitHub

Example placeholders:
```
YOUR_HUBSPOT_PRIVATE_APP_TOKEN
YOUR_SLACK_WEBHOOK_URL
YOUR_GROQ_API_KEY
YOUR_GOOGLE_SHEET_ID
YOUR_EMAIL@example.com
```

---

## 🚀 Setup Guide

### Prerequisites
- Self-hosted n8n instance (v2.0+)
- HubSpot free account
- Google account with Sheets + Gmail access
- Groq account — free tier
- Slack workspace with incoming webhook

### Step 1 — Create HubSpot Private App

1. Go to `app.hubspot.com` → Settings → Integrations → Private Apps
2. Create app named `n8n CRM Monitor`
3. Add scopes: `crm.objects.contacts.read`, `crm.objects.deals.read`, `crm.objects.owners.read`, `crm.schemas.deals.read`, `crm.schemas.contacts.read`, `sales-email-read`
4. Copy the access token — starts with `pat-eu1-` for EU accounts

### Step 2 — Import the workflow

In n8n go to **Workflows → Import from file** and select:
```
workflow/CRM_Pipeline_Health_Monitor.json
```

### Step 3 — Configure credentials

| Credential | Type | Used in |
|-----------|------|---------|
| HubSpot private app token | HTTP Bearer Auth | All HubSpot nodes |
| Groq API key | HTTP Bearer Auth | Generate AI Summary |
| Google Sheets OAuth2 | OAuth2 | Log nodes + Read Last Week Log |
| Gmail OAuth2 | OAuth2 | Send AI Summary by Email |
| Slack Webhook URL | Incoming webhook URL in Send Slack Report node | Send Slack Report |

### Step 4 — Create Google Sheet

Create a sheet named `CRM Health Monitor` with two tabs: `audit_log` and `risk_details`. Use the column structure from the **Google Sheets Structure** section above.

### Step 5 — Configure

In **Set Run Mode** node set value to `production`. Set your monthly target in **Run Health Checks**:
```javascript
const MONTHLY_TARGET = 150000; // Change to your target
```

### Step 6 — Activate

Set Schedule Trigger to Monday 7AM in your timezone and activate.

---

## ⚠️ Known Limitations

- Demo mode intentionally inflates issue counts because thresholds are set to 0
- `notes_last_activity` only reflects activity logged in HubSpot — WhatsApp, phone calls, or emails outside HubSpot will not appear. The report says "no logged activity in CRM" not "no activity happened"
- Designed for a single deal pipeline — multi-pipeline support requires code modification
- HubSpot Free does not expose a full Leads object — contacts are treated as leads in this setup
- Forecast confidence is rule-based, not a statistical model
- The workflow treats contacts created within the configured window as recent leads — adjust the date filter in Fetch Recent Contacts for longer history

---

## 🗺️ Roadmap

- [ ] Multi-pipeline support
- [ ] Per-owner email digest
- [ ] Deal velocity tracking (average days per stage over time)
- [ ] Lead source quality analysis
- [ ] Webhook-triggered real-time alerts on deal stage changes
- [ ] Salesforce connector as alternative CRM source
- [ ] Dashboard visualization from `audit_log` data
- [ ] WhatsApp/phone follow-up gap detection through manual activity tagging
- [ ] Environment variables for run mode and thresholds

---

## 📁 Folder Structure

```
crm-pipeline-health-monitor/
├── workflow/
│   └── CRM_Pipeline_Health_Monitor.json
├── README.md
├── sample-data/
│   └── CRM_Health_Monitor_sample.xlsx
└── screenshots/
    ├── n8n-workflow.png
    ├── slack-report.png
    ├── email-report.png
    ├── audit-log.png
    └── risk-details.png
```

---

## 👤 Author

**Becher Zribi** — Data Automation & Quality Analyst
Building automation workflows for Tunisian businesses and beyond.

[GitHub](https://github.com/Bechir02) · [LinkedIn](https://linkedin.com/in/becher-zribi)

---

## 📄 License

MIT License — free to use, modify, and distribute.
