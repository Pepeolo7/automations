# Contract Analysis AI — Automated Extraction & Review

## 📋 Summary

**Problem:** Legal teams spend 2–4 hours per contract on manual review — extracting parties, dates, obligations, and risk clauses from dense legal documents. Expiring contracts with auto-renewal clauses often go unnoticed, leading to unwanted renewals and missed renegotiation windows.

**Solution:** Dual-purpose workflow — a webhook receives uploaded contracts for instant AI-powered analysis (key term extraction, risk scoring, clause identification), while a weekly schedule monitors the contract database for upcoming expirations and auto-renewals, alerting stakeholders with actionable summaries.

**Impact:**
- Contract review time reduced from 2–4 hours to under 2 minutes
- Risk scoring (1–10) with automatic legal team escalation for high-risk contracts
- 18 data points extracted automatically per contract
- Zero missed renewals with 90-day advance warning
- Full searchable contract repository in Supabase

## 🔄 Flow

```
┌── Analysis Path ─────────────────────────────────────────────┐
│                                                              │
│  Contract Upload Webhook                                     │
│       ↓                                                      │
│  Parse Contract Upload                                       │
│       ↓                                                      │
│  AI Agent: Analyze Contract ← GPT-4o                         │
│       ↓                                                      │
│  Parse & Score Analysis                                      │
│       ↓                                                      │
│  Supabase: Store Contract                                    │
│       ↓                                                      │
│  Check: Needs Legal Review?                                  │
│  ├── YES (risk ≥ 7 or HIGH risks) → Slack: Legal Alert       │
│  └── NO  (low risk)               → Slack: Auto-Approved     │
│       ↓                                                      │
│  Respond: Analysis Result                                    │
└──────────────────────────────────────────────────────────────┘

┌── Renewal Monitoring Path ───────────────────────────────────┐
│                                                              │
│  Weekly Schedule (Mon 9AM)                                   │
│       ↓                                                      │
│  Supabase: Fetch Contracts Expiring ≤ 90 days                │
│       ↓                                                      │
│  Parse Expiring Contracts                                    │
│       ↓                                                      │
│  Filter: Has Expiring?                                       │
│       ↓                                                      │
│  Slack: Weekly Renewal Report                                │
└──────────────────────────────────────────────────────────────┘
```

## 📊 Extracted Data Points (18)

| # | Field | Description |
|---|-------|-------------|
| 1 | contract_type | NDA, MSA, SLA, SOW, Employment, Lease, License, Amendment |
| 2 | parties | Names and roles of all parties |
| 3 | effective_date | When the contract begins |
| 4 | expiration_date | When the contract ends |
| 5 | auto_renewal | Whether auto-renews, notice period, renewal term |
| 6 | total_value | Amount, currency, payment terms |
| 7 | key_terms | Named clauses with summaries |
| 8 | obligations | Per-party obligations with deadlines |
| 9 | termination | Conditions, notice period, penalties |
| 10 | liability | Cap, indemnification, insurance requirements |
| 11 | confidentiality | Duration, scope, exceptions |
| 12 | ip_ownership | Owner, license type, restrictions |
| 13 | governing_law | Jurisdiction, dispute resolution |
| 14 | risks | Severity, description, recommendation per risk |
| 15 | missing_clauses | Standard clauses that should be present |
| 16 | language | Document language |
| 17 | overall_risk_score | 1–10 composite risk rating |
| 18 | executive_summary | 3–5 sentence plain-language summary |

## ⚠️ Risk Detection Rules

| Risk Pattern | Severity | Recommendation |
|-------------|----------|----------------|
| Auto-renewal without opt-out notice | 🔴 HIGH | Add notice period clause |
| No liability cap | 🔴 HIGH | Negotiate cap at contract value |
| Unlimited indemnification | 🔴 HIGH | Limit to direct damages |
| IP assignment without limits | 🔴 HIGH | Scope to project deliverables |
| Data processing without GDPR | 🔴 HIGH | Add DPA or processing clauses |
| Non-compete > 12 months | 🟡 MEDIUM | Negotiate narrower scope |
| Missing governing law | 🟡 MEDIUM | Add jurisdiction clause |
| One-sided termination | 🟡 MEDIUM | Add mutual termination rights |
| Payment terms > 60 days | 🟡 MEDIUM | Negotiate to net-30 |

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_WEBHOOK_ID` | n8n contract webhook ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential |
| `YOUR_SUPABASE_URL` | Supabase project REST URL |
| `YOUR_SUPABASE_ANON_KEY` | Supabase anon key |
| `YOUR_SLACK_WEBHOOK_URL` | Slack legal team webhook |

### Supabase Setup

```sql
CREATE TABLE contracts (
  id BIGSERIAL PRIMARY KEY,
  contract_id TEXT UNIQUE NOT NULL,
  file_name TEXT,
  file_url TEXT,
  contract_type TEXT,
  parties JSONB,
  effective_date DATE,
  expiration_date DATE,
  auto_renewal JSONB,
  total_value JSONB,
  governing_law JSONB,
  risk_score INTEGER,
  high_risks INTEGER DEFAULT 0,
  executive_summary TEXT,
  full_analysis JSONB,
  submitted_by TEXT,
  department TEXT,
  status TEXT DEFAULT 'pending_review', -- pending_review, approved, rejected, expired
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for renewal monitoring
CREATE INDEX idx_contracts_expiry ON contracts(expiration_date, status)
  WHERE status = 'approved';

-- Index for risk-based queries
CREATE INDEX idx_contracts_risk ON contracts(risk_score DESC);
```

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Set up Supabase contracts table
3. Configure OpenAI and Supabase credentials
4. Replace all `YOUR_*` placeholders
5. Test by uploading a sample contract via webhook
6. Activate both triggers (webhook + weekly schedule)

## 🔗 Integration Options

- **Email intake**: Add a mailbox trigger to auto-process contracts received as attachments
- **Google Drive**: Monitor a shared folder for new contract uploads
- **Slack slash command**: `/analyze-contract [url]` for ad-hoc analysis
- **PDF extraction**: Pre-process with Apache Tika or pdf-parse for OCR'd contracts
