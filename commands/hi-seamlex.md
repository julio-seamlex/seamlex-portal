---
description: Start a Seamlex session — works out where you are in the customer lifecycle, sets the workspace up the first time, and loads only the context that step needs.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Start a Seamlex session

One command to open any working session, including the first one. It answers "where are we?" before you
have to, then pulls down the context for that step and hands you to the right command. Run it at the start
of every session.

**The session opens with the config.** The first thing this command does — and the first thing the
customer sees — is a short summary of the settings the session will run on.

**Settings are fixed and live in this file.** The **Configuration** section at the foot of this file is the
single source of truth for the whole plugin: every Seamlex command and agent resolves its
`{{PLACEHOLDER}}` tokens from that table. There is nothing to fill in, nothing to generate, and no config
is ever written into the customer's workspace. To change how the plugin behaves for a customer, edit that
table and ship a new version of the plugin — never work around a wrong value locally. Nothing is fetched
from Confluence to configure the session.

**The lifecycle has four steps, in order:** `setup` → `discovery` → `refinement` → `status`. A company
sits at exactly one of them at a time. **Nothing records which one** — no state file is kept, and the
plugin writes no settings into the workspace. The step is worked out fresh each session from what is
actually there: the workspace folders, the Discovery Brief, and the board. Evidence cannot go stale the
way a recorded step can.

`$ARGUMENTS` may name a step (`setup`, `discovery`, `refinement`/`alcance` — `requirement` also works, `status`) to override
what you infer — use it when the customer wants to jump ahead or go back over something, or to re-run the
setup checks after fixing an Atlassian connection. Say plainly that you are overriding.

## Steps

1. **Load the config, and summarize it first.** Before anything else — before any Atlassian call, before
   any other output — resolve `{{COMPANY}}`, `{{PROGRAM}}`, `{{LOCALE}}`,
   `{{DETAIL}}`, `{{CLOUD_ID}}`, `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`, `{{TYPE_EPIC}}`,
   `{{LABEL_REQUEST}}`, `{{DRAFTS_DIR}}` from the **Configuration** table at the foot of this file.

   Your **very first message to the customer** is a summary of that table and nothing else: who they are
   (`{{COMPANY}}`, `{{PROGRAM}}`, `{{MY_ROLE}}`, `{{LOCALE}}`), the Atlassian workspace (`{{SITE_URL}}`,
   `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`), the issue types and labels in use (`{{TYPE_EPIC}}`,
   `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`, `{{LABEL_REQUEST}}`), who they work with at Seamlex
   (`{{SEAMLEX_CONTACT}}`, `{{ESCALATION}}`, `{{CADENCE}}`), and how the agents will behave
   (`{{CONFIRM_WRITES}}`, `{{DETAIL}}`, `{{DRAFTS_DIR}}`). Keep it short — a handful of lines or a small
   table — and pitch it at `{{DETAIL}}`. Note plainly that the config is fixed and read-only.

   Only after that summary is delivered do you move on to step 2.

2. **Work out which step they are on**, from the cheapest signal upward. Stop at the first that answers.
   - `{{DRAFTS_DIR}}/` does not exist → **`setup`**. The workspace has never been set up: go to step 3 and
     run it now, in this session. Do not send the customer off to another command.
   - No `{{DRAFTS_DIR}}/discovery/discovery-brief.md`, or one whose ten sections are not all covered →
     **`discovery`**.
   - The brief is complete and the board shows `{{JIRA_PROJECT}}` issues labelled `{{LABEL_REQUEST}}` in
     flight — assigned, in progress, or recently updated → **`status`**.
   - The brief is complete and there is no such work moving yet → **`refinement`**.
   Where the signals genuinely conflict — a complete brief and an empty board and a workspace full of
   request drafts, say — show what you saw and ask with `AskUserQuestion` rather than silently picking.
   Say which signals you read, so the customer can correct you in one line if you land wrong.

