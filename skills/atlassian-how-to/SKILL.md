---
name: atlassian-how-to
description: How Seamlex uses Jira and Confluence — the house rules every Atlassian tool call in a Seamlex session follows. The structure of a customer's Confluence space, the index page written so Claude can retrieve pages later (claude-client-config is its root instance), what a well-completed Jira task looks like — summary, description, status, comments, labels, linked to its Confluence page — the Jira labelling convention, the title convention on Confluence (every page tied to a task carries its Jira key in the title; Confluence labels are not used because the Atlassian MCP cannot set them), and the CQL/JQL retrieval patterns to find pages by title and ancestor and tasks by key, label or text. Loaded by /hi-seamlex at session start; applies to every read and write in Jira and Confluence made by any Seamlex command or skill.
---

# What this skill is for

Every Seamlex command reads and writes the same Jira project and the same Confluence space, and every
one of them needs the same answers: where does this page go, what is it called, how is the task it
belongs to left, and how will someone — or Claude, next session — find it again. The commands say
**what** to write and when; this skill says **where it goes, what it is called, how it is linked and
how it is found**. When a command and this skill disagree on one of
those, this skill wins; when this skill is silent, the command decides.

It is loaded by `/hi-seamlex` at the start of the session, after the `claude-client-config` page, and
stays in context for every later command. If a command finds it is not in context, it loads it itself
(`Skill` tool, `seamlex-portal:atlassian-how-to`, or this file directly).

Every `{{PLACEHOLDER}}` below resolves from the `claude-client-config` page `/hi-seamlex` loaded —
`{{JIRA_PROJECT}}`, `{{CONF_PARENT}}`, `{{CONF_SCOPE_PAGE}}`, `{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`,
`{{TYPE_RELEVAMIENTO}}`, `{{TYPE_TASK}}`, `{{SEAMLEX_CONTACT}}`, `{{CONFIRM_WRITES}}`, `{{DETAIL}}`,
`{{PROGRAM}}`, `{{COMPANY}}` — or from the session: `{{CLOUD_ID}}` and `{{CONF_SPACE}}` from `/hi-seamlex`,
`{{LOCALE}}` and `{{USER_NAME}}` from `atlassianUserInfo`. If the config page is not in context, stop
and ask the customer to run `/hi-seamlex`; nothing here works against a guessed space or project.

# The Confluence space — structure

One customer, one space, one shape. Everything the plugin and the Seamlex delivery team write lands in
a place this tree already has; nothing is created where the tree has no place for it.

```
{{CONF_SPACE}}
├── claude-client-config                         root index — settings + map of the project's knowledge (Seamlex)
├── <signed scope page>  = {{CONF_SCOPE_PAGE}}     the contract: what is in, what is written as out (Seamlex)
├── Discovery Brief — {{PROGRAM}}                 §1–10, the project's understanding of the business (Seamlex + customer)
├── {{CONF_PARENT}}                               section index for relevamientos (e.g. "Relevamientos")
│   ├── Minuta <KEY> — <task summary as in Jira>    one per relevamiento task   (/seamlex-refinar)
│   │   └── Transcript <KEY> — <task summary>       only when the transcript did not fit a Jira comment
│   └── …
├── Procesos                                      section index + one page per business process (Seamlex + customer)
├── Glosario                                      the customer's vocabulary, one page or one per area
├── Referencia                                    section index + anything else the config lists — org charts, samples, policies
├── Features no identificados — {{PROGRAM}}       what was raised outside the signed scope (/seamlex-refinar)
└── Delivery                                      section index — what the delivery team derives from the minutas
    ├── Análisis funcional <KEY>                    (seamlex-deliver-team /gate-funcional)
    └── High-level design <KEY>                     (seamlex-deliver-team /high-level-design)
```

The section names are the defaults; a customer's space may call them differently, and the
`claude-client-config` page says which page plays which role. **Follow the config page, then this tree,
never a guess.** When the config page does not name a section that a write needs — there is no
`{{CONF_PARENT}}`, no *Delivery* page — say so and ask where it goes; do not create the section on your
own initiative.

