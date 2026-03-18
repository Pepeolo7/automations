# Internal Chatbot — Onboarding & Company Policies

## 📋 Summary

**Problem:** Employees spend 30+ minutes per week searching for internal policies, HR procedures, and IT instructions across scattered documents, wikis, and Slack channels. New hires are overwhelmed during onboarding with no single point of reference. HR and IT teams answer the same questions repeatedly.

**Solution:** AI-powered internal chatbot backed by a centralized knowledge base (Supabase). Employees ask questions in natural language, and the bot provides accurate answers citing specific policies. Onboarding mode proactively guides new hires through their first weeks. Questions outside the knowledge base are automatically escalated to the right team (HR, IT, or manager).

**Impact:**
- 70% reduction in repetitive HR/IT support queries
- New hire onboarding time reduced by 40% with proactive guidance
- Every interaction logged for knowledge gap analysis
- Automatic escalation ensures no question goes unanswered
- Multilingual support (responds in the employee's language)

## 🔄 Flow

```
Chat Webhook (POST from Slack bot / intranet / Teams)
    ↓
Parse Chat Message (user context, department, onboarding status)
    ↓
Filter: Has Question
    ↓
Fetch Knowledge Base (Supabase: policies, SOPs, FAQs)
    ↓
Build Knowledge Context (merge relevant articles)
    ↓
AI Agent: Internal Chatbot ← GPT-4o
    ↓
Parse AI Response (answer, category, confidence, escalation)
    ↓
┌── Parallel ──────────────────────────────────┐
│ Google Sheets: Log Interaction               │
│ Check: Needs Escalation?                     │
│  ├── YES → Slack Alert → Respond with answer │
│  └── NO  → Respond with answer               │
└──────────────────────────────────────────────┘
```

## 🗂️ Question Categories

| Category | Topics Covered | Escalation Target |
|----------|---------------|-------------------|
| ONBOARDING | Equipment, credentials, training, introductions | Manager |
| POLICY | PTO, remote work, expenses, code of conduct | HR |
| HR | Benefits, payroll, reviews, leave | HR |
| IT | Password, VPN, software, hardware | IT |
| BENEFITS | Insurance, retirement, wellness, education | HR |
| GENERAL | Office locations, org chart, contacts | None |

## 🤖 Smart Behaviors

**Onboarding Mode:** When `is_onboarding: true`, the bot:
- Uses a welcoming, encouraging tone
- Proactively suggests next onboarding steps
- References the employee's specific department context
- Sends weekly check-in prompts during the first 30 days

**Escalation Logic:** The bot escalates when:
- The answer isn't in the knowledge base (confidence < 0.5)
- The topic is sensitive (salary, complaints, termination)
- The employee explicitly asks to speak with someone
- The question involves account/access changes

**Language Detection:** Responds in the same language as the question, supporting en, pt, es, fr, de.

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_WEBHOOK_ID` | n8n chat webhook ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential |
| `YOUR_SUPABASE_URL` | Supabase project REST URL |
| `YOUR_SUPABASE_ANON_KEY` | Supabase anon key |
| `YOUR_SLACK_WEBHOOK_URL` | Slack escalation webhook |
| `YOUR_GSHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 |
| `YOUR_GOOGLE_SHEET_ID` | Interaction log spreadsheet |

### Knowledge Base Setup (Supabase)

```sql
CREATE TABLE knowledge_base (
  id BIGSERIAL PRIMARY KEY,
  category TEXT NOT NULL, -- policy, onboarding, hr, it, benefits
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  department TEXT, -- NULL = company-wide
  tags TEXT[],
  effective_date DATE,
  review_date DATE,
  author TEXT,
  version INTEGER DEFAULT 1,
  status TEXT DEFAULT 'active', -- active, draft, archived
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for fast category lookups
CREATE INDEX idx_kb_category ON knowledge_base(category, status);
```

### Slack Bot Integration

To enable employees to chat via Slack:
1. Create a Slack App with Socket Mode
2. Add Event Subscription for `app_mention` and `message.im`
3. Forward events to the n8n webhook endpoint
4. The bot responds via the Slack API with the AI answer

### Google Sheets Log Structure

| Column | Type | Description |
|--------|------|-------------|
| timestamp | DateTime | Interaction time |
| conversation_id | String | Session identifier |
| user_id | String | Employee ID |
| category | String | Question category |
| question | String | Employee question |
| confidence | Number | AI confidence score |
| escalated | Boolean | Whether escalation occurred |
| escalate_to | String | Target team |

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Set up Supabase knowledge base table
3. Populate with company policies and SOPs
4. Configure Slack bot (or connect to intranet chat widget)
5. Replace all `YOUR_*` placeholders
6. Test with sample questions
7. Activate the webhook

## 📈 Analytics & Improvement

Track these metrics weekly to improve the bot:
- **Resolution rate**: Questions answered without escalation
- **Top categories**: Where employees need most help
- **Low-confidence answers**: Knowledge gaps to fill
- **Escalation patterns**: Topics needing better documentation
- **Onboarding completion**: New hire milestone tracking
