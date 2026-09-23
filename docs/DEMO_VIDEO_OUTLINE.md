# Ansura — 5–6 Minute Demo Video Outline

Target: hit all four judging criteria explicitly and on camera — don't make
the judges infer it. Extended from the original 5:00 cut to add the
multi-channel (Streamlit + MCP) and observability beats below — trim the
business-value recap if you need to land back at 5:00.

**0:00–0:30 — Hook + problem statement**
"Business users at an insurer need three different specialists to get an
answer: a SQL analyst for numbers, a document reviewer for policy text, and
a data engineer for 'why is this dashboard wrong.' We built one agent that
replaces all three — meet Ansura." Show the chat UI empty, ready.

**0:30–1:15 — Agent 1: Self-Service Analytics (Innovation + Technical Excellence)**
Type: *"What's our average loss ratio by policy type, and which policy type
has the most fraud-flagged claims?"* Show the answer, then expand "Generated
SQL" to prove it's real governed SQL against `ANALYTICS_SEMANTIC_VIEW`, not
a canned response. Narrate: native Snowflake `SEMANTIC VIEW`, no BI tool.

**1:15–2:15 — Agent 2: Document Q&A / RAG (Technical Excellence + UX)**
Type: *"What are the exclusion clauses for water damage in [policy ID]?"*
Show the answer with the source document cited, expand "Sources" to show
the citation. Narrate: Cortex Search auto-embeds and indexes document
chunks — no external vector DB, no manual embedding pipeline.

**2:15–3:15 — Agent 3: Data Quality Root Cause (Innovation + Business Value)**
Type: *"Why did the data quality check on [column] fail last week?"* Show a
conversational root-cause answer pulling from `DQ_RESULTS`/`DQ_RULES`.
Narrate: this used to mean opening a DQ dashboard and cross-referencing
rule logs manually — now it's one question.

**3:15–3:45 — Prove it's unified, not three separate demos**
Ask one more question that's ambiguous on purpose, and point out the same
chat box routed it correctly without the user picking a tool. Briefly show
the architecture diagram from `docs/ARCHITECTURE.md`.

**3:45–4:20 — Agent 4/channel proof: Streamlit + MCP with Claude (Innovation + Technical Excellence)**
State it plainly: "This isn't just a chat app — the exact same agent is
exposed as an MCP server, so it already works as a tool inside Claude."
Switch to Claude's Connectors UI (or Claude Desktop/claude.ai) with the
"Insurance AI Hub" custom connector already connected, and ask it the same
kind of question live (e.g. *"What's our average loss ratio by policy
type?"*). Show the answer coming back fully formed — not just generated SQL
text, but the executed, orchestrated result. Narrate: this is
`ENTERPRISE_AI_MCP_SERVER` (`sql/07_create_mcp_server.sql`) proxying the
same `ENTERPRISE_AI_AGENT` object the Streamlit app calls via
`CORTEX_AGENT_RUN` — one governed agent, two channels, zero duplicated
logic or re-implemented tools.

**4:20–5:00 — Dashboards: Agent Accuracy & Usage (Business Value + UX)**
Switch to the Streamlit app's **Dashboard** page, **"Agent Accuracy &
Usage"** tab. Two things to show on camera:
- *Usage — all channels* section: point at the three metrics (Total
  Queries, via Streamlit/Direct, via MCP) and the "Queries by channel"
  chart — this is the direct, visual proof that the Claude/MCP question you
  just asked and the Streamlit questions from earlier both landed on the
  same agent. Say explicitly: "Same agent, same governance, counted in one
  place."
- *Accuracy — Streamlit only* section: show the 👍/👎 helpful-rate per tool,
  sourced from `AGENT_INTERACTION_LOG` — this is the UX feedback loop
  judges can see is real, not just a mockup.

**5:00–5:30 — Observability in Snowflake: what's called, what it costs (Technical Excellence + Business Value)**
This is the trust/governance story, and it's worth a beat on its own: every
call to the agent — regardless of whether it came from Streamlit or from
Claude over MCP — is captured automatically by Snowflake's *native* AI
Observability (`SNOWFLAKE.LOCAL.GET_AI_OBSERVABILITY_EVENTS`), with zero
custom logging code required for this layer. Narrate what that trace data
carries per call: which tool/agent span actually ran (Self-Service
Analytics, Document Q&A, or Data Quality), the caller/channel, latency —
and, natively in the same event stream, per-call **token consumption** and
the underlying **Cortex model used** for that step. Say plainly: "Nothing
here is a black box — every answer is traceable back to the exact tool,
model, and token cost that produced it, inside Snowflake's own governance
boundary." If you want this on screen rather than just spoken: open
Snowsight's trace/observability view for `ENTERPRISE_AI_AGENT`, or run a
quick `SELECT * FROM TABLE(SNOWFLAKE.LOCAL.GET_AI_OBSERVABILITY_EVENTS(...))`
in a worksheet and point at the raw event JSON to show the token/model
attributes are already being captured by the platform (`tool_name` and
`channel` are what this project's own `sql/08_unified_agent_observability.sql`
view currently surfaces on the dashboard; token/model are the natural next
columns to add to that same view).

**5:30–6:00 — Business value recap + Close**
State concretely: "This removes SQL/BI dependency for three workflows,
cutting time-to-answer from [X] to seconds, makes every answer auditable
via generated SQL and citations, and now makes every call — from any
channel — auditable down to the tool, model, and token cost that answered
it." Recap the three agents + unified orchestration + multi-channel (Streamlit,
MCP/Claude) access — "That's Ansura" — mention GitHub repo and architecture
doc are included, thank the judges.

## Filming tips

- Screen-record Snowsight full-window at 1080p+, hide any credentials/PII.
- Keep each typed question on screen long enough to read (2-3 sec pause
  before hitting enter).
- Pre-warm the Cortex Search service and semantic views before recording
  (first query after creation can be slower while it indexes).
- Have a backup take of each segment in case an answer is inconsistent —
  LLM-generated SQL can occasionally phrase differently between runs.
- Have the Claude Connectors UI (or Claude Desktop) already signed in as
  `CLAUDE_MCP_USER` with the "Insurance AI Hub" connector showing
  "Connected" *before* you start recording the 3:45–4:20 segment — the
  OAuth sign-in flow itself is not worth showing on camera.
- Ask at least one question through Streamlit and one through Claude/MCP
  *before* recording the dashboard segment (4:20–5:00), so the "via
  Streamlit/Direct" vs "via MCP" split on screen isn't zero.
- Caveat for the 5:00–5:30 observability segment: this project's own
  dashboard view (`sql/08_unified_agent_observability.sql`) currently
  surfaces tool/channel/latency only — token consumption and model-used are
  present in Snowflake's raw `GET_AI_OBSERVABILITY_EVENTS` trace data but
  are not yet pulled into a view/chart. Narrate this segment from the raw
  event table (or Snowsight's trace UI) rather than implying the polished
  dashboard already has a token/cost chart, so the claim on camera stays
  accurate.
