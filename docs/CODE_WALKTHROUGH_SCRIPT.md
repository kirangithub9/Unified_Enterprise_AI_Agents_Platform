# Ansura: Code Walkthrough Script (Plain-English Version)

This is the narration for the code part of the demo. It goes with
`DEMO_VIDEO_OUTLINE.md`, which shows the product in action. This script
explains how it's built, in words anyone can follow, even without a
technical background.

**Format:** **SHOW** tells you what to put on screen. **SAY** is what you
read out loud. The whole script takes about 6 minutes.

---

## 0. The big picture (≈30s)

**SHOW:** The project folders in your editor (`semantic_model/`, `sql/`,
`streamlit/`).

**SAY:**
> "Imagine a big insurance company. People there have three kinds of
> questions. Some are about numbers, like 'How many claims did we get?'
> Some are about documents, like 'What does this policy say about water
> damage?' And some are about data problems, like 'Why is this report
> wrong?' Today, each kind of question goes to a different expert, and the
> answer can take days.
>
> Ansura is one chat box that answers all three. Everything is built inside
> Snowflake, the database where the company's data already lives. The data
> never leaves, and we didn't need to add any extra tools. Let me show you
> the pieces, one at a time."

---

## 1. Teaching the AI what the data means (≈60s)

**SHOW:** `semantic_model/01_create_semantic_view_analytics.sql`, starting
at line 21.

**SAY:**
> "An AI can't answer questions about data it doesn't understand. A table
> called CLAIMS with a column called FRAUD_FLAG means nothing to it at
> first. So this file works like a **dictionary** for the AI.
>
> It lists the tables, and it adds other words people use for them. For
> example, when someone says 'clients' or 'policyholders', they mean the
> CUSTOMERS table."

**SHOW:** `RELATIONSHIPS` at line 37.

**SAY:**
> "It also explains how the tables connect. Every claim belongs to a
> policy, and every policy belongs to a customer. Think of it like a family
> tree, so the AI knows how to link things together correctly."

**SHOW:** The `METRICS` section at line 115, pointing at `fraud_claim_count`
on line 135.

**SAY:**
> "Most importantly, it gives exact definitions. 'Number of fraud claims'
> is defined here, once, as a precise formula. So when anyone asks about
> fraud, the AI always uses this same formula and never guesses. That's
> why its answers can be trusted."

---

## 2. Making documents searchable (≈60s)

**SHOW:** `sql/02_document_search_agent2.sql`, lines 48–54.

**SAY:**
> "The second part answers questions about policy documents. Documents are
> long, and it's hard to search a whole document at once. So we cut each
> one into smaller pieces, like cutting a textbook into **index cards**.
> Each card holds about 1,200 characters, roughly two paragraphs.
>
> Neighbouring cards share a little bit of the same text, about 200
> characters. That way, a sentence that falls on the edge of a card is
> never cut in half and lost."

**SAY:**
> "We also found a problem while building this. The important parts, like
> the list of things a policy does NOT cover, were stored in a different
> place from the main text. If we'd only used the main text, the AI could
> never find them. So we combined everything before making the cards."

**SHOW:** `CREATE OR REPLACE CORTEX SEARCH SERVICE` at line 73.

**SAY:**
> "Then this one command creates a **search engine** for those cards. It
> works like a smart librarian. You ask a question in normal words, and it
> finds the cards that match the *meaning* of your question, even if the
> exact words are different. It also keeps itself up to date whenever the
> documents change."

---

## 3. Finding the cause of data problems (≈75s)

**SHOW:** `sql/03_dq_agent_enhancements.sql`, the comment at the top.

**SAY:**
> "The third part is about data quality. Companies run automatic checks on
> their data, like 'Is every email address valid?' When a check fails,
> someone has to find out why, and that usually takes hours.
>
> We wanted the AI to answer two questions: 'Which column got worse the
> most?' and 'Which reports are affected?' But there was a problem. The
> data had only **one day** of check results. You can't tell if something
> got worse if you only have one day to look at, just like you can't tell
> if you grew taller from one measurement."

**SHOW:** Lines 54–55 (the CUSTOMERS.EMAIL rows).

**SAY:**
> "So we added two earlier weeks of check results. We based them on the
> real current numbers instead of making them up randomly. The email
> column's quality score goes from 99 to 94 to 88, which is a clear drop
> the AI can spot."

**SHOW:** `DQ_DOWNSTREAM_IMPACT` at line 120.

**SAY:**
> "Next, we built a table that works like a **map**. It says: if this
> column has bad data, these reports depend on it and will show wrong
> numbers too. Now the AI can say which reports are in trouble."

