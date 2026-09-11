---
description: Refine the next task in the Jira plan — picks the first open task by plan order as of today, reads what Jira already says about it, runs a business-language session, and leaves one Confluence page, the pending items as sub-tasks, and the task marked in progress or done.
---

# Refine the next planned task

Hand this to the **seamlex-product-owner** agent for *how it interviews* — the need under the request,
real actors from discovery, small batches with `AskUserQuestion`, nothing invented. **What it works on is
decided here, not by the agent's own opening step**: the Jira plan says which task is up today, so skip
the agent's contract-epic list and follow the steps below.

Settings come from the plugin's fixed config, the **Configuration** section of
[`commands/hi-seamlex.md`](${CLAUDE_PLUGIN_ROOT}/commands/hi-seamlex.md) — nothing to generate and nothing
to check first. Resolve `{{CLOUD_ID}}`, `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`, `{{CONF_PARENT}}`,
`{{TYPE_EPIC}}`, `{{TYPE_QUESTION}}`, `{{SEAMLEX_CONTACT}}`, `{{CONFIRM_WRITES}}`, `{{DETAIL}}`,
`{{LOCALE}}`, `{{DRAFTS_DIR}}`. If the Atlassian tools are not available, say so and stop — the plan, the
task and every output live in Jira and Confluence, and there is nothing local to fall back on. Point at
`/hi-seamlex setup`.

## Step 1 — Pick the task from the plan, as of today

The Jira plan is the order of work. Take **today's date** from the session and find the first open task in
plan order:

1. `searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`:
   ```
   project = {{JIRA_PROJECT}} AND statusCategory != Done AND issuetype not in subTaskIssueTypes()
   ORDER BY "Start date" ASC, duedate ASC, Rank ASC
   ```
   If the site rejects `"Start date"` (the field name differs between project types), drop it and order by
   `duedate ASC, Rank ASC`. Ask for the fields you will need next: summary, status, issuetype, start date,
   due date, parent, assignee, labels.
2. Pick, in this order:
   - the first task whose **start date is on or before today** — something already due to be in flight
     comes before anything future;
   - if none has started, the first task whose start date is still ahead — the next thing the plan brings;
   - if nothing carries a date, the first by `Rank`.
   Sub-tasks are never the pick; the task they hang off is.
3. Show the pick and the two tasks behind it, with key, summary, type, dates and status, and say which
   rule chose it ("started 2026-09-08, still To Do" beats "not started, ranked first"). Ask the customer to
   confirm with `AskUserQuestion` before reading further — the plan may be stale, and they will know.

`$ARGUMENTS` may name a task directly — a Jira key, or words from its summary — in which case go straight
to it and say you are skipping the plan order. If the plan has no open task at all, say so and stop; there
is nothing to refine.

## Step 2 — Read the task before asking anything

The session must be shaped by what Jira already knows. Before the first question:

1. `getJiraIssue` on the pick, with description, comments, sub-tasks, issue links, attachments list, and
   parent. Then `getJiraIssueRemoteIssueLinks` for pages already attached. If it has a parent epic, read
   that too — its description is often where the contract wording lives.
2. Look for an existing refinement page: `searchConfluenceUsingCql` with
   `space = "{{CONF_SPACE}}" AND title ~ "<KEY>"`. If one exists, this is a **resumed session** — read it,
   and continue from its open items rather than starting over.
3. Read the discovery brief at `{{DRAFTS_DIR}}/discovery/discovery-brief.md` if it exists — the areas and
   roles (§3), processes (§4), actors (§6) and pains (§7) are the ground every question stands on. If
   there is no brief, say so and go on; the session will be weaker for it.
4. **Reflect it back in five to eight lines before anything else**: what the task asks for in the
   customer's words, what Jira already settles (decisions in comments, attached documents, closed
   sub-tasks), what is still open, and which two or three things you intend to spend the session on. Ask
   them to correct you. A session that opens with generic questions on a task Jira already half-describes
   wastes the customer's time and tells them you did not read it.
5. If the task is still in its initial state, move it to **In progress** now: `getTransitionsForJiraIssue`
   to find the transition, `transitionJiraIssue` to apply it. When `{{CONFIRM_WRITES}}` is `always`, ask
   once. The task is in progress because the session is running — the board should say so.

## Step 3 — Run the session in the customer's language

This is a consulting conversation with a business person, not a configuration questionnaire. The task
description will often be written in Salesforce terms — the plan was built by the delivery team. **Never
put those terms to the customer as questions.** Translate every one into what it means for their
operation, ask that, and let the delivery team map the answer back to the platform. `{{DETAIL}}` =
`business` makes this mandatory; even at `technical`, ask the business question first.

