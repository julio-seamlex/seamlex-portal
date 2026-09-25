---
description: Start a Seamlex session — signs you in to Atlassian and GitHub, picks the customer's documentation repo, and loads the project's README.md and config.yml into the session.
allowed-tools: Read, AskUserQuestion, Skill, mcp__atlassian__getAccessibleAtlassianResources, mcp__github__get_file_contents, mcp__github__search_repositories
---

# Start a Seamlex session

Run this at the start of every session. It does six things and nothing else: makes sure you are
signed in to Atlassian and GitHub, settles which GitHub repo this session works in, finds and reads
that repo's `README.md` and `config.yml`, keeps their content in the session for `/seamlex-refinar` to
use, and loads the house rules for Jira and GitHub so every read and write that command makes follows
them.

Every call here is a read. This command writes nothing — not to the workspace, not to Jira, not to
GitHub.

## Steps

1. **Authenticate to Atlassian (only if needed).** Call `getAccessibleAtlassianResources`. Its full name
   is namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
   — so match on the base name after the last `__`; the prefix differs if the customer already has an
   Atlassian server configured elsewhere, and either one works.
   - If it returns resources, the user is already authenticated — do not prompt for anything, move on.
   - If it prompts for browser sign-in, walk the user through it. Seamlex never sees their credentials.
   - If the tool is not available at all, the MCP server is not connected. Tell the user to restart Claude
     and approve the `atlassian` server (see "Connecting Atlassian" in
     [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md)) and stop here — Jira needs it and nothing below works
     without it either.

   Keep the `cloudId` from the result; every later Jira call needs it.

2. **Check the GitHub connection.** Call `get_file_contents` on any known path (or `search_repositories`
   with no query) to confirm a `github` MCP tool answers. Unlike Atlassian, this server is **not**
   bundled by the plugin — each person adds their own, so the tool's namespace prefix is whatever they
   named it when adding it (typically `mcp__github__get_file_contents`); match on the base name after
   the last `__`.
   - If it answers, move on without announcing anything.
   - If no GitHub tool is available at all, or it fails with an authorization error, the person hasn't
     added their own `github` MCP server yet (or it has no working token). Tell them once, plainly:
     documentation lives on GitHub and needs their own personal access token — a plugin can't safely
     hand out a token from a shared config file, so this is a one-time step they run themselves. Point
     at "Connecting GitHub" in [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md) (`claude mcp add --transport
     http github https://api.githubcopilot.com/mcp/ --header "Authorization: Bearer <token>" --scope
     user`, then restart Claude) and stop — nothing below works without it. Do not repeat this check's
     explanation every session once it is set; if it is already working, say nothing about it.

3. **Settle which GitHub repo this session works in.** Seamlex documentation repos follow the pattern
   `{{GITHUB_ORG}}/seamlex-docs-<customer-slug>` in the Seamlex-owned org.
   - If the user's own instructions (a `CLAUDE.md`, a prior message this session) already name the repo,
     use it.
   - Otherwise call `search_repositories` scoped to the Seamlex org for `seamlex-docs-`.
     - Exactly one match → use it. Say which one, in a single line.
     - More than one → ask with `AskUserQuestion` (header `Repo`), listing each repo by name, and use
       the one they pick.
     - None → tell the user no documentation repo was found for them and to check with their Seamlex
       contact, and stop.

4. **Download `README.md` and `config.yml` and keep them in the session.** Call `get_file_contents` on
   both at the repo root. Keep their content in context for the rest of the session: they are the source
   of truth `/seamlex-refinar` and the Seamlex skills read from, so do not summarize them away or drop
   them. Do not list their contents back to the user.
   - If either file is missing, tell the user the repo has no `README.md`/`config.yml` yet, name the repo
     you looked in, and stop. Do not create one.

5. **Load the house rules for Jira and GitHub.** Load the `seamlex-portal:delivery-how-to` skill
   with the `Skill` tool; if the Skill tool is not available, read
   `../skills/delivery-how-to/SKILL.md` directly. It carries the structure of the documentation repo,
   the shape of the root index the config files follow, what a well-completed Jira task looks like,
   the Jira labels, and the retrieval patterns to find files by path and tasks by key — every read and
   write `/seamlex-refinar` makes in Jira and GitHub follows it. Keep it in context for the session; do
   not summarise it to the customer.

6. **Close the setup.** In two lines at most: say the setup has finished, naming the repo (org and name)
   and welcome the customer to the Seamlex portal.

## Where the configuration lives

**`config.yml` in the customer's GitHub repo is the configuration.** `/seamlex-refinar` and the Seamlex
skills resolve the `{{PLACEHOLDER}}` tokens in its instructions from the file loaded in step 4 — the
Jira project, the sub-task issue type, Jira labels, the Seamlex contacts, the write-confirmation and
detail preferences, and whatever else `config.yml` sets. `README.md` holds no settings — it is the
purpose and the map, not the values — so nothing about behaviour is read from it directly. The pair's
shape — `README.md`'s purpose and map, `config.yml`'s one-key-per-setting — is the root instance of the
index the `delivery-how-to` skill defines
(`../skills/delivery-how-to/references/memory-template.md`); Seamlex maintains both, and no command
ever edits `config.yml`.

Two things come from the session rather than the files: `{{CLOUD_ID}}` is the `cloudId` from step 1, and
`{{GITHUB_ORG}}`/`{{GITHUB_REPO}}` are the repo settled in step 3 — the files live inside it. Who is
signed in, and the language to answer in, come from `atlassianUserInfo`; the company and program come
from `scope.md` and the Discovery Brief when `config.yml` does not name them.

No state is kept per-workspace. Discovery notes live under `seamlex/`, and never come back into this file.
To change how the plugin behaves for a customer, edit their `config.yml` in GitHub —
there is no plugin release involved.

Safe to commit — it holds no secrets. Authentication to Jira happens through the Atlassian MCP server's
own browser login; authentication to GitHub happens through the personal access token set up per
"Connecting GitHub" in `SETUP.md`. **Never put an API token or password in `config.yml` or anywhere in
the documentation repo.**