**SHOW:** `VW_DQ_COLUMN_HEALTH_TRENDS` at line 148.

**SAY:**
> "Finally, this view does the maths ahead of time. It works out how much
> each column's score changed. The AI just has to read the answer instead
> of working it out itself, which makes it faster and more reliable."

**SHOW:** `sql/04_data_quality_agent.sql`, line 34.

**SAY:**
> "This file is another dictionary, like the first one, but for data
> quality. Notice this note in the definition. It tells the AI to 'use the
> latest check date for current questions'. These notes are like
> instructions a teacher writes on a worksheet: they help the AI avoid
> common mistakes."

---

## 4. The receptionist: one agent, three tools (≈60s)

**SHOW:** `sql/05_create_unified_agent.sql`, line 24.

**SAY:**
> "Now we have three tools: one for numbers, one for documents, and one for
> data problems. This file joins them into **one AI agent**.
>
> Think of it as a **receptionist** at the front desk. You don't need to
> know which department to call. You just ask your question, and the
> receptionist sends it to the right place."

**SHOW:** The `tools:` block at lines 38–50, pointing at each `description`.

**SAY:**
> "How does it know where to send each question? Each tool has a short job
> description. The numbers tool says 'use me for counts, totals, and
> averages'. The documents tool says 'use me for what a policy document
> says'. The agent reads your question, compares it to these descriptions,
> and picks the best match."

**SHOW:** `budget` at lines 34–36.

**SAY:**
> "We also set a limit: each question gets at most 45 seconds and a fixed
> amount of AI processing. It's like a spending limit on a card. It keeps
> costs under control."

---

## 5. The chat app (≈60s)

**SHOW:** `streamlit/app.py`, the `call_agent` function at line 103.

**SAY:**
> "This is the chat screen people actually use. It's intentionally simple.
> It takes the question you typed, sends it to the agent, and shows the
> answer. All the smart work happens in the agent, not in this app."

**SHOW:** Lines 150–174.

**SAY:**
> "When the answer comes back, the app also grabs two extra things to
> show. For number questions, it shows the **exact database query** the AI
> ran, so anyone can check the work, like showing your working in a maths
> exam. For document questions, it shows the **sources**, meaning which
> document the answer came from."

**SHOW:** The 👍/👎 buttons at lines 231–237.

**SAY:**
> "Every question and answer is saved. Users can click thumbs up or thumbs
> down, and we count those votes on the dashboard. That shows us how often
> the AI is actually helpful."

---

## 6. Using the same agent from Claude (≈50s)

**SHOW:** `sql/07_create_mcp_server.sql`, lines 22–30.

**SAY:**
> "People don't only work in our chat app. Some use AI assistants like
> Claude. MCP is a standard way to plug tools into AI assistants, a bit
> like how USB lets you plug any device into any computer.
>
> This file creates that plug. It connects Claude to the **same agent**
> the chat app uses. We didn't build anything twice. It's one brain with
> two doors to reach it."

**SHOW:** `MCP_CLAUDE_ROLE` at line 110.

**SAY:**
> "For safety, Claude signs in as a special user that can only read the
> data this agent needs and nothing else. It's like giving a visitor a key
> card that only opens one room."

---

## 7. Keeping track of everything (≈40s)

**SHOW:** `sql/08_unified_agent_observability.sql`, line 40.

**SAY:**
> "Finally, we want to know how the agent is being used. Snowflake
> automatically keeps a record of every question the agent answers,
> similar to a security camera log. This file reads that log and sorts
> each question by which tool answered it and whether it came from our
> chat app or from Claude."

**SHOW:** `streamlit/pages/1_Dashboard.py`, or the dashboard page running.

**SAY:**
> "The dashboard turns that log into charts. With one look, you can see
> how many questions came from each place, which tools get used most, and
> how often people found the answers helpful."

---

## 8. Wrap-up (≈15s)

**SAY:**
> "To sum up: we taught the AI what the data means, made the documents
> searchable, filled in missing data so it could explain data problems,
> and put one smart receptionist in front of all of it. You can reach it
> from our chat app or from Claude, and every answer can be checked. That's
> Ansura."

---

## Cheat sheet: which views each agent uses, and why

Keep this list open while recording, or show it as a slide if someone
asks which views each part uses.

### 📊 Self-Service Analytics Agent (number questions)

| What it uses | Type | Purpose, in simple words |
|---|---|---|
| `ANALYTICS_SEMANTIC_VIEW` | Semantic view (the "dictionary") | Teaches the AI what the business data means: other names for each table, how the tables connect, and exact formulas for things like "loss ratio" and "fraud claims". |
| `CUSTOMERS`, `POLICIES`, `CLAIMS`, `BILLING`, `AGENTS`, `AT_RISK_POLICIES` | Tables | The raw data. The dictionary above points to these tables. |

