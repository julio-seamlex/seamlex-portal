---
description: Start a Seamlex session — completes the config on first run, works out where you are in the customer lifecycle, sets the workspace up, and loads only the context that step needs.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Start a Seamlex session

One command to open any working session, including the first one. It fills in the config before anything
else, then answers "where are we?" before you have to, pulls down the context for that step and hands you
to the right command. Run it at the start of every session.

**The session opens with the config.** The first thing this command does — and the first thing the
customer sees — is the config: completing it if it is still blank, then a short summary of the settings
the session will run on.

**The settings live in one file.** [`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md)
is the single source of truth for the whole plugin: every Seamlex command and agent resolves its
`{{PLACEHOLDER}}` tokens from that table. It ships blank; **this command is what completes it** (step 1),
and once a row has a value nothing overwrites it.

**The lifecycle has four steps, in order:** `setup` → `discovery` → `requirement` → `status`. A company
sits at exactly one of them at a time. **Nothing records which one** — no state file is kept. The step is
worked out fresh each session from what is actually there: the workspace folders, the Discovery Brief, and
the board. Evidence cannot go stale the way a recorded step can.

`$ARGUMENTS` may name a step (`setup`, `discovery`, `requirement`/`requerimiento`, `status`) to override
what you infer — use it when the customer wants to jump ahead or go back over something, or to re-run the
setup checks after fixing an Atlassian connection. Say plainly that you are overriding. `$ARGUMENTS` may
also be `config`, which re-opens the config interview for the rows the customer wants to change.

## Steps

1. **Complete the config — before anything else.** This is the very first thing you do in every session,
   ahead of any lifecycle inference, any folder check, any other output.

   1. **Read the config.** Look for `{{DRAFTS_DIR}}/seamlex.config.md` in the customer's workspace first
      (try `seamlex/seamlex.config.md`, the default), then fall back to
      `${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md`. Whichever you find first is the config for this
      session.

   2. **If every value is filled, skip to sub-step 5.** A filled config is never re-asked and never
      overwritten. The exception is `/hi-seamlex config`, which asks which rows to change and reruns the
      interview for those alone.

   3. **Fill the blank rows.** Work through the table by the **Source** column, cheapest first:
      - `atlassian` rows — resolve them from the live site rather than asking. Call
        `getAccessibleAtlassianResources` for `{{SITE_URL}}` and `{{CLOUD_ID}}`, `getVisibleJiraProjects`
        for `{{JIRA_PROJECT}}`, `getConfluenceSpaces` for `{{CONF_SPACE}}`, `getPagesInConfluenceSpace`
        for `{{CONF_PARENT}}`, and `getJiraProjectIssueTypesMetadata` for `{{TYPE_EPIC}}`,
        `{{TYPE_STORY}}` and `{{TYPE_QUESTION}}` — taking the type names exactly as the project spells
        them, which may not be English. Where a call returns exactly one candidate, take it and say so.
        Where it returns several, put the real options to the customer with `AskUserQuestion` rather than
        picking. Where there is no matching issue type at all, write `n/a`.
      - `ask` rows — put them to the customer with `AskUserQuestion`, grouped so this is two or three
        questions, not fifteen. Offer the **Notes** default as the first option where there is one. §4 is
        Seamlex-owned: say that blank is fine there and leave it blank rather than guessing.
      - `default` rows — take the default named in **Notes** without asking.
      Tool names are namespaced by the plugin's server — e.g.
      `mcp__plugin_seamlex-portal_atlassian__getAccessibleAtlassianResources` — so match on the base name
      after the last `__`; the prefix differs if the customer already has an Atlassian server configured
      elsewhere, and either one works.

      > If the Atlassian tools are unavailable, do not fail and do not invent values. Ask the customer for
      > the `atlassian` rows they know, leave the rest blank, say plainly which rows are unverified, and
      > offer to rerun `/hi-seamlex setup` once the connection is back.

   4. **Write the completed config back.** Edit the **Value** cells in place, changing nothing else — the
      headings, notes and **Source** column stay as they are. Try
      `${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md` first; if the plugin directory is not writable
      (the usual case for an installed plugin), write the completed file to
      `{{DRAFTS_DIR}}/seamlex.config.md` in the customer's workspace instead and say which of the two you
      wrote. Show the values for approval before writing.

   5. **Summarize the config.** Your **first message about the session** is a summary of that table and
      nothing else: who they are (`{{COMPANY}}`, `{{PROGRAM}}`, `{{MY_ROLE}}`, `{{LOCALE}}`), the
      Atlassian workspace (`{{SITE_URL}}`, `{{JIRA_PROJECT}}`, `{{CONF_SPACE}}`), the issue types and
      labels in use (`{{TYPE_EPIC}}`, `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`, `{{LABEL_REQUEST}}`), who
      they work with at Seamlex (`{{SEAMLEX_CONTACT}}`, `{{ESCALATION}}`, `{{CADENCE}}`), and how the
      agents will behave (`{{CONFIRM_WRITES}}`, `{{DETAIL}}`, `{{DRAFTS_DIR}}`). Keep it short — a handful
      of lines or a small table — and pitch it at `{{DETAIL}}`. Say where the file lives and that
      `/hi-seamlex config` changes any of it.

   Only after that summary is delivered do you move on to step 2.

2. **Work out which step they are on**, from the cheapest signal upward. Stop at the first that answers.
   - `{{DRAFTS_DIR}}/` does not exist → **`setup`**. The workspace has never been set up: go to step 3 and
     run it now, in this session. Do not send the customer off to another command.
   - No `{{DRAFTS_DIR}}/discovery/discovery-brief.md`, or one whose nine sections are not all covered →
     **`discovery`**.
   - The brief is complete and the board shows `{{JIRA_PROJECT}}` issues labelled `{{LABEL_REQUEST}}` in
     flight — assigned, in progress, or recently updated → **`status`**.
   - The brief is complete and there is no such work moving yet → **`requirement`**.
   Where the signals genuinely conflict — a complete brief and an empty board and a workspace full of
   request drafts, say — show what you saw and ask with `AskUserQuestion` rather than silently picking.
   Say which signals you read, so the customer can correct you in one line if you land wrong.

3. **If the step is `setup`, set the workspace up.** Confirm the Atlassian connection works, check the
   config against the live site, and create the local folders the other commands write into. Run this once
   per workspace; `/hi-seamlex setup` re-runs it on demand.

   1. **Check the Atlassian connection.** Call `getAccessibleAtlassianResources` (matching the base name
      after the last `__`, as in step 1).
      - If the tool is not available at all, the MCP server is not connected. Tell the customer to restart
        Claude and approve the `atlassian` server, and see the "Connecting Atlassian" section of
        [`SETUP.md`](${CLAUDE_PLUGIN_ROOT}/SETUP.md). Still do sub-steps 3 and 4 — the folders are useful
        offline — and say plainly that nothing was verified against Atlassian this session.
      - If it prompts for browser sign-in, walk them through it. Seamlex never sees their credentials.
      - Confirm the site that comes back matches `{{SITE_URL}}` and `{{CLOUD_ID}}`. If it does not, stop
        and report the difference: the config points at a site this user cannot reach, and every later
        write would land in the wrong place or fail.

   2. **Verify the config against the live site.** These are all reads; do them without asking.
      - `atlassianUserInfo` — confirm who the customer is signed in as.
      - `getVisibleJiraProjects` — confirm `{{JIRA_PROJECT}}` is visible to them.
      - `getJiraProjectIssueTypesMetadata` on that project — confirm the issue type names
        (`{{TYPE_EPIC}}`, `{{TYPE_STORY}}`, `{{TYPE_QUESTION}}`) actually exist. Where one does not, name
        it, name the nearest available type the agents will fall back to, and offer to correct that row in
        the config.
      - `getConfluenceSpaces` — confirm `{{CONF_SPACE}}` exists, and `getConfluencePage` on
        `{{CONF_PARENT}}` that the discovery parent page is reachable.
      - `searchJiraIssuesUsingJql` with `project = {{JIRA_PROJECT}} ORDER BY created DESC` — a read-only
        probe; report how many issues are visible.
      Report each check as pass or mismatch. A mismatch is a finding about the config: offer to fix the
      row now — `/hi-seamlex config` does the same later — rather than working around it.

   3. **Create the local working folders.**
      `mkdir -p {{DRAFTS_DIR}}/discovery {{DRAFTS_DIR}}/requests`
      These hold drafts only — discovery notes and requirement drafts, before they become Jira issues.

   Once the folders exist, setup is done and the step is `discovery` — carry on into step 4 with that,
   rather than ending the session on a bare setup report.

   > If the Atlassian tools become unavailable partway through setup, do not fail. Nothing here depends on
   > a write succeeding; say which checks did not run and offer to rerun `/hi-seamlex setup` once the
   > connection is back.

4. **Load the context that step needs — and only that step's.** Pulling everything is slow and buries the
   thing they came for.

   | Step | What to load |
   |---|---|
   | `setup` | Nothing further. The config *is* the context. |
   | `discovery` | `{{DRAFTS_DIR}}/discovery/discovery-brief.md`, plus the published brief in `{{CONF_SPACE}}` if there is one. Note which of the nine sections are complete and which are open. |
   | `requirement` | The Discovery Brief (the pains and actors every story anchors to), local drafts in `{{DRAFTS_DIR}}/requests/`, and open `{{TYPE_EPIC}}` issues in `{{JIRA_PROJECT}}` carrying `{{LABEL_REQUEST}}` — so a new requirement can be checked against them for duplicates. |
   | `status` | Everything in `{{JIRA_PROJECT}}` labelled `{{LABEL_REQUEST}}` via `searchJiraIssuesUsingJql`, ordered by updated, with assignee and status. Do not summarize it here — that is the liaison's job. |

   Every read is read-only. Apart from the config it completes in step 1 and the two empty folders in step
   3, this command writes nothing — not to the workspace, not to Jira, not to Confluence. If the Atlassian
   tools are unavailable, do not fail: say clearly which signals could not be checked and carry on from
   the local files alone.

5. **Report, then hand off.** Short and concrete: which step, the signals you read to land on it, what
   context you loaded, and the one command to run next — `/seamlex-discovery`, `/seamlex-request` or
   `/seamlex-status`. If the customer already said what they came to do, run that command's agent now
   rather than making them type it.

> The step is a statement about the engagement — "discovery is done, we are taking requirements now" — so
> when you name it, name it as a reading of the evidence rather than a fact. Going *back* a step is fine
> and sometimes right; a second discovery round after a reorg is a real thing, not a mistake, and
> `/hi-seamlex discovery` is all it takes.

## Configuration

The config table itself lives in
[`config/seamlex.config.md`](${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md) — with the completed copy at
`{{DRAFTS_DIR}}/seamlex.config.md` in the workspace where the plugin directory is read-only. That file
carries the placeholders, the notes on each row, and the **Source** column step 1 works from. It holds no
secrets: authentication to Jira and Confluence happens through the Atlassian MCP server's own browser
login. **Never put an API token or password in it.**
