---
description: Start a Seamlex session — signs you in to Atlassian, picks the Confluence space, and loads the project's seamlex-portal-memory page into the session.
allowed-tools: Read, AskUserQuestion, Skill, mcp__atlassian__getAccessibleAtlassianResources, mcp__atlassian__getConfluenceSpaces, mcp__atlassian__searchConfluenceUsingCql, mcp__atlassian__getConfluencePage
---

# Start a Seamlex session

Run this at the start of every session. It does five things and nothing else: makes sure you are signed
in to Atlassian, settles which Confluence space this session works in, finds the `seamlex-portal-memory`
page in that space, keeps its content in the session for `/seamlex-refinar` to use, and loads the
house rules for Jira and Confluence so every read and write that command makes follows them.

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

3. **Find the `seamlex-portal-memory` page in that space.** Call `searchConfluenceUsingCql` with
   `space = "<space key>" AND title = "seamlex-portal-memory"`. This page is the index of the project's key
   files and aspects — the things to keep in session memory.
   - One hit → take its page id.
   - Several hits → prefer the one whose title matches exactly; if still ambiguous, ask with
     `AskUserQuestion` (header `Config page`), showing each page's title and parent.
   - No hit → tell the user the space has no `seamlex-portal-memory` page yet, name the space you searched,
     and stop. Do not create one.

4. **Download the page and keep it in the session.** Call `getConfluencePage` on that page id and read the
   whole body. Keep it in context for the rest of the session: it is the source of truth `/seamlex-refinar` and
   the Seamlex skills read from, so do not summarize it away or drop it. Do not list the page's contents
   back to the user.

5. **Load the house rules for Jira and Confluence.** Load the `seamlex-portal:atlassian-how-to` skill
   with the `Skill` tool; if the Skill tool is not available, read
   `../skills/atlassian-how-to/SKILL.md` directly. It carries the structure of the Confluence space,
   the shape of the index page the config page follows, what a well-completed Jira task looks like,
   the Jira labels, and the CQL/JQL patterns to find pages by title and tasks by key — every read and write
   `/seamlex-refinar` makes in Jira and Confluence follows it. Keep it in context for the session; do not
   summarise it to the customer.

6. **Close the setup.** In two lines at most: say the setup has finished, naming the space and the config
   page (title and link), and welcome the customer to the Seamlex portal.

## Where the configuration lives

**The `seamlex-portal-memory` page is the configuration.** `/seamlex-refinar` and the Seamlex skills resolve
the `{{PLACEHOLDER}}` tokens in its instructions from the page loaded in step 4 — the Jira project, the
Confluence space and parent pages, the signed scope page, issue types and Jira labels, the Seamlex contacts,
the write-confirmation and detail preferences, and whatever else the page indexes. Nothing is read from
this file: it holds no tables, no values, and no per-customer state. The page's shape — purpose, settings
table, one row per page with its exact title, type and what to take from it — is the root
instance of the index page the `atlassian-how-to` skill defines
(`../skills/atlassian-how-to/references/memory-template.md`); Seamlex maintains it, and no command
ever edits it.

Two things come from the session rather than the page: `{{CLOUD_ID}}` is the `cloudId` from step 1, and
`{{CONF_SPACE}}` is the space chosen in step 2 — the page lives inside it. Who is signed in, and the
language to answer in, come from `atlassianUserInfo`; the company and program come from the
signed scope page and the Discovery Brief when the config page does not name them.

No state is kept per-workspace. Discovery notes live under `seamlex/`, and never come back into this file.
To change how the plugin behaves for a customer, edit their `seamlex-portal-memory` page in Confluence —
there is no plugin release involved.

Safe to commit — it holds no secrets. Authentication to Jira and Confluence happens through the Atlassian
MCP server's own browser login, never through this file. **Never put an API token or password in here.**
