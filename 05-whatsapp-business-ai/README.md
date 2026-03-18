# WhatsApp Business AI — Automated Customer Service

## 📋 Summary

**Problem:** WhatsApp is the #1 customer communication channel in many markets, but handling high message volume manually leads to slow response times, inconsistent answers, and missed conversations outside business hours.

**Solution:** AI-powered WhatsApp automation that receives messages via the WhatsApp Cloud API, enriches them with CRM data, generates contextual responses using GPT-4o, and routes complex issues to human agents with full context.

**Impact:**
- First response time reduced from 2-4 hours to under 30 seconds (24/7)
- 60-70% of inquiries resolved automatically without human intervention
- Every interaction logged with sentiment analysis for quality monitoring
- Seamless escalation preserves full conversation context

## 🔄 Flow

```
WhatsApp Cloud API Webhook
    ↓
Parse Message (extract sender, text, type)
    ↓
Filter: Valid Messages Only
    ↓
CRM: Lookup Contact (enrich with plan, LTV, history)
    ↓
Build AI Context (merge message + customer data)
    ↓
AI Agent: Generate Response ← GPT-4o
    ↓
Parse AI Response (reply, category, sentiment, escalate flag)
    ↓
┌─── Switch: Escalate? ───┐
│                          │
├── YES                    ├── NO
│   ↓                      │   ↓
│   Slack: Alert Team      │   WhatsApp: Send Reply
│   ↓                      │   ↓
│   WhatsApp: Ack + ETA    │   Google Sheets: Log
│   ↓                      │
│   Google Sheets: Log     │
└──────────────────────────┘
```

## 🤖 AI Classification

| Field | Values | Purpose |
|-------|--------|---------|
| `category` | billing, technical, feature_question, general, complaint, sales | Route & analytics |
| `sentiment` | positive, neutral, negative | Priority scoring |
| `escalate` | true/false | Human handoff trigger |
| `language` | ISO code (pt, en, es, fr...) | Response language matching |

## 🔀 Escalation Triggers

The AI agent escalates automatically when:
- Billing disputes or refund requests
- Account deletion requests
- Technical issues it cannot diagnose
- Negative sentiment with specific complaints
- Questions outside the knowledge base
- Customer explicitly asks for a human

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description | Where to Find |
|-------------|-------------|---------------|
| `YOUR_WEBHOOK_ID` | n8n webhook ID | Auto-generated on first save |
| `YOUR_WHATSAPP_CREDENTIAL_ID` | WhatsApp Cloud API Bearer token | Meta Business Suite → WhatsApp → API Setup |
| `YOUR_CRM_CREDENTIAL_ID` | CRM API key header auth | Your CRM's API settings |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential | n8n Credentials |
| `YOUR_GSHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 | n8n Credentials |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook | Slack App → Incoming Webhooks |
| `YOUR_GOOGLE_SHEET_ID` | Tracking spreadsheet ID | Google Sheets URL |

### WhatsApp Cloud API Setup

1. Create a Meta Business App at [developers.facebook.com](https://developers.facebook.com)
2. Add WhatsApp product to your app
3. Configure webhook URL: `https://YOUR_N8N_DOMAIN/webhook/whatsapp-webhook`
4. Subscribe to `messages` webhook field
5. Copy the permanent access token
6. Verify webhook with the challenge token

### Google Sheets Structure

Create a sheet named `whatsapp_log` with columns:

| Column | Type | Description |
|--------|------|-------------|
| timestamp | DateTime | Interaction timestamp |
| from | String | Phone number |
| customer_name | String | Contact name |
| message | String | Original message |
| ai_reply | String | AI-generated response |
| category | String | AI classification |
| sentiment | String | Sentiment analysis |
| escalated | Boolean | Whether it was escalated |
| language | String | Detected language |

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Configure all credentials (WhatsApp, CRM, OpenAI, Google Sheets, Slack)
3. Replace all `YOUR_*` placeholders
4. Set up the WhatsApp webhook in Meta Business Suite
5. Test with a message to your WhatsApp Business number
6. Activate the workflow

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| Webhook not receiving messages | Verify webhook URL in Meta, check n8n is accessible publicly |
| AI response in wrong language | Check system prompt language detection rules |
| CRM lookup failing | Verify API URL and phone number format (include country code) |
| Messages not sending | Check WhatsApp access token, verify phone number ID |
| Rate limiting | Meta allows 80 messages/second — add Wait node if batch sending |

## 📈 Metrics to Track

- Auto-resolution rate (non-escalated / total)
- Average response time
- Sentiment distribution over time
- Top categories by volume
- Escalation rate by category
