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

## 2. Connect Atlassian and configure

Run:

```
/seamlex-setup
```

This will:

1. **Connect to Atlassian.** The first time, Claude asks you to approve the `atlassian` MCP server, then
   opens your browser to sign in to Atlassian. You are signing in to *your own* Atlassian account —
   Seamlex never sees your credentials, and the plugin never stores them. You can revoke access any time
   from your Atlassian account settings.
2. **Find your project.** It reads which Jira projects and Confluence spaces you can see and proposes the
   right ones, rather than asking you to look up keys.
3. **Read your real issue types.** Epic, Story and whatever your project uses for questions — taken from
   the project itself, not assumed.
4. **Ask the rest.** Your company, industry, program name, and how much detail you want in answers.
   A handful of questions. It never asks for delivery-side details like your Seamlex contact or the
   sprint cadence — Seamlex fills those in.
5. **Verify.** It runs a read-only query against your project and reports what it can see.

The result is `seamlex/config.md` in your workspace. You can edit it by hand at any time, and it holds no
secrets, so it is safe to commit.

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
| New engagement, discovery not done yet | `/seamlex-discovery` |
| You have something you need built | `/seamlex-request` |
| You have a question | `/seamlex-ask` |
| You want to know where things stand | `/seamlex-status` |

## Troubleshooting

**"Config missing or has unfilled placeholders"** — run `/seamlex-setup`. If `seamlex/config.md` exists,
it will only fill the gaps, never overwrite what you've set.

**An agent can't find the discovery brief** — that's fine; it will say so and carry on. Discovery makes
requirements sharper but isn't a hard prerequisite.

**Wrong issue type when raising a requirement** — open `seamlex/config.md` §3 and set the type names to
match your project exactly, then try again.

**A write to Jira half-succeeded** — the agent will tell you exactly which issues were created and which
weren't, and stop rather than retrying. Give that list to your Seamlex contact.
