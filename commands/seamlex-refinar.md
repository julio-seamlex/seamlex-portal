---
description: Run a "relevamiento" from the current sprint with the Product Owner — finds the relevamiento tasks in the open sprint, gathers everything the client config points at, interviews the customer in business language only, and leaves a comment and the transcript on the task, one GitHub file "relevamientos/<KEY>.md", the pending items as sub-tasks of the task, and the task linked to the file.
---

# Relevamiento of a sprint task

**Before Step 1, load the `seamlex-portal:business-analysis` skill with the `Skill` tool.** If the
Skill tool is not available, read `../skills/business-analysis/SKILL.md` directly.
The split is simple: the skill decides **how the conversation goes** — business language no matter how
technical the task is written, the need under the request, real actors from discovery, small batches
with `AskUserQuestion`, a verbatim transcript, nothing invented — and this command decides **what it
works on** (the current sprint says which relevamientos are up) and **what it leaves behind** (the five
outputs in Step 4). Its translation table is the rule every question here passes through.

**The house rules for Jira and GitHub are the `seamlex-portal:delivery-how-to` skill**, which
`/hi-seamlex` loaded at the start of the session. If it is not in context, load it now with the `Skill`
tool (or read `../skills/delivery-how-to/SKILL.md`). Where a file goes, what it is called, which
Jira labels a task carries, how the task and the file link, how a task is left and how it is found
later all come from there; Step 4 names what it writes and points at the skill for the how.

Settings come from **`config.yml` in the customer's GitHub repo** that `/hi-seamlex` loaded into the
session. If it is not in context, stop and ask the customer to run `/hi-seamlex`. Resolve from it
`{{JIRA_PROJECT}}`, `{{TYPE_TASK}}` (the project's
sub-task type, used for pending items; default whatever the project calls it — `Subtarea`, `Sub-task`),
`{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`, `{{SEAMLEX_CONTACT}}`, `{{CONFIRM_WRITES}}`, `{{DETAIL}}`,
`{{DRAFTS_DIR}}`, and `{{PROGRAM}}` and `{{COMPANY}}` when it names them. `{{CLOUD_ID}}`, `{{GITHUB_ORG}}` and `{{GITHUB_REPO}}` are the cloud id, org and repo `/hi-seamlex` settled on;
`{{LOCALE}}` and `{{USER_NAME}}` come from `atlassianUserInfo`. If the Atlassian or GitHub tools are not
available, say which one and stop — the sprint and the task live in Jira, every output's home is GitHub,
and there is nothing local to fall back on.

## Business language is the skill's rule, not this command's option

The *speak the customer's language* rule and its technical-term → business-question table live in the
`business-analysis` skill. Nothing in the steps below relaxes it: however a task is written — record
type, flow, LWC, integration — the customer only ever hears questions about their operation, and platform
vocabulary appears in exactly one place, the minuta's closing *Para el equipo de delivery* section.

## Step 1 — Find the relevamiento tasks in the current sprint

1. `searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`:
   ```
   project = {{JIRA_PROJECT}} AND sprint in openSprints()
   AND labels = "relevamiento" AND statusCategory != Done
   ORDER BY Rank ASC
   ```
   Ask for summary, status, issuetype, sprint, parent, assignee, labels, duedate.
2. Show the result as a numbered list — key, summary, status, parent, due date. Then:
   - **One task** → say it is the only relevamiento in the sprint and take it.
   - **Several** → ask with `AskUserQuestion` (header `Relevamiento`) which one to run, defaulting to the
     first not yet in progress. One task per session; the others are offered again at the close.
   - **None** → say there is no relevamiento open in the current sprint, name the sprint and the project
     you searched, and stop. If there is no open sprint at all, say that instead.

`$ARGUMENTS` may name a task directly — a Jira key, or words from its summary — in which case go
straight to it, still checking it carries the `relevamiento` label; if it does not, say so and ask
whether to go on anyway.

## Step 2 — Gather everything the config points at, before asking anything

