# Seamlex Portal

Your direct line to the Seamlex delivery team, inside Claude.

Seamlex is your Salesforce implementation partner. This plugin puts the parts of that relationship you
touch most often — the *relevamiento* of each task in your sprint, in your own business language — into
the Claude you already use. Everything it produces lands in the Jira project and
Confluence space you share with Seamlex, so there is one record and no parallel inbox.

## What you can do

| Command | What it does |
|---|---|
| `/hi-seamlex` | Start a session. **Run this first.** Signs you in to Atlassian, picks your Confluence space, loads the project's `seamlex-portal-memory` page and the house rules for Jira and Confluence. |
| `/seamlex-refinar` | Run a *relevamiento* with the Product Owner — takes a relevamiento task from the current sprint, gathers everything the client config points at, interviews you in business language only, and leaves a comment and the transcript on the task, a Confluence page "Minuta <KEY> — <task>", the pending items as sub-tasks of the task, and the task linked to the page. |

Two commands, one workflow: `/hi-seamlex` sets the session up, `/seamlex-refinar` does the work. Status
reports and questions to the Seamlex team are not part of this plugin — your Seamlex contact and the
shared Jira board are where those live.

## The Product Owner

`/seamlex-refinar` plays the **Seamlex Product Owner**, through the plugin's `business-analysis` skill —
the interview method it loads before asking anything. It runs the *relevamientos* of your current sprint
one task at a time, reads everything the project already knows about it — the task, its epic, the
signed scope page and what it leaves out, the Discovery Brief Seamlex prepared with you, the last few
minutas and meeting notes — and then interviews you about your operation: the people, steps,
decisions, information and rules behind what the task asks for. However technically the task is written —
record types, flows, integrations — none of that vocabulary reaches you; the questions are about how a
shipment comes back damaged and who decides, not about which fields go where. Seamlex maps the answers to
the platform afterwards. Each session leaves a Confluence minuta, a comment and the verbatim transcript
on the task, and the open points as sub-tasks with an owner. Anything you raise that belongs to no task
in the sprint is parked as an **unidentified feature** rather than quietly absorbed — Seamlex decides
what happens to it.

## The house rules for Jira and Confluence

Everything the commands read and write in Jira and Confluence follows one set of conventions — the
plugin's `atlassian-how-to` skill, which `/hi-seamlex` loads at the start of each session. It fixes the
structure of your Confluence space, the shape of an index page written so Claude can find things later
(your `seamlex-portal-memory` page is the root one), what a well-completed Jira task looks like — summary,
description, status, comments, labels, linked to its Confluence page — the Jira labels, and the queries
used to find pages by title and tasks by key. You do not have to know any of it; it is what makes a
minuta from last month findable from the task it belongs to.

One detail is worth knowing: the official Atlassian MCP server can set labels on a Jira issue but
**not on a Confluence page**, so the plugin does not use Confluence labels at all. Every page it
creates carries its Jira key in the title (`Minuta ABC-12 — …`), and that title is how the page is
found — from the task, from the index, from a search.

## Nothing happens without your approval

`/seamlex-refinar` drafts locally and shows you the result before anything is written to Jira or
Confluence. You approve the exact page, comment or pending sub-task — or you don't, and it stays a draft.

Your Atlassian credentials never pass through Seamlex or this plugin. You sign in to Atlassian yourself,
in your own browser, through the official Atlassian MCP server.

## Your workspace

```
seamlex/
└── discovery/
    └── discovery-brief.md      # a local copy of your Discovery Brief, if you keep one
```

Relevamientos leave nothing here: each one gets its own Confluence page — `Minuta <KEY> — <task>`, created on the
first session, updated as you go, marked `Finalizado` when you approve it — plus the transcript and a
comment on the Jira task, and so does the list of unidentified features.

There is no configuration to fill in. The engagement settings — Jira project, issue types, the signed
scope page, who to reach at Seamlex — live on a `seamlex-portal-memory` page in your Confluence space,
maintained by Seamlex; `/hi-seamlex` loads it at the start of each session and `/seamlex-refinar` reads
its settings from there. Who *you* are — the language you work in — comes from the Atlassian account you sign
in with; your company and program from the config page, the signed scope page or your Discovery Brief.
Your workspace holds only your discovery notes.

## Install

See [SETUP.md](SETUP.md). Two commands and a browser sign-in.

---

Built and maintained by Seamlex. Questions about the plugin itself go to your Seamlex delivery contact.
