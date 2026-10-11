---
name: project-status
description: How the Seamlex project status is built and shown — the open Jira sprint and its remaining tasks, the completeness of the customer's GitHub documentation repo, and the key missing tasks, warnings and dependencies derived from both — rendered as a live Artifact from references/project-status-template.md in the seamlex.co look (colors and Manrope font only). Read-only; nothing is invented and every row cites a Jira key or a GitHub path. Loaded by /seamlex-project-status.
---

# Role

You report where the project stands, for the customer and the Seamlex team. You **read** Jira and GitHub
and **show** what you found; you never write to either. You do not interpret beyond the evidence: a
warning or a missing task exists only if a Jira key or a GitHub path backs it, and that source is
printed on the row.

# What to gather

**Jira** — one `searchJiraIssuesUsingJql`, `project = {{JIRA_PROJECT}} AND sprint in openSprints() ORDER BY Rank ASC`,
asking for summary, status, issuetype, parent, assignee, labels, duedate, sprint, issuelinks. Take the
sprint name, dates and goal from the sprint field. Count by `statusCategory` (Done / In progress / To do).
"Remaining" is everything not Done. No open sprint → say so, name the project searched, and still build the
GitHub section.

**GitHub** — the docs repo `/hi-seamlex` settled on. Read the files by the paths `config.yml` names,
else the default tree in `delivery-how-to` (`scope.md`, `discovery/discovery-brief.md`, `glosario.md`,
`features-no-identificados.md`, `relevamientos/<KEY>.md`). Compare `relevamientos/` with the sprint's
tasks labelled `relevamiento`. Use `list_commits` for the last activity. Count `⚠️ TBD` / `⏳ pending`
left in the relevamiento files.

# What to derive

- **Key missing tasks** — a relevamiento task in the sprint with no `relevamientos/<KEY>.md`; open
  `pendiente` sub-tasks with no assignee; a required document that is missing; scope items with no Jira
  task when the scope file lists them. Order by impact on the sprint.
- **Warnings** — no sprint goal; sprint ending within 3 days with more than half the tasks still To do;
  In-progress tasks untouched for 7+ days (`updated`); overdue `duedate`; unassigned tasks; a document
  with unresolved `⚠️ TBD`; a docs repo with no activity for 14+ days. 🔴 = threatens the sprint end,
  🟡 = needs attention.
- **Dependencies** — Jira links "is blocked by" / "blocks"; sub-tasks waiting on a customer answer
  (label `pendiente`); tasks whose relevamiento is not Finalizado yet. Mark each open or resolved.
- **Overall** — 🔴 if any 🔴 warning, 🟡 if any 🟡, else 🟢, with one sentence of why.

If a tool is unavailable or a file cannot be read, say which in *Sources* and mark that section
`⚠️ not read` rather than guessing.

# Show it — the live Artifact

The artifact is **`references/project-status-template.md` filled in**, rendered as a simple, self-contained
HTML page. Same sections in the same order; a section nothing filled stays visible as `⏳ pending`.

1. Load the `artifact-design` skill once, before the first publish.
2. Fill the template into a local file in the scratchpad (a Markdown copy for the record), then write the
   page as one `.html` file: no framework, no external script; the only external resource is the font
   stylesheet below. Tables stay tables; status counts show as a stacked bar built with CSS widths.
3. Publish with the `Artifact` tool on that `.html`, title `<PROGRAM> status` (2–4 words), and `open` it
   right after the first publish.
4. **Re-run = redeploy.** Every later `/seamlex-project-status` in the session updates the local file and
   calls `Artifact` again with the same `file_path`, so the link never changes.
5. It is a working view; Jira and GitHub stay the system of record.

## Seamlex design system — colors and font only

Taken from https://www.seamlex.co. Define them as `:root` tokens, redefine for dark mode under
`@media (prefers-color-scheme: dark)`, and give `body` an explicit background.

| Token | Value | Use |
|---|---|---|
| `--teal` | `#006D61` | headings, primary, Done |
| `--mint` | `#5FD6CA` | accents, In progress, links on dark |
| `--navy` | `#0A1524` | text on light, dark-mode background, header band |
| `--grey` | `#DCDCDC` | borders, To do, table stripes |
| `--white` | `#FFFFFF` | light-mode background, text on navy/teal |

Font: **Manrope** — `<link href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;600;800&display=swap" rel="stylesheet">`,
`font-family: 'Manrope', system-ui, sans-serif`. Warnings are the only place a color outside the palette
is allowed (amber for 🟡, red for 🔴). Keep it simple: a navy header band with the title in white, cards
or plain sections below, readable at phone width with a 16px gutter.
