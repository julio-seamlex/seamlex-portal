---
description: Run a "relevamiento" from the current sprint with the Product Owner — finds the relevamiento tasks in the open sprint, gathers everything the client config points at, interviews the customer in business language only, and leaves a comment and the transcript on the task, one Confluence page "Minuta <task>", the pending items as sub-tasks of the task, and the task linked to the page.
---

# Relevamiento of a sprint task

**Before Step 1, load the `seamlex-portal:business-analysis` skill with the `Skill` tool.** If the
Skill tool is not available, read `${CLAUDE_PLUGIN_ROOT}/skills/business-analysis/SKILL.md` directly.
The split is simple: the skill decides **how the conversation goes** — business language no matter how
technical the task is written, the need under the request, real actors from discovery, small batches
with `AskUserQuestion`, a verbatim transcript, nothing invented — and this command decides **what it
works on** (the current sprint says which relevamientos are up) and **what it leaves behind** (the five
outputs in Step 4). Its translation table is the rule every question here passes through.

Settings come from the **`claude-client-config` Confluence page** that `/hi-seamlex` loaded into the
session. If the page is not in context, stop and ask the customer to run `/hi-seamlex`. Resolve from it
`{{JIRA_PROJECT}}`, `{{CONF_PARENT}}`, `{{TYPE_RELEVAMIENTO}}` (the issue type — or, if the page says
so, the label — that marks a relevamiento; default `Relevamiento`), `{{TYPE_TASK}}` (the project's
sub-task type, used for pending items; default whatever the project calls it — `Subtarea`, `Sub-task`),
`{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`, `{{SEAMLEX_CONTACT}}`, `{{CONFIRM_WRITES}}`, `{{DETAIL}}`,
`{{DRAFTS_DIR}}`, and `{{PROGRAM}}` and `{{COMPANY}}` when it names them. `{{CLOUD_ID}}` and `{{CONF_SPACE}}` are the cloud id and space `/hi-seamlex` settled on;
`{{LOCALE}}` and `{{USER_NAME}}` come from `atlassianUserInfo`. If the Atlassian tools are not available,
say so and stop — the sprint, the task and every output live in Jira and Confluence, and there is nothing
local to fall back on.

## Business language is the skill's rule, not this command's option

The *speak the customer's language* rule and its technical-term → business-question table live in the
`business-analysis` skill. Nothing in the steps below relaxes it: however a task is written — record
type, flow, LWC, integration — the customer only ever hears questions about their operation, and platform
vocabulary appears in exactly one place, the minuta's closing *Para el equipo de delivery* section.

## Step 1 — Find the relevamiento tasks in the current sprint

1. `searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`:
   ```
   project = {{JIRA_PROJECT}} AND sprint in openSprints()
   AND issuetype = "{{TYPE_RELEVAMIENTO}}" AND statusCategory != Done
   ORDER BY Rank ASC
   ```
   If the config marks relevamientos by **label** rather than type, use `labels = "{{TYPE_RELEVAMIENTO}}"`
   instead of `issuetype`. If the site rejects the type name, read the project's types once with
   `getJiraProjectIssueTypesMetadata`, match by name case-insensitively, and say which one you used. Ask
   for summary, status, issuetype, sprint, parent, assignee, labels, duedate.
2. Show the result as a numbered list — key, summary, status, parent, due date. Then:
   - **One task** → say it is the only relevamiento in the sprint and take it.
   - **Several** → ask with `AskUserQuestion` (header `Relevamiento`) which one to run, defaulting to the
     first not yet in progress. One task per session; the others are offered again at the close.
   - **None** → say there is no relevamiento open in the current sprint, name the sprint and the project
     you searched, and stop. If there is no open sprint at all, say that instead.

`$ARGUMENTS` may name a task directly — a Jira key, or words from its summary — in which case go
straight to it, still checking it is a `{{TYPE_RELEVAMIENTO}}`; if it is not, say so and ask whether to
go on anyway.

## Step 2 — Gather everything the config points at, before asking anything

The `claude-client-config` page is an **index of the project's knowledge** — the signed scope page, the
Discovery Brief, process documents, previous minutas, glossaries, whatever Seamlex has listed there. Use
it as the map: the relevamiento must open already knowing what the project knows, and the customer must
never be asked something a page already answers.

1. **Read the task**: `getJiraIssue` with description, comments, sub-tasks, issue links, attachments list
   and parent; `getJiraIssueRemoteIssueLinks` for pages already attached. If it hangs off an epic, read
   the epic — that is usually where the contract wording lives.