`README.md` is an **index of the project's knowledge** — the signed scope file, the Discovery Brief,
glossaries, whatever Seamlex has listed in its Map table. Use it as the map: the relevamiento must open
already knowing what the project knows, and the customer must never be asked something a file already
answers. **How** the GitHub repo is read — which files, what to take from each, including what is in
and out of scope, the last minutas — is the skill's *What you know before you ask*; this step keeps
only what is specific to the task.

1. **Read the task**: `getJiraIssue` with description, comments, sub-tasks, issue links, attachments list
   and parent; `getJiraIssueRemoteIssueLinks` for pages already attached. If it hangs off an epic, read
   the epic — that is usually where the contract wording lives.
2. **Walk the repo** the way the skill says — `get_file_contents` on `scope.md`, the Discovery Brief
   file (or the local `{{DRAFTS_DIR}}/discovery/discovery-brief.md` if it exists), `glosario.md` and any
   other reference file `README.md` lists, `features-no-identificados.md`, and the **last two or three
   minutas** — the files under `relevamientos/` — most recent first.
3. **Search for what the index does not list by name**: `search_code` in `{{GITHUB_ORG}}/{{GITHUB_REPO}}`
   with the task's key terms, and `searchJiraIssuesUsingJql` with
   `project = {{JIRA_PROJECT}} AND text ~ "<key terms>"` for related tasks, earlier relevamientos and
   open questions. A file already at `relevamientos/<KEY>.md` for this task (the
   skill's *this task's file* pattern: `get_file_contents` on the fixed path directly) means this is a
   **resumed session** — read it and continue from its open items rather than starting over.
4. **Reflect it back in five to eight lines, in business terms, before the first question**: what the
   task asks for as the customer would say it, what the project already knows (from the repo's files,
   the brief, earlier comments), what is still open, and the two or three things the session will spend
   its time on. Ask them to correct you. A relevamiento that opens with generic questions on a task the
   project half-documents wastes the customer's time and shows nobody read the material.
5. If the task is still in its initial state, move it to **In progress** now — `getTransitionsForJiraIssue`,
   then `transitionJiraIssue`; ask once when `{{CONFIRM_WRITES}}` is `always`.

## Step 3 — Interview, the way the skill says

How the interview runs — the business-language rule, the need under the request, real actors, small
batches, follow the energy, what design needs covered in business terms, `⚠️ TBD` with an owner — is the
skill's *How you interview*. What this command adds is where its inputs and outputs connect to the steps
around it:

- Options for each `AskUserQuestion` batch come from the discovery brief and the material gathered in
  Step 2. An unknown with an owner becomes a pending sub-task in 4d.
- Something that belongs to another task in the sprint or the plan is noted against that key. Something
  that belongs to no task goes to `features-no-identificados.md` in `{{GITHUB_ORG}}/{{GITHUB_REPO}}`,
  created from `../skills/business-analysis/references/unidentified-features.md` if needed, and is
  routed to `{{SEAMLEX_CONTACT}}`.
- The verbatim transcript the skill keeps — question, answer, time of each batch — is what 4c attaches to
  the task.
- The interview ends with the skill's closing summary and the customer's explicit yes. Step 4 does not
  start on silence.

## Step 4 — Leave five things behind

Every session ends with exactly these outputs, in Jira and GitHub. Show the whole set for one approval
when `{{CONFIRM_WRITES}}` is `always`, then write in this order — the file first, because everything else
points at it.

### 4a. The GitHub file `relevamientos/<KEY>.md`

One file per relevamiento, in `{{GITHUB_ORG}}/{{GITHUB_REPO}}` at `relevamientos/<KEY>.md` — the key
upper-cased as Jira writes it, nothing else in the path. The path is how the minuta is found — no
search needed — and the file's own header repeats the task summary for a human reading it on GitHub.

- `create_or_update_file` **as soon as there is something worth saving**, committing again section by
  section as the session runs, without asking again each time — sessions get interrupted, and nothing
  gathered should depend on reaching the end. On a resumed session, update the existing file; never
  create a second one for the same task — find it first with the skill's *this task's file* pattern
  (`get_file_contents` on `relevamientos/<KEY>.md` directly).
- **No labels or topics on the file** — nothing depends on them; the path is how the file is found.
  If this is the first file under a top-level path `README.md`'s Map table does not mention, name that
  for Seamlex to add; otherwise nothing else needs updating.
