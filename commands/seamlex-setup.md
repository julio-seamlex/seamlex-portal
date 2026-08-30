---
description: Connect this workspace to Seamlex — verify the Atlassian connection, adopt your company's shared config from Confluence if it exists, otherwise create seamlex/config.md interactively and publish it.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Set up the Seamlex Portal

Produce `seamlex/config.md` — the single file every Seamlex agent reads — and confirm the Atlassian
connection works. Run this once per workspace, before anything else.

**The config is company-wide, not per-user.** It lives once in the customer's Confluence space, on a page
carrying the label `seamlex-portal-config`, and that page is the source of truth. The first person at a
company creates it; everyone after them adopts it. So: look in Confluence *before* creating anything, and
publish back at the end.

## Steps

1. **Check the Atlassian connection.** Call `getAccessibleAtlassianResources`. Its full name is
   namespaced by the plugin's server — `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources`
   — so match on the base name after the last `__`; the prefix differs if the customer already has an
   Atlassian server configured elsewhere, and either one works.
   - If the tool is not available at all, the MCP server is not connected. Tell the customer to restart
     Claude and approve the `atlassian` server, and see the "Connecting Atlassian" section of
     [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md). You can still work **offline**: skip steps 2, 3 and 7,
     create the config from the template, ask for everything, and say plainly that the shared config could
     not be checked and nothing will be published this session.
   - If it prompts for browser sign-in, walk them through it. Seamlex never sees their credentials.
   - On success, record the site URL as `{{SITE_URL}}` and the id as `{{CLOUD_ID}}`. If more than one
     site comes back, ask which one Seamlex works in.

2. **Resolve the space to look in.** `getConfluenceSpaces` → propose `{{CONF_SPACE}}`; if several come
   back, ask which one. If `seamlex/config.md` already exists locally, read its `{{CLOUD_ID}}` and
   `{{CONF_SPACE}}` and use those instead of asking again.

3. **Look for your company's published config.** `searchConfluenceUsingCql` with
   `label = "seamlex-portal-config" AND space = "{{CONF_SPACE}}"`, passing `{{CLOUD_ID}}`. If the space is
   still ambiguous, search on the label alone and show the customer the hits so they can pick.

   **Found — this is not the first session for this company.** Fetch the page with `getConfluencePage`,
   extract the config (see *Page format* below), then
   `mkdir -p seamlex/discovery seamlex/requests` and write it to `seamlex/config.md`.
   - **No local config yet** — write it, then report which values came from Confluence, and who last
     updated the page and when.
   - **A local config exists and differs** — **Confluence wins.** Show a short row-by-row diff of what
     will change, copy the current file to `seamlex/config.md.local.bak` so nothing is lost, then write
     the imported version. Say that the page is the source of truth and that any refinement they want to
     keep gets published back in step 7.
   - **Never re-ask what the page already answers.** After importing, only rows still holding `<...>`
     are open — fill them by discovery (step 4) first, then by asking (step 5). If none are left, go
     straight to step 6.
   - If more than one labelled page comes back, list them and ask which is current; do not merge them.

   **Not found — this is the first session for this company.** Create the config from the shipped
   template:
   `mkdir -p seamlex/discovery seamlex/requests && cp "${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.template.md" seamlex/config.md`
   (if `seamlex/config.md` already exists, keep it and just fill its remaining `<...>` rows — never
   overwrite a config the customer has filled in when there is no page to defer to).

4. **Discover, don't interrogate.** For rows still unresolved, use the connection to propose values
   instead of asking blind:
   - `getVisibleJiraProjects` → propose `{{JIRA_PROJECT}}`; if several, ask which.
   - `getJiraProjectIssueTypesMetadata` on that project → read the *actual* issue type names and fill
     `{{TYPE_EPIC}}`, `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`. If a type does not exist, set the nearest
     available one and note the substitution in the row's Notes column.
   - `getPagesInConfluenceSpace` → propose `{{CONF_PARENT}}`, or use the space root.
   - `atlassianUserInfo` → confirm who the customer is signed in as.

5. **Ask only what you cannot discover.** Use `AskUserQuestion` in small batches for the §1 rows
   (company, industry, program, locale, role) and §5 rows (confirmation policy, detail level) that are
   still `<...>`. Recommend `{{CONFIRM_WRITES}}` = `always` and explain why: no issue or page is ever
   created in their Jira or Confluence without them seeing it first. Remember these answers are recorded
   company-wide, so ask for the company's position, not a personal preference.
   **Never ask about §4** (Seamlex contact, escalation, board URL, cadence) — Seamlex completes those
   later, and they usually arrive already filled from the shared page. Leave the rows as they are and say
   so in passing, so the blanks don't look like an oversight.
   Use [`config/seamlex.config.example.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.example.md) as a
   worked reference.

6. **Verify.** No `<...>` left in `seamlex/config.md` outside §4, whose rows are expected to stay as
   `<filled by Seamlex — leave blank>`; `seamlex/discovery/` and `seamlex/requests/` exist;
   a read-only probe succeeds — run `searchJiraIssuesUsingJql` with
   `project = {{JIRA_PROJECT}} ORDER BY created DESC` and report how many issues are visible. Confirm
   no secrets were written into the config.

7. **Publish the config to Confluence**, so the next colleague — and any later refinement — starts from
   it instead of from the template. Ask for explicit approval first, showing the exact page title and
   space: this is a write, and `{{CONFIRM_WRITES}}` defaults to `always`.
   - **No page existed** — `createConfluencePage` in space `{{CONF_SPACE}}` under `{{CONF_PARENT}}`
     (the space root if unset), titled `Seamlex Portal Config — {{COMPANY}}`, passing `{{CLOUD_ID}}`.
     Then apply the label `seamlex-portal-config`. **The label is what the next workspace searches on**,
     so if no available tool can add a label, say so plainly and ask the customer to add it to the page
     by hand before their colleagues run setup.
   - **A page existed and something changed** — `updateConfluencePage` on that same page id. Never create
     a second page. If nothing changed, skip the write and say so.
   - Give the customer the page URL.

8. **Finish** with a short summary — including the shared config page URL, and that colleagues running
   `/seamlex-setup` in their own workspace will now pick it up automatically — and what to do next:
   - `/seamlex-discovery` — the first working session, if discovery hasn't happened yet
   - `/seamlex-request` — turn an idea into an epic with user stories
   - `/seamlex-ask` — ask the Seamlex team a question
   - `/seamlex-status` — see where your work stands

## Page format

The page holds the config file verbatim, so it round-trips without loss: one short intro line saying what
the page is and that it is maintained by `/seamlex-setup`, then the entire contents of `seamlex/config.md`
inside a fenced ` ```markdown ` code block. Publish it that way, and read that code block back on import.

If the code block is missing — someone edited the page into plain Confluence tables — reconstruct the rows
from the rendered tables instead, and tell the customer the page will be re-normalised into a code block
the next time it is published.

> If the Atlassian tools become unavailable partway through, do not fail the command. The config is
> already saved locally; say the connection is not up, that the shared page was not read or written, and
> offer to rerun `/seamlex-setup` once it is back.