3. **If the step is `setup`, set the workspace up.** Confirm the Atlassian connection works, check the
   fixed config against the live site, and create the local folders the other commands write into. Run
   this once per workspace; `/hi-seamlex setup` re-runs it on demand.

   1. **Check the Atlassian connection.** Call `getAccessibleAtlassianResources`. Its full name is
      namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
      — so match on the base name after the last `__`; the prefix differs if the customer already has an
      Atlassian server configured elsewhere, and either one works.
      - If the tool is not available at all, the MCP server is not connected. Tell the customer to restart
        Claude and approve the `atlassian` server, and see the "Connecting Atlassian" section of
        [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md). Still do sub-steps 3 and 4 — the folders are useful
        offline — and say plainly that nothing was verified against Atlassian this session.
      - If it prompts for browser sign-in, walk them through it. Seamlex never sees their credentials.
      - Confirm the site that comes back matches `{{SITE_URL}}` and `{{CLOUD_ID}}`. If it does not, stop
        and report the difference: the config points at a site this user cannot reach, and every later
        write would land in the wrong place or fail.

   2. **Verify the config against the live site.** These are all reads; do them without asking.
      - `atlassianUserInfo` — confirm who the customer is signed in as.
      - `getVisibleJiraProjects` — confirm `{{JIRA_PROJECT}}` is visible to them.
      - `getJiraProjectIssueTypesMetadata` on that project — confirm the issue type names
        (`{{TYPE_EPIC}}`, `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`) actually exist. Where one does not, name
        it, name the nearest available type the agents will fall back to, and say the row should be
        corrected in this file's Configuration table.
      - `getConfluenceSpaces` — confirm `{{CONF_SPACE}}` exists, and `getConfluencePage` on
        `{{CONF_PARENT}}` that the discovery parent page is reachable, and on `{{CONF_SCOPE_PAGE}}` that
        the signed scope page is reachable.
      - `searchJiraIssuesUsingJql` with `project = {{JIRA_PROJECT}} ORDER BY created DESC` — a read-only
        probe; report how many issues are visible.
      Report each check as pass or mismatch. A mismatch is a finding about the plugin's config, not
      something to ask the customer to fix locally.

   3. **Create the local working folders.**
      `mkdir -p {{DRAFTS_DIR}}/discovery`
      This holds the discovery notes and nothing else. Scope refinement keeps its drafts on Confluence,
      not here, and no config file is ever written into the workspace.

   Once the folders exist, setup is done and the step is `discovery` — carry on into step 4 with that,
   rather than ending the session on a bare setup report.

   > If the Atlassian tools become unavailable partway through setup, do not fail. Nothing here depends on
   > a write succeeding; say which checks did not run and offer to rerun `/hi-seamlex setup` once the
   > connection is back.

4. **Load the context that step needs — and only that step's.** Pulling everything is slow and buries the
   thing they came for.

   | Step | What to load |
   |---|---|
   | `setup` | Nothing further. The config *is* the context. |
   | `discovery` | `{{DRAFTS_DIR}}/discovery/discovery-brief.md`, plus the published brief in `{{CONF_SPACE}}` if there is one. Note which of the ten sections are complete and which are open. |
   | `refinement` | The signed scope page `{{CONF_SCOPE_PAGE}}` (the epics to be delivered), the Discovery Brief (the areas, processes, actors and pains every statement anchors to), and the `Epic — <contract epic title>` pages in `{{CONF_SPACE}}` with their Status rows — the refinement drafts live there, not in the workspace, and they are what says which epics are ready for design and which are not. |
   | `status` | Everything in `{{JIRA_PROJECT}}` labelled `{{LABEL_REQUEST}}` via `searchJiraIssuesUsingJql`, ordered by updated, with assignee and status. Do not summarize it here — that is the project manager's job. |

   Every read is read-only. Apart from creating the two empty folders in step 3, this command writes
   nothing — not to the workspace, not to Jira, not to Confluence. If the Atlassian tools are unavailable,
   do not fail: say clearly which signals could not be checked and carry on from the local files alone.

5. **Report, then hand off.** Short and concrete: which step, the signals you read to land on it, what
   context you loaded, and the one command to run next — `/seamlex-discovery`, `/refine-project-scope` or
   `/seamlex-status`. If the customer already said what they came to do, run that command's agent now
   rather than making them type it.

> The step is a statement about the engagement — "discovery is done, we are refining the scope now" — so
> when you name it, name it as a reading of the evidence rather than a fact. Going *back* a step is fine
> and sometimes right; a second discovery round after a reorg is a real thing, not a mistake, and
> `/hi-seamlex discovery` is all it takes.

## Configuration

**This section parameterizes the whole plugin.** Every Seamlex command and agent reads it first and
resolves the `{{PLACEHOLDER}}` tokens in its instructions from the **Value** column below.