| If the task says… | Do not ask | Ask instead |
|---|---|---|
| record type | "Which record types do you need?" | "Are there kinds of *<order / case / account>* that are handled differently — different information, different steps, or different people? Walk me through one of each." |
| field / custom field | "Which fields?" | "What do you need to know about a *<thing>* to do your job? Who looks at it, and what do they decide with it?" |
| object / custom object | "Do you need a custom object?" | "What is the thing you need to keep track of here — one per customer, one per shipment, one per contract? What happens to it over time?" |
| validation rule | "What validations?" | "What should never be allowed to be saved or move forward? What mistakes cost you time today because they slip through?" |
| flow / automation / trigger | "Which flows?" | "After *<event>* happens, what does someone do by hand that is always the same? Who does it, how long does it take, what goes wrong when they forget?" |
| approval process | "What approval steps?" | "Who has to say yes before this goes ahead, above what limit, and what happens when they are away or say no?" |
| page layout / screen / component | "What should the layout show?" | "When *<actor>* is doing this, what do they need in front of them, and what do they go looking for somewhere else today?" |
| profile / permission set / sharing | "Who gets which permission set?" | "Who must be able to see this, who must not, and who can change it versus only read it?" |
| picklist / values | "What picklist values?" | "What are the valid options here, who decides them, and how often do they change?" |
| queue / assignment rule | "What assignment rules?" | "How do you decide today who takes this? Is it by region, by workload, by who is on shift — and who overrides it?" |
| report / dashboard | "What reports do you need?" | "What questions do you need answered every week or month about this, who asks them, and what do they do with the answer?" |
| integration / API / sync | "Which systems to integrate?" | "Where does this information live today, who copies it across, and how do you know when it is wrong?" |
| email template / notification / alert | "What notifications?" | "Who needs to be told when this happens, how soon, and what do they need to know to act?" |
| status / stage / stage picklist | "What statuses?" | "What are the steps this goes through from start to finished, who moves it to the next one, and where does it get stuck?" |
| SLA / entitlement | "What SLA levels?" | "What have you promised customers about how fast this gets handled, and what happens when you miss it?" |

Beyond the translation:

- **Interview in small batches** with `AskUserQuestion` — two to four questions, options drawn from the
  discovery brief and from what the task already says, always with an "I don't know / someone else owns
  that" way out. An unknown with an owner is a finding, and becomes a sub-task in Step 4.
- **Ask what it is for before how it works.** Every request gets "what would you do with that", "what
  breaks today without it", "who benefits". Write down the need; the delivery team chooses the mechanism.
- **Every actor is a real role** from discovery §3. "A user" makes the statement untestable.
- **Follow the energy.** When the customer gets specific and animated, stay there and ask three more.
- **Cover what design needs** — actors, the process today and after, trigger and outcome, rules and
  exceptions, steps and who moves them, who is told, volume and frequency, the information involved and
  where it comes from, who sees it, what will be measured, and what is explicitly *not* part of this
  task. Anything that would send an architect back to the customer later is asked now.
- **Park what is not this task.** Something that belongs to another task in the plan is noted against
  that key and left there. Something that belongs to no task at all is appended to the
  `Features no identificados — {{PROGRAM}}` page in `{{CONF_SPACE}}` (created from
  `${CLAUDE_PLUGIN_ROOT}/templates/unidentified-features.md` if needed), told plainly to sit outside the
  signed scope, and routed to `{{SEAMLEX_CONTACT}}`. Never folded in, never dropped.

## Step 4 — Leave three things behind

Every session ends with exactly these outputs, in Jira and Confluence, nothing on the customer's disk.

### 4a. One Confluence page — the functional result of the conversation

One page per task, in `{{CONF_SPACE}}` under `{{CONF_PARENT}}`, titled
`Relevamiento — <KEY> — <task summary as it reads in Jira>`. The key in the title is the trace back to
the plan; keep the Jira wording even if a better title came up in conversation.

- **Create it as soon as there is something worth saving** — `createConfluencePage`, one confirmation when
  `{{CONFIRM_WRITES}}` is `always` — and `updateConfluencePage` section by section as the session runs,
  without asking again each time. Sessions get interrupted; nothing gathered should depend on reaching
  the end. On a resumed session, update the existing page; never create a second one for the same key.
