# Invoice & Billing AI — Fiscal Compliance Automation

## 📋 Summary

**Problem:** Managing invoices across multiple tax jurisdictions (EU VAT, Brazilian ISS/ICMS, US Sales Tax, UK VAT) requires manual compliance checks, and chasing failed payments with dunning emails is time-consuming and inconsistent.

**Solution:** Dual-trigger workflow — Stripe webhooks process invoices in real-time with automatic fiscal compliance validation, while a daily schedule identifies overdue invoices and generates AI-powered dunning emails with escalation logic based on attempt count.

**Impact:**
- 100% fiscal compliance validation on every invoice (multi-region)
- AI-generated dunning emails with tone progression (soft → firm) reduce manual collections effort by 80%
- Automatic overdue detection eliminates missed follow-ups
- Full audit trail via Google Sheets for accounting/finance teams

## 🔄 Flow

```
┌── Stripe Webhook (invoice events) ──┐     ┌── Daily 9AM Schedule ──┐
│                                       │     │                        │
↓                                       │     ↓                        │
Parse Stripe Event                      │     Stripe: Fetch Overdue    │
↓                                       │     ↓                        │
Filter: Invoice Events                  │     Parse Overdue List       │
↓                                       │     │                        │
Route by Event Type                     │     │                        │
├── payment_succeeded ─┐                │     │                        │
│   ↓                  │                │     │                        │
│   Fiscal Compliance  │                │     │                        │
│   Engine             │                │     │                        │
│   ↓                  │                │     │                        │
│   ├── Slack Log      │                │     │                        │
│   ├── GSheets Log    │                │     │                        │
│                      │                │     │                        │
├── payment_failed ────┤────────────────┤─────┤                        │
├── overdue ───────────┘                │     │                        │
│   ↓                                   │     ↓                        │
│   AI Agent: Dunning Email ← GPT-4o ←─┘─────┘                        │
│   ↓                                                                  │
│   Send Dunning Email                                                 │
└──────────────────────────────────────────────────────────────────────┘
```

## 🌍 Fiscal Compliance Engine

The compliance engine automatically detects the tax jurisdiction and validates:

| Region | Tax Label | Standard Rate | Validation |
|--------|-----------|--------------|------------|
| PT (Portugal/EU) | IVA | 23% | NIF, line items, rates |
| BR (Brazil) | ISS/ICMS | Variable | CNPJ, nota fiscal fields |
| US (United States) | Sales Tax | Variable | State-level rates |
| UK (United Kingdom) | VAT | 20% | VAT number, MTD format |

### Compliance Checks
- ✅ Customer tax ID present
- ✅ Line items with descriptions
- ✅ Valid tax rates per region
- ✅ Customer address on record
- ✅ Invoice number format

## 🤖 AI Dunning Logic

| Attempt | Tone | Subject Pattern | Escalation |
|---------|------|----------------|------------|
| 1st | Soft, informational | "Quick note about your payment" | No |
| 2nd | Friendly reminder | "Action needed: update payment method" | No |
| 3rd+ | Firm, professional | "Important: payment required within 48h" | Yes → Slack alert |

The AI agent also:
- Detects customer language from name/email
- Adjusts formality based on invoice amount
- Never uses threatening language
- Always includes self-service billing portal link

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description | Where to Find |
|-------------|-------------|---------------|
| `YOUR_WEBHOOK_ID` | n8n webhook ID | Auto-generated on first save |
| `YOUR_STRIPE_CREDENTIAL_ID` | Stripe API key (Bearer auth) | Stripe Dashboard → Developers → API Keys |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential | n8n Credentials |
| `YOUR_GSHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 | n8n Credentials |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook | Slack App → Incoming Webhooks |
| `YOUR_GOOGLE_SHEET_ID` | Invoice tracking spreadsheet | Google Sheets URL |
| `YOUR_EMAIL_API_URL` | Transactional email endpoint | SendGrid/SES/Resend API |
| `YOUR_BILLING_PORTAL_URL` | Stripe Customer Portal URL | Stripe Settings → Billing → Customer Portal |

### Stripe Webhook Setup

1. Go to Stripe Dashboard → Developers → Webhooks
2. Add endpoint: `https://YOUR_N8N_DOMAIN/webhook/stripe-invoice-webhook`
3. Select events: `invoice.payment_succeeded`, `invoice.payment_failed`, `invoice.overdue`
4. Copy the signing secret for verification

### Google Sheets Structure

Create a sheet named `invoices` with columns:

| Column | Type | Description |
|--------|------|-------------|
| timestamp | DateTime | Processing timestamp |
| invoice_number | String | Generated invoice number |
| customer | String | Customer name |
| email | String | Customer email |
| subtotal | Number | Pre-tax amount |
| tax | Number | Tax amount |
| total | Number | Grand total |
| currency | String | ISO currency code |
| region | String | Fiscal region |
| status | String | Payment status |
| compliance | String | COMPLIANT / FLAGGED |

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Configure Stripe webhook in Stripe Dashboard
3. Set up all credentials (Stripe, OpenAI, Google Sheets, Slack)
4. Replace all `YOUR_*` placeholders
5. Test with Stripe test mode webhook
6. Activate both triggers

## 📈 Metrics to Track

- Collection rate (recovered / total failed)
- Average days to payment recovery
- Compliance validation pass rate
- Dunning email open/response rates
- Escalation rate by attempt number
