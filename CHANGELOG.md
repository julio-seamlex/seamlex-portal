# Changelog

## 2.0.0

> Documentation moves off Confluence and onto GitHub. Jira stays exactly as it was — this is a
> documentation-store swap, not a move off Atlassian. **Breaking**: `/hi-seamlex` now signs in to a
> GitHub-bundled MCP server (a `GITHUB_PAT` personal access token) and resolves settings from
> `config.yml` in the customer's GitHub repo instead of a `seamlex-portal-memory` Confluence page.

- **`skills/atlassian-how-to/` is renamed `skills/delivery-how-to/`** and rewritten: the Confluence
  space tree, title convention and CQL cookbook become a GitHub repo tree
  (`README.md`, `config.yml`, `scope.md`, `discovery/`, `relevamientos/<KEY>.md`, `glosario.md`,
  `features-no-identificados.md`), a fixed-path convention, and path/search/commit-history lookups.
  The Jira sections (labels, JQL cookbook, "a well-completed Jira task") are unchanged beyond
  repointing "linked to its Confluence page" at GitHub.
- **The single `seamlex-portal-memory` config page becomes two files**: `README.md` (purpose + map,
  human-readable) and `config.yml` (settings, machine-readable) in the customer's own GitHub repo.
  `CONF_PARENT` and `CONF_SCOPE_PAGE` settings are dropped — paths are now fixed by the repo layout.
- **`commands/hi-seamlex.md`** now checks the `github` MCP server alongside `atlassian`, resolves the
  customer's `seamlex-docs-<slug>` repo instead of a Confluence space, and reads `README.md`/`config.yml`
  from it.
- **`commands/seamlex-refinar.md`** §4a commits `relevamientos/<KEY>.md` via `create_or_update_file`
  instead of `createConfluencePage`/`updateConfluencePage`. §4c always writes the transcript as its own
  committed file (`relevamientos/<KEY>-transcript.md`) — the old length-based fallback to a Confluence
  child page is gone, since GitHub has no comment-length limit to work around. §4e's two-way link is
  now a GitHub blob URL in the Jira comment, and a `jira_key`/`jira_url` front-matter pair in the file
  — said plainly as a readable link, not a live backlink the way Confluence sometimes offered.
- **`minuta-template.md`** gains YAML front matter (`jira_key`, `jira_url`, `type: minuta`) ahead of its
  existing header table.
- **`.mcp.json`** bundles a `github` HTTP MCP server (`https://api.githubcopilot.com/mcp/`,
  PAT-authenticated) alongside the existing `atlassian` SSE server.
- `README.md` and `SETUP.md` are rewritten for the GitHub workflow; `SETUP.md` gains a "Connecting
  GitHub" section (token creation, `GITHUB_PAT`, restart) alongside "Connecting Atlassian".
- **Follow-up, not in this release**: the sibling `seamlex-deliver-team` plugin still reads Confluence
  pages this plugin no longer writes and needs matching changes in its own repo.

## 1.18.1

> The minuta template's 13 thin, prose-heavy sections are replaced by 7 sections centered on a
> **Requirements** table — every discrete ask gets a stable ID and a traceable source instead of loose
> "Functional detail" subsections.

- **`minuta-template.md` is remade around a Requirements table** (REQ-ID, capability, actor, statement,
  source, MoSCoW, complexity 1–10, in/out of scope, open questions), now the document's core. *The pain
  it resolves* and *Success measure* fold into *What this task delivers*; *Processes in scope* becomes
  *Process context*, kept only for a process whose shape actually changes; *Functional detail*, *Data*,
  *Visibility and permissions*, *Reporting* and *Integrations and dependencies* fold into one
  *Cross-cutting details* section; *Scope boundaries* and *Open questions* merge into *Scope & open
  questions* (assumptions plus a session-level table, now with a *Jira key* column).
- **New *Para el equipo de delivery* closing section** — `SKILL.md` and `/seamlex-refinar` already
  described it as the one place platform vocabulary may appear, but the template never actually had it;
  it does now.
- **The *Ready for design* checklist is rewritten** to walk the Requirements table (every row has a
  REQ-ID, a real actor, a testable statement, a source, a MoSCoW and an in/out-of-scope call) and
  *Cross-cutting details*, instead of the old per-section checks.
- `commands/seamlex-refinar.md` §4a and `skills/business-analysis/SKILL.md`'s references table are
  reworded to describe the new shape; `skills/atlassian-how-to/references/jira-task-checklist.md`'s
  stale "the epic template" wording is fixed to "the minuta template".