- **Shape**: `${CLAUDE_PLUGIN_ROOT}/templates/epic-template.md`, with the header adapted — **Jira task**
  (key and link), **Plan dates** (start, due), **Parent epic** if any, **Refined with**, **Session
  date(s)**, and a **Status** row that reads `En progreso` or `Finalizado`. Write the body in `{{LOCALE}}`
  and in business terms: what the customer said, what it means for their operation, the rules and
  exceptions, who does what, the information involved — no Salesforce vocabulary in the body. Where the
  delivery team needs the mapping, put it in a short closing section *For the delivery team* and nowhere
  else.
- **Never leave a section blank.** What was not answered is `⚠️ TBD — <the question> — <owner>` in the
  *Open questions* table, and each of those becomes a sub-task in 4b.
- Put the page URL on the task with `addCommentToJiraIssue` — one comment, the link and a three-line
  summary of what was settled. That comment is how anyone opening the task in Jira finds the result.

### 4b. The pending items, as sub-tasks under the task

Everything the session left open goes under the task in Jira, one item each, so it shows on the board and
has an owner:

- **What becomes a sub-task**: every `⚠️ TBD` on the page; every decision the customer deferred to someone
  else; every document or example they offered to send; every point that needs a Seamlex decision (a
  parked feature, a conflict with another task). Not the things that were answered — those live on the
  page.
- **Type**: the sub-task type the project exposes — read it once with
  `getJiraProjectIssueTypesMetadata` (`Subtarea`, `Sub-task`, or whatever the project calls it). If the
  task is a `{{TYPE_EPIC}}` and cannot carry sub-tasks, create `{{TYPE_QUESTION}}` issues with the epic
  as parent instead, and say so.
- **Shape**: summary `Pendiente: <the question in one line>`; description with why it matters, what it
  unblocks, who can answer, needed by, and a link to the page section it comes from — the shape of
  `${CLAUDE_PLUGIN_ROOT}/templates/question-template.md`. Assign to the owner when `lookupJiraAccountId`
  resolves them; otherwise leave it unassigned and name them in the description. Label
  `{{LABEL_REQUEST}}` plus `{{LABELS_EXTRA}}`.
- **Check first** against the sub-tasks read in Step 2 — a resumed session must not create the same
  pending item twice. Close, with a comment, any existing sub-task the session answered.
- Show the full list — new, still open, closed today — and get one approval for the batch when
  `{{CONFIRM_WRITES}}` is `always`. Then `createJiraIssue` each one, and write the keys back into the
  page's *Open questions* table.

### 4c. The task itself — finished, or still in progress

Before touching the task, show the customer a summary — what this task delivers in three to six
sentences, the actors and the process it changes, the rules and exceptions captured, what is explicitly
out, and every pending item with its owner — and ask directly whether it describes what they need and
whether anything missing would make it useless. Iterate until they say yes. Do not proceed on silence.

Then decide the state honestly:

- **Finalizado** — the customer approved the summary, the page covers what design needs (walk the
  *Ready for design* checklist at the foot of the template and say which lines are true), and no pending
  sub-task **blocks** design. Set the page's Status row to `Finalizado`, and transition the task to its
  done state (`getTransitionsForJiraIssue`, then `transitionJiraIssue`; the name is whatever the project
  uses — `Done`, `Finalizada`, `Listo`). Open sub-tasks that are informational — an example to send, a
  number to confirm — do not stop this; say which they are.
- **En progreso** — anything else: the summary was not approved, a blocking question is open, the session
  ended early. Leave the task in progress, leave the page at `En progreso`, and list what has to happen
  before the next session closes it. A task is never marked done to make the board look better.

When `{{CONFIRM_WRITES}}` is `always`, the transition to done is a separate, explicit yes; never infer it
from approval of the summary.

Close by showing what was left behind — the page URL, the sub-task keys, the task's state — and the next
task the plan brings, from the same query as Step 1, so the customer can pick it up with
`/seamlex-refinar` or `/seamlex-refinar <KEY>`.

## Failure handling

If any write fails, stop, report exactly what succeeded and what did not, and do not retry blindly — a
page without its sub-tasks is recoverable; a task marked done with nothing behind it is not. If the
Atlassian tools drop out mid-session, stop refining and show the customer everything gathered since the
last successful save so they can keep it themselves, then point them at `/hi-seamlex setup`.

> Atlassian tools come from the MCP server bundled with this plugin and are namespaced by it —
> `mcp__plugin_seamlex-portal_atlassian__searchJiraIssuesUsingJql`. Match on the base name after the last
> `__`; the prefix changes if the server is configured elsewhere, and either one works.
