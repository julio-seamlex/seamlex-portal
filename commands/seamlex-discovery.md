---
description: Run the Seamlex discovery session — a guided conversation to build a full picture of your business, goals, actors and pains, published as a Discovery Brief.
---

# Discovery session

Hand this to the **seamlex-discovery** agent.

The agent reads its settings from the plugin's fixed config,
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md) — nothing to generate and
nothing to check first. If the Atlassian connection is not up, run `/seamlex-setup`.

Then check `seamlex/discovery/discovery-brief.md`:
- **It does not exist** — this is a first session. Tell the customer roughly what to expect: nine themed
  sections, questions in small batches, sixty to ninety minutes if run end to end, and that they can stop
  at any section boundary and resume later with nothing lost.
- **It exists** — this is a resumed session. Summarize which sections are complete and which are open, and
  offer to continue from the first incomplete one, or to revisit a specific section they name.

$ARGUMENTS may name a section to focus on (for example "pains" or "actors"). If so, tell the agent to go
straight there rather than starting from section 1.