- Also fixed in this pass, found by a project-wide consistency check: `{{CONF_SCOPE_PAGE}}` was used by
  `/seamlex-refinar` but missing from its own placeholder resolve-list; the undefined `{{INDUSTRY}}`
  token is dropped from `discovery-brief.md`; the *Type* vocabulary in `atlassian-how-to/SKILL.md` is
  now the full, correct 6-value list everywhere it's given; `/hi-seamlex`'s `allowed-tools` now lists
  the Atlassian and `Skill` tools its own steps call; and a stray "epic" reference in
  `unidentified-features.md`'s example row is reworded for the current task-based relevamiento flow.

## 1.18.0

> A relevamiento used to be identified by a configurable Jira issue type or custom field
> (`{{TYPE_RELEVAMIENTO}}`, default `Relevamiento`), with a label-based fallback already documented for
> customers who preferred it. That fallback is now the only way: a relevamiento is a task carrying the
> `relevamiento` Jira label, full stop — no issue type to configure, no custom field, no per-customer
> setting.

- **`TYPE_RELEVAMIENTO` is gone** from the `seamlex-portal-memory` settings table, the placeholder list
  in `atlassian-how-to`, and every command that resolved it. `{{TYPE_TASK}}` (the sub-task type for
  pending items) is unaffected — it stays a real, configurable issue type.
- **Every JQL query that found relevamientos by type now finds them by label**: `labels =
  "relevamiento"` replaces `issuetype = "{{TYPE_RELEVAMIENTO}}"` in `/seamlex-refinar` Step 1 and in the
  `atlassian-how-to` JQL cookbook — unconditionally, since there is no longer a type-vs-label branch to
  choose between. The `getJiraProjectIssueTypesMetadata` fallback that validated a customer's issue-type
  name against their project is dropped with it; a label needs no such lookup.
- **The *Labels — Jira only* table's `relevamiento` row** drops its "when the config marks them by
  label" qualifier — it is now Seamlex's unconditional convention, alongside `pendiente` and
  `no-identificado`.
- `SETUP.md` and the `jira-task-checklist.md` heading are reworded to describe a relevamiento by its
  label rather than a type that no longer exists to check.
- **No fallback for existing engagements**: customers whose relevamiento tickets don't yet carry the
  `relevamiento` label need Seamlex to add it; `/seamlex-refinar` finds nothing for a sprint until they
  do.
- **`epic-template.md` is renamed `minuta-template.md`** and rewritten to drop the contract-epic framing
  it has kept since 1.12.0 removed epic/story generation — it now plainly describes what it has actually
  shaped since then, the `Minuta <KEY> — <task>` page body, including its *Ready for design* checklist
  used by `/seamlex-refinar`'s closing step. The vestigial epic header table (Program, Contract epic,
  Signed scope page…) and the unused *Key user stories* section are gone. `SKILL.md` and
  `/seamlex-refinar` point at the new name.
- **`user-story-template.md` is removed.** The plugin has offered no way to produce a standalone
  Given/When/Then user story since 1.12.0; this drops the last dangling reference and the file itself.
  `SKILL.md`'s references table loses its row.

## 1.17.0

> The Atlassian MCP server cannot set labels on a Confluence page — `createConfluencePage` and
> `updateConfluencePage` have no label parameter — so 1.16.0 promised labels it could never set.
> Confluence labels are gone from the convention altogether: the Jira key lives in the page title and
> every page is found by title and by its place under a parent. Jira labels are unchanged.

- **Page titles carry the Jira key**: `Minuta <KEY> — <exact Jira summary>` and
  `Transcript <KEY> — <summary>` (was `Minuta <summary>` / `Transcript — Minuta <summary>`), the same
  `<Type> <KEY>` shape the delivery team already uses. `title ~ "ABC-12"` finds every page of a task;
  the prefix tells the kinds apart. No fallback for the old titles — spaces created before this
  version are renamed by Seamlex.
