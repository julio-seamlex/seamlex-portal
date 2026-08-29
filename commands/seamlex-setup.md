---
description: Connect this workspace to Seamlex — verify the Atlassian connection, discover your Jira project and Confluence space, and create seamlex/config.md interactively.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Set up the Seamlex Portal

Create `seamlex/config.md` — the single file every Seamlex agent reads — and confirm the Atlassian
connection works. Run this once, before anything else.

## Steps

1. **If `seamlex/config.md` already exists**, read it and report which rows still hold unresolved `<...>`
   placeholders. Offer to fill just those. Never overwrite a config the customer has filled in.

2. **Otherwise create it** from the shipped template:
   `mkdir -p seamlex/discovery seamlex/requests && cp "${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.template.md" seamlex/config.md`

3. **Check the Atlassian connection.** Call `getAccessibleAtlassianResources`. Its full name is
   namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
   — so match on the base name after the last `__`; the prefix differs if the customer already has an
   Atlassian server configured elsewhere, and either one works.
   - If the tool is not available at all, the MCP server is not connected. Tell the customer to restart
     Claude and approve the `atlassian` server, and see the "Connecting Atlassian" section of
     [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md). Stop here — the rest depends on it.
   - If it prompts for browser sign-in, walk them through it. Seamlex never sees their credentials.
   - On success, record the site URL as `{{SITE_URL}}` and the id as `{{CLOUD_ID}}`. If more than one
     site comes back, ask which one Seamlex works in.

4. **Discover, don't interrogate.** Use the connection to propose values instead of asking blind:
   - `getVisibleJiraProjects` → propose `{{JIRA_PROJECT}}`; if several, ask which.
   - `getJiraProjectIssueTypesMetadata` on that project → read the *actual* issue type names and fill
     `{{TYPE_EPIC}}`, `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`. If a type does not exist, set the nearest
     available one and note the substitution in the row's Notes column.
   - `getConfluenceSpaces` → propose `{{CONF_SPACE}}`; `getPagesInConfluenceSpace` → propose
     `{{CONF_PARENT}}`, or use the space root.
   - `atlassianUserInfo` → confirm who the customer is signed in as.

5. **Ask only what you cannot discover.** Use `AskUserQuestion` in small batches for §1 (company,
   industry, program, locale, role), §4 (Seamlex contact, escalation, board URL, cadence) and §5
   (confirmation policy, detail level). Recommend `{{CONFIRM_WRITES}}` = `always` and explain why: no
   issue or page is ever created in their Jira or Confluence without them seeing it first.
   Use [`config/seamlex.config.example.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.example.md) as a
   worked reference.

6. **Verify.** No `<...>` left in `seamlex/config.md`; `seamlex/discovery/` and `seamlex/requests/` exist;
   a read-only probe succeeds — run `searchJiraIssuesUsingJql` with
   `project = {{JIRA_PROJECT}} ORDER BY created DESC` and report how many issues are visible. Confirm
   no secrets were written into the config.

7. **Finish** with a short summary and what to do next:
   - `/seamlex-discovery` — the first working session, if discovery hasn't happened yet
   - `/seamlex-request` — turn an idea into an epic with user stories
   - `/seamlex-ask` — ask the Seamlex team a question
   - `/seamlex-status` — see where your work stands
