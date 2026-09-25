---
name: delivery-how-to
description: How Seamlex uses Jira and GitHub — the house rules every Jira and GitHub tool call in a Seamlex session follows. The structure of a customer's documentation repo, the root index file written so Claude can retrieve files later (README.md and config.yml are its root instance), what a well-completed Jira task looks like — summary, description, status, comments, labels, linked to its GitHub file — the Jira labelling convention, the file-path convention on GitHub (every file tied to a task carries its Jira key in the path and its front matter), and the JQL retrieval patterns for Jira plus the path/search patterns for GitHub. Loaded by /hi-seamlex at session start; applies to every read and write in Jira and GitHub made by any Seamlex command or skill.
---

# What this skill is for

Every Seamlex command reads and writes the same Jira project and the same GitHub documentation repo,
and every one of them needs the same answers: where does this file go, what is it called, how is the
task it belongs to left, and how will someone — or Claude, next session — find it again. The commands
say **what** to write and when; this skill says **where it goes, what it is called, how it is linked
and how it is found**. When a command and this skill disagree on one of those, this skill wins; when
this skill is silent, the command decides.

It is loaded by `/hi-seamlex` at the start of the session, after `README.md` and `config.yml`, and
stays in context for every later command. If a command finds it is not in context, it loads it itself
(`Skill` tool, `seamlex-portal:delivery-how-to`, or this file directly).

Every `{{PLACEHOLDER}}` below resolves from `config.yml` in the customer's GitHub repo —
`{{JIRA_PROJECT}}`, `{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`, `{{TYPE_TASK}}`,
`{{CONFIRM_WRITES}}`, `{{DETAIL}}`, `{{DRAFTS_DIR}}`, `{{PROGRAM}}`, `{{COMPANY}}` — or from the
session: `{{CLOUD_ID}}`, `{{GITHUB_ORG}}` and `{{GITHUB_REPO}}` from `/hi-seamlex`, `{{LOCALE}}` and
`{{USER_NAME}}` from `atlassianUserInfo`. If `config.yml` is not in context, stop and ask the customer
to run `/hi-seamlex`; nothing here works against a guessed repo or project.

# The GitHub repo — structure

One customer, one repo, one shape. Everything the plugin and the Seamlex delivery team write lands in
a place this tree already has; nothing is created where the tree has no place for it.

```
{{GITHUB_ORG}}/{{GITHUB_REPO}}
├── README.md                          root index — purpose + pointer to config.yml (Seamlex)
├── config.yml                         settings, machine-readable (Seamlex)
├── scope.md                           the contract: what is in, what is written as out (Seamlex)
├── discovery/
│   └── discovery-brief.md             the project's understanding of the business (Seamlex + customer)
├── relevamientos/
│   ├── <KEY>.md                       one per relevamiento task   (/seamlex-refinar)
│   └── <KEY>-transcript.md            always written alongside it (/seamlex-refinar)
├── glosario.md                        the customer's vocabulary
└── features-no-identificados.md       what was raised outside the signed scope (/seamlex-refinar)
```

The paths above are the defaults; a customer's repo may lay things out differently, and `config.yml`
says which file plays which role if it does. **Follow `config.yml`, then this tree, never a guess.**
When `config.yml` does not name a path a write needs, say so and ask where it goes; do not invent a
new top-level folder on your own initiative.

| Rule | Why |
|---|---|
| **One file per task.** Before creating a file tied to a Jira key, check whether it already exists (`get_file_contents` on the exact path). If it exists, update it. | A second minuta file for the same task splits the record; the delivery team reads the wrong one. |
| **Paths are `relevamientos/<KEY>.md`** — the Jira key upper-cased, exactly as Jira writes it, no summary in the filename. | The path *is* the identity: no title search, no ambiguity, no risk of the filename drifting from the Jira summary as it would with a Confluence-style title. |
| **Every file tied to a task opens with YAML front matter** naming `jira_key`, `jira_url` and `type` (`minuta` / `transcript`), followed by the same header table the file used to open with in Confluence (**Tarea Jira**, **Sprint**, **Épica**, **Relevado con**, **Fecha(s)**, **Estado**). | Front matter is what a script or a future Claude session greps for; the header table is what a human reads first when they open the file on GitHub. |
| **No GitHub labels or topics.** Nothing the plugin writes or reads depends on a repo label or topic; a file is found by its path, never by metadata GitHub's UI happens to offer. | Keeps one convention (path) instead of two half-maintained ones (path and label), the same reasoning that dropped Confluence labels for the same files. |
| **Children go under their folder**: minutas and transcripts under `relevamientos/`. | Anyone — human or Claude — can list the folder and see every relevamiento the project has. |
| **The body is in `{{LOCALE}}` and in business terms**; platform vocabulary lives only in a file's closing *Para el equipo de delivery* section. | The customer reads these files; the `business-analysis` rule applies on the file as in the room. |

