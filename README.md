# AI Automation Workflows

Production-ready n8n automation workflows powered by AI agents (GPT-4o). Built for Customer Success, Operations, Legal, HR, and Finance teams.

Each workflow includes a complete `workflow.json` (importable into n8n) and detailed documentation with architecture diagrams, configuration guides, and setup instructions.

## 📂 Workflows

| # | Folder | Name | Description | Trigger |
|---|--------|------|-------------|---------|
| 01 | `cs-scheduled-cancellation` | Win-Back Automation | AI-generated retention emails for scheduled subscription cancellations with discount logic and tone adaptation | Schedule (hourly) |
| 02 | `potential-partners` | Partner Request Router | Classifies partner inquiries (BIG_DEAL / STARTUP / NONPROFIT / AFFILIATE) and auto-responds or escalates | Schedule (6h) |
| 03 | `summary-feedback` | Feedback Response Engine | Categorizes negative product feedback (7 types), generates personalized multilingual responses | Schedule (6h) |
| 04 | `notion-demo-analysis` | Demo Lead Scoring | Analyzes demo requests, scores lead urgency (hot/warm/cold), syncs enriched data to Notion | Schedule (hourly) |
| 05 | `whatsapp-business-ai` | WhatsApp Business AI | Full customer service via WhatsApp Cloud API — AI responses, CRM enrichment, sentiment analysis, smart escalation | Webhook (real-time) |
| 06 | `automated-dashboards` | AI Dashboard Reports | Pulls data from 4+ sources (CRM, support, billing, product), generates executive summaries with trend detection | Schedule (daily 8AM) |
| 07 | `invoice-billing-ai` | Invoice & Billing AI | Stripe webhook processing with multi-region fiscal compliance (EU/BR/US/UK) + AI-powered dunning emails with tone escalation | Webhook + Schedule |
| 08 | `document-management-ai` | Document Management AI | Intelligent document classification (10 categories), metadata extraction, vector embeddings for semantic search, sensitivity alerts | Webhook |
| 09 | `internal-chatbot` | Internal Chatbot | Employee-facing AI assistant for onboarding, company policies, HR, and IT — with knowledge base and smart escalation | Webhook |
| 10 | `contract-analysis-ai` | Contract Analysis AI | Extracts 18 data points from contracts, scores risk (1–10), flags missing clauses, monitors renewals on a 90-day horizon | Webhook + Schedule |

## 🏗️ Architecture

All workflows follow a consistent pattern:

```
Trigger (Webhook / Schedule / Event)
    ↓
Ingest & Parse (Code node — normalize input)
    ↓
Filter / Route (Switch node — classify flow)
    ↓
AI Agent (GPT-4o — analysis, generation, classification)
    ↓
Post-Process (Code node — structure output)
    ↓
Act (API calls — email, Slack, CRM, database)
    ↓
Log (Google Sheets / Supabase — audit trail)
```

## 🚀 Quick Start

