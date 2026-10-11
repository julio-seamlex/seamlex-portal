<!--
  Project status template. /seamlex-project-status fills this in and renders it as the live Artifact.
  Placeholders are {{LIKE_THIS}}; <!-- repeat --> marks a row/item that repeats.
  A section nothing has filled yet stays visible and reads "⏳ pending".
  Every row cites its source — a Jira key or a GitHub path. Nothing is invented.
-->

# {{PROGRAM}} — Project status

**Customer:** {{COMPANY}} · **Jira project:** {{JIRA_PROJECT}} · **Docs repo:** {{GITHUB_ORG}}/{{GITHUB_REPO}}
**As of:** {{DATE}} · **Overall:** {{RAG}} — one sentence saying why (🟢 on track · 🟡 attention · 🔴 at risk)

## 1. Sprint status

**{{SPRINT_NAME}}** · {{SPRINT_START}} → {{SPRINT_END}} ({{DAYS_LEFT}} days left)
**Goal:** {{SPRINT_GOAL}} (or `⚠️ no sprint goal set`)

| Done | In progress | To do | Total | % done |
|---|---|---|---|---|
| {{N_DONE}} | {{N_PROGRESS}} | {{N_TODO}} | {{N_TOTAL}} | {{PCT}} |

### Remaining tasks
| Key | Summary | Type | Status | Assignee | Due |
|---|---|---|---|---|---|
| {{KEY}} | {{SUMMARY}} | {{TYPE}} | {{STATUS}} | {{ASSIGNEE or "unassigned"}} | {{DUE}} | <!-- repeat, status category != Done, in Rank order -->

## 2. GitHub documentation status

| Document | Path | State | Note |
|---|---|---|---|
| Signed scope | {{PATH}} | ✅ present · ⚠️ stale/incomplete · ❌ missing | {{last commit date, TBD count}} |
| Discovery Brief | {{PATH}} | … | … |
| Glosario | {{PATH}} | … | … |
| Features not identified | {{PATH}} | … | … |
| Relevamiento per sprint task | `relevamientos/<KEY>.md` | {{x of y tasks covered}} | {{keys without a file}} |

**Last activity on the docs repo:** {{DATE}} — {{commit message}}

## 3. Key missing tasks
Things the project needs and nobody has yet. Each row says what is missing and where that was seen.

| # | Missing | Why it matters | Seen in |
|---|---|---|---|
| 1 | {{what}} | {{impact}} | {{KEY or path}} | <!-- repeat, most important first -->

## 4. Warnings
| Level | Warning | Source |
|---|---|---|
| 🔴 / 🟡 | {{e.g. 6 of 9 tasks still To do with 2 days left}} | {{KEY(s) or path}} | <!-- repeat -->

## 5. Dependencies
| Item | Depends on | Kind | Status |
|---|---|---|---|
| {{KEY}} | {{KEY, customer answer, external team, scope decision}} | blocked by · waiting on customer · external | {{open / resolved}} | <!-- repeat -->

## Sources
Jira query run, files read, and anything that could not be read ({{MCP tool unavailable, file missing}}).