- **No Confluence labels.** The *Labels* section of `atlassian-how-to` is now *Labels — Jira only*:
  the Jira table stays (`{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`, `relevamiento`, `pendiente`,
  `no-identificado`, set in `createJiraIssue`'s `additional_fields.labels`); the Confluence table is
  removed, and the skill says plainly that nothing is written, read, promised or checked on a page
  label. The structure rule *Pages carry their labels* becomes *No Confluence labels*; the index page
  loses its *Labels* / *Etiquetas* column; *Before any write* no longer says "labels in the call".
- **Retrieval order is index → key in the title → title prefix under an ancestor → text.** Every CQL
  shape in the skill, the index template and the `business-analysis` skill is by `title` and
  `ancestor`; *the one minuta of a task* is `title ~ "<KEY>" AND title ~ "Minuta"`, then the exact
  title, then the task's `getJiraIssueRemoteIssueLinks`. Same order in `/seamlex-refinar` step 3
  (resumed-session detection) and 4a.
- `references/jira-task-checklist.md` — the *Confluence page* block asks for the keyed title and
  drops the labels line. `references/memory-template.md` — keyed example titles, no label
  column, title/ancestor CQL only. README and SETUP explain the limitation in one paragraph and one
  troubleshooting entry.
- **For the delivery team**: `seamlex-deliver-team` (`/gate-funcional`, `/high-level-design`) finds
  minutas with `label = "minuta" AND label = "<key>"` — it has to switch to `title ~ "<KEY>"`.

## 1.16.0

> The plugin does one thing: the relevamiento. `/seamlex-status` and questions to Seamlex are gone, and
> the house rules for Jira and Confluence are the `atlassian-how-to` skill, loaded by `/hi-seamlex` at
> session start and followed by every Atlassian read and write `/seamlex-refinar` makes.

- **`/seamlex-status` is removed** — no delivery status, no questions to Seamlex. The plugin is
  `/hi-seamlex` (session setup) and `/seamlex-refinar` (the relevamiento). Status and questions live
  with the Seamlex contact and the shared board. `TYPE_QUESTION`, `ESCALATION`, `BOARD_URL` and
  `CADENCE` are no longer read from the config page.
- **`question-template.md` is replaced by `pendiente-template.md`** under
  `skills/business-analysis/references/` — the description of a `Pendiente:` sub-task, shaped for what
  a relevamiento leaves open: where it was raised, owner, needed by, whether it blocks design, what is
  open, why it matters, what is already believed, what was offered, when it is done. `/seamlex-refinar`
  4d and the `business-analysis` *References* table point at it.
- **New skill `skills/atlassian-how-to/SKILL.md`** — how Seamlex uses Jira and Confluence: the
  **structure of a customer's Confluence space** (`seamlex-portal-memory` as root index, the signed
  scope page, the Discovery Brief, `{{CONF_PARENT}}` with one `Minuta <task>` per relevamiento and its
  `Transcript` child, *Procesos*, *Glosario*, *Referencia*, `Features no identificados`, *Delivery*)
  and the rules that keep it that way — one page per task, exact-title naming, the Jira key as a link
  in every header table, children under their parent; the **index page written for retrieval**
  (purpose, settings table on the root, one row per page with exact title, type, labels, what to take
  from it and when to read it, the CQL shapes at the foot) and what breaks it; **what a
  well-completed Jira task looks like**, field by field — summary prefix, type and parent,
  description, status honesty, the result and transcript comments, labels, assignee, the two-way link
  to the Confluence page, dates; the **labelling convention** on both sides (Confluence: `index`,
  `config`, `scope`, `discovery`, `minuta`, `transcript`, `proceso`, `glosario`, `referencia`,
  `no-identificado`, `analisis-funcional`, `hld` and the Jira key lower-cased on every page tied to a
  task; Jira: `{{LABEL_REQUEST}}`, `{{LABELS_EXTRA}}`, `relevamiento`, `pendiente`,
  `no-identificado`); the **retrieval patterns** — index → label → title → ancestor → text — as CQL
  and JQL cookbooks; and the **rules of interaction with the Atlassian tools**: search before any
  write, a place in the tree, title and labels in the call, page first then Jira, re-read what landed,
  walk the checklist before done, stop on failure.
- **Two references under the skill**: `references/memory-template.md` — the root
  `seamlex-portal-memory` with its settings table and a section index, with filled example rows — and
  `references/jira-task-checklist.md` — the *complete task* walk, one block per kind (relevamiento,
  `Pendiente:` sub-task).
- **`/hi-seamlex` gains a step 5**: loads the skill with the `Skill` tool (or reads the file) after the
  config page, so it is in context for every later command. *Where the configuration lives* says the
  config page is the root instance of the skill's index page and is never edited by a command.
- **`/seamlex-refinar` names the skill** as required and loads it if it is not in context. 4a sets
  `minuta` + `<jira-key>` (+ `transcript` on the child) at creation and finds an existing minuta by
  label before title; 4d adds the `pendiente` label; the state decision walks the checklist's
  *Relevamiento* block first.