### 📄 Document Q&A Agent (document questions)

This agent doesn't use a view. It uses a search engine instead.

| What it uses | Type | Purpose, in simple words |
|---|---|---|
| `POLICY_DOCUMENTS` | Table | The original policy documents (overview, coverage, exclusions). |
| `DOCUMENT_CHUNKS` | Table | The same documents cut into small "index cards" so they're easy to search. |
| `POLICY_DOCUMENT_SEARCH_SVC` | Cortex Search service | The "smart librarian". It finds the cards that match the meaning of a question and returns them as sources. |

### 🛡️ Data Quality Agent (data problem questions)

| What it uses | Type | Purpose, in simple words |
|---|---|---|
| `DQ_SEMANTIC_VIEW` | Semantic view (the "dictionary") | Teaches the AI what the data-quality tables mean and how to measure things like failures, pass rates, and score drops. |
| `DQ_RULES` | Table | The list of checks, like "email must be valid". |
| `DQ_RESULTS` | Table | Whether each check passed or failed on each day, with examples of bad values. |
| `DQ_COLUMN_HEALTH` | Table | A health report card for each column (missing values, duplicates, errors), one per check date. |
| `DQ_SCORES` | Table | Weekly overall quality score for each table. |
| `VW_DQ_COLUMN_HEALTH_TRENDS` | View | Works out ahead of time how much each column's score went up or down, so "which column dropped the most?" is easy to answer. |
| `DQ_DOWNSTREAM_IMPACT` | Table | The "map" of which reports and dashboards depend on which columns, so the agent can say what's affected. |

### Views the agents don't use (for the dashboard only)

The agents never read these. They power the charts on the Dashboard page.

| View | Dashboard tab | Purpose, in simple words |
|---|---|---|
| `VW_PORTFOLIO_RISK_DASHBOARD` | Portfolio & Risk | Totals by policy type, plan, and region: premium, loss ratio, claims, fraud, and money at risk. |
| `VW_TREND_ANALYSIS` | Trend Analysis | Claims per month: how many, how much, fraud count, and how long they take to settle. |
| `VW_CHURN_TREND` | Trend Analysis | Per month: how many policies might be cancelled and how much money that puts at risk. |
| `VW_AGENT_ACCURACY_METRICS` | Agent Accuracy & Usage | For each agent, the number of questions, thumbs up and down, helpful rate, and average speed. Chat app only. |
| `VW_AGENT_USAGE_OVER_TIME` | Agent Accuracy & Usage | Questions per day for each agent. Chat app only. |
| `VW_AGENT_OBSERVABILITY_CALLS` | (used by the two views below) | Reads Snowflake's automatic log of every question and records which agent answered, how long it took, and where it came from (chat app or Claude). |
| `VW_AGENT_USAGE_ALL_CHANNELS` | Agent Accuracy & Usage | Questions per day, split by agent and by channel. Includes Claude. |
| `VW_AGENT_CHANNEL_SPLIT` | Agent Accuracy & Usage | Total questions from the chat app compared with Claude. |

**One line to say on camera:**
> "Two of our three agents each read from their own dictionary, called a
> semantic view: one for business data and one for data quality. The
> document agent uses a search engine instead. All the other views exist
> only to draw the dashboard charts."

---

## Cheat sheet: the tools

A **tool** is a skill the agent can use. Think of the agent as a
receptionist and the tools as the departments it can send your question
to. The tools are defined in `sql/05_create_unified_agent.sql`.

### The three main tools (the ones users see)

| Tool name | Tool type | What it's connected to | What it does, in simple words |
|---|---|---|---|
| **Self-Service Analytics Agent** | `cortex_analyst_text_to_sql` | `ANALYTICS_SEMANTIC_VIEW` | Turns an everyday question into a database query, runs it, and explains the result. |
| **Document Q&A Agent** | `cortex_search` | `POLICY_DOCUMENT_SEARCH_SVC` | Searches the policy documents, reads the 5 best-matching cards, and answers with sources. |
| **Data Quality Agent** | `cortex_analyst_text_to_sql` | `DQ_SEMANTIC_VIEW` | Same process as the analytics tool, but for data-quality questions like "why did this check fail?" |

**How the agent chooses a tool:** each tool has a short job description.
The agent compares your question to those descriptions and picks the best
match. This is called **orchestration**, and it's set to `auto`, so nobody
has to pick a tool by hand.

**Settings for the tools:**
- The two number tools run their queries on the `COMPUTE_WH` warehouse,
  which is the computer that runs the database queries. Each query gets up
  to 60 seconds.
