# CS Scheduled Cancellation - Win-Back Automation

## 📋 Summary

**Problem:** Customers schedule subscription cancellations, leaving limited time to retain them. Manual review of each cancellation is time-consuming and delays response.

**Solution:** Automated workflow that monitors Slack for scheduled cancellation notifications from Stripe, uses AI to analyze cancellation reasons, generates personalized win-back emails, and sends them via Intercom.

**Impact:** 
- Response time reduced from 3-7 days to under 1 hour
- Consistent, personalized retention messaging
- Automatic categorization of cancellation reasons for analytics

## 🔄 Flow

```
Hourly Trigger
    ↓
Slack: Get Messages (#cs-scheduled-cancellation channel)
    ↓
Parse Cancellation Data (extract customer info)
    ↓
Filter: Scheduled Only
    ↓
AI Agent: Generate Email ← OpenAI (gpt-4o)
    ↓
Parse AI Response (subject, body, tag)
    ↓
Intercom: Search Contact
    ↓
Filter: Has Contact
    ↓
Intercom: Send Email
    ↓
Slack: Notify Success
```

## 📊 Data Extracted

| Field | Description |
|-------|-------------|
| `customer` | Customer name |
| `email` | Customer email |
| `mrr` | Monthly recurring revenue |
| `plan_type` | Monthly/Annual |
| `plan` | Pro/Business |
| `effective_date` | When cancellation takes effect |
| `reason` | Cancellation reason category |
| `comment` | Customer's comment |
| `days_until_cancellation` | Calculated urgency metric |
| `isLegacy` | Whether on legacy pricing |

## 🏷️ Cancellation Tags (AI-assigned)

| Tag | Description |
|-----|-------------|
| `competitor` | Switched to another service |
| `not_needed` | No longer needs the product |
| `dont_use` | Not using the product |
| `too_expensive` | Price-related |
| `product_issue` | Technical/quality issues |
| `billing` | Billing problems |
| `other` | Uncategorized |

## 🛠️ Retention Tools

| Tool | When to Use |
|------|-------------|
| **COMEBACK50** | 50% off for 6 months - price-related cancellations within 60 days |
| **Configuration call** | Corporate emails with setup/complexity issues |
| **Direct help** | Specific technical issues mentioned |
| **Feedback request** | Unclear reasons or long-term scheduled cancellations |

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description | Where to Find |
|-------------|-------------|---------------|
| `YOUR_STRIPE_CHANNEL_ID` | Slack channel ID | Right-click channel → View channel details → scroll to bottom |
| `YOUR_SLACK_CREDENTIAL_ID` | n8n Slack credential ID | n8n Credentials |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n OpenAI credential ID | n8n Credentials |
| `YOUR_INTERCOM_CREDENTIAL_ID` | n8n Intercom Bearer Auth ID | n8n Credentials |
| `YOUR_INTERCOM_ADMIN_ID` | Intercom Admin ID | Intercom Settings → Your profile |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook | Slack App → Incoming Webhooks |
| `YOUR_CALENDLY_LINK` | Your Calendly username | calendly.com/YOUR_USERNAME |

### Intercom Setup

1. Create Bearer Auth credential in n8n with your Intercom Access Token
2. Add headers in HTTP Request nodes:
   - `Content-Type: application/json`
   - `Intercom-Version: 2.11`

## 📦 Import Instructions

1. Copy the `workflow.json` content
2. In n8n, go to Workflows → Import from File
3. Paste or select the JSON file
4. Update all placeholders
5. Test with Manual Trigger first
6. Activate when ready

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| No messages found | Check channel ID, verify Slack credential has `channels:history` scope |
| AI response parsing fails | Check OpenAI credential, verify model access |
| Contact not found in Intercom | Customer may not exist in Intercom yet |
| Email not sending | Verify Intercom Admin ID, check API token permissions |

## 📈 Metrics to Track

- Response time (trigger → email sent)
- Win-back success rate by tag
- Discount code usage (COMEBACK50)
- Configuration call bookings