- `skills/business-analysis/SKILL.md` — *The map* says the config page follows the skill's index
  template; *How to find what the index does not name* points at the label patterns and the full
  cookbook. Nothing changes in how the interview goes.
- README, SETUP and the plugin manifests describe two commands and one workflow; README gains *The house
  rules for Jira and Confluence*.

## 1.15.0

> The templates move under the `business-analysis` skill, as its `references/`, and are used from there.

- **`templates/` is now `skills/business-analysis/references/`** — `epic-template.md`,
  `question-template.md`, `unidentified-features.md`, `discovery-brief.md` and `user-story-template.md`
  live next to the skill that shapes what is written with them.
- **New section *References* in `skills/business-analysis/SKILL.md`** names the directory and what each
  file shapes — the minuta body, the pending items and questions, the `Features no identificados` page,
  the Discovery Brief format the skill reads, the user story.
- Paths are relative to the file that names them — `references/<file>.md` from the skill,
  `../skills/business-analysis/references/<file>.md` (and `../skills/business-analysis/SKILL.md`) from
  the commands — instead of `${CLAUDE_PLUGIN_ROOT}/…`. Nothing changes in what either command writes.

## 1.14.0

> The `business-analysis` skill knows how to read Confluence. The Product Owner opens every session
> already holding the project, its scope and out of scope, and what the last meetings settled.

- **New section *What you know before you ask — Confluence* in `skills/business-analysis/SKILL.md`**:
  the `seamlex-portal-memory` page as the map of the project's knowledge; a table of what to read and what
  to take from each — the signed scope page (what is in, what is written as excluded, assumptions, phase
  boundaries), the Discovery Brief (§1–10, with §9 scope, constraints and non-negotiables), the **last two
  or three meeting notes** (the `Minuta` pages under `{{CONF_PARENT}}` and any meeting-notes page the
  config lists or that turns up by title — `Minuta`, `Meeting notes`, `Acta`, `Reunión`), the process,
  glossary and reference pages, and `Features no identificados`; the CQL shapes to find what the index
  does not name; and what goes into the reflection before the first question.
- *How you interview* → *Never ask what the project already knows* now points at that section instead
  of "the command gathers it; you read it". The skill's frontmatter description names the Confluence
  reading and the scope / out-of-scope knowledge so the skill is picked for them.
- **`/seamlex-refinar` Step 2 points at the skill** for how Confluence is read and keeps only what is
  task-specific — the Jira issue and its epic, the resumed-session check, the transition to In progress.
  Its item 2 names the `Features no identificados` page and the last two or three meeting notes
  explicitly so the two files agree. Nothing changes in what the command writes.

## 1.13.0

> No agents. The Product Owner is the `business-analysis` skill, loaded by `/seamlex-refinar`.

- **`seamlex-product-owner` is now the `business-analysis` skill** — `skills/business-analysis/SKILL.md`
  carries the role, the *speak the customer's language* rule, the technical-term → business-question
  table, *How you interview*, the closing and what is written, unchanged. The agent file is gone.
- **`/seamlex-refinar` loads the skill with the `Skill` tool before Step 1** (or reads the file directly
  when that tool is unavailable) instead of handing the session to a sub-agent resolved by name. Why: a
  test in Cowork ran the 1.12.0 command against a cached 1.11.0 agent — the old "refine contract epics"
  persona with no translation table — and the command could not do what it described. A skill loaded
  in-process by the command cannot drift from it that way.
- **Works from the Chat tab as well as Cowork** — nothing in the plugin runs as a sub-agent any more, so
  SETUP no longer restricts it to Cowork.
- The comment `/seamlex-refinar` leaves on the task (4b) says the relevamiento was run by the Seamlex
  Product Owner (Claude) with the customer, no longer "the Product Owner agent". Nothing else changes in
  behaviour or in what the command writes.

## 1.12.0

> One agent, three commands. The Product Owner agent is behaviour only and `/seamlex-refinar` is the
> workflow; the discovery session leaves the plugin, and the Project Manager is folded into
> `/seamlex-status`, which now also takes questions.

- **`seamlex-product-owner` describes only how the Product Owner behaves**: the goal is the functional
  understanding of the business behind a task, and the customer is spoken to in business language no
  matter how technically the task is written. The *speak the customer's language* rule and its
  technical-term → business-question table move from the command into the agent, so they live in one
  place.