| Rule | Why |
|---|---|
| **Every section parent is an index page** in the shape of *The index page* below — never an empty container. | The parent is how the section is found and how Claude knows what each child is without opening it. |
| **One page per task.** Before creating a page tied to a Jira key, search for it (*Retrieval patterns* → *pages about a task*). If it exists, update it. | A second `Minuta` for the same task splits the record; the delivery team reads the wrong one. |
| **Titles are `<Type> <KEY> — <exact Jira summary>`** — `Minuta ABC-12 — <summary>`, `Transcript ABC-12 — <summary>`, `Análisis funcional <KEY>`. The Jira key in upper case, then an em dash, then the Jira wording even when a better title came up. | The title is the only handle a page has: `title ~ "ABC-12"` finds every page of a task, the prefix tells the kinds apart, the summary keeps it readable from the board. |
| **Every page tied to a task opens with a header table** whose first row is the Jira key **written as a link** to the issue (`https://<site>/browse/<KEY>`). | Confluence renders it as a Jira link and Jira lists the page under the issue — half of the two-way link. |
| **No Confluence labels.** Nothing the plugin writes or reads depends on a page label; a page is found by title and by ancestor, never by label. | The Atlassian MCP's `createConfluencePage` / `updateConfluencePage` have no label parameter, so a convention built on labels would never hold. |
| **Children go under their parent**: minutas under `{{CONF_PARENT}}`, transcripts under their minuta, delivery documents under *Delivery*. | `ancestor =` searches and `getConfluencePageDescendants` depend on it. |
| **The body is in `{{LOCALE}}` and in business terms**; platform vocabulary lives only in a page's closing *Para el equipo de delivery* section, or in the *Delivery* pages. | The customer reads these pages; the `business-analysis` rule applies on the page as in the room. |

# The index page — written so Claude can retrieve from it

An index page is a table of pages with, for each one, enough to decide whether to open it and how to
find it. `claude-client-config` is the **root** index — it also carries the settings — and every
section parent (`{{CONF_PARENT}}`, *Procesos*, *Referencia*, *Delivery*) is a **section** index in the
same shape without the settings. The full body is in `references/index-page-template.md`; the shape:

1. **Purpose** — one paragraph: what this index covers and what it deliberately does not.
2. **Settings** — root page only. A two-column table, one setting per row, key exactly as the commands
   name it (`JIRA_PROJECT`, `CONF_PARENT`, `CONF_SCOPE_PAGE`, `TYPE_RELEVAMIENTO`, `TYPE_TASK`,
   `LABEL_REQUEST`, `LABELS_EXTRA`, `SEAMLEX_CONTACT`, `CONFIRM_WRITES`, `DETAIL`, `DRAFTS_DIR`,
   `PROGRAM`, `COMPANY`), value, and a short meaning. A missing setting is a missing row, not a blank value.
3. **Pages** — the map. One row per page: **Title (exact)** · **Link** · **Type** ·
   **What to take from it** · **Read when**. *Type* uses the fixed vocabulary in the table below;
   *Read when* is one of `every session` / `when the task touches <area>` / `on demand`.
4. **How to find what is not listed** — the two or three CQL shapes that cover this section, ready to
   run (space, ancestor, title prefix filled in).
5. **Maintained by / last updated** — who owns the page and when it last changed.

What makes it retrievable, and what breaks it:

| Do | Don't |
|---|---|
| Titles typed **exactly** as the page is titled — search is `title = "…"`. | A paraphrase, an abbreviation, a title with the emoji dropped. |
| One page per row; the *What to take from it* cell says what the page settles, in one or two lines. | Prose between rows, several pages in one cell, "see below". |
| The same *Type* words everywhere — `scope`, `discovery`, `minuta`, `proceso`, `glosario`, `referencia`, `analisis-funcional`, `hld`. | A new type invented per row. |
| A Jira key, wherever it appears, as a link to the issue. | A bare key Confluence cannot resolve. |
| A row for every page under this section — a page that is not listed is a page nobody will read. | Index rows for pages that no longer exist. |

