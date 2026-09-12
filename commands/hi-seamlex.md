---
description: Start a Seamlex session — signs you in to Atlassian, picks the Confluence space, and loads the project's claude-client-config page into the session.
allowed-tools: Read, AskUserQuestion
---

# Start a Seamlex session

Run this at the start of every session. It does four things and nothing else: makes sure you are signed
in to Atlassian, settles which Confluence space this session works in, finds the `claude-client-config`
page in that space, and keeps its content in the session for every later command to use.

Every call here is a read. This command writes nothing — not to the workspace, not to Jira, not to
Confluence.

## Steps

1. **Authenticate to Atlassian (only if needed).** Call `getAccessibleAtlassianResources`. Its full name
   is namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
   — so match on the base name after the last `__`; the prefix differs if the customer already has an
   Atlassian server configured elsewhere, and either one works.
   - If it returns resources, the user is already authenticated — do not prompt for anything, move on.
   - If it prompts for browser sign-in, walk the user through it. Seamlex never sees their credentials.
   - If the tool is not available at all, the MCP server is not connected. Tell the user to restart Claude
     and approve the `atlassian` server (see "Connecting Atlassian" in
     [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md)) and stop here — nothing below works without it.

   Keep the `cloudId` from the result; every later call needs it.

2. **Choose the Confluence space.** Call `getConfluenceSpaces` with that `cloudId`.
   - Exactly one space visible → use it. Say which one, in a single line.
   - More than one → ask with `AskUserQuestion` (header `Space`), listing each space by name and key, and
     use the one they pick.
   - None → tell the user their Atlassian account has no Confluence space visible and stop.

3. **Find the `claude-client-config` page in that space.** Call `searchConfluenceUsingCql` with
   `space = "<space key>" AND title = "claude-client-config"`. This page is the index of the project's key
   files and aspects — the things to keep in session memory.
   - One hit → take its page id.
   - Several hits → prefer the one whose title matches exactly; if still ambiguous, ask with
     `AskUserQuestion` (header `Config page`), showing each page's title and parent.
   - No hit → tell the user the space has no `claude-client-config` page yet, name the space you searched,
     and stop. Do not create one.

4. **Download the page and keep it in the session.** Call `getConfluencePage` on that page id and read the
   whole body. Keep it in context for the rest of the session: it is the source of truth the other Seamlex
   commands and agents read from, so do not summarize it away or drop it. Do not list the page's contents
   back to the user.

5. **Close the setup.** In two lines at most: say the setup has finished, naming the space and the config
   page (title and link), and welcome the customer to the Seamlex portal.

## Where the configuration lives

**The `claude-client-config` page is the configuration.** Every other Seamlex command and agent resolves
the `{{PLACEHOLDER}}` tokens in its instructions from the page loaded in step 4 — the Jira project, the
Confluence space and parent pages, the signed scope page, issue types and labels, the Seamlex contacts,
the write-confirmation and detail preferences, and whatever else the page indexes. Nothing is read from
this file: it holds no tables, no values, and no per-customer state.

Two things come from the session rather than the page: `{{CLOUD_ID}}` is the `cloudId` from step 1, and
`{{CONF_SPACE}}` is the space chosen in step 2 — the page lives inside it. Who is signed in, and the
language to answer in, come from `atlassianUserInfo`; the company, program and industry come from the
signed scope page and the Discovery Brief when the config page does not name them.

No state is kept per-workspace. Discovery notes live under `seamlex/`, and never come back into this file.
To change how the plugin behaves for a customer, edit their `claude-client-config` page in Confluence —
there is no plugin release involved.

Safe to commit — it holds no secrets. Authentication to Jira and Confluence happens through the Atlassian
MCP server's own browser login, never through this file. **Never put an API token or password in here.**
