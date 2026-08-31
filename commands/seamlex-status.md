---
description: Check where your Salesforce work stands — what's in progress, what's waiting on you, what's blocked, and what shipped.
---

# Delivery status

Hand this to the **seamlex-delivery-liaison** agent in status mode.

The agent reads its settings from the plugin's fixed config, the **Configuration** section of
[`commands/hi-seamlex.md`](${CLAUDE_PLUGIN_ROOT}/commands/hi-seamlex.md) — nothing to generate and
nothing to check first. If the Atlassian connection is not up, run `/hi-seamlex setup`.

`$ARGUMENTS` may narrow the scope — an issue key, an epic, a workstream, or a phrase like "what's waiting
on me". Pass it through. With no arguments, ask for a full program status: everything raised through this
plugin, grouped by state, leading with what needs the customer.