- **The agent's own contract-epic refinement is removed** — no epic list with states, no
  `Epic — <title>` pages, no *Ready for design* gate, no Jira epic + stories offer. `/seamlex-refinar`
  is the only way the agent runs; the command decides what task is worked on and what is left behind.
- `/seamlex-refinar` Step 3 no longer restates the interview guidance; it points at the agent and keeps
  only where the interview's inputs and outputs connect to the other steps. `{{DETAIL}}` is now listed
  among the settings the command resolves.
- README and the hand-off from the Project Manager agent describe the relevamiento of the sprint
  instead of contract-epic refinement. `templates/user-story-template.md` is no longer referenced.
- **The discovery session is removed from the plugin** — `seamlex-discovery-agent` and
  `/seamlex-discovery` are gone. The Discovery Brief is prepared with your Seamlex consultant and
  published to Confluence; the Product Owner and Project Manager keep reading it from there (and from
  `seamlex/discovery/discovery-brief.md` when a local copy exists). `templates/discovery-brief.md` stays
  as the document's format.
- **`seamlex-project-manager` is folded into `/seamlex-status`** — the agent file is gone; its role,
  configuration step, operating principles and both modes now live in the command.
- **`/seamlex-ask` is removed.** Questions to Seamlex go through `/seamlex-status`: a question in its
  arguments runs Mode B — searched on the board and in Confluence first, filed as a tracked question
  only when the team's answer is needed.

## 1.11.0

> `/seamlex-refinar` becomes a *relevamiento* of the current sprint, run by the Product Owner agent, in
> business language only, and leaves five things behind on Jira and Confluence.

- **`/seamlex-refinar` is executed by the `seamlex-product-owner` agent** and no longer follows the plan
  by date: it queries the **open sprint** for tasks of type `{{TYPE_RELEVAMIENTO}}` (default
  `Relevamiento`; a label when the config says so), takes the only one or asks which to run, and
  `/seamlex-refinar <KEY>` goes straight to a task.
- **Business language is the rule that overrides everything.** However technical the scope item — record
  type, trigger, LWC, integration, data model — none of it reaches the customer; the translation table
  grows and applies even at `{{DETAIL}}` = `technical`. Platform vocabulary is allowed only in the
  minuta's *Para el equipo de delivery* section.
- **The `seamlex-portal-memory` page is used as the index of the project's knowledge**: before the first
  question the agent reads the task and its epic, every page the config links (scope, Discovery Brief,
  process and reference pages), searches Confluence and Jira for related material, and reflects it back in
  business terms. The interview aims at the functional understanding of the business around the task.
- **Five outputs per session**: a comment on the task saying the Product Owner agent ran the
  relevamiento; the verbatim conversation **transcript** as a dedicated comment (or a child page when too
  long — the MCP server has no file upload); the pending items as `{{TYPE_TASK}}` issues (sub-tasks under
  the relevamiento by default); a Confluence page titled **`Minuta <task summary>`** under
  `{{CONF_PARENT}}`; and the Jira task **linked to the page** in both directions (page URL in the comment,
  issue link in the minuta header, checked with `getJiraIssueRemoteIssueLinks`).
- README describes the command and the relevamiento outputs.

## 1.10.0

> The configuration moves out of the plugin and into Confluence. `/hi-seamlex` loads the customer's
> `seamlex-portal-memory` page, and every command and agent reads its settings from that page.

- **`/hi-seamlex` is now a five-step session start**: authenticate to Atlassian, choose the Confluence
  space (asking only when more than one is visible), find the `seamlex-portal-memory` page in it, load the
  page into the session, and close with a two-line "setup finished, welcome to the Seamlex portal". It
  writes nothing, creates nothing, and no longer echoes the page's contents back.
- **The Configuration section of `commands/hi-seamlex.md` is gone.** The tables that shipped with the
  plugin — Atlassian workspace, issue types, Seamlex contacts, agent behaviour — and the *Who you are*
  rows all live on the customer's `seamlex-portal-memory` page now. Changing a setting is an edit to the
  page, not a plugin release.
- **Every agent and command resolves its `{{PLACEHOLDER}}` tokens from the loaded page** and stops with
  "run `/hi-seamlex` first" when the page is not in the session. `{{CLOUD_ID}}` and `{{CONF_SPACE}}` come
  from the cloud id and space `/hi-seamlex` settled on; `{{LOCALE}}` and `{{USER_NAME}}` from
  `atlassianUserInfo`; `{{COMPANY}}`, `{{PROGRAM}}` and `{{INDUSTRY}}` from the page when it names them,
  otherwise from the signed scope page title and the Discovery Brief. `{{MY_ROLE}}` is removed.