- **Shape**: YAML front matter (`jira_key`, `jira_url`, `type: minuta`) followed by a header table —
  **Tarea Jira** (key, as a link to the issue), **Sprint**, **Épica** if
  any, **Relevado con** (name, role, per person), **Fecha(s)**, **Estado** (`En progreso` /
  `Finalizado`) — then the body from `../skills/business-analysis/references/minuta-template.md`
  adapted to the task: what this task delivers, the pain it resolves and the success measure; the actors;
  the **Requirements** table at its core — one row per discrete requirement with REQ-ID, capability,
  actor, statement, source, MoSCoW, complexity (1–10) and its own in/out-of-scope call and open
  questions; process context only where a process's shape actually changes; cross-cutting details (data,
  visibility, reporting, integrations); a **Scope & open questions** section with assumptions and a
  session-level table (question, owner, needed by, Jira key once created) for anything that doesn't
  belong to a single requirement row; what was raised here but belongs elsewhere; and the closing
  *Para el equipo de delivery* section. Everything in `{{LOCALE}}` and in business terms.
- The only place platform vocabulary may appear is a short closing section **Para el equipo de
  delivery**, and only where the delivery team genuinely needs the mapping.
- **Never leave a section blank.** What was not answered is `⚠️ TBD — <the question> — <owner>`, either
  in a requirement's own open-questions cell or in the *Scope & open questions* table, and each of those
  becomes a task in 4d.
- **Link the file to the Jira task from the file side**: `jira_key` and `jira_url` in the front matter,
  and the Jira key in the header table written as a link to the issue
  (`https://<site>/browse/<KEY>`). GitHub does not list the task under the file the way Confluence
  sometimes listed a page under an issue — this is a readable, greppable link, not a live backlink; say
  so plainly rather than implying otherwise. This is half of the link in 4e.

### 4b. A comment on the task saying the relevamiento was run

`addCommentToJiraIssue`, one comment, in `{{LOCALE}}`, that says: the relevamiento was run by the Seamlex
Product Owner (Claude) with `{{USER_NAME}}` on `<date>`; a three-line summary of what was settled; the URL
of the minuta file; the keys of the pending tasks created (fill after 4d); and whether the task was left
`Finalizado` or `En progreso`. Anyone opening the task in Jira finds the whole result from that one
comment.

### 4c. The conversation transcript, committed alongside the minuta

The verbatim transcript kept in Step 3 is always written as its own file, so the minuta can be checked
against what was actually said.

- `create_or_update_file` on `relevamientos/<KEY>-transcript.md`, containing the questions and answers
  in order with their times, verbatim — no length limit to work around, unlike a Jira comment, so this
  is the only path, not a fallback.
- Leave a **second, dedicated comment** on the task with `addCommentToJiraIssue`, headed
  `Transcript del relevamiento — <date>`, holding only the transcript file's URL — never the transcript
  text itself, since the file already holds it.
- The transcript is never summarized, edited or cleaned up beyond fixing typos in its own
  questions; the customer's words stay as given.

### 4d. The pending items, as sub-tasks of the relevamiento

Everything the session left open goes into Jira, one item each, **as a sub-task of the relevamiento
task**, so it shows on the board under its parent, has an owner, and is found from the task itself
without following a link:

- **What becomes a task**: every `⚠️ TBD` in the minuta; every decision the customer deferred to someone
  else; every document or example they offered to send; every point that needs a Seamlex decision (a
  parked feature, a conflict with another task). Not the things that were answered — those live on the
  file.
- **Type and parent**: always a **sub-task** with the relevamiento task as `parent` — never a standalone
  issue linked with `relates to`. Read the project's types once with `getJiraProjectIssueTypesMetadata`
  and use `{{TYPE_TASK}}` from the config, or, if the config does not name one, the type the project
  flags as a sub-task (`Subtarea`, `Sub-task`, whatever it is called); if `{{TYPE_TASK}}` names a type
  that is not a sub-task type, say so and use the project's sub-task type anyway. Pass the parent key in
  `createJiraIssue` (`parent`), and confirm with `getJiraIssue` on the relevamiento that the new keys show
  under its sub-tasks. If the project has no sub-task type at all, stop and say so before creating
  anything — the customer decides whether to add one or accept linked tasks instead. Sub-tasks inherit
  the sprint from their parent; do not set the sprint field yourself.
