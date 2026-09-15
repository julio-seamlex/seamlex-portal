# Seamlex Portal

Your direct line to the Seamlex delivery team, inside Claude.

Seamlex is your Salesforce implementation partner. This plugin puts the parts of that relationship you
touch most often — framing what your business needs, raising requirements, asking questions, checking
where work stands — into the Claude you already use. Everything it produces lands in the Jira project and
Confluence space you share with Seamlex, so there is one record and no parallel inbox.

## What you can do

| Command | What it does |
|---|---|
| `/hi-seamlex` | Start a session. **Run this first.** Sets the workspace up, then works out where you are in the lifecycle and loads the context for it. |
| `/seamlex-refinar` | Run a *relevamiento* with the Product Owner — takes a relevamiento task from the current sprint, gathers everything the client config points at, interviews you in business language only, and leaves a comment and the transcript on the task, a Confluence page "Minuta <task>", the pending items as sub-tasks of the task, and the task linked to the page. |
| `/seamlex-status` | See what's in progress, what's waiting on you, and what's blocked — or ask the Seamlex team a question. |

`/hi-seamlex` is the one to reach for when you are not sure what to run: it works out which step you are
on — setup, scope refinement or status — pulls down just that step's context, and points you at the
right command.

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

`/seamlex-status` plays the **Seamlex Project Manager**. It answers "where is my request" from the live board rather than from memory, tells you
plainly what is stale or blocked, and leads with what is waiting on you. It also handles questions:
searching Jira and Confluence for an existing answer first, and filing a tracked question when it
genuinely needs the team.

## Nothing happens without your approval

Every command drafts locally and shows you the result before anything is written to Jira or Confluence. You
approve the exact epic, story, comment or page — or you don't, and it stays a draft in your workspace.

Your Atlassian credentials never pass through Seamlex or this plugin. You sign in to Atlassian yourself,
in your own browser, through the official Atlassian MCP server.

## Your workspace

```
seamlex/
└── discovery/
    └── discovery-brief.md      # a local copy of your Discovery Brief, if you keep one
```

Relevamientos leave nothing here: each one gets its own Confluence page — `Minuta <task>`, created on the
first session, updated as you go, marked `Finalizado` when you approve it — plus the transcript and a
comment on the Jira task, and so does the list of unidentified features.

There is no configuration to fill in. The engagement settings — Jira project, issue types, the signed
scope page, who to reach at Seamlex — live on a `claude-client-config` page in your Confluence space,
maintained by Seamlex; `/hi-seamlex` loads it at the start of each session and every command reads its
settings from there. Who *you* are — the language you work in — comes from the Atlassian account you sign
in with; your company and program from the config page, the signed scope page or your Discovery Brief.
Your workspace holds only your discovery notes. Which step of the lifecycle you are on isn't recorded
anywhere — `/hi-seamlex` works it out each session from your brief and your board.

## Install

See [SETUP.md](SETUP.md). Two commands and a browser sign-in.

---

Built and maintained by Seamlex. Questions about the plugin itself go to your Seamlex delivery contact.