- `/hi-seamlex setup` no longer exists as a separate mode; every reference now points at `/hi-seamlex`.
- README, SETUP and the discovery-brief template describe the Confluence page as the source of settings;
  SETUP's troubleshooting covers a missing `seamlex-portal-memory` page.

## 1.8.0

> `/refine-project-scope` is replaced by `/seamlex-refinar`. Refinement now follows the Jira plan, one task
> per session, and leaves three things behind: a Confluence page, the pending items as sub-tasks, and the
> task's state.

- **`/seamlex-refinar` replaces `/refine-project-scope`** — the session no longer opens with the contract
  epic list. It reads the Jira plan as of today's date and takes the first open task in plan order (start
  date, then due date, then rank; sub-tasks never), shows the pick and the two behind it, and asks the
  customer to confirm. `/seamlex-refinar <KEY>` skips the plan order.
  - **The task is read before the first question** — description, comments, sub-tasks, links, parent
    epic, any existing refinement page — and reflected back in a few lines, so the session spends its time
    on what Jira does not already settle. A task still in its initial state is moved to *In progress* when
    the session starts.
  - **Consultative, in the customer's language** — Salesforce terms in the task ("record type", "validation
    rule", "flow", "permission set"…) are never put to the customer as questions. The command carries a
    translation table: each platform term maps to the business question that actually gets the answer.
  - **Three outputs, always** — one Confluence page `Relevamiento — <KEY> — <summary>` under
    `{{CONF_PARENT}}` with the functional result of the conversation, saved as the session runs and linked
    from the task with a comment; every open item as a sub-task under the task (type read from the project
    metadata, `{{TYPE_QUESTION}}` under an epic that cannot carry sub-tasks), de-duplicated against what
    already hangs there; and the task transitioned to done only after an explicit approval of the summary
    with no blocking pending item — otherwise it stays in progress and the page says so.
  - The parking rule is unchanged: what belongs to another task is noted against that key; what belongs to
    no task goes to `Features no identificados`.
- The lifecycle step `refinement` in `/hi-seamlex` now hands off to `/seamlex-refinar`.

## 1.7.0

> Refinement now starts from the signed contract scope. `/seamlex-request` is gone; `/refine-project-scope`
> takes its place, and two agents are renamed. See the migration note at the foot of this entry.

- **`/seamlex-request` is replaced by `/refine-project-scope`** — the plugin no longer starts from a blank
  requirement. The new command works from the signed contract scope page (`{{CONF_SCOPE_PAGE}}`) and
  refines the epics that were actually contracted, one at a time.
  - **Every session opens with the epic list** — every epic on the contract page, numbered, with the state
    of its functional refinement in parentheses: `sin comenzar`, `en progreso`, `finalizado`. Nothing
    records the state; it is read each time from the evidence — the published `Functional understanding`
    pages in `{{CONF_SPACE}}`, the local drafts, and their open `⚠️ TBD` items.
  - **Features no identificados** — anything the customer raises that fits no contract epic is parked in
    `{{DRAFTS_DIR}}/scope/unidentified-features.md` with its context and impact, told plainly to be outside
    the signed scope, and routed to `{{SEAMLEX_CONTACT}}`. It is never folded into the epic at hand and
    never dropped.
  - **One page per epic** — when an epic is refined, the agent shows a summary of what it understood, and
    only after an explicit yes publishes a `Functional understanding — <epic> — {{COMPANY}}` page to
    `{{CONF_SPACE}}`, updating that same page on a re-refinement. Raising the epic and its stories in Jira
    is now a separate, optional step after the page.
  - **Refinement drafts live in Confluence, not in the workspace** — an epic's page is created marked
    `Draft` at the start of its refinement and updated as the session runs, so a session can be picked up
    from the page itself and the customer can read it between sessions. The Status row is the state:
    `Draft` reads as `en progreso`, `Reviewed by customer` as `finalizado`. `{{DRAFTS_DIR}}/requests/` is
    gone and no `scope/` folder replaces it — the workspace now holds the discovery brief and nothing else.
  - **One epic artefact, traceable end to end** — `epic-template.md` and the short-lived
    `functional-understanding.md` are merged into a single epic page. It is titled `Epic — <exact contract
    epic title>`, names the contract item and links the signed scope page in its header, and carries the
    Jira key once the epic is raised: the trace runs contract → page → Jira, with the same title
    throughout. The Jira description links back to the page rather than duplicating it.
  - **`Ready for design` is the definition of done** — the epic page ends with a checklist (actors, process
    delta, rules, statuses, notifications, failure paths, data and validation, visibility, reporting,
    integrations, scope boundaries, stories with acceptance criteria, no blocking `⚠️ TBD`, customer
    approval). An epic only counts as `finalizado` when its Status says `Ready for design` and that
    checklist is fully ticked, so refinement carries enough detail to be designed and implemented without
    coming back to the customer.
  - `unidentified-features.md` is a new template; the lifecycle step `requirement` is now `refinement`
    (`/hi-seamlex requirement` still works).
