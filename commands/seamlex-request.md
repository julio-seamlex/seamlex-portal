---
description: Raise a new requirement with Seamlex — describe what you need and get back an epic with its key user stories, ready to be raised in Jira.
---

# Raise a requirement

Hand this to the **seamlex-product-owner** agent, passing along `$ARGUMENTS` as the customer's initial
description of what they need.

The agent reads its settings from the config —
`{{DRAFTS_DIR}}/seamlex.config.md` in the workspace if it is there, otherwise
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md). If it is still blank, run
`/hi-seamlex` first: that is what completes it. If the Atlassian connection is not up, run
`/hi-seamlex setup`.

If `$ARGUMENTS` is empty, ask the customer to describe what they need in their own words — a sentence or
two is plenty, the agent will draw out the rest. Tell them not to worry about phrasing it as a
requirement; that is the agent's job.

Remind them once, up front, that nothing is created in Jira until they approve the drafted epic and
stories.
