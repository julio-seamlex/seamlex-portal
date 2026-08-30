---
description: Connect this workspace to Seamlex — verify the Atlassian connection, check it against the plugin's fixed config, and create the local working folders.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Set up the Seamlex Portal

Confirm the Atlassian connection works, check it against the settings the plugin already ships with, and
create the local folders the other commands write into. Run this once per workspace, before anything else.

**There is nothing to fill in.** The configuration is fixed and ships with the plugin at
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md). Every Seamlex command and
agent reads that file and only that file. It is **read-only**: this command never edits it, never copies it
into the workspace, and never publishes it anywhere. If a value in it is wrong for this customer, the fix is
to change the plugin — say so rather than working around it locally.

## Steps

1. **Read the config.** Read [`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md)
   and resolve `{{COMPANY}}`, `{{PROGRAM}}`, `{{SITE_URL}}`, `{{CLOUD_ID}}`, `{{JIRA_PROJECT}}`,
   `{{CONF_SPACE}}`, `{{CONF_PARENT}}`, the §3 issue types, and `{{DRAFTS_DIR}}`. Show the customer a short
   summary of what the plugin is configured for — company, program, Jira project, Confluence space — so a
   mismatch is caught in the first ten seconds rather than three commands later.

2. **Check the Atlassian connection.** Call `getAccessibleAtlassianResources`. Its full name is
   namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
   — so match on the base name after the last `__`; the prefix differs if the customer already has an
   Atlassian server configured elsewhere, and either one works.
   - If the tool is not available at all, the MCP server is not connected. Tell the customer to restart
     Claude and approve the `atlassian` server, and see the "Connecting Atlassian" section of
     [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md). Still do step 4 — the folders are useful offline — and
     say plainly that nothing was verified against Atlassian this session.
   - If it prompts for browser sign-in, walk them through it. Seamlex never sees their credentials.
   - Confirm the site that comes back matches `{{SITE_URL}}` and `{{CLOUD_ID}}`. If it does not, stop and
     report the difference: the config points at a site this user cannot reach, and every later write would
     land in the wrong place or fail.

3. **Verify the config against the live site.** These are all reads; do them without asking.
   - `atlassianUserInfo` — confirm who the customer is signed in as.
   - `getVisibleJiraProjects` — confirm `{{JIRA_PROJECT}}` is visible to them.
   - `getJiraProjectIssueTypesMetadata` on that project — confirm the §3 type names (`{{TYPE_EPIC}}`,
     `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`) actually exist. Where one does not, name it, name the nearest
     available type the agents will fall back to, and say the config row should be corrected in the plugin.
   - `getConfluenceSpaces` — confirm `{{CONF_SPACE}}` exists, and `getConfluencePage` on `{{CONF_PARENT}}`
     that the discovery parent page is reachable.
   - `searchJiraIssuesUsingJql` with `project = {{JIRA_PROJECT}} ORDER BY created DESC` — a read-only probe;
     report how many issues are visible.
   Report each check as pass or mismatch. A mismatch is a finding about the plugin's config, not something
   to ask the customer to fix locally.

4. **Create the local working folders.**
   `mkdir -p {{DRAFTS_DIR}}/discovery {{DRAFTS_DIR}}/requests`
   These hold drafts only — discovery notes and requirement drafts, before they become Jira issues. No
   config file is written here.

5. **Record the lifecycle step.** Write `{{DRAFTS_DIR}}/state.md` if it does not exist, using
   [`templates/state.md`](${CLAUDE_PLUGIN_ROOT}/templates/state.md): `{{STAGE}}` = `discovery`,
   `{{STAGE_UPDATED}}` = today's date, `{{STAGE_NOTE}}` = one line saying setup completed. This file is
   local and per-workspace — it is the one piece of state Seamlex keeps outside the fixed config, and
   `/hi-seamlex` reads it at the start of every session. If it already exists, leave it alone.

6. **Finish** with a short summary — what was verified, anything that mismatched, and what to do next:
   - `/hi-seamlex` — start any later session; it works out which step you are on and loads the context
   - `/seamlex-discovery` — the first working session, if discovery hasn't happened yet
   - `/seamlex-request` — turn an idea into an epic with user stories
   - `/seamlex-ask` — ask the Seamlex team a question
   - `/seamlex-status` — see where your work stands

> If the Atlassian tools become unavailable partway through, do not fail the command. Nothing here depends
> on a write succeeding; say which checks did not run and offer to rerun `/seamlex-setup` once the
> connection is back.