- **Agents renamed** — `seamlex-discovery` is now `seamlex-discovery-agent`, so the agent no longer shares
  a name with the `/seamlex-discovery` command. Behaviour is unchanged, and the command is unchanged.
- **`seamlex-delivery-liaison` is now `seamlex-project-manager`** — same agent, same two modes (status from
  the live board, and questions to the Seamlex team); the name says what the customer actually deals with.
  `/seamlex-status` and `/seamlex-ask` are unchanged.

- **Discovery states its objective, and has two new sections** — the agent now opens by naming the seven
  things the session must answer: company and business model, industry and business context, organization
  structure (the areas and roles that make sense for the project), the business processes the project
  touches, the actors who will use the system, the pains, and the goals and expectations. Two sections were
  added to carry the new ones — **§3 Organization structure — areas and roles** and **§4 Business processes
  related to the project** — and the Governance and ways of working section was dropped, so discovery is
  now ten sections, seventy to ninety minutes. Actors (§6) are grounded in those areas and process steps,
  and goals (§8) capture the customer's spoken expectations alongside the measurable outcomes. The
  Discovery Brief template follows: matching sections in, governance out.
- **Discovery never starts over** — a new step 0b looks for work already done in both places: the local
  draft and the published Discovery Brief page in `{{CONF_SPACE}}`, found by title. If only the page
  exists, its content is read back into the local draft and the session continues from there; if both
  exist, the more complete one is the base and the differences are put to the customer rather than
  silently overwritten. Publishing a resumed session updates that same page instead of creating a second.
- **Signed scope page in the config** — a new `{{CONF_SCOPE_PAGE}}` row under **Atlassian workspace** points
  at the Confluence page holding the detail of the scope the customer signed, by page ID, the same way
  `{{CONF_PARENT}}` does. `/hi-seamlex setup` checks it is reachable alongside the other Confluence values.

### Migrating from 1.5.x

- `/seamlex-request` no longer exists. Use `/refine-project-scope`; it needs `{{CONF_SCOPE_PAGE}}` to point
  at the signed scope page, which `/hi-seamlex setup` verifies.
- Anything left in `seamlex/requests/` is not read by the plugin any more. Keep the folder if you want the
  history; refinement drafts now live in `{{CONF_SPACE}}` as `Epic — <contract epic title>` pages.
- Agents renamed: `seamlex-discovery` → `seamlex-discovery-agent`, `seamlex-delivery-liaison` →
  `seamlex-project-manager`. If you call an agent by name in your own notes or hooks, update it. Every
  command is unchanged apart from the one replacement above.

## 1.6.0

Never shipped. Cut on 2026-09-02 and reverted the same day; the version number was retired rather than
reused, so 1.5.0 is followed by 1.7.0.

## 1.5.0

- **One entry point** — `/seamlex-setup` is gone; `/hi-seamlex` absorbs it. The first run of `/hi-seamlex`
  in a workspace finds no `state.md`, runs the connection check, the config verification against the live
  site and the folder creation itself, then carries straight on into discovery. `/hi-seamlex setup` re-runs
  just those checks.
- **No lifecycle state file** — `seamlex/state.md` and its template are gone. `/hi-seamlex` infers the
  step each session from the signals that cannot go stale: whether the workspace folders exist, whether the
  Discovery Brief is complete, and whether labelled work is moving on the board. It says which signals it
  read, and `/hi-seamlex <step>` overrides it. The plugin now writes nothing to the workspace but two empty
  draft folders.
- **One config, in one place** — `config/seamlex.config.md` is deleted. The full table, with its notes and
  section headings, now lives in the **Configuration** section of `commands/hi-seamlex.md`, and every
  command and agent resolves its placeholders from there. Nothing to keep in sync, and nothing to fail when
  a command is loaded without its sibling files.

