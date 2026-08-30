# Changelog

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
