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
| `/seamlex-discovery` | A guided session covering your business model, industry, areas and roles, business processes, actors, pains, goals and expectations. Resumes from an existing brief. |
| `/refine-project-scope` | Refine the epics in your signed contract scope, one at a time, into Confluence epic pages ready to be designed and built. |
| `/seamlex-ask` | Ask the Seamlex team a question. |
| `/seamlex-status` | See what's in progress, what's waiting on you, and what's blocked. |

`/hi-seamlex` is the one to reach for when you are not sure what to run: it works out which step you are
on — setup, discovery, scope refinement or status — pulls down just that step's context, and points you at the
right command.

You can also talk to the agents directly — "run discovery with me", "let's refine the quoting epic",
"what's the status of the quoting work" — without remembering a command.

## The three agents

**Discovery** runs the first working session of an engagement. Ten themed sections, questions in small
batches, covering your business model, industry, how you are organised, the processes this project
touches, your current systems, the people involved, what hurts today, and what success looks like. It produces a Discovery Brief published to Confluence — the document every
Seamlex architect and developer reads before touching your org. Sessions are resumable; stop whenever you
like and nothing is lost.

**Product Owner** refines your signed contract scope, drafting straight into Confluence. It opens with every epic in the contract and where
its refinement stands — sin comenzar, en progreso, finalizado — then takes one epic at a time and works it
into its own Confluence page, titled exactly as the contract names it: real actors, the process delta,
rules, statuses and edge cases, data, visibility, reporting, what is explicitly out of scope, and the user
stories that carry the value. When an epic is done you get a summary to check, and the page moves to
**Ready for design** — a checklist it must pass, so nothing bounces back from the architect later. The
contract epic, its page and the Jira key that comes out of it all carry the same title, so the work traces
back to what you signed. Anything you raise that fits no epic in the contract is parked as an
**unidentified feature** rather than quietly absorbed — Seamlex decides what happens to it.

**Project Manager** answers "where is my request" from the live board rather than from memory, tells you
plainly what is stale or blocked, and leads with what is waiting on you. It also handles questions:
searching Jira and Confluence for an existing answer first, and filing a tracked question when it
genuinely needs the team.

## Nothing happens without your approval

Every agent drafts locally and shows you the result before anything is written to Jira or Confluence. You
approve the exact epic, story, comment or page — or you don't, and it stays a draft in your workspace.

Your Atlassian credentials never pass through Seamlex or this plugin. You sign in to Atlassian yourself,
in your own browser, through the official Atlassian MCP server.

## Your workspace

```
seamlex/
└── discovery/
    └── discovery-brief.md      # your discovery notes, resumable
```

Scope refinement leaves nothing here: each contract epic gets its own Confluence page — created as a draft
on the first session, updated as you go, marked `Ready for design` when you approve it — and so does the
list of unidentified features.

There is no configuration to fill in. The settings — your company, program, Jira project, Confluence space
and issue types — ship with the plugin in the **Configuration** section of `commands/hi-seamlex.md`, and
every command reads them from there. Your workspace holds only your discovery notes. Which step of the
lifecycle you are on isn't recorded anywhere — `/hi-seamlex` works it out each session from your brief and
your board.

## Install

See [SETUP.md](SETUP.md). Two commands and a browser sign-in.

---

Built and maintained by Seamlex. Questions about the plugin itself go to your Seamlex delivery contact.
