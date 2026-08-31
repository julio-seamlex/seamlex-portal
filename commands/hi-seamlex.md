---
description: Start a Seamlex session — works out where you are in the customer lifecycle and loads only the context that step needs.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Start a Seamlex session

One command to open any working session. It answers "where are we?" before you have to, then pulls down
the context for that step and hands you to the right command. Run it at the start of every session.

**The session opens with the config.** The first thing this command does — and the first thing the
customer sees — is a short summary of the settings the session will run on.

**Settings are fixed.** The source of truth is
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md), which is read-only: this
command reads it and never writes to it. Nothing is fetched from Confluence to configure the session.
Because some environments load this command on its own, without the files next to it, the same values are
inlined under **Configuration** at the foot of this file — use those when the file is not reachable. Never
treat an unreadable config file as an unconfigured workspace.

**The lifecycle has four steps, in order:** `setup` → `discovery` → `requirement` → `status`. A company
sits at exactly one of them at a time, and the step is recorded locally in `{{DRAFTS_DIR}}/state.md` — the
one piece of state that is per-workspace rather than fixed.

`$ARGUMENTS` may name a step (`setup`, `discovery`, `requirement`/`requerimiento`, `status`) to override
the recorded step — use it when the customer wants to jump ahead or go back over something. Say plainly
that you are overriding, and do not rewrite `state.md` just because of an override.

## Steps

1. **Load the config, and summarize it first.** Before anything else — before reading `state.md`, before
   any Atlassian call, before any other output — read
   [`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md) and resolve `{{COMPANY}}`,
   `{{PROGRAM}}`, `{{LOCALE}}`, `{{DETAIL}}`, `{{CLOUD_ID}}`, `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`,
   `{{TYPE_EPIC}}`, `{{LABEL_REQUEST}}`, `{{DRAFTS_DIR}}`. **If that file is not reachable, resolve them
   from the Configuration table at the foot of this file and carry on without comment** — the values are
   the same. A missing file says nothing about whether the workspace is set up, so do not stop and do not
   send the customer to `/seamlex-setup` over it.

   Your **very first message to the customer** is a summary of that file and nothing else: who they are
   (`{{COMPANY}}`, `{{PROGRAM}}`, `{{MY_ROLE}}`, `{{LOCALE}}`), the Atlassian workspace (`{{SITE_URL}}`,
   `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`), the issue types and labels in use (`{{TYPE_EPIC}}`,
   `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`, `{{LABEL_REQUEST}}`), who they work with at Seamlex
   (`{{SEAMLEX_CONTACT}}`, `{{ESCALATION}}`, `{{CADENCE}}`), and how the agents will behave
   (`{{CONFIRM_WRITES}}`, `{{DETAIL}}`, `{{DRAFTS_DIR}}`). Keep it short — a handful of lines or a small
   table — and pitch it at `{{DETAIL}}`. Note plainly that the config is fixed and read-only.

   Only after that summary is delivered do you move on to step 2.

2. **Read the recorded step** from `{{DRAFTS_DIR}}/state.md`.
   - **No such file** — the workspace has not been set up. Say so, run `/seamlex-setup`, and stop.
   - **It exists** — take `{{STAGE}}` as the answer, then sanity-check it against what you can see, since
     a stale step is the common failure:
     - `{{STAGE}}` is `discovery` but the Discovery Brief is complete → propose `requirement`.
     - `{{STAGE}}` is `requirement` and the board shows work in flight → propose `status`.
     - `{{STAGE}}` is missing or unreadable → infer the step from the signals in step 3, tell the customer
       the file is incomplete, and offer to record it in step 5.
   Where the recorded step and the evidence disagree, show both and ask with `AskUserQuestion` — never
   silently pick.

3. **Load the context that step needs — and only that step's.** Pulling everything is slow and buries the
   thing they came for.

   | Step | What to load |
   |---|---|
   | `setup` | Nothing further. The config *is* the context. |
   | `discovery` | `{{DRAFTS_DIR}}/discovery/discovery-brief.md`, plus the published brief in `{{CONF_SPACE}}` if there is one. Note which of the nine sections are complete and which are open. |
   | `requirement` | The Discovery Brief (the pains and actors every story anchors to), local drafts in `{{DRAFTS_DIR}}/requests/`, and open `{{TYPE_EPIC}}` issues in `{{JIRA_PROJECT}}` carrying `{{LABEL_REQUEST}}` — so a new requirement can be checked against them for duplicates. |
   | `status` | Everything in `{{JIRA_PROJECT}}` labelled `{{LABEL_REQUEST}}` via `searchJiraIssuesUsingJql`, ordered by updated, with assignee and status. Do not summarize it here — that is the liaison's job. |

   Every read is read-only. Nothing in this command writes to Jira or Confluence. If the Atlassian tools
   are unavailable, do not fail: say clearly which signals could not be checked and carry on from the local
   files alone.

4. **Report, then hand off.** Short and concrete: which step, what it was read from and when it was last
   changed, what context you loaded, and the one command to run next — `/seamlex-setup`,
   `/seamlex-discovery`, `/seamlex-request` or `/seamlex-status`. If the customer already said what they
   came to do, run that command's agent now rather than making them type it.

5. **Record a step change, if one happened.** Only when the step you settled on differs from the recorded
   one — and only with explicit approval, since `{{CONFIRM_WRITES}}` defaults to `always`. Update
   `{{DRAFTS_DIR}}/state.md`: set `{{STAGE}}`, `{{STAGE_UPDATED}}` to today, `{{STAGE_NOTE}}` to one line on
   why, and add a row to the History table. This is a local file — nothing is published. If they decline,
   leave it alone and say the recorded step stays where it was.

> Advancing the step is a statement about the engagement — "discovery is done, we are taking requirements
> now" — so make what it means clear before asking. Moving *back* a step is fine and sometimes right; a
> second discovery round after a reorg is a real thing, not a mistake.

## Configuration

The values from [`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md), inlined so
this command works when that file is not shipped alongside it. **Edit the config file, then copy the
change here** — the file is the source of truth and the two must not drift.

| Placeholder | Value |
|---|---|
| `{{COMPANY}}` | Northwind Logistics |
| `{{INDUSTRY}}` | Third-party logistics (3PL), B2B |
| `{{PROGRAM}}` | Northwind CRM Transformation |
| `{{LOCALE}}` | en_US |
| `{{MY_ROLE}}` | Head of Commercial Operations |
| `{{SITE_URL}}` | https://seamlex.atlassian.net |
| `{{CLOUD_ID}}` | 87285af3-bdbc-4c75-bc0c-2ec7805668fb |
| `{{JIRA_PROJECT}}` | SCRUM |
| `{{CONF_SPACE}}` | MST |
| `{{CONF_PARENT}}` | 327858 |
| `{{TYPE_EPIC}}` | Epic |
| `{{TYPE_STORY}}` | Historia |
| `{{TYPE_QUESTION}}` | Tarea |
| `{{LABEL_REQUEST}}` | customer-request |
| `{{LABELS_EXTRA}}` | phase-1 |
| `{{SEAMLEX_CONTACT}}` | Ana Rivas (Delivery Lead) |
| `{{ESCALATION}}` | delivery@seamlex.com |
| `{{BOARD_URL}}` | https://seamlex.atlassian.net/jira/software/projects/SCRUM/boards/1 |
| `{{CADENCE}}` | 2-week sprints, starting Wednesdays |
| `{{CONFIRM_WRITES}}` | `always` |
| `{{DETAIL}}` | business |
| `{{DRAFTS_DIR}}` | `seamlex` |