Claude reads the index before searching, and the search shapes at its foot before inventing one. When
Claude creates a page under a section whose index it can edit, it adds the row in the same session;
when it cannot (the `claude-client-config` page is Seamlex's — **never edit it**), it says which row
Seamlex should add.

# A well-completed Jira task

A task is complete when someone who was not in the room can open it in Jira and, from the task alone,
learn what it was for, what happened, where the record is, and what is still open. Field by field:

| Field | The rule |
|---|---|
| **Summary** | `<Prefix>: <the outcome in one line>`, in `{{LOCALE}}`, in business words — never platform vocabulary. Prefixes in use: none for a relevamiento (its Jira summary is also the minuta's title, so it is kept as it reads), `Pendiente:` for a pending item. One outcome per task; a summary with "and" is usually two tasks. |
| **Type and parent** | The type the config names (`{{TYPE_RELEVAMIENTO}}`, `{{TYPE_TASK}}`), checked once against `getJiraProjectIssueTypesMetadata`. Pending items are **sub-tasks of the task they came from** — `parent` set, sprint inherited, never a standalone issue linked with *relates to*. Anything the plugin has to say about an existing task is a **comment** on that task, never a new issue. |
| **Description** | What is being asked, why it matters, who it is for, and what *done* looks like — so the assignee does not have to come back to ask. Pending items take the shape of `../business-analysis/references/pendiente-template.md`. Always a link to the Confluence page it comes from, and to the section when there is one. |
| **Status** | Read the project's real workflow with `getTransitionsForJiraIssue` — never assume names. Map it to the four states the plugin reasons in: **to do → in progress → waiting on the customer / blocked → done**. Move a task to *in progress* when the work starts, not when it ends. **Done is earned**: a relevamiento is done only when the customer approved the summary, the minuta says `Finalizado`, and no pending item *blocks* design; a pending item is done only when its answer is written back on the minuta. Never done to make the board look better. |
| **Comments** | Every write that changes the task leaves a comment saying what changed and why — a status move, a new sub-task, an answer. Per session, one **result comment**: who ran it (the Seamlex role, Claude, with `{{USER_NAME}}`), the date, three lines of what was settled, the URL of the Confluence page, the keys created, and the state the task was left in. When there is a transcript, a **second, dedicated comment** headed `Transcript del relevamiento — <date>`, or the URL of its Confluence child page when it did not fit. A pending item that gets answered is closed **with a comment holding the answer**, not silently. |
| **Labels** | `{{LABEL_REQUEST}}` and `{{LABELS_EXTRA}}` on everything the plugin creates, plus the kind label from the *Labels* table. Set at creation — `createJiraIssue` takes them in `additional_fields.labels`. |
| **Assignee** | The owner, when `lookupJiraAccountId` resolves them; otherwise unassigned and the owner named in the first line of the description. Never assigned to whoever is signed in by default. |
| **Linked to its Confluence page** | Two directions. **Jira → Confluence**: the page URL in the result comment (and a remote link when the server offers a tool for it — the bundled one does not; say when the comment is the only Jira-side link). **Confluence → Jira**: the Jira key in the page's header table as a link to the issue, which makes Jira list the page under the issue. After the page is saved, `getJiraIssueRemoteIssueLinks` — if the page does not appear, say so; the comment link still holds. |
| **Dates** | Only dates that exist — a due date the customer gave, a sprint end. Never an ETA invented to fill the field. |

The copy-and-walk checklist, one block per kind — relevamiento, pending sub-task — is in
`references/jira-task-checklist.md`. A command walks the block that applies **before** it calls the
task done, and says which lines are not true rather than closing over them.

# Labels — Jira only

Jira labels are how an issue is found without knowing its summary. The plugin sets them **when the
issue is created** (`createJiraIssue`, `additional_fields: {"labels": [...]}`), never retro-fitted
quietly later. All labels are lower-case, ASCII, hyphenated.

| Label | On | Set by |
|---|---|---|
| `{{LABEL_REQUEST}}` | every issue the plugin creates — it is how *everything I raised* is answered | the command |
| `{{LABELS_EXTRA}}` | the same, when the config sets it | the command |
| `relevamiento` | relevamiento tasks, when the config marks them by label rather than by type (`{{TYPE_RELEVAMIENTO}}`) | Seamlex |
| `pendiente` | every `Pendiente:` sub-task | `/seamlex-refinar` |
| `no-identificado` | any issue Seamlex raises from the unidentified-features page | Seamlex |

**Confluence pages carry no labels the plugin knows about.** The Atlassian MCP cannot set them, so
nothing is written, read, promised or checked on that side. A page is identified by its **title** —
the `<Type> <KEY> — <summary>` convention above — and by its **place in the tree**. If a customer's
space happens to have labels on its pages, they are ignored.

# Retrieval patterns

The order is always the same: **the index first, then the key in the title, then the title prefix
under an ancestor, then text.** The index says what exists; the key in the title finds a task's pages;
the prefix (`Minuta`, `Transcript`, `Análisis funcional`) tells the kinds apart; ancestor walks a
section; the full exact title confirms; text is the last resort and returns noise. Never guess a page
id or an issue key. Never answer from what was read in an earlier turn — the board and the space move;
re-query.

**Confluence — `searchConfluenceUsingCql` on `{{CLOUD_ID}}`, always scoped to `space = "{{CONF_SPACE}}"`**

| Looking for | CQL |
|---|---|
| The root index | `space = "{{CONF_SPACE}}" AND title = "claude-client-config"` |
| A section index | `space = "{{CONF_SPACE}}" AND title = "<section title as the root index lists it>"` |
| Every page tied to a task | `space = "{{CONF_SPACE}}" AND title ~ "<JIRA-KEY>"` — the key upper-cased as it is in the title; the prefix of each title tells minuta from transcript from analysis |
| The one minuta of a task | `space = "{{CONF_SPACE}}" AND title ~ "<JIRA-KEY>" AND title ~ "Minuta"` — then confirm the exact title `Minuta <KEY> — <exact Jira summary>`. If that returns nothing, `getJiraIssueRemoteIssueLinks` on the task: the header-table link makes Jira list the page. Nothing there → the page does not exist; create it |
| The latest minutas | `space = "{{CONF_SPACE}}" AND ancestor = <{{CONF_PARENT}} id> AND title ~ "Minuta" ORDER BY lastmodified DESC` |
| All pages of one kind | the section's ancestor plus the title prefix — `ancestor = <Delivery id> AND title ~ "Análisis funcional"`, `ancestor = <Procesos id>` for the processes, `title ~ "Glosario"` for the glossary |
| Everything under a section | `space = "{{CONF_SPACE}}" AND ancestor = <parent page id>` — or `getConfluencePageDescendants` on the parent |
| Other meeting notes | `space = "{{CONF_SPACE}}" AND (title ~ "Minuta" OR title ~ "Meeting notes" OR title ~ "Acta" OR title ~ "Reunión") ORDER BY lastmodified DESC` |
| Pages about a part of the business | `space = "{{CONF_SPACE}}" AND text ~ "<the customer's own term>"` — last resort; read titles before opening anything |
| Changed recently | `space = "{{CONF_SPACE}}" AND lastmodified >= now("-14d") ORDER BY lastmodified DESC` |

**Jira — `searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`, always scoped to `project = {{JIRA_PROJECT}}`**

| Looking for | JQL |
|---|---|
| Relevamientos of the open sprint | `project = {{JIRA_PROJECT}} AND sprint in openSprints() AND issuetype = "{{TYPE_RELEVAMIENTO}}" AND statusCategory != Done ORDER BY Rank ASC` — `labels = "relevamiento"` instead of `issuetype` when the config marks them by label |
| The pending items of a task | `project = {{JIRA_PROJECT}} AND parent = <KEY> ORDER BY status` — or `getJiraIssue` on the task, sub-tasks included |
| Every open pending item | `project = {{JIRA_PROJECT}} AND labels = "pendiente" AND statusCategory != Done ORDER BY created ASC` |
| Pending items owned by someone | `… AND labels = "pendiente" AND assignee = <accountId>` — `lookupJiraAccountId` first |
| Everything the plugin created | `project = {{JIRA_PROJECT}} AND labels = {{LABEL_REQUEST}} ORDER BY updated DESC` |
| One epic's tasks — the siblings of a relevamiento | `project = {{JIRA_PROJECT}} AND parent = <EPIC-KEY> ORDER BY status` |
| Earlier relevamientos, any sprint | `project = {{JIRA_PROJECT}} AND issuetype = "{{TYPE_RELEVAMIENTO}}" ORDER BY updated DESC` |
| Related work by words | `project = {{JIRA_PROJECT}} AND text ~ "<key terms>"` — last resort |

Ask each query for the fields the next step needs — `summary, status, issuetype, parent, assignee,
labels, sprint, duedate, updated` — rather than re-fetching one issue at a time.

**Reading rules.** `getConfluencePage` for the body — read it, do not summarise it away; the material
stays in context for the session. Read a page's comments (`getConfluencePageFooterComments`,
`getConfluencePageInlineComments`) when it looks contested. `getJiraIssue` with description, comments,
sub-tasks, links and parent; `getJiraIssueRemoteIssueLinks` for the pages attached to it. Content
written to Confluence follows `getContentFormatGuide` — call it once before the first page write of a
session rather than guessing the markup.

# Rules of interaction with the Atlassian tools

These hold for every read and write in a Seamlex session, whichever command is running.

**Before any write**

1. **It was searched for.** The page or issue about to be created was looked for with the patterns
   above — by key in the title, then by exact title, then through the task's remote links. If it
   exists, update it; a second copy is never the answer.
2. **It has a place in the tree** — a parent the structure names and the config resolves. No place →
   ask, do not invent a section.
3. **The title follows the convention** — `<Type> <KEY> — <exact Jira summary>` for a page tied to a
   task — the header table opens with the Jira key as a link, and the body is in `{{LOCALE}}` in
   business terms. Jira labels are in the `createJiraIssue` call; a Confluence page gets none.
4. **Its counterpart is known**: a page knows its task, a task will get the page URL in a comment. A
   write that has no counterpart is the exception, and the command says why.
5. **`{{CONFIRM_WRITES}}`** — when `always`, the exact text is shown and approved first; a transition
   to done is a separate, explicit yes, never inferred from approval of a summary. Drafting locally and
   showing before writing is the default for anything with a customer-facing text.

**Order of writes.** The Confluence page first — it is created as soon as there is something worth
saving and updated as the session runs — then the Jira sub-tasks, then the result comment (which needs
the page URL and the sub-task keys), then the transition. Everything points at the page, so the page
exists before anything points at it.

**After any write**

6. **Re-read what landed** — `getJiraIssue` on the task to see its sub-tasks and comment,
   `getConfluencePage` on the page, `getJiraIssueRemoteIssueLinks` for the link — and report from the
   re-read, not from the intent.
7. **Close the loop on the index**: when a page was created under a section whose index Claude may
   edit, add the row; otherwise name the row for Seamlex to add. Never edit `claude-client-config`.
8. **Walk the checklist** in `references/jira-task-checklist.md` for the task's kind before calling it
   done, and say which lines are not true.

**When something fails.** Stop. Report exactly what succeeded and what did not, with keys and URLs. Do
not retry blindly — a minuta without its sub-tasks is recoverable; a task marked done with nothing
behind it is not. If the tools drop out mid-session, show the customer everything gathered since the
last successful save so they keep it themselves, and point them at `/hi-seamlex`.

**Naming of the tools.** They come from the Atlassian MCP server bundled with the plugin and are
namespaced by it — `mcp__plugin_seamlex-portal_atlassian__searchJiraIssuesUsingJql`. Match on the base
name after the last `__`; the prefix differs if the customer has the server configured elsewhere, and
either one works. If they are not available at all, say plainly that Jira and Confluence cannot be
reached rather than answering from guesswork, and stop.

# References

| Reference | What it shapes |
|---|---|
| `references/index-page-template.md` | The body of an index page — the root `claude-client-config` with its settings table, and a section index — with a filled example row. |
| `references/jira-task-checklist.md` | The *complete task* checklist, one block per kind — relevamiento, `Pendiente:` sub-task — walked before a task is called done. |
| `../business-analysis/references/pendiente-template.md` | The description of a pending item — owned by the `business-analysis` skill, named here because *A well-completed Jira task* points at it. |
