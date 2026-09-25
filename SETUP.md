# Setup

Two steps, about five minutes.

## 1. Install the plugin

In Claude, add the Seamlex marketplace and install the plugin:

```
/plugin marketplace add julio-seamlex/seamlex-portal
/plugin install seamlex-portal@seamlex
```

Then restart Claude so the plugin's commands, skills and Atlassian connection load. GitHub is a
separate, one-time step — see "Connecting GitHub" below — since it needs your own personal access
token, not something the plugin can bundle for you.

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

The plugin works from both the **Chat** and the **Cowork** tab — nothing in it runs as a sub-agent.

> **This is a private repository.** Your Seamlex contact will grant your GitHub account read access
> before you install. Claude Code uses your existing GitHub credentials; the Claude desktop app's
> "Add from a repository" may not be able to reach a private repo at all — if it fails to sync, install
> through Claude Code instead. If `marketplace add` reports that the repository cannot be found,
> that grant hasn't landed yet — tell your contact rather than retrying.
>
> If you authenticate to GitHub with the `gh` CLI, `gh auth status` should show the account that was
> granted access. Claude uses your existing GitHub credentials to fetch the plugin — from the machine
> running Claude, so check `gh auth status` there and not on some other laptop.

## 2. Connect Atlassian and GitHub

Run:

```
/hi-seamlex
```

`/hi-seamlex` is the only command you need to start with: run it at the start of every session.

**There is nothing to fill in.** Your engagement's settings — Jira project, issue type names, Seamlex
contacts, the signed scope file — live in `README.md` and `config.yml` in your GitHub documentation repo,
maintained by Seamlex. `/hi-seamlex` finds that repo and loads both files into the session, and every
other command reads its settings from there. Your language and who you are come from the Atlassian
account you sign in with; your company and program come from `config.yml` and your Discovery Brief when
the config does not name them (the first time, before a brief exists, you are asked for your company name
once). Nothing is written to your workspace.

This will:

1. **Connect to Atlassian.** The first time, Claude asks you to approve the `atlassian` MCP server, then
   opens your browser to sign in to Atlassian. You are signing in to *your own* Atlassian account —
   Seamlex never sees your credentials, and the plugin never stores them. You can revoke access any time
   from your Atlassian account settings.
2. **Check the GitHub connection.** You need your own `github` MCP server added — see
   "Connecting GitHub" below if this is your first run.
3. **Show you what the session is running on** — who you are signed in as, your company and program as
   read from the repo, the Jira project, the documentation repo — each tagged with where it came from, so
   anything wrong is obvious immediately and correctable in one line.
4. **Check it against your site.** That the Jira project is visible to you and the documentation repo is
   reachable, that the sub-task type really exists, and that relevamiento tasks in your project carry the
   `relevamiento` label. Anything that doesn't match is reported as a mismatch to take back to Seamlex,
   not something for you to patch locally.
5. **Verify.** It runs a read-only query against your project and reports what it can see.
6. **Create your working folder** — `seamlex/discovery/` for your discovery notes. Scope refinement drafts live in GitHub, not in your workspace.

Everything written to your workspace is your own work: drafts, and nothing else. No settings, no state, no
secrets, safe to commit.

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
  `/hi-seamlex` again will prompt you to sign in.
- **Check your access.** You need access to the Jira project Seamlex shares with
  you. If the setup checks see no projects, ask your Seamlex contact to confirm your invitation.

If your organization proxies or restricts outbound connections, your IT team may need to allow
`mcp.atlassian.com`.

## Connecting GitHub

Unlike Atlassian, the plugin does **not** ship the GitHub connection for you — GitHub's remote MCP
server needs a personal access token, and Claude Code cannot safely substitute a token from an
environment variable into a *shared* config file (that would mean a plugin could ask Claude to hand
your secret to a server it names, so it's blocked). You add your own `github` MCP server once, with
your own token, and it stays in your personal Claude config — never in this repo, never shared with
anyone else who uses the plugin.

If `/hi-seamlex` reports that GitHub tools are unavailable:

- **Create a token.** In GitHub, go to Settings → Developer settings → Personal access tokens, and
  create one scoped to `repo` (read/write access to the documentation repo Seamlex shares with you).
- **Add the server**, once, at **user** scope so it's available wherever you run Claude:
  ```
  claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
    --header "Authorization: Bearer <your token>" --scope user
  ```
  **Never use `--scope project`** for this — that would write your token into a file shared with
  everyone who uses this plugin.
- **Restart Claude.** MCP servers load at startup; the new server will not be live until you do.
- **Check your access.** You need read/write access to the documentation repo Seamlex shares with you.
  If the setup checks can't reach it, ask your Seamlex contact to confirm you were added as a
  collaborator.
- **`claude mcp list`** shows every server you've added and which scope it's in, if you want to confirm
  `github` landed at `user` scope rather than `local` or `project`.

> **On command names:** Claude also lists these fully qualified, as
> `/seamlex-portal:hi-seamlex`. Both forms work — type the short one.

## What to do next

| | |
|---|---|
| First run, or not sure where the engagement got to | `/hi-seamlex` |
| You are ready to run the relevamiento of a task in the current sprint | `/seamlex-refinar` |

## Troubleshooting

**"No documentation repo found" / "No `README.md`/`config.yml`"** — `/hi-seamlex` searched for a
`seamlex-docs-` repo in the Seamlex org and found none, or found the repo but not those two files.
Seamlex creates and maintains both; tell your Seamlex contact which account you were signed in as.

**A setting is wrong or missing** — the wrong Jira project, an issue type that doesn't exist in your
project, a blank entry a command asks about. These live in `config.yml` in your GitHub repo,
not in your workspace or the plugin — send the entry to your Seamlex contact and they fix the file; the
next `/hi-seamlex` picks it up.

**The company, program or language is wrong** — these are read live, not shipped. The program is named
in `scope.md` or `config.yml`, the language is your Atlassian profile's locale, the company comes from
your Discovery Brief. Fix the source (rename it in the file, change your Atlassian language, correct the
brief header) and the next session picks it up — or just tell the command in one line for the current
session.

**"Which step am I on?"** — nothing records it. `/hi-seamlex` works it out each session from your
workspace, your Discovery Brief and your board, and tells you which signals it read. If it lands wrong,
say so, or name the step yourself: `/hi-seamlex refinement`.

**A command can't find the discovery brief** — that's fine; it will say so and carry on. The Discovery
Brief is prepared with your Seamlex consultant; it makes scope refinement sharper but isn't a hard
prerequisite.

**Wrong issue type when raising an epic or story** — the issue types in `config.yml` do
not match your project. Send the name that is off to your Seamlex contact for a fix on the file.

**The minuta has no labels or topics on GitHub** — expected. The plugin does not use them at all; the
file is found by its path (`relevamientos/ABC-12.md`) and by the Jira key in its front matter. Nothing
to fix.

**A write to Jira half-succeeded** — the command will tell you exactly which issues were created and which
weren't, and stop rather than retrying. Give that list to your Seamlex contact.
