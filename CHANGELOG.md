# Changelog

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
