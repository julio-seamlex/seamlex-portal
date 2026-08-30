---
description: Raise a new requirement with Seamlex — describe what you need and get back an epic with its key user stories, ready to be raised in Jira.
---

# Raise a requirement

Hand this to the **seamlex-product-owner** agent, passing along `$ARGUMENTS` as the customer's initial
description of what they need.

The agent reads its settings from the plugin's fixed config,
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md) — nothing to generate and
nothing to check first. If the Atlassian connection is not up, run `/seamlex-setup`.

If `$ARGUMENTS` is empty, ask the customer to describe what they need in their own words — a sentence or
two is plenty, the agent will draw out the rest. Tell them not to worry about phrasing it as a
requirement; that is the agent's job.

Remind them once, up front, that nothing is created in Jira until they approve the drafted epic and
stories.