- **Shape**: summary `Pendiente: <the open point in one line>`, in business terms; description in the
  shape of `../skills/business-analysis/references/pendiente-template.md` — where it was raised, the
  owner, needed by, whether it blocks design, what is open, why it matters, what is already believed,
  what was offered, and when it is done — with a link to the minuta section it comes from. Assign to the owner when
  `lookupJiraAccountId` resolves them; otherwise leave it unassigned and name them in the description.
  Label `{{LABEL_REQUEST}}` plus `pendiente` plus `{{LABELS_EXTRA}}` — the skill's Jira label table.
- **Check first** against the relevamiento's existing sub-tasks read in Step 2 — a resumed session must
  not create the same pending item twice. Close, with a comment, any existing pending sub-task the session
  answered.
- Show the full list — new, still open, closed today — get the approval, then `createJiraIssue` each one
  under the relevamiento and write the keys back into whichever table the open item came from — a
  requirement's *Open questions* cell, or the *Scope & open questions* table's *Jira key* column — and
  into the comment from 4b.

### 4e. The Jira task linked to the GitHub file

The task and the minuta must point at each other:

- **Jira → GitHub**: the minuta file's blob URL is in the comment from 4b. If the server exposes a tool
  to add a remote link to an issue, use it too so the file shows under the issue's *Links*; the MCP
  server bundled with the plugin does not, so say when the comment is the only Jira-side link.
- **GitHub → Jira**: the Jira key is in the minuta's front matter (`jira_key`) and its header (4a) as a
  link to the issue. Unlike Confluence, GitHub has no mechanism to list the task under the file — this
  direction is readable and greppable from the file itself, not a live backlink, and the command says
  so rather than implying parity with the old behaviour.

### Then decide the task's state honestly

Walk the *Relevamiento task* block of `../skills/delivery-how-to/references/jira-task-checklist.md`
from a fresh `getJiraIssue` and say which lines are not true; the task is never closed over one.

- **Finalizado** — the customer approved the summary, the minuta covers what design needs (walk the
  *Ready for design* checklist at the foot of the minuta template and say which lines are true), and no
  pending task **blocks** design. Set the minuta's *Estado* to `Finalizado` and transition the task to its
  done state (`getTransitionsForJiraIssue`, then `transitionJiraIssue`; the name is whatever the project
  uses — `Done`, `Finalizada`, `Listo`). Pending tasks that are informational — an example to send, a
  number to confirm — do not stop this; say which they are.
- **En progreso** — anything else: the summary was not approved, a blocking question is open, the session
  ended early. Leave the task in progress, leave the minuta at `En progreso`, and list what has to happen
  before the next session closes it. A relevamiento is never marked done to make the board look better.

When `{{CONFIRM_WRITES}}` is `always`, the transition to done is a separate, explicit yes; never infer it
from approval of the summary.

Close by showing what was left behind — the minuta file URL, the two comment links, the pending sub-task
keys, the task's state — and the other relevamientos still open in the sprint, from the same query as
Step 1, so the customer can pick the next one up with `/seamlex-refinar` or `/seamlex-refinar <KEY>`.

## Failure handling

If any write fails, stop, report exactly what succeeded and what did not, and do not retry blindly — a
minuta without its pending sub-tasks is recoverable; a task marked done with nothing behind it is not. If
the Atlassian or GitHub tools drop out mid-session, stop the relevamiento and show the customer everything
gathered since the last successful save — the transcript included — so they can keep it themselves, then
point them at `/hi-seamlex`.

> Atlassian tools come from the MCP server bundled with this plugin and are namespaced by it —
> `mcp__plugin_seamlex-portal_atlassian__searchJiraIssuesUsingJql`. GitHub tools come from a `github`
> MCP server each person adds themselves (per `SETUP.md`, since a token can't be bundled) — typically
> `mcp__github__create_or_update_file`. Match on the base name after the last `__` for either; the
> prefix varies by how the server was added, and any of them works.
