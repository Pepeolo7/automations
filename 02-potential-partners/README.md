# Potential Partners Automation v4

## 📋 Summary

**Problem:** Partner inquiries via Typeform require manual triage and response. Many are routine requests (startup discounts, affiliate signups) that consume CS time.

**Solution:** AI-powered classification system that routes partner inquiries to appropriate responses: auto-emails for standard requests, Slack alerts for strategic opportunities.

**Impact:**
- 70% of partner inquiries handled automatically
- Big deals flagged immediately for manual attention
- Spam filtered automatically

## 🔄 Flow

```
Schedule Trigger (every 6 hours)
    ↓
Slack: Get Messages (#potential-partners)
    ↓
Parse Typeform Data
    ↓
Filter: Not Yet Replied (check thread_reply_count)
    ↓
Filter: Not Spam
    ↓
AI Agent: Classify Deal ← OpenAI (gpt-4o)
    ↓
Parse Classification
    ↓
Switch: By Category
    ├── BIG_DEAL → Slack Alert (manual)
    ├── STARTUP → Intercom Email (STARTUP30)
    ├── NONPROFIT → Intercom Email (proof request)
    └── AFFILIATE → Intercom Email (program details)
```

## 📊 Categories

| Category | Action | Response |
|----------|--------|----------|
| **BIG_DEAL** | Slack Alert | Manual follow-up required |
| **STARTUP** | Auto Email | STARTUP30 code (30% off first year) |
| **NONPROFIT** | Auto Email | Request proof for 30% permanent discount |
| **AFFILIATE** | Auto Email | Program details (20% recurring commission) |

## 🚫 Spam Detection

Automatically filtered if request contains:
- "guest post" / "guest blog"
- "link insertion" / "backlink"
- SEO/content marketing spam patterns

## 🎯 BIG_DEAL Criteria

AI classifies as BIG_DEAL when:
- Reseller/Distributor requests
- 10+ licenses mentioned
- Enterprise context
- Strategic/co-marketing partnerships
- Large audience (10k+)
- Corporate email + vague request
- "Evaluating" or "pilot" mentions

## 📬 Email Templates

### Startup (STARTUP30)
- 30% off first year
- Valid for Pro and Business plans
- Direct link to pricing page

### Non-Profit/Education
- 30% permanent discount
- Requires proof (501c3, .edu email)
- Applied within 24h after verification

### Affiliate
- 20% recurring commission
- 90-day cookie duration
- Monthly payouts
- Link to affiliate signup

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_PARTNERS_CHANNEL_ID` | Slack #potential-partners channel ID |
| `YOUR_SLACK_CREDENTIAL_ID` | n8n Slack credential ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n OpenAI credential ID |
| `YOUR_INTERCOM_CREDENTIAL_ID` | n8n Intercom Bearer Auth ID |
| `YOUR_INTERCOM_ADMIN_ID` | Intercom Admin ID |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook for alerts |

## 📦 Import Instructions

1. Copy the `workflow.json` content
2. In n8n, go to Workflows → Import from File
3. Update all placeholders
4. Test with Manual Trigger
5. Activate schedule

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| Duplicate responses | Check `reply_count` filter is working |
| Wrong classification | Adjust AI prompt or lower temperature |
| Contact not found | Customer may not exist in Intercom |
| Spam getting through | Add more spam keywords to filter |

## 📈 Metrics to Track

- Inquiries by category distribution
- Auto-response rate
- BIG_DEAL conversion rate
- Response time for manual reviews
- Spam filter accuracy
