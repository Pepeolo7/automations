# n8n Automations

Customer Success automation workflows, built with n8n.

## 📂 Workflows

| Folder | Name | Description | Trigger |
|--------|------|-------------|---------|
| `01-cs-scheduled-cancellation` | Win-Back Automation | AI-powered retention emails for scheduled cancellations | Schedule (hourly) |
| `02-potential-partners` | Partner Requests | Routes partner inquiries (Typeform) to Intercom/Slack | Schedule (6h) |
| `03-summary-feedback` | Summary Feedback | Processes AI Summary feedback, sends personalized responses | Schedule (6h) |
| `04-notion-demo-analysis` | Demo Lead Analysis | Analyzes demo requests, scores leads, syncs to Notion | Schedule (hourly) |

## 🚀 Quick Start

1. Import the `.json` file into n8n
2. Configure credentials (see each workflow's README)
3. Update placeholders marked with `YOUR_*`
4. Test with manual trigger
5. Activate schedule

## 🔧 Required Credentials

| Credential | Used By |
|------------|---------|
| Slack API | All workflows |
| OpenAI API | All workflows (AI Agent) |
| Intercom API | 01, 02, 03 |
| Notion API | 04 |

## 📋 Credential Setup

### Slack API
1. Create a Slack App at api.slack.com
2. Add OAuth scopes: `channels:history`, `chat:write`
3. Install to workspace
4. Copy Bot User OAuth Token

### Intercom API
1. Go to Settings → Developers → Developer Hub
2. Create new app or use existing
3. Generate Access Token
4. Note your Admin ID (Settings → Your profile)

### OpenAI API
1. Go to platform.openai.com
2. Create API key
3. Recommended model: `gpt-4o`

### Notion API
1. Go to notion.so/my-integrations
2. Create new integration
3. Copy Internal Integration Token
4. Share database with integration

## ⚠️ Important Notes

- All sensitive data has been replaced with placeholders
- Test workflows with manual trigger before activating schedules
- Check n8n execution history for debugging
- Workflows use HTTP Request nodes for Slack API (more reliable than native node)

## 📄 License

Internal use only - Customer Success Team
