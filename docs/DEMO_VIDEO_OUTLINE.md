# Ansura: Demo Video Outline (5–6 Minutes)

This is the plan for the product demo video. It shows what Ansura does. The
companion script, `CODE_WALKTHROUGH_SCRIPT.md`, explains how it's built.

**Goal:** the judges score four things: **Innovation**, **Technical
Excellence**, **Business Value**, and **User Experience**. Each section
below says which one it covers. Say the point out loud. Don't expect the
judges to work it out on their own.

**Format:** **DO** is what happens on screen. **SAY** is what you read out
loud. If you need to finish in 5 minutes, shorten the last section.

---

## 0:00–0:30 · The problem

**DO:** Show the Ansura chat screen, empty and ready.

**SAY:**
> "At an insurance company, people have three kinds of questions, and each
> kind needs a different expert. Number questions go to a data analyst.
> Questions about what a policy says go to someone who reads the
> documents. And 'why is this report wrong?' goes to a data engineer.
> Getting an answer can take days.
>
> We built one assistant that does all three jobs. Meet Ansura."

---

## 0:30–1:15 · Number questions *(Innovation + Technical Excellence)*

**DO:** Type:
> *"What's our average loss ratio by policy type, and which policy type has
> the most fraud-flagged claims?"*

Show the answer. Then open the **"Generated SQL"** box.

**SAY:**
> "You ask in plain English, and you get a real answer from real data. This
> box shows the exact database query the AI ran, so anyone can check its
> work, like showing your working in a maths exam.
>
> The AI isn't guessing. It uses a 'dictionary' we built in Snowflake,
> called a semantic view, which gives it exact definitions for business
> terms like 'loss ratio'. No reporting tool is needed, and nobody has to
> write SQL."

---

## 1:15–2:15 · Document questions *(Technical Excellence + User Experience)*

**DO:** Type:
> *"What are the exclusion clauses for water damage in policy POL-1023?"*

Show the answer. Then open the **"Sources"** box.

**SAY:**
> "Now a question about what a policy document says. Ansura finds the right
> part of the right document and answers in plain words. The Sources box
> shows exactly which document the answer came from, so you don't have to
> take the AI's word for it.
>
> Behind the scenes, Snowflake's built-in search engine, Cortex Search,
> does the heavy lifting. It understands the *meaning* of the question, not
> just the keywords. We didn't need an outside search database or any
> extra setup."

---

## 2:15–3:15 · Data problem questions *(Innovation + Business Value)*

**DO:** Type:
> *"Why did the DQ check on CUSTOMERS.EMAIL fail last week?"*

Show the answer. If you have time, follow up with:
> *"Is any downstream reporting impacted by the CUSTOMERS data quality
> issues?"*

**SAY:**
> "Companies run automatic checks on their data, like 'every email address
> must be valid'. When a check fails, someone normally has to dig through
> dashboards and logs for hours to find out why.
>
> Now it's one question. Ansura explains what went wrong, shows how the
> score dropped over time, and even tells you which reports are affected
> by the bad data."

---

## 3:15–3:45 · One assistant, not three separate demos

**DO:** Ask a question that doesn't say which tool to use, for example:
> *"How healthy is our customer data?"*

(Test this before recording to make sure it goes to the Data Quality
Agent.) Then briefly show the architecture diagram from
`docs/ARCHITECTURE.md`.

**SAY:**
> "Notice that I never chose a tool. The question mentions customers, but
> it's really about data quality, and Ansura worked that out by itself.
> It's one chat box. Behind it, the assistant acts like a receptionist and
> sends each question to the right expert automatically."

---

## 3:45–4:20 · The same assistant inside Claude *(Innovation + Technical Excellence)*

**DO:** Switch to Claude (claude.ai or Claude Desktop). The "Insurance AI
Hub" connector should already be connected. Ask:
> *"What's our average loss ratio by policy type?"*

Show the complete answer.

**SAY:**
> "People don't only work in our chat app. Many already use AI assistants
> like Claude. So we connected Ansura to Claude using MCP, a standard
> 'plug' for connecting tools to AI assistants, a bit like USB.
>
> This is the exact same assistant, not a copy. Claude gets a complete,
> checked answer, not just a query. It's one brain with two doors, and
> we didn't build anything twice."

---

## 4:20–5:00 · The dashboard *(Business Value + User Experience)*

**DO:** Open the app's **Dashboard** page and go to the **"Agent Accuracy &
Usage"** tab. Show two things:

1. **Usage, all channels (top half):** point at the three numbers (Total
   Queries, via Streamlit/Direct, via MCP) and the "Queries by channel"
   chart.
2. **Accuracy, chat app only (bottom half):** point at the 👍/👎 helpful
   rate for each tool.

**SAY:**
> "Here's proof that it really is one assistant. The question I just asked
> in Claude and the ones I asked in our chat app are all counted here,
> together. Same assistant, same rules, counted in one place.
>
> Down here are the thumbs up and thumbs down votes from real users. They
> show how often each tool gives a helpful answer, so we can keep
> improving it."

---

## 5:00–5:30 · Every answer can be traced *(Technical Excellence + Business Value)*

**DO:** Open Snowsight's trace view for `ENTERPRISE_AI_AGENT`, or run this
in a worksheet and point at the raw results:
```sql
SELECT * FROM TABLE(SNOWFLAKE.LOCAL.GET_AI_OBSERVABILITY_EVENTS(
  'INSURANCE_AI_HUB', 'PUBLIC', 'ENTERPRISE_AI_AGENT', 'CORTEX AGENT'));
```

**SAY:**
> "Businesses need to trust AI, and that means knowing what it did.
> Snowflake automatically keeps a record of every question Ansura answers,
> whether it came from our chat app or from Claude. We didn't write any
> extra code for this.
>
> For each question, the record shows which tool answered, where the
> question came from, how long it took, which AI model was used, and how
> much processing it cost. Nothing is a black box. Every answer can be
> traced back to exactly what produced it."

⚠️ **Keep this accurate:** our dashboard currently shows *which tool*,
*which channel*, and *how long*. The AI model and processing cost are in
Snowflake's raw record, but **not on our dashboard yet**. Show them from
the raw record or the trace view, and don't say the dashboard has a cost
chart.

---

## 5:30–6:00 · Wrap-up

**SAY:**
> "To sum up: Ansura lets anyone get answers about numbers, documents, and
> data problems without writing code or waiting on another team. Answers
> that used to take [X] now take seconds. Every answer shows its working,
> through the SQL or the source document. And every question, from any
> channel, is recorded down to the tool, the model, and the cost.
>
> Three expert tools, one assistant, available in our app and in Claude.
> That's Ansura. The code and the architecture document are in our GitHub
> repo. Thank you!"

*(Fill in [X] with a real before-and-after time, for example "a day".)*

---

## Filming tips

- **Record clearly:** record the full browser window at 1080p or higher.
  Hide passwords, keys, and any personal data.
- **Give viewers time to read:** after typing a question, pause for 2–3
  seconds before pressing Enter.
- **Warm up first:** ask each type of question once before recording. The
  first question after setup can be slow.
- **Record backup takes:** the AI can word its answer a little differently
  each time, so record each section twice in case one comes out oddly.
- **Sign in to Claude beforehand:** before recording the Claude section,
  sign in as `CLAUDE_MCP_USER` and make sure the "Insurance AI Hub"
  connector shows "Connected". Don't record the sign-in steps.
- **Make sure the dashboard has data:** before recording the dashboard
  section, ask at least one question in the chat app and one in Claude, so
  neither count shows zero.
