# Notion Demo Analysis

## 📋 Summary

**Problem:** Demo requests from Typeform need to be analyzed, prioritized, and added to Notion for sales follow-up. Manual data entry is time-consuming and inconsistent.

**Solution:** Automated pipeline that parses demo requests from Slack, uses AI to analyze urgency and use case, scores leads, and creates entries in Notion database.

**Impact:**
- Sales team has prioritized lead list in Notion
- AI-generated insights for each lead
- Consistent scoring methodology
- Zero manual data entry

## 🔄 Flow

```
Hourly Trigger
    ↓
Slack: Get Messages (Typeform channel)
    ↓
Filter: Typeform Only ("Request Demo / Enterprise Plan")
    ↓
Parse Typeform Data
    ↓
AI Agent: Analyze Lead ← OpenAI (gpt-4o)
    ↓
Parse AI Response
    ↓
Notion: Create Entry
```

## 📊 Data Extracted

| Field | Source | Description |
|-------|--------|-------------|
| `company` | Typeform | Company name |
| `full_name` | Typeform | Contact name |
| `email` | Typeform | Contact email |
| `company_size` | Typeform | Number of employees |
| `tell_us_more` | Typeform | Request details |
| `tech_stack` | Typeform | Current tools |
| `score` | Calculated | Lead score (1-5) |
| `urgency` | AI | hot/warm/cold |
| `use_case` | AI | Primary use case summary |
| `key_notes` | AI | Important details for sales call |

## 📈 Lead Scoring

| Company Size | Score |
|--------------|-------|
| 100+ | 5 (Enterprise) |
| 50-99 | 4 (Mid-Market) |
| 20-49 | 3 (SMB+) |
| 5-19 | 2 (SMB) |
| < 5 | 1 (Startup) |

## 🎯 Urgency Levels

| Level | Indicators |
|-------|------------|
| **HOT** | Explicit timeline, "evaluating", "ready to buy", urgent language |
| **WARM** | Genuine interest, specific use case, reasonable company size |
| **COLD** | Vague interest, very small company, tire-kicker signals |

## 📋 Notion Database Structure

| Column | Type | Source |
|--------|------|--------|
| Company | Title | Typeform |
| Size | Rich Text | Typeform |
| Urgency | Select | AI (hot/warm/cold) |
| Use-case | Rich Text | AI |
| Tech stack | Rich Text | Typeform |
| Key Notes | Rich Text | AI |
| Potential | Number | Score (1-5) |
| Country | Manual | Optional |

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_TYPEFORM_CHANNEL_ID` | Slack channel where Typeform posts |
| `YOUR_SLACK_CREDENTIAL_ID` | n8n Slack credential ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | n8n OpenAI credential ID |
| `YOUR_NOTION_CREDENTIAL_ID` | n8n Notion credential ID |
| `YOUR_NOTION_DATABASE_ID` | Notion database ID (32 chars from URL) |

### Notion Setup

1. Create a database with the columns listed above
2. Share database with your Notion integration
3. Get database ID from URL:
   ```
   https://www.notion.so/DATABASE_ID?v=VIEW_ID
   ```
4. Column names in n8n must match exactly (case-sensitive)

## 📦 Import Instructions

1. Copy the `workflow.json` content
2. In n8n, go to Workflows → Import from File
3. Update all placeholders
4. Verify Notion column names match
5. Test with Manual Trigger
6. Activate schedule

## 🔍 Troubleshooting

| Issue | Solution |
|-------|----------|
| Notion create fails | Check column names match exactly |
| No data parsed | Verify Typeform message format |
| Wrong database | Check database ID vs page ID |
| AI analysis generic | Add more context to prompt |

## 📈 Metrics to Track

- Leads by urgency distribution
- Average lead score over time
- Conversion rate by urgency level
- Time from request to Notion entry