## 1.4.2

- **`/hi-seamlex` no longer depends on a sibling file** — some environments load a command on its own,
  without `config/` next to it, and the command was reporting a fixed config as a missing one and sending
  the customer to `/seamlex-setup`. The values are now inlined in the command under **Configuration**, used
  whenever `config/seamlex.config.md` is not reachable. The config file stays the source of truth; edit it
  and copy the change across.

## 1.4.1

- **`/hi-seamlex` opens with the config** — step 1 reads `config/seamlex.config.md` and summarizes it back
  as the very first message, before `state.md`, before any Atlassian call, before any other output. If the
  file cannot be read it says so and stops, rather than carrying on.

## 1.4.0

- **Fixed configuration** — the config is no longer generated. It ships with the plugin at
  `config/seamlex.config.md` and every command and agent reads it there, read-only. Nothing is written to
  `seamlex/config.md`, and the setup interview is gone.
- **`/seamlex-setup` verifies instead of asking** — it shows what the plugin is configured for, checks the
  Jira project, Confluence space and issue type names against the live site, runs a read-only probe, and
  creates the local working folders. A wrong value is reported as a mismatch to fix in the plugin, not
  patched per workspace.
- **No Confluence config page** — the shared-config flow (the `seamlex-portal-config` label, adoption,
  publishing, the `.local.bak` backup) is removed. There is nothing to share; everyone runs the same
  shipped config.
- **Lifecycle state moved local** — the step lives in `seamlex/state.md`, from
  `templates/state.md`, with a history table. `/hi-seamlex` reads and updates it, still only with approval,
  and still checks it against the brief, the drafts and the board.
- **Templates** — `seamlex.config.template.md` and `seamlex.config.example.md` are gone; there is nothing
  left to template.

## 1.3.0

- **`/hi-seamlex`** — one command to open a session. It refreshes the config from the shared
  Confluence page, works out which of the four lifecycle steps the company is on, loads the context that
  step needs, and hands off to the matching command.
- **Lifecycle state** — the config gains §6, recording the current step, when it changed and why. It lives
  on the same company-wide page, so where the engagement got to is a shared fact rather than something each
  person reconstructs.
- **Evidence over record** — a step recorded in §6 is checked against the brief, the drafts and the board;
  when they disagree the customer is shown both and asked, and a step change is written back only with
  approval.

## 1.2.0

- **Shared config** — the configuration is now company-wide. `/seamlex-setup` searches Confluence for a
  page labelled `seamlex-portal-config` before creating anything and adopts it when found, so the second
  person at a company answers almost nothing and everyone's settings agree.
- **Publishing** — at the end of setup, the config is published (or updated) on that page, with the
  customer's approval, as a verbatim markdown block that round-trips cleanly.
- **Conflicts** — the Confluence page is the source of truth; a differing local file is shown as a diff,
  backed up to `seamlex/config.md.local.bak`, and replaced.
- **Offline** — with Atlassian unavailable, setup still works from the template and says the shared config
  was neither read nor written.

## 1.1.0

- **Discovery** — the agent now opens a fresh session with an overview of the three blocks of nine
  sections and the 60-90 minute shape, so the customer knows what they are signing up for.
- **Solution Domains handoff** — after publishing the brief, discovery checks Confluence for a
  Solution Domains page and points the customer at their Seamlex consultant when it is missing.
- **Seamlex-owned config** — Seamlex contact, escalation, board URL and sprint cadence are marked as
  filled in by Seamlex: `/seamlex-setup` no longer asks for them, and the liaison and product-owner
  agents treat them as optional.
- **Docs** — `SETUP.md` install instructions.

## 1.0.0

Initial release.

- **Agents** — `seamlex-discovery` (nine-section discovery session, resumable, publishes a Discovery Brief
  to Confluence), `seamlex-product-owner` (requirement → epic + key user stories, raised in Jira),
  `seamlex-delivery-liaison` (live status from the board, and questions to the Seamlex team).
- **Commands** — `/seamlex-setup`, `/seamlex-discovery`, `/seamlex-request`, `/seamlex-ask`,
  `/seamlex-status`.
- **Templates** — discovery brief, epic, user story, question.
- **Atlassian MCP** — official server bundled via `.mcp.json`; customer authenticates in their own browser.
- **Config** — single `seamlex/config.md` per workspace, discovered and filled by `/seamlex-setup`.
