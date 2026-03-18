# Automated Dashboards — AI-Generated Reports

## 📋 Summary

**Problem:** Stakeholders need daily operational reports (sales pipeline, support health, billing metrics, product usage), but manually pulling data from 4+ sources and formatting reports takes 1–2 hours every morning.

**Solution:** Scheduled n8n workflow that pulls data from CRM, support platform, billing system, and product analytics at 8 AM, feeds it to GPT-4o for analysis and trend detection, then generates a formatted dashboard report delivered via Slack and email.

**Impact:**
- Executive summary ready before morning standup (zero manual effort)
- Cross-source trend detection humans often miss (e.g., support spike correlating with billing changes)
- Historical tracking via Google Sheets for week-over-week comparison
- On-demand refresh via webhook for ad-hoc reporting needs

## 🔄 Flow

```
Schedule Trigger (8 AM weekdays) ─┐
                                   ├─→ Prepare Date Parameters
Manual Refresh Webhook ───────────┘         ↓
                                   ┌── Parallel Data Pull ──┐
                                   │  CRM Pipeline          │
                                   │  Support Metrics        │
                                   │  Billing Data           │
                                   │  Product Analytics      │
                                   └─────────┬──────────────┘
                                             ↓
                                   Merge All Data Sources
                                             ↓
                                   AI Agent: Analyze & Summarize ← GPT-4o
                                             ↓
                                   Format Dashboard (HTML)
                                             ↓
                                   ┌─── Deliver ───┐
                                   │ Slack Channel  │
                                   │ Email (PDF)    │
                                   │ Google Sheets  │
                                   └────────────────┘
```

## 📊 Report Sections

| Section | Source | Metrics |
|---------|--------|---------|
| Sales Pipeline | CRM API | New leads, conversion rate, pipeline value, deals closing this week |
| Support Health | Support API | Open tickets, avg response time, CSAT, ticket categories |
| Revenue | Billing API | MRR, churn rate, new subscriptions, failed payments |
| Product Usage | Analytics API | DAU/MAU, feature adoption, retention cohorts |
| AI Insights | GPT-4o | Trend analysis, anomalies, recommended actions |

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description | Where to Find |
|-------------|-------------|---------------|
| `YOUR_WEBHOOK_ID` | n8n webhook ID for manual refresh | Auto-generated on first save |
| `YOUR_CRM_API` | CRM analytics endpoint base URL | Your CRM API documentation |
| `YOUR_SUPPORT_API` | Support platform API base URL | Support tool API settings |
| `YOUR_BILLING_API` | Billing/Stripe API endpoint | Stripe Dashboard → Developers |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential | n8n Credentials |
| `YOUR_GSHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 | n8n Credentials |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook | Slack App → Incoming Webhooks |
| `YOUR_GOOGLE_SHEET_ID` | Historical tracking spreadsheet | Google Sheets URL |

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Configure all API credentials
3. Replace all `YOUR_*` placeholders
4. Customize data source endpoints for your stack
5. Test with manual webhook trigger
6. Activate the schedule

## 📈 Customization

- Adjust the cron expression for different schedules (e.g., `0 8,17 * * 1-5` for morning + end-of-day)
- Add/remove data sources in the parallel pull section
- Modify the AI prompt to focus on specific KPIs relevant to your business
- Change Slack channel per team (sales gets pipeline focus, support gets ticket focus)
