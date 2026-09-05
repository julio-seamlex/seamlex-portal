---
description: Refine the contract scope — walk the epics you signed for, one at a time, and detail each on its own Confluence page until it is ready to be designed and built. Opens with every epic and where its refinement stands.
---

# Refine project scope

Hand this to the **seamlex-product-owner** agent. The agent works **from the signed contract scope**, not
from a blank page: the epics to be delivered are already agreed, and the job is to refine each one until
the delivery team knows what it means.

It reads its settings from the plugin's fixed config, the **Configuration** section of
[`commands/hi-seamlex.md`](${CLAUDE_PLUGIN_ROOT}/commands/hi-seamlex.md) — nothing to generate and nothing
to check first. If the Atlassian connection is not up, run `/hi-seamlex setup`.

## Always open with the epic list

**Every time this command starts**, before anything else, read the signed scope page `{{CONF_SCOPE_PAGE}}`
in `{{CONF_SPACE}}` with `getConfluencePage` and show **every epic in the contract as a numbered list**,
each with the state of its functional refinement in parentheses:

```
1. Quoting and approvals            (finalizado)
2. Partner onboarding portal        (en progreso)
3. Renewals and churn alerts        (sin comenzar)
...
Features no identificados (2)
```

**One page per contract epic, in Confluence, not on the customer's disk.** The page is titled
`Epic — <exact contract epic title>` — that exact title is the trace back to what was signed. It is created
as a draft when refinement starts, grows as the sessions run, and reaches `Ready for design` when it holds
everything an architect and a developer need. Nothing about refinement is written to the workspace.

The state is never recorded as such — read it off that page, per epic:
- **finalizado** — the epic's page exists, its **Status** row says `Ready for design`, its *Ready for
  design* checklist is fully ticked, and no open `⚠️ TBD` blocks design.
- **en progreso** — the page exists but is still `Draft`, or the checklist and its `⚠️ TBD` items are not
  closed.
- **sin comenzar** — no page for that epic.

Close the list with the count of entries on the `Features no identificados` page in `{{CONF_SPACE}}`, so
nothing parked there is forgotten. Say which signals you read, so the customer can correct you in one
line.

If the scope page cannot be read — no Atlassian connection, or `{{CONF_SCOPE_PAGE}}` unset or wrong — say
so plainly and stop rather than inventing an epic list. The contract is the whole basis of this command.

## Then refine one epic

Ask which epic to take, defaulting to the first that is not `finalizado`. `$ARGUMENTS` may name one
directly — a number from the list, or words from its title — in which case go straight there.

Refine **one epic at a time**. Requirements the customer raises that do not belong to any epic in the
contract go to **features no identificados** — never silently folded into the epic at hand, and never
dropped. They are out of the signed scope until Seamlex says otherwise.

The agent keeps the epic's Confluence page up to date as it goes. When the refinement is done it shows a
summary of what it understood, walks the *Ready for design* checklist, and — after explicit approval —
marks the page `Ready for design`. Raising the epic and its stories in Jira is a separate, optional step
that links back to the page. Then it offers the next epic from the list.
