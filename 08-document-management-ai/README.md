# Document Management AI — Intelligent Classification & Search

## 📋 Summary

**Problem:** Organizations handle thousands of documents (contracts, invoices, policies, reports) across departments, but without consistent classification, files get lost, misfiled, or lack proper access controls. Finding a specific document means searching through folders, emails, and shared drives manually.

**Solution:** AI-powered document pipeline that automatically classifies incoming files into 10 categories, extracts metadata (entities, keywords, sensitivity level), generates vector embeddings for semantic search, and stores everything in Supabase with automatic retention policies and access control alerts.

**Impact:**
- Documents classified in under 5 seconds with 95%+ accuracy
- Semantic search finds documents by meaning, not just keywords
- Restricted documents trigger immediate security alerts
- Full audit trail for compliance (who uploaded what, when)
- Automatic retention period recommendations per document type

## 🔄 Flow

```
┌── Upload Path ───────────────────────────────────────────────┐
│                                                              │
│  Document Upload Webhook                                     │
│       ↓                                                      │
│  Parse Document Metadata                                     │
│       ↓                                                      │
│  AI Agent: Classify Document ← GPT-4o                        │
│       ↓                                                      │
│  Enrich & Build Search Index                                 │
│       ↓                                                      │
│  ┌── Parallel ──────────────────┐                           │
│  │ Supabase: Store Record       │                           │
│  │ OpenAI: Generate Embedding   │                           │
│  └──────────┬───────────────────┘                           │
│             ↓                                                │
│  Check: Restricted Document?                                 │
│  ├── YES → Slack Alert + Respond                            │
│  └── NO  → Respond Success                                  │
└──────────────────────────────────────────────────────────────┘

┌── Search Path ───────────────────────────────────────────────┐
│                                                              │
│  Document Search Webhook                                     │
│       ↓                                                      │
│  Generate Query Embedding                                    │
│       ↓                                                      │
│  Supabase: Vector Similarity Search                          │
│       ↓                                                      │
│  Return Ranked Results                                       │
└──────────────────────────────────────────────────────────────┘
```

## 🗂️ Document Categories

| Category | Examples | Default Retention |
|----------|----------|-------------------|
| CONTRACT | NDAs, SLAs, MSAs, service agreements | 10 years |
| INVOICE | Bills, receipts, POs | 7 years |
| REPORT | Analysis, audits, reviews | 5 years |
| POLICY | SOPs, compliance docs, procedures | Permanent |
| HR | Offer letters, reviews, timesheets | 7 years |
| LEGAL | Court docs, regulatory filings | Permanent |
| MARKETING | Campaigns, proposals, brand assets | 3 years |
| TECHNICAL | Architecture docs, APIs, RFCs | 5 years |
| CORRESPONDENCE | Emails, memos, minutes | 3 years |
| OTHER | Unclassified | 1 year |

## 🔒 Sensitivity Levels

| Level | Description | Auto-Actions |
|-------|-------------|-------------|
| `public` | Can be shared externally | None |
| `internal` | Company-wide access | Logged |
| `confidential` | Department-restricted | Access logged + notification |
| `restricted` | Named-person access only | Immediate Slack alert + audit |

## 🔍 Semantic Search

The search pipeline uses OpenAI embeddings (`text-embedding-3-small`) stored in Supabase with `pgvector` to enable:
- Natural language queries ("find the NDA we signed with Acme last quarter")
- Cross-language search (query in English, find Portuguese documents)
- Similarity threshold (0.7 default, adjustable)
- Category/department filters combinable with semantic search

## ⚙️ Configuration

### Placeholders to Replace

| Placeholder | Description |
|-------------|-------------|
| `YOUR_WEBHOOK_ID` | n8n upload webhook ID |
| `YOUR_SEARCH_WEBHOOK_ID` | n8n search webhook ID |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential |
| `YOUR_SUPABASE_URL` | Supabase project REST URL |
| `YOUR_SUPABASE_ANON_KEY` | Supabase anon/public key |
| `YOUR_SLACK_WEBHOOK_URL` | Slack incoming webhook |

### Supabase Setup

```sql
-- Enable vector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Documents table
CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  doc_id TEXT UNIQUE NOT NULL,
  file_name TEXT,
  file_url TEXT,
  file_type TEXT,
  category TEXT NOT NULL,
  subcategory TEXT,
  title TEXT,
  summary TEXT,
  keywords TEXT[],
  entities TEXT[],
  sensitivity TEXT DEFAULT 'internal',
  language TEXT DEFAULT 'en',
  retention_years INTEGER DEFAULT 7,
  storage_path TEXT,
  search_index TEXT,
  embedding vector(1536),
  uploaded_by TEXT,
  department TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Vector similarity search function
CREATE OR REPLACE FUNCTION match_documents(
  query_embedding vector(1536),
  match_threshold FLOAT DEFAULT 0.7,
  match_count INT DEFAULT 10
)
RETURNS TABLE (
  doc_id TEXT,
  title TEXT,
  category TEXT,
  summary TEXT,
  similarity FLOAT
)
LANGUAGE plpgsql AS $$
BEGIN
  RETURN QUERY
  SELECT
    d.doc_id, d.title, d.category, d.summary,
    1 - (d.embedding <=> query_embedding) AS similarity
  FROM documents d
  WHERE 1 - (d.embedding <=> query_embedding) > match_threshold
  ORDER BY d.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;
```

## 📦 Import Instructions

1. Import `workflow.json` into n8n
2. Set up Supabase database with the SQL above
3. Configure OpenAI and Supabase credentials
4. Replace all `YOUR_*` placeholders
5. Test with a sample document upload
6. Integrate with your file upload pipeline (Google Drive, Dropbox, email)