- The whole agent gets up to 45 seconds and 16,000 tokens (a measure of
  how much text the AI processes) per question. It's like a spending limit.

### Helper tools the agent uses behind the scenes

The agent adds these by itself. They don't appear in our SQL files.

| Helper | What it does |
|---|---|
| `system_execute_sql` | Runs the database query that a number tool wrote. The app grabs this query and shows it in the "Generated SQL" box. |
| `data_to_chart` | Can turn results into a chart. |

### The MCP tool (for Claude)

Defined in `sql/07_create_mcp_server.sql`.

| Tool name | Tool type | What it does, in simple words |
|---|---|---|
| `enterprise_ai_agent` | `CORTEX_AGENT_RUN` | The "plug" that lets Claude talk to our agent. Claude sends a question in a field called `text`, and the full agent answers it, choosing between the same three tools. |

---

## Cheat sheet: the Streamlit app

Streamlit is a way to build simple web pages with Python. Our app runs
**inside Snowflake**, so it uses the logged-in user's access and no
passwords are stored in the code.

### The files

| File | What it is |
|---|---|
| `streamlit/app.py` | The **chat page**, the main screen people use. |
| `streamlit/pages/1_Dashboard.py` | The **dashboard page**, with charts in three tabs. |
| `streamlit/environment.yml` | The list of Python packages the app needs. |
| `streamlit/secrets.toml.example` | A template for running the app on your own computer while developing. It's not used inside Snowflake. |
| `streamlit/assets/*.png` | The Ansura logo and chat avatar. |

### What the chat page (`app.py`) does, step by step

1. **Shows the screen:** the Ansura title and logo, a sidebar with example
   questions to try, and a "Clear conversation" button.
2. **You type a question.**
3. **`call_agent`** sends the question to the agent. The agent picks a
   tool and sends back the answer.
4. **Reads the answer** and pulls out four things:
   - the answer text
   - which tool answered, shown as "via 📊 Self-Service Analytics Agent"
   - the database query, if there was one, shown in a **"Generated SQL"**
     box you can open
   - the sources, if any, shown in a **"Sources"** box you can open
5. **`TOOL_LABELS`** turns the tool names the agent sends back (such as
   `Document_Q_A_Agent`) into friendly labels with emojis.
6. **`log_interaction`** saves the question, answer, tool used, and speed
   into the `AGENT_INTERACTION_LOG` table. If saving fails, the chat keeps
   working.
7. **👍 / 👎 buttons** call **`record_feedback`**, which marks that saved
   answer as helpful or not helpful.

### What the dashboard page (`1_Dashboard.py`) does

| Tab | What it shows | Where the data comes from |
|---|---|---|
| **Portfolio & Risk** | Total premium, loss ratio, money at risk, fraud claims, plus bar charts | `VW_PORTFOLIO_RISK_DASHBOARD` |
| **Trend Analysis** | Monthly line charts for claims, fraud, settle time, and policies that might be cancelled | `VW_TREND_ANALYSIS`, `VW_CHURN_TREND` |
| **Agent Accuracy & Usage** | Top half: questions from **both** the chat app and Claude. Bottom half: 👍/👎 helpful rate, from the **chat app only** | Top: `VW_AGENT_CHANNEL_SPLIT`, `VW_AGENT_USAGE_ALL_CHANNELS`. Bottom: `VW_AGENT_ACCURACY_METRICS`, `VW_AGENT_USAGE_OVER_TIME` |

Dashboard data refreshes every 60 seconds.

### How it all connects

```
You ──► Chat page (app.py) ──► ENTERPRISE_AI_AGENT ──┬─► Analytics tool ──► ANALYTICS_SEMANTIC_VIEW
                │                     ▲              ├─► Document tool ───► POLICY_DOCUMENT_SEARCH_SVC
                │                     │              └─► Data Quality tool ► DQ_SEMANTIC_VIEW
                │          Claude ──► MCP server
                ▼
     AGENT_INTERACTION_LOG ──► Dashboard page (1_Dashboard.py)
     Snowflake's own log ────► Dashboard page (counts Claude too)
```

---

## Shorter version (≈3 minutes)

Keep sections 0, 1, 2 (skip the "we also found a problem" part), 4, 5, and
6. Skip sections 3 and 7, and add one sentence instead: "We also built a
data-quality helper and a usage tracker, and both are in the repo."

## Recording tips

- Make your editor font big (16–18pt) so the code is readable on video.
- Scroll to the right spot first, pause, and then start talking.
- Never show passwords, secret keys, or your account URL on screen.
- Check that the line numbers still match if you change any files before
  recording.
