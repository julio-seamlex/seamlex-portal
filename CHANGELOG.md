# Changelog

## 1.6.0

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