**It is fixed and ships with the plugin.** Nothing generates it, nothing overwrites it, and no copy is
made in the customer's workspace — the agents read it in place at
`${CLAUDE_PLUGIN_ROOT}/commands/hi-seamlex.md`. To change how the plugin behaves for a customer, edit this
table in the plugin and ship it; setup (step 3 above) only checks these values against the live Atlassian
site and reports any mismatch.

No state is kept per-workspace. Discovery notes and request drafts live under `seamlex/`, and never come
back into this file.

Safe to commit — it holds no secrets. Authentication to Jira and Confluence happens through the Atlassian
MCP server's own browser login, never through this file.
**Never put an API token or password in here.**

### 1. Who you are

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Company name | `{{COMPANY}}` | Northwind Logistics | Demo customer. |
| Industry | `{{INDUSTRY}}` | Third-party logistics (3PL), B2B | |
| Program name | `{{PROGRAM}}` | Northwind CRM Transformation | Sales + Service Cloud, phase 1. |
| Locale | `{{LOCALE}}` | en_US | |
| Your role | `{{MY_ROLE}}` | Head of Commercial Operations | |

### 2. Atlassian workspace

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Atlassian site | `{{SITE_URL}}` | https://seamlex.atlassian.net | |
| Cloud ID | `{{CLOUD_ID}}` | 87285af3-bdbc-4c75-bc0c-2ec7805668fb | Resolved once via `getAccessibleAtlassianResources`; cached here so agents skip the lookup. |
| Jira project key | `{{JIRA_PROJECT}}` | SCRUM | My Software Team. |
| Confluence space key | `{{CONF_SPACE}}` | MST | |
| Discovery parent page | `{{CONF_PARENT}}` | 327858 | My Software Team home. |
| Signed scope page | `{{CONF_SCOPE_PAGE}}` | 5373953 | Page in `{{CONF_SPACE}}` holding the detail of the scope the customer signed. Correct it by editing this table and shipping a new plugin version. |

### 3. Issue types and fields

> Names must match the Jira project exactly. Setup reads the project metadata and reports any row that
> does not match, so a wrong value is caught before the first issue is raised. If the project does not
> have a given type, write `n/a` and the agents will fall back to the nearest available type and say so.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Epic type | `{{TYPE_EPIC}}` | Epic | |
| Story type | `{{TYPE_STORY}}` | Historia | This project's issue types are in Spanish. |
| Question type | `{{TYPE_QUESTION}}` | Tarea | No Question type in this project — questions are raised as Tarea. |
| Request label | `{{LABEL_REQUEST}}` | customer-request | |
| Extra labels | `{{LABELS_EXTRA}}` | phase-1 | |

### 4. Working with Seamlex

> **Seamlex owns this section.** These are delivery-side facts, filled in when the plugin is shipped to a
> customer. Everything works without them: questions and requests are simply raised unassigned, and status
> answers skip the board link and the cadence.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Seamlex contact | `{{SEAMLEX_CONTACT}}` | Ana Rivas (Delivery Lead) | Default assignee for questions and new requests. |
| Escalation contact | `{{ESCALATION}}` | delivery@seamlex.com | For anything urgent or blocked. |
| Status board | `{{BOARD_URL}}` | https://seamlex.atlassian.net/jira/software/projects/SCRUM/boards/1 | Linked when reporting status. |
| Sprint cadence | `{{CADENCE}}` | 2-week sprints, starting Wednesdays | Sets expectations in status answers. |

### 5. How the agents should behave

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Write confirmation | `{{CONFIRM_WRITES}}` | `always` | `always` (recommended) or `summary`. Whether every Jira/Confluence write is shown for approval first. |
| Detail level | `{{DETAIL}}` | business | `business` (plain language) or `technical` (Salesforce terms welcome). |
| Local drafts | `{{DRAFTS_DIR}}` | `seamlex` | Folder in the customer's workspace holding the discovery notes. Scope-refinement drafts live in Confluence, not here. |

### What lives in your workspace

```
seamlex/
└── discovery/
    └── discovery-brief.md      # your discovery notes — resumable across sessions
```

That is all of it. The scope refinement leaves nothing here: each contract epic's page, and the
features no identificados page, are drafted straight into `{{CONF_SPACE}}` as pages marked `Draft` until
the customer approves them. The configuration is not in the workspace either — it stays in the plugin.
Nothing is marked reviewed, and nothing reaches Jira, until you approve it.
