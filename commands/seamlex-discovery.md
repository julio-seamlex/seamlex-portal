---
description: Run the Seamlex discovery session — a guided conversation to build a full picture of your business, goals, actors and pains, published as a Discovery Brief.
---

# Discovery session

Hand this to the **seamlex-discovery-agent**.

The agent reads its settings from the **`claude-client-config` Confluence page** that `/hi-seamlex`
loaded into the session — the workspace, issue types, labels and Seamlex contacts all live there; the
signed-in user and their language come from `atlassianUserInfo`. Nothing to generate and nothing to
check first. If that page is not in context, or the Atlassian connection is not up, run `/hi-seamlex`.

The session exists to answer seven things about the customer: their company and business model, their
industry and business context, the organization structure — the areas and roles that matter to this
project, the business processes the project touches, the actors who will use the system, the pains, and
the goals and expectations for the project. Everything else the agent asks serves those.

**Never start over.** Look for work already done, in both places:
- `seamlex/discovery/discovery-brief.md` — the local draft.
- The published Discovery Brief page in the Confluence space from the config, found by title. If the page
  exists but the local draft does not, the agent reads the page back into the local draft and continues
  from there — and publishes back to that same page rather than creating a second one.

Then:
- **Nothing found** — this is a first session. Tell the customer roughly what to expect: ten themed
  sections, questions in small batches, seventy to ninety minutes if run end to end, and that they
  can stop at any section boundary and resume later with nothing lost.
- **Something found** — this is a resumed session. Summarize which sections are complete and which are
  open, and offer to continue from the first incomplete one, or to revisit a specific section they name.

$ARGUMENTS may name a section to focus on (for example "pains", "actors", "processes" or "org"). If so,
tell the agent to go straight there rather than starting from section 1.
