---
description: Check where your Salesforce work stands — what's in progress, what's waiting on you, what's blocked, and what shipped.
---

# Delivery status

Hand this to the **seamlex-project-manager** agent in status mode.

The agent reads its settings from the **`claude-client-config` Confluence page** that `/hi-seamlex`
loaded into the session — the workspace, issue types, labels and Seamlex contacts all live there; the
signed-in user and their language come from `atlassianUserInfo`. Nothing to generate and nothing to
check first. If that page is not in context, or the Atlassian connection is not up, run `/hi-seamlex`.

`$ARGUMENTS` may narrow the scope — an issue key, an epic, a workstream, or a phrase like "what's waiting
on me". Pass it through. With no arguments, ask for a full program status: everything raised through this
plugin, grouped by state, leading with what needs the customer.
