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
| `/seamlex-discovery` | A guided session that builds a full picture of your business, goals, actors and pains. |
| `/seamlex-request` | Describe what you need; get back an epic with its key user stories. |
| `/seamlex-ask` | Ask the Seamlex team a question. |
| `/seamlex-status` | See what's in progress, what's waiting on you, and what's blocked. |

`/hi-seamlex` is the one to reach for when you are not sure what to run: it works out which step you are
on — setup, discovery, requirements or status — pulls down just that step's context, and points you at the
right command.

You can also talk to the agents directly — "run discovery with me", "I have a new requirement",
"what's the status of the quoting work" — without remembering a command.

## The three agents

**Discovery** runs the first working session of an engagement. Nine themed sections, questions in small
batches, covering your business model, industry, current systems, the people involved, what hurts today,
and what success looks like. It produces a Discovery Brief published to Confluence — the document every
Seamlex architect and developer reads before touching your org. Sessions are resumable; stop whenever you
like and nothing is lost.

**Product Owner** takes a requirement in whatever shape you have it — a sentence, a frustration, a
half-formed idea — and turns it into a well-formed epic with three to seven user stories: real personas,
testable acceptance criteria, explicit scope. It checks for duplicates first and anchors every story to a
pain from discovery, so what gets built traces back to why you wanted it.

**Delivery Liaison** answers "where is my request" from the live board rather than from memory, tells you
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
├── discovery/
│   └── discovery-brief.md      # your discovery notes, resumable
└── requests/
    └── <slug>.md               # requirement drafts, before they become Jira issues
```

There is no configuration to fill in. The settings — your company, program, Jira project, Confluence space
and issue types — ship with the plugin in the **Configuration** section of `commands/hi-seamlex.md`, and
every command reads them from there. Your workspace holds only your own work: drafts on their way to Jira and Confluence. Which step of the
lifecycle you are on isn't recorded anywhere — `/hi-seamlex` works it out each session from your brief and
your board.

## Install

See [SETUP.md](SETUP.md). Two commands and a browser sign-in.

---

Built and maintained by Seamlex. Questions about the plugin itself go to your Seamlex delivery contact.