1. Import the `.json` file into n8n
2. Configure credentials (see each workflow's README)
3. Update placeholders marked with `YOUR_*`
4. Test with manual trigger
5. Activate schedule or webhook

## 🔧 Technology Stack

| Component | Technology |
|-----------|-----------|
| Orchestration | [n8n](https://n8n.io) (self-hosted or cloud) |
| AI Model | OpenAI GPT-4o |
| Database | Supabase (PostgreSQL + pgvector) |
| Payments | Stripe API |
| Messaging | Slack API, WhatsApp Cloud API |
| Support | Intercom API |
| CRM | Notion / HubSpot / Salesforce (configurable) |
| Tracking | Google Sheets |
| Email | SendGrid / AWS SES / Resend |
| Vector Search | OpenAI Embeddings + Supabase pgvector |

## 🔑 Required Credentials

| Credential | Used By |
|------------|---------|
| OpenAI API | All workflows (AI Agent) |
| Slack API | All workflows (notifications) |
| Intercom API | 01, 02, 03 |
| Notion API | 04 |
| WhatsApp Cloud API | 05 |
| Google Sheets OAuth2 | 06, 07, 09 |
| Stripe API | 07 |
| Supabase API | 08, 09, 10 |

## 📋 Credential Setup

### OpenAI API
1. Go to [platform.openai.com](https://platform.openai.com)
2. Create API key
3. Recommended model: `gpt-4o`

### Slack API
1. Create a Slack App at [api.slack.com](https://api.slack.com)
2. Add OAuth scopes: `channels:history`, `chat:write`
3. Install to workspace
4. Copy Bot User OAuth Token

### Intercom API
1. Go to Settings → Developers → Developer Hub
2. Create new app or use existing
3. Generate Access Token
4. Note your Admin ID (Settings → Your profile)

### Stripe API
1. Go to [Stripe Dashboard](https://dashboard.stripe.com) → Developers → API Keys
2. Copy Secret Key
3. Configure webhooks for invoice events

### Supabase
1. Create project at [supabase.com](https://supabase.com)
2. Copy Project URL and anon key from Settings → API
3. Enable `pgvector` extension for document search
4. Run the SQL migrations from each workflow's README

### WhatsApp Cloud API
1. Create a Meta Business App at [developers.facebook.com](https://developers.facebook.com)
2. Add WhatsApp product
3. Configure webhook URL and subscribe to `messages`
4. Copy permanent access token

## ⚠️ Important Notes

- All sensitive data has been replaced with `YOUR_*` placeholders
- Test workflows with manual trigger before activating schedules
- Check n8n execution history for debugging
- Workflows use HTTP Request nodes for Slack API (more reliable than native node)
- AI prompts are tuned for GPT-4o — adjust temperature/tokens if using other models

## 📈 Coverage by Department

| Department | Workflows |
|-----------|-----------|
| Customer Success | 01, 02, 03, 05 |
| Sales | 04, 06 |
| Finance & Billing | 07 |
| Operations | 06, 08 |
| HR & People | 09 |
| Legal & Compliance | 10 |
| IT & Infrastructure | 08, 09 |
| Marketing | 02 (partner/affiliate) |

## 🧠 AI Consulting Templates (`ai-consulting/`)

Ready-to-use system prompts and n8n workflow templates for enterprise AI implementation across **10 departments**. Each prompt is production-ready with placeholders for client customization.

| Department | Prompts | Workflows | Focus |
|---|---|---|---|
| [Suporte Disruptivo](ai-consulting/suporte-disruptivo/) | 3 | 2 | Emotion detection, proactive outreach, quality analysis |
| [Comercial/Vendas](ai-consulting/comercial/) | 4 | 1 | Sales coaching, auto-proposal, churn prediction, mystery shopper |
| [RH](ai-consulting/rh/) | 3 | 1 | Turnover prediction, autonomous onboarding, continuous evaluation |
| [Jurídico](ai-consulting/juridico/) | 3 | — | Regulatory monitor, compliance audit, due diligence |
| [Finanças](ai-consulting/financas/) | 3 | 1 | Cash flow prediction, fraud detection, narrative reports |
| [Operações](ai-consulting/operacoes/) | 4 | 1 | Smart reorder, route optimization, predictive maintenance |
| [Marketing](ai-consulting/marketing/) | 4 | — | Hyper-segmentation, dynamic pricing, brand sentiment |
| [Gestão Executiva](ai-consulting/gestao-executiva/) | 4 | 1 | Morning briefing, scenario simulation, board reports |
| [TI](ai-consulting/ti/) | 3 | — | Auto-healing, predictive security, license optimization |
| [Formação](ai-consulting/formacao/) | 3 | — | AI mentor, scenario training, knowledge capture |
| **Total** | **34** | **7** | |

→ See [`ai-consulting/README.md`](ai-consulting/README.md) for full documentation.

## 📄 License

MIT License — free to use, modify, and distribute.
