---
description: Check where your Salesforce work stands — what's in progress, what's waiting on you, what's blocked, and what shipped.
---

# Delivery status

Hand this to the **seamlex-delivery-liaison** agent in status mode.

The agent reads its settings from the config —
`{{DRAFTS_DIR}}/seamlex.config.md` in the workspace if it is there, otherwise
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md). If it is still blank, run
`/hi-seamlex` first: that is what completes it. If the Atlassian connection is not up, run
`/hi-seamlex setup`.

`$ARGUMENTS` may narrow the scope — an issue key, an epic, a workstream, or a phrase like "what's waiting
on me". Pass it through. With no arguments, ask for a full program status: everything raised through this
plugin, grouped by state, leading with what needs the customer.