# The index files — written so Claude can retrieve from them

`README.md` plus `config.yml` together play the role the single `seamlex-portal-memory` Confluence
page used to: `README.md` is the **purpose and map**, human-readable; `config.yml` is the **settings**,
machine-readable. Splitting them is a GitHub-native move — a settings file that used to be a table on
a wiki page is now just a file a command can parse directly, no CQL involved. The full templates are
in `references/memory-template.md`.

`README.md`:

1. **Purpose** — one paragraph: what this repo covers and what it deliberately does not.
2. **Map** — a short table, one row per top-level path: **Path** · **What it holds** · **Read when**
   (`every session` / `when the task touches <area>` / `on demand`). GitHub's own file tree already
   shows what exists; this table says what each part is *for*, which the tree cannot.
3. **Maintained by / last updated** — who owns the repo and when it last changed.

`config.yml` — one key per setting, spelled exactly as the commands name it (`JIRA_PROJECT`,
`TYPE_TASK`, `LABEL_REQUEST`, `LABELS_EXTRA`, `CONFIRM_WRITES`, `DETAIL`,
`DRAFTS_DIR`, `PROGRAM`, `COMPANY`). A setting that does not apply is a missing key, not a blank
value. No plugin release is needed to change it — an edit to the customer's own `config.yml` is
enough, exactly as an edit to `seamlex-portal-memory` used to be.

Claude reads `README.md` and `config.yml` before searching, and the repo's own file tree before
inventing a search. When Claude creates a file, it did not need to add itself to an index the way a
Confluence page did — the file tree *is* the index — but a `README.md` whose Map table has gone stale
(a new top-level path with no row) should be flagged, not silently left wrong.

# A well-completed Jira task

A task is complete when someone who was not in the room can open it in Jira and, from the task alone,
learn what it was for, what happened, where the record is, and what is still open. Field by field:

| Field | The rule |
|---|---|
| **Summary** | `<Prefix>: <the outcome in one line>`, in `{{LOCALE}}`, in business words — never platform vocabulary. Prefixes in use: none for a relevamiento (its Jira summary is also the minuta file's header, so it is kept as it reads), `Pendiente:` for a pending item. One outcome per task; a summary with "and" is usually two tasks. |
| **Type and parent** | A relevamiento keeps whatever issue type the project already uses for it — it is found by the `relevamiento` label, not by its type. A pending item's type is `{{TYPE_TASK}}` from the config, checked once against `getJiraProjectIssueTypesMetadata`. Pending items are **sub-tasks of the task they came from** — `parent` set, sprint inherited, never a standalone issue linked with *relates to*. Anything the plugin has to say about an existing task is a **comment** on that task, never a new issue. |
| **Description** | What is being asked, why it matters, who it is for, and what *done* looks like — so the assignee does not have to come back to ask. Pending items take the shape of `../business-analysis/references/pendiente-template.md`. Always a link to the GitHub file it comes from, and to the section when there is one. |
| **Status** | Read the project's real workflow with `getTransitionsForJiraIssue` — never assume names. Map it to the four states the plugin reasons in: **to do → in progress → waiting on the customer / blocked → done**. Move a task to *in progress* when the work starts, not when it ends. **Done is earned**: a relevamiento is done only when the customer approved the summary, the minuta file says `Finalizado`, and no pending item *blocks* design; a pending item is done only when its answer is written back on the minuta. Never done to make the board look better. |
| **Comments** | Every write that changes the task leaves a comment saying what changed and why — a status move, a new sub-task, an answer. Per session, one **result comment**: who ran it (the Seamlex role, Claude, with `{{USER_NAME}}`), the date, three lines of what was settled, the URL of the GitHub file, the keys created, and the state the task was left in. When there is a transcript, a **second, dedicated comment** headed `Transcript del relevamiento — <date>` with the URL of the transcript file — never the transcript text itself, since the file already holds it. A pending item that gets answered is closed **with a comment holding the answer**, not silently. |
| **Labels** | `{{LABEL_REQUEST}}` and `{{LABELS_EXTRA}}` on everything the plugin creates, plus the kind label from the *Labels* table. Set at creation — `createJiraIssue` takes them in `additional_fields.labels`. |
| **Assignee** | The owner, when `lookupJiraAccountId` resolves them; otherwise unassigned and the owner named in the first line of the description. Never assigned to whoever is signed in by default. |
| **Linked to its GitHub file** | Two directions, both weaker than a native cross-link and said plainly rather than implied. **Jira → GitHub**: the file's blob URL in the result comment (and a remote link when the server offers a tool for it — the bundled one does not; say when the comment is the only Jira-side link). **GitHub → Jira**: the Jira key in the file's front matter (`jira_key:`) and in its header table as a link to the issue — GitHub cannot list the task under the file the way Confluence used to list a page under an issue, so this direction is readable and greppable, not a live backlink. |
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
| `relevamiento` | every relevamiento task — it is how the plugin finds them, not the issue type | Seamlex |
| `pendiente` | every `Pendiente:` sub-task | `/seamlex-refinar` |
| `no-identificado` | any issue Seamlex raises from the unidentified-features file | Seamlex |

**GitHub files carry no labels or topics the plugin knows about.** A file is identified by its
**path** — `relevamientos/<KEY>.md` — and, secondarily, by the `jira_key` in its front matter. If a
customer's repo happens to have topics or labels on its issues, they are ignored; this plugin does not
use GitHub Issues at all, only the repo's files.

# Retrieval patterns

**Jira — `searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`, always scoped to `project = {{JIRA_PROJECT}}`**

| Looking for | JQL |
|---|---|
| Relevamientos of the open sprint | `project = {{JIRA_PROJECT}} AND sprint in openSprints() AND labels = "relevamiento" AND statusCategory != Done ORDER BY Rank ASC` |
| The pending items of a task | `project = {{JIRA_PROJECT}} AND parent = <KEY> ORDER BY status` — or `getJiraIssue` on the task, sub-tasks included |
| Every open pending item | `project = {{JIRA_PROJECT}} AND labels = "pendiente" AND statusCategory != Done ORDER BY created ASC` |
| Pending items owned by someone | `… AND labels = "pendiente" AND assignee = <accountId>` — `lookupJiraAccountId` first |
| Everything the plugin created | `project = {{JIRA_PROJECT}} AND labels = {{LABEL_REQUEST}} ORDER BY updated DESC` |
| One epic's tasks — the siblings of a relevamiento | `project = {{JIRA_PROJECT}} AND parent = <EPIC-KEY> ORDER BY status` |
| Earlier relevamientos, any sprint | `project = {{JIRA_PROJECT}} AND labels = "relevamiento" ORDER BY updated DESC` |
| Related work by words | `project = {{JIRA_PROJECT}} AND text ~ "<key terms>"` — last resort |

Ask each query for the fields the next step needs — `summary, status, issuetype, parent, assignee,
labels, sprint, duedate, updated` — rather than re-fetching one issue at a time.

**GitHub — always scoped to `{{GITHUB_ORG}}/{{GITHUB_REPO}}`**

The order is always the same: **`README.md`/`config.yml` first, then the fixed path, then search,
then commit history.** The index says what exists; the fixed path finds a task's file directly — no
fuzzy matching needed, unlike Confluence's title search — and search and history are for what the
fixed paths do not cover.

| Looking for | How |
|---|---|
| The root index | `get_file_contents` on `README.md` |
| The settings | `get_file_contents` on `config.yml` |
| The one minuta of a task | `get_file_contents` on `relevamientos/<KEY>.md` directly. Not found → the file does not exist; create it. |
| Its transcript | `get_file_contents` on `relevamientos/<KEY>-transcript.md` |
| The latest minutas | `list_commits` scoped to `relevamientos/`, most recent first, or `get_file_contents` on the directory listing plus a handful of the most recently touched files |
| Everything under a folder | `get_file_contents` on the folder path (returns the directory listing) |
| Pages about a part of the business | `search_code` scoped to the repo, with the customer's own term — last resort; read filenames before opening anything |
| Changed recently | `list_commits` on the repo, most recent first |

Never guess a path. Never answer from what was read in an earlier turn — the board and the repo move;
re-query. `getJiraIssue` with description, comments, sub-tasks, links and parent; `getJiraIssueRemoteIssueLinks`
for anything already attached to it, understanding that a GitHub file will not show up there the way a
Confluence page sometimes did — the result comment is the source of truth for that link now.

# Rules of interaction with the Jira and GitHub tools

These hold for every read and write in a Seamlex session, whichever command is running.

**Before any write**

1. **It was searched for.** The file about to be created was looked for at its fixed path first. If it
   exists, update it; a second copy is never the answer.
2. **It has a place in the tree** — a path the structure above names and `config.yml` confirms. No
   place → ask, do not invent a folder.
3. **The path follows the convention** — `relevamientos/<KEY>.md` for a file tied to a task — the
   front matter opens with `jira_key`, the header table's Jira key is a link to the issue, and the
   body is in `{{LOCALE}}` in business terms.
4. **Its counterpart is known**: a file knows its task (front matter), a task will get the file's URL
   in a comment. A write that has no counterpart is the exception, and the command says why.
5. **`{{CONFIRM_WRITES}}`** — when `always`, the exact text is shown and approved first; a transition
   to done is a separate, explicit yes, never inferred from approval of a summary. Drafting locally and
   showing before writing is the default for anything with a customer-facing text.

**Order of writes.** The GitHub file first — it is committed as soon as there is something worth
saving and updated as the session runs — then the Jira sub-tasks, then the result comment (which needs
the file's URL and the sub-task keys), then the transition. Everything points at the file, so the file
exists before anything points at it.

**After any write**

6. **Re-read what landed** — `getJiraIssue` on the task to see its sub-tasks and comment,
   `get_file_contents` on the committed file — and report from the re-read, not from the intent.
7. **The file tree is the index**, so there is no separate row to add — but if `README.md`'s Map table
   has gone stale (a new top-level path it does not mention), say so; Seamlex updates it.
8. **Walk the checklist** in `references/jira-task-checklist.md` for the task's kind before calling it
   done, and say which lines are not true.

**When something fails.** Stop. Report exactly what succeeded and what did not, with keys and URLs. Do
not retry blindly — a minuta without its sub-tasks is recoverable; a task marked done with nothing
behind it is not. If the tools drop out mid-session, show the customer everything gathered since the
last successful save so they keep it themselves, and point them at `/hi-seamlex`.

**Naming of the tools.** Jira tools come from the Atlassian MCP server bundled with the plugin,
namespaced by it — `mcp__plugin_seamlex-portal_atlassian__searchJiraIssuesUsingJql`. GitHub tools are
different: the plugin cannot bundle a working GitHub connection (a personal access token can't be
substituted into a shared config file), so each person adds their own `github` MCP server themselves —
its namespace prefix is whatever they named it, typically `mcp__github__get_file_contents` for the
server name `SETUP.md` has them use. Match on the base name after the last `__` for either kind — the
prefix always varies for GitHub and sometimes varies for Jira if the customer has a server configured
elsewhere, and either one works. If a server is not available at all, say plainly which one (Jira or
GitHub) cannot be reached rather than answering from guesswork, and stop.

# References

| Reference | What it shapes |
|---|---|
| `references/memory-template.md` | The body of `README.md` and `config.yml` — the root index with its map and settings, with a filled example. |
| `references/jira-task-checklist.md` | The *complete task* checklist, one block per kind — relevamiento, `Pendiente:` sub-task — walked before a task is called done. |
| `../business-analysis/references/pendiente-template.md` | The description of a pending item — owned by the `business-analysis` skill, named here because *A well-completed Jira task* points at it. |
