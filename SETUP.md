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
/seamlex-setup
```

**There is nothing to fill in.** Seamlex ships the plugin already configured for your engagement — your
company, program, Jira project, Confluence space and issue type names all live in the plugin's own
`config/seamlex.config.md`. Setup does not ask you for them and does not create a config in your
workspace; it confirms the connection works and that what the plugin expects matches what your Atlassian
site actually has.

This will:

1. **Connect to Atlassian.** The first time, Claude asks you to approve the `atlassian` MCP server, then
   opens your browser to sign in to Atlassian. You are signing in to *your own* Atlassian account —
   Seamlex never sees your credentials, and the plugin never stores them. You can revoke access any time
   from your Atlassian account settings.
2. **Show you what the plugin is configured for** — company, program, Jira project, Confluence space — so
   anything wrong is obvious immediately.
3. **Check it against your site.** That the Jira project and Confluence space are visible to you, and that
   the issue type names — Epic, Story, and whatever your project uses for questions — really exist.
   Anything that doesn't match is reported as a mismatch to take back to Seamlex, not something for you
   to patch locally.
4. **Verify.** It runs a read-only query against your project and reports what it can see.
5. **Create your working folders** — `seamlex/discovery/` and `seamlex/requests/` for drafts, and
   `seamlex/state.md` recording which lifecycle step you are on.

Everything written to your workspace is your own work: drafts and the lifecycle step. No settings, no
secrets, safe to commit.

> **If a setting is wrong for you** — the wrong project key, an issue type that doesn't exist — tell your
> Seamlex contact. The fix ships in the next version of the plugin, so it is right for everyone at once
> rather than in one person's local file.

## Connecting Atlassian

The plugin ships with the official Atlassian MCP server configured. If `/seamlex-setup` reports that
Atlassian tools are unavailable:

- **Restart Claude.** MCP servers load at startup; a freshly installed plugin's server won't be live until
  you do.
- **Approve the server.** Claude asks once, on first use, whether to trust the `atlassian` server from
  this plugin. If you declined, re-enable it in your MCP settings.
- **Check you're signed in.** The connection uses a browser OAuth flow. If it expired, running
  `/seamlex-setup` again will prompt you to sign in.
- **Check your access.** You need access to the Jira project and Confluence space Seamlex shares with
  you. If the setup command sees no projects, ask your Seamlex contact to confirm your invitation.

If your organization proxies or restricts outbound connections, your IT team may need to allow
`mcp.atlassian.com`.

> **On command names:** Claude also lists these fully qualified, as
> `/seamlex-portal:seamlex-setup`. Both forms work — type the short one.

## What to do next

| | |
|---|---|
| Not sure where the engagement got to | `/hi-seamlex` |
| New engagement, discovery not done yet | `/seamlex-discovery` |
| You have something you need built | `/seamlex-request` |
| You have a question | `/seamlex-ask` |
| You want to know where things stand | `/seamlex-status` |

## Troubleshooting

**"Config missing or has unfilled placeholders"** — the config ships with the plugin, so this means the
install is incomplete or out of date. Reinstall or update the plugin and restart Claude; if it persists,
tell your Seamlex contact which rows are blank.

**A setting is wrong** — the wrong Jira project, a Confluence space you can't see, an issue type that
doesn't exist in your project. `/seamlex-setup` reports these as mismatches. They are fixed in the plugin
by Seamlex, not in your workspace — send the mismatch to your Seamlex contact.

**"Which step am I on?"** — that lives in `seamlex/state.md` in your workspace, and `/hi-seamlex` keeps it
current. Delete it and the next `/hi-seamlex` works the step out again from your brief and your board.

**An agent can't find the discovery brief** — that's fine; it will say so and carry on. Discovery makes
requirements sharper but isn't a hard prerequisite.

**Wrong issue type when raising a requirement** — §3 of the plugin's config does not match your project.
Run `/seamlex-setup` to see exactly which name is off, and send that to your Seamlex contact for a fix.

**A write to Jira half-succeeded** — the agent will tell you exactly which issues were created and which
weren't, and stop rather than retrying. Give that list to your Seamlex contact.
