# Setup

Two steps, about five minutes.

## 1. Install the plugin

In Claude, add the Seamlex marketplace and install the plugin:

```
/plugin marketplace add julio-seamlex/seamlex-portal
/plugin install seamlex-portal@seamlex
```

Then restart Claude so the plugin's agents, commands and Atlassian connection load.

### Where to run this

**Claude Code** (terminal CLI, the Claude Code desktop app, or the VS Code / JetBrains extension) — type
the two commands above straight into the chat input. Typing just `/plugin` opens a menu that does the
same thing from a list. "Restart Claude" means quitting the app fully and reopening it, not just closing
the window; MCP servers only load at startup.

**Claude desktop app / claude.ai** — install through the UI instead of the commands:

1. **Customize** in the left sidebar → the **Plugins** tab
2. Under **Personal plugins**, click **+** → **Add marketplace**
3. Choose **Add from a repository** and paste `https://github.com/julio-seamlex/seamlex-portal`
4. Install **seamlex-portal** once the marketplace syncs

Then use the plugin from the **Cowork** tab, not the Chat tab. Sub-agents run only in Cowork, and every
Seamlex command delegates to one of the three agents, so in plain chat the commands will appear but do
nothing.

> **This is a private repository.** Your Seamlex contact will grant your GitHub account read access
> before you install. Claude Code uses your existing GitHub credentials; the Claude desktop app's
> "Add from a repository" may not be able to reach a private repo at all — if it fails to sync, install
> through Claude Code instead. If `marketplace add` reports that the repository cannot be found,
> that grant hasn't landed yet — tell your contact rather than retrying.
>
> If you authenticate to GitHub with the `gh` CLI, `gh auth status` should show the account that was
> granted access. Claude uses your existing GitHub credentials to fetch the plugin — from the machine
> running Claude, so check `gh auth status` there and not on some other laptop.

## 2. Connect Atlassian

Run:

```
/hi-seamlex
```

`/hi-seamlex` is the only command you need to start with: the first time you run it, it sets the workspace
up before doing anything else. (`/hi-seamlex setup` re-runs just those checks later.)

**The first run fills in the config.** The plugin ships `config/seamlex.config.md` blank, and step 1 of
`/hi-seamlex` completes it before anything else: it reads your site, Jira project, Confluence space and
issue type names off Atlassian, and asks you for the handful of things only you know — your company,
programme, role and how you want the agents to behave. That happens once. Every later session reads the
filled config and gets straight to work, and `/hi-seamlex config` changes a value later.

This will:

1. **Connect to Atlassian.** The first time, Claude asks you to approve the `atlassian` MCP server, then
   opens your browser to sign in to Atlassian. You are signing in to *your own* Atlassian account —
   Seamlex never sees your credentials, and the plugin never stores them. You can revoke access any time
   from your Atlassian account settings.
2. **Fill in and show you the config** — company, program, Jira project, Confluence space — asking only
   for what it cannot read off your site, then summarizing it back so anything wrong is obvious
   immediately.
3. **Check it against your site.** That the Jira project and Confluence space are visible to you, and that
   the issue type names — Epic, Story, and whatever your project uses for questions — really exist.
   Anything that doesn't match is reported as a mismatch, with an offer to correct that row in the config
   there and then.
4. **Verify.** It runs a read-only query against your project and reports what it can see.
5. **Create your working folders** — `seamlex/discovery/` and `seamlex/requests/` for drafts.

Everything written to your workspace is your own work — drafts, plus the completed config where the plugin
directory is read-only. No state, no secrets, safe to commit.

> **If a setting is wrong for you** — the wrong project key, an issue type that doesn't exist — tell your
> Seamlex contact. The fix ships in the next version of the plugin, so it is right for everyone at once
> rather than in one person's local file.

## Connecting Atlassian

The plugin ships with the official Atlassian MCP server configured. If `/hi-seamlex` reports that
Atlassian tools are unavailable:

- **Restart Claude.** MCP servers load at startup; a freshly installed plugin's server won't be live until
  you do.
- **Approve the server.** Claude asks once, on first use, whether to trust the `atlassian` server from
  this plugin. If you declined, re-enable it in your MCP settings.
- **Check you're signed in.** The connection uses a browser OAuth flow. If it expired, running
  `/hi-seamlex setup` again will prompt you to sign in.
- **Check your access.** You need access to the Jira project and Confluence space Seamlex shares with
  you. If the setup checks see no projects, ask your Seamlex contact to confirm your invitation.

If your organization proxies or restricts outbound connections, your IT team may need to allow
`mcp.atlassian.com`.

> **On command names:** Claude also lists these fully qualified, as
> `/seamlex-portal:hi-seamlex`. Both forms work — type the short one.

## What to do next

| | |
|---|---|
| First run, or not sure where the engagement got to | `/hi-seamlex` |
| New engagement, discovery not done yet | `/seamlex-discovery` |
| You have something you need built | `/seamlex-request` |
| You have a question | `/seamlex-ask` |
| You want to know where things stand | `/seamlex-status` |

## Troubleshooting

**"Config missing or has unfilled rows"** — run `/hi-seamlex`; completing the config is the first thing it
does. If the file itself is missing, the install is incomplete or out of date: reinstall or update the
plugin and restart Claude.

**A setting is wrong** — the wrong Jira project, a Confluence space you can't see, an issue type that
doesn't exist in your project. `/hi-seamlex setup` reports these as mismatches and offers to fix the row;
`/hi-seamlex config` reopens any of them on demand.

**"Which step am I on?"** — nothing records it. `/hi-seamlex` works it out each session from your
workspace, your Discovery Brief and your board, and tells you which signals it read. If it lands wrong,
say so, or name the step yourself: `/hi-seamlex requirement`.

**An agent can't find the discovery brief** — that's fine; it will say so and carry on. Discovery makes
requirements sharper but isn't a hard prerequisite.

**Wrong issue type when raising a requirement** — §3 of the config does not match your project. Run
`/hi-seamlex setup` to see exactly which name is off, and let it correct the row.

**A write to Jira half-succeeded** — the agent will tell you exactly which issues were created and which
weren't, and stop rather than retrying. Give that list to your Seamlex contact.
