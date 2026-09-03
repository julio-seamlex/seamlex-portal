# Changelog

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
