# Summary Feedback Automation

## 📋 Summary

**Problem:** Users submit feedback about AI Summary quality via Typeform. Manual review and response to each piece of feedback is time-consuming, especially for negative feedback that requires immediate attention.

**Solution:** AI-powered feedback triage that categorizes feedback, prioritizes based on user plan, and generates personalized responses via Intercom.

**Impact:**
- Negative feedback addressed within hours instead of days
- Consistent, helpful responses tailored to feedback type
- Priority handling for paying customers
- Automatic categorization for analytics

## 🔄 Flow

```
Schedule Trigger (every 6 hours)
    ↓
Slack: Get Messages (#summary-feedback)
    ↓
Parse Feedback Data
    ↓
Filter: Negative Only (rating = down)
    ↓
AI Agent: Categorize & Respond ← OpenAI (gpt-4o)
    ↓
Parse AI Response
    ↓
Intercom: Search Contact
    ↓
Filter: Has Contact
    ↓
Intercom: Send Email
    ↓
Slack: Notify
```

## 📊 Feedback Categories

| Category | Keywords/Patterns | Response Focus |
|----------|-------------------|----------------|
| **SUMMARY_TOO_SHORT** | "too short", "missing", "sparse", "more details" | Detailed Notes view, custom templates, Ask AI |
| **INACCURACY** | "inaccurate", "wrong", "missed conclusion" | Apologize, request specifics, offer review |
| **FORMAT_REQUEST** | "word document", "pdf", "export" | Explain export options, log feature request |
| **TRANSLATION_ISSUE** | "translate", "language", "wrong language" | Check settings, offer regenerate |
| **TRANSCRIPTION_QUALITY** | "voice recognition", "name wrong" | Vocabulary feature for custom names |
| **FEATURE_REQUEST** | "action items", "who says what" | Thank them, confirm logged |
| **VAGUE_NEGATIVE** | Generic negative without specifics | Ask for specifics |

## 🎯 Priority Levels

| Priority | Criteria |
|----------|----------|
| **HIGH** | Paying customer (Pro/Business) + accuracy/quality issue |
| **MEDIUM** | Free user + specific complaint |
| **LOW** | Vague feedback or feature request |

## 📊 Data Extracted

| Field | Description |
|-------|-------------|
| `feedback_text` | User's feedback comment |
| `rating` | up/down (thumbs) |
| `user_email` | Customer email |
| `user_role` | free/pro/business/trial |
| `meeting_id` | UUID of the meeting |
| `company_size` | Company size (number) |
| `is_negative` | Boolean (rating = down) |
| `is_paying` | Boolean (pro/business/trial) |

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_FEEDBACK_CHANNEL_ID` | Slack #summary-feedback channel ID |
| `YOUR_SLACK_CREDENTIAL_ID` | n8n Slack credential ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n OpenAI credential ID |
| `YOUR_INTERCOM_CREDENTIAL_ID` | n8n Intercom Bearer Auth ID |
| `YOUR_INTERCOM_ADMIN_ID` | Intercom Admin ID |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook for notifications |

## 📦 Import Instructions

1. Copy the `workflow.json` content
2. In n8n, go to Workflows → Import from File
3. Update all placeholders
4. Test with Manual Trigger
5. Activate schedule

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| No feedback parsed | Check Typeform message format in Slack |
| Wrong category | Adjust AI prompt temperature |
| Positive feedback being processed | Check rating filter logic |
| Missing user_role | Typeform may not include this field |

## 📈 Metrics to Track

- Feedback volume by category
- Response rate (emails sent / negative feedback)
- Priority distribution
- Resolution time by category