2. **Walk the config page** and read every entry that can bear on this task — with `getConfluencePage`
   for the pages it links (`{{CONF_SCOPE_PAGE}}`, the Discovery Brief page, any process, glossary or
   reference page it lists) and with the local `{{DRAFTS_DIR}}/discovery/discovery-brief.md` if it
   exists. From the brief, the areas and roles (§3), processes (§4), actors (§6) and pains (§7) are the
   ground every question stands on.
3. **Search for what the config does not list by name**: `searchConfluenceUsingCql` in `{{CONF_SPACE}}`
   with the task's key and its key terms (`title ~ "<KEY>"`, `text ~ "<term>"`), and
   `searchJiraIssuesUsingJql` with `project = {{JIRA_PROJECT}} AND text ~ "<key terms>"` for related
   tasks, earlier relevamientos and open questions. A page titled `Minuta <this task's summary>` already
   in the space means this is a **resumed session** — read it and continue from its open items rather
   than starting over.
4. **Reflect it back in five to eight lines, in business terms, before the first question**: what the
   task asks for as the customer would say it, what the project already knows (from the config's pages,
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
  that belongs to no task goes to the `Features no identificados — {{PROGRAM}}` page in `{{CONF_SPACE}}`,
  created from `${CLAUDE_PLUGIN_ROOT}/templates/unidentified-features.md` if needed, and is routed to
  `{{SEAMLEX_CONTACT}}`.
- The verbatim transcript the skill keeps — question, answer, time of each batch — is what 4c attaches to
  the task.
- The interview ends with the skill's closing summary and the customer's explicit yes. Step 4 does not
  start on silence.

## Step 4 — Leave five things behind

Every session ends with exactly these outputs, in Jira and Confluence. Show the whole set for one approval
when `{{CONFIRM_WRITES}}` is `always`, then write in this order — the page first, because everything else
points at it.

### 4a. The Confluence page `Minuta <task summary>`

One page per relevamiento, in `{{CONF_SPACE}}` under `{{CONF_PARENT}}`, titled exactly
`Minuta <task summary as it reads in Jira>`. Keep the Jira wording even if a better title came up in
conversation — the title is how the minuta is found from the board.

- `createConfluencePage` **as soon as there is something worth saving**, then `updateConfluencePage`
  section by section as the session runs, without asking again each time — sessions get interrupted, and
  nothing gathered should depend on reaching the end. On a resumed session, update the existing page;
  never create a second one for the same task.
- **Shape**: a header table — **Tarea Jira** (key, as a link to the issue), **Sprint**, **Épica** if
  any, **Relevado con** (name, role, per person), **Fecha(s)**, **Estado** (`En progreso` /
  `Finalizado`) — followed by the body from `${CLAUDE_PLUGIN_ROOT}/templates/epic-template.md` adapted to
  the task: what this task delivers, the pain it resolves, the actors, the processes today and after,
  the functional detail with its rules and exceptions, the information involved, who sees what, what will
  be measured, what is explicitly out, what was raised here but belongs elsewhere, and a **Pendientes**
  table (question, owner, Jira key once created). Everything in `{{LOCALE}}` and in business terms.
- The only place platform vocabulary may appear is a short closing section **Para el equipo de
  delivery**, and only where the delivery team genuinely needs the mapping.
- **Never leave a section blank.** What was not answered is `⚠️ TBD — <the question> — <owner>` in the
  *Pendientes* table, and each of those becomes a task in 4d.
- **Link the page to the Jira task from the page side**: the Jira key in the header must be written as a
  link to the issue (`https://<site>/browse/<KEY>`), so Confluence renders it as a Jira link and Jira lists
  the page under the issue's Confluence content. This is half of the link in 4e.

### 4b. A comment on the task saying the relevamiento was run

`addCommentToJiraIssue`, one comment, in `{{LOCALE}}`, that says: the relevamiento was run by the Seamlex
Product Owner (Claude) with `{{USER_NAME}}` on `<date>`; a three-line summary of what was settled; the URL
of the minuta; the keys of the pending tasks created (fill after 4d); and whether the task was left
`Finalizado` or `En progreso`. Anyone opening the task in Jira finds the whole result from that one
comment.

### 4c. The conversation transcript, attached to the task

The verbatim transcript kept in Step 3 goes on the task, so the minuta can be checked against what was
actually said.

- The Atlassian MCP server has no file-upload tool, so the transcript is attached as a **second,
  dedicated comment** on the task with `addCommentToJiraIssue`, headed `Transcript del relevamiento —
  <date>` and containing the questions and answers in order with their times.
- If the transcript is too long for one comment (the site rejects it), create a child page of the minuta
  titled `Transcript — Minuta <task summary>` with `createConfluencePage`, put the transcript there, and
  leave a comment on the task with that page's URL instead. Say which of the two you did.
- The transcript is never summarized, edited or cleaned up beyond fixing typos in its own
  questions; the customer's words stay as given.

### 4d. The pending items, as sub-tasks of the relevamiento

Everything the session left open goes into Jira, one item each, **as a sub-task of the relevamiento
task**, so it shows on the board under its parent, has an owner, and is found from the task itself
without following a link:

- **What becomes a task**: every `⚠️ TBD` in the minuta; every decision the customer deferred to someone
  else; every document or example they offered to send; every point that needs a Seamlex decision (a
  parked feature, a conflict with another task). Not the things that were answered — those live on the
  page.
- **Type and parent**: always a **sub-task** with the relevamiento task as `parent` — never a standalone
  issue linked with `relates to`. Read the project's types once with `getJiraProjectIssueTypesMetadata`
  and use `{{TYPE_TASK}}` from the config, or, if the config does not name one, the type the project
  flags as a sub-task (`Subtarea`, `Sub-task`, whatever it is called); if `{{TYPE_TASK}}` names a type
  that is not a sub-task type, say so and use the project's sub-task type anyway. Pass the parent key in
  `createJiraIssue` (`parent`), and confirm with `getJiraIssue` on the relevamiento that the new keys show
  under its sub-tasks. If the project has no sub-task type at all, stop and say so before creating
  anything — the customer decides whether to add one or accept linked tasks instead. Sub-tasks inherit
  the sprint from their parent; do not set the sprint field yourself.
- **Shape**: summary `Pendiente: <the question in one line>`, in business terms; description in the shape
  of `${CLAUDE_PLUGIN_ROOT}/templates/question-template.md` — why it matters, what it unblocks, who can
  answer, needed by, and a link to the minuta section it comes from. Assign to the owner when
  `lookupJiraAccountId` resolves them; otherwise leave it unassigned and name them in the description.
  Label `{{LABEL_REQUEST}}` plus `{{LABELS_EXTRA}}`.
- **Check first** against the relevamiento's existing sub-tasks read in Step 2 — a resumed session must
  not create the same pending item twice. Close, with a comment, any existing pending sub-task the session
  answered.
- Show the full list — new, still open, closed today — get the approval, then `createJiraIssue` each one
  under the relevamiento and write the keys back into the minuta's *Pendientes* table and into the
  comment from 4b.

### 4e. The Jira task linked to the Confluence page

The task and the minuta must point at each other:

- **Jira → Confluence**: the minuta URL is in the comment from 4b. If the server exposes a tool to add a
  remote link to an issue, use it too so the page shows under the issue's *Links*; the MCP server bundled
  with the plugin does not, so say when the comment is the only Jira-side link.
- **Confluence → Jira**: the Jira key in the minuta header is a link to the issue (4a), which makes Jira
  list the page under the issue. Confirm with `getJiraIssueRemoteIssueLinks` after the page is saved;
  if the page does not show up, say so — the comment link still holds, and the customer can attach the
  page by hand from Jira's *Link* menu.

### Then decide the task's state honestly

- **Finalizado** — the customer approved the summary, the minuta covers what design needs (walk the
  *Ready for design* checklist at the foot of the epic template and say which lines are true), and no
  pending task **blocks** design. Set the minuta's *Estado* to `Finalizado` and transition the task to its
  done state (`getTransitionsForJiraIssue`, then `transitionJiraIssue`; the name is whatever the project
  uses — `Done`, `Finalizada`, `Listo`). Pending tasks that are informational — an example to send, a
  number to confirm — do not stop this; say which they are.
- **En progreso** — anything else: the summary was not approved, a blocking question is open, the session
  ended early. Leave the task in progress, leave the minuta at `En progreso`, and list what has to happen
  before the next session closes it. A relevamiento is never marked done to make the board look better.

When `{{CONFIRM_WRITES}}` is `always`, the transition to done is a separate, explicit yes; never infer it
from approval of the summary.

Close by showing what was left behind — the minuta URL, the two comment links, the pending sub-task keys,
the task's state — and the other relevamientos still open in the sprint, from the same query as Step 1, so
the customer can pick the next one up with `/seamlex-refinar` or `/seamlex-refinar <KEY>`.

## Failure handling

If any write fails, stop, report exactly what succeeded and what did not, and do not retry blindly — a
minuta without its pending sub-tasks is recoverable; a task marked done with nothing behind it is not. If the
Atlassian tools drop out mid-session, stop the relevamiento and show the customer everything gathered
since the last successful save — the transcript included — so they can keep it themselves, then point
them at `/hi-seamlex`.

> Atlassian tools come from the MCP server bundled with this plugin and are namespaced by it —
> `mcp__plugin_seamlex-portal_atlassian__searchJiraIssuesUsingJql`. Match on the base name after the last
> `__`; the prefix changes if the server is configured elsewhere, and either one works.
