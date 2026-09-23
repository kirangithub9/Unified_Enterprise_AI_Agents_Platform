# Architecture Documentation — Ansura

*Ansura — a Unified Enterprise AI Agents Platform (Snowflake x Capgemini
Hackathon, Challenge 6)*

## 1. Problem

**Problem statement:** Insurance business users need self-service answers
from three disconnected sources — structured operational data, unstructured
policy documents, and data-quality pipelines — without waiting on a SQL
analyst, document reviewer, or data engineer each time.

Business users at insurers need answers from structured operational data
(policies, claims, billing), unstructured documents (policy contracts,
exclusion clauses), and data-quality pipelines — but today each requires a
different specialist (a SQL/BI analyst, a document reviewer, a data
engineer). Ansura collapses all three into one natural-language
conversation.

## 2. High-level architecture

```
                                        ┌────────────────────────────────┐
                                        │     Streamlit-in-Snowflake     │
                                        │        Chat UI (app.py)        │
                                        └────────────────────────────────┘
                                                         │ REST: agents/{name}:run
                                                         ▼
                                        ┌────────────────────────────────┐
                                        │      ENTERPRISE_AI_AGENT       │
                                        │    (Snowflake Cortex Agent)    │
                                        │  auto-orchestration / routing  │
                                        └────────────────────────────────┘
                                                         │
                  ┌──────────────────────────────────────┬──────────────────────────────────────┐
                  ▼                                      ▼                                      ▼
┌──────────────────────────────────┐   ┌──────────────────────────────────┐   ┌──────────────────────────────────┐
│   Self-Service Analytics Agent   │   │        Document Q&A Agent        │   │        Data Quality Agent        │
│          Cortex Analyst          │   │      Cortex Search Service       │   │          Cortex Analyst          │
│          (text-to-SQL)           │   │      (RAG, auto-embeddings)      │   │          (text-to-SQL)           │
└──────────────────────────────────┘   └──────────────────────────────────┘   └──────────────────────────────────┘
                  ▼                                      ▼                                      ▼
ANALYTICS_SEMANTIC_VIEW                POLICY_DOCUMENT_SEARCH_SVC             DQ_SEMANTIC_VIEW
over ANALYTICS schema                  over DOCUMENTS.DOCUMENT_CHUNKS         over DATA_QUALITY schema
(CUSTOMERS, POLICIES,                  (chunked from POLICY_DOCUMENTS         (DQ_RULES, DQ_RESULTS,
CLAIMS, BILLING, AGENTS,               .CONTENT_TEXT via                      DQ_COLUMN_HEALTH, DQ_SCORES,
AT_RISK_POLICIES)                      SPLIT_TEXT_RECURSIVE_CHARACTER)        VW_DQ_COLUMN_HEALTH_TRENDS,
                                                                              DQ_DOWNSTREAM_IMPACT)
```

Tool names above are the exact wording from the requirement doc
(`tool_spec.name` in `sql/05_create_unified_agent.sql`). The agent runtime
sanitizes spaces/`&` to underscores in anything it actually returns
(`tool_use.name`, logs, traces): `Self-Service_Analytics_Agent`,
`Document_Q_A_Agent`, `Data_Quality_Agent` — confirmed live, see the
"Resolved during build" note below.

Everything runs natively inside Snowflake — one platform, one governance
boundary, no data leaves the account and no external vector database or
orchestration service is required.

## 3. Component design

**Agent 1 — Self-Service Analytics.** A native `SEMANTIC VIEW`
(`ANALYTICS_SEMANTIC_VIEW`) declares business-friendly dimensions, facts, and
metrics (loss ratio, total premium, fraud claim count, revenue at risk,
churn probability, etc.) over six operational tables, with relationships
declared explicitly so Cortex Analyst always joins correctly. Cortex Analyst
translates a natural-language question into governed SQL against this view.

**Agent 2 — Document Q&A (RAG).** `POLICY_DOCUMENTS.CONTENT_TEXT` (already
extracted document text) is chunked with
`SNOWFLAKE.CORTEX.SPLIT_TEXT_RECURSIVE_CHARACTER` into `DOCUMENT_CHUNKS`. A
`CORTEX SEARCH SERVICE` indexes `CHUNK_TEXT`, with `POLICY_ID`,
`DOCUMENT_TYPE`, and `DOCUMENT_TITLE` as filterable attributes. Cortex
Search generates and manages embeddings internally — no separate embedding
pipeline to maintain.

**Agent 3 — Data Quality Root Cause.** A second semantic view
(`DQ_SEMANTIC_VIEW`) sits over `DQ_RULES`, `DQ_RESULTS`, `DQ_COLUMN_HEALTH`,
`DQ_SCORES`, `VW_DQ_COLUMN_HEALTH_TRENDS` (per-column score trend/delta), and
`DQ_DOWNSTREAM_IMPACT` (lineage to affected dashboards/reports), letting a
business user ask "why did X fail", "which column dropped the most", or "is
any downstream reporting impacted" and get a conversational, evidence-backed
answer instead of opening a data-quality dashboard.

**Orchestration.** All three are registered as `tool_resources` on one
`AGENT` object (`ENTERPRISE_AI_AGENT`). Cortex Agents' orchestration model
decides per-question which tool(s) to invoke — the "unified" requirement is
satisfied at the platform level, not by the user having to pick an agent.

**Front end.** A Streamlit-in-Snowflake app calls
`/api/v2/databases/.../agents/ENTERPRISE_AI_AGENT:run` via
`_snowflake.send_snow_api_request`, renders the answer, shows which tool
answered, and — for transparency/trust — expands the generated SQL or
document citations inline. Every question/answer is logged to
`AGENT_INTERACTION_LOG` (including 👍/👎 feedback), which feeds the
dashboard below.

**Snowflake Intelligence dashboards.** A second Streamlit page reads three
views: `VW_PORTFOLIO_RISK_DASHBOARD` (premium/loss-ratio/revenue-at-risk by
policy type and region), `VW_TREND_ANALYSIS` / `VW_CHURN_TREND`
(month-over-month claims, fraud, and churn), and
`VW_AGENT_ACCURACY_METRICS` (query volume and helpful-rate per tool, from
the interaction log above).

**MCP Integration.** `ENTERPRISE_AI_MCP_SERVER` (`CREATE MCP SERVER`)
exposes `ENTERPRISE_AI_AGENT` itself as a single `CORTEX_AGENT_RUN` MCP
tool, so any MCP-compatible client (tested live with Claude's Connectors
UI) gets the same fully-executed, orchestrated answers as the Streamlit
app — not just generated SQL — across all three capabilities.

## 4. Benefits

- **Faster answers** — questions that took hours (waiting on an analyst, a
  document review, or a DQ ticket) now resolve in seconds, in one
  conversation.
- **No new tools to learn** — business users ask in plain English; no SQL,
  no BI dashboard navigation, no ticket queue.
- **Trustworthy by design** — every answer shows its generated SQL or
  source document citation, so it can be verified, not just trusted.
- **One integration, many channels** — a single Cortex Agent (and its MCP
  server) already serves Streamlit, and any MCP-compatible client, without
  separate integrations per capability.