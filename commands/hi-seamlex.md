---
description: Start a Seamlex session — works out where you are in the customer lifecycle from your company's Confluence config and loads only the context that step needs.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Start a Seamlex session

One command to open any working session. It answers "where are we?" before you have to, then pulls down
the context for that step and hands you to the right command. Run it at the start of every session — it is
cheap, and it is the only way to be sure you are working from what the rest of the company sees.

**The lifecycle has four steps, in order:** `setup` → `discovery` → `requirement` → `status`. A company
sits at exactly one of them at a time, and the step is recorded in §6 of the company-wide config page in
Confluence — the page labelled `seamlex-portal-config`, the same one `/seamlex-setup` publishes. That page
is the source of truth, not the local file and not this conversation.

`$ARGUMENTS` may name a step (`setup`, `discovery`, `requirement`/`requerimiento`, `status`) to override
what the config says — use it when the customer wants to jump ahead or go back over something. Say plainly
that you are overriding the recorded step, and do not rewrite §6 just because of an override.

## Steps

1. **Refresh the config from Confluence.** This is a read, so do it without asking.
   - No `seamlex/config.md` locally → the company is at step `setup`. Stop here and run `/seamlex-setup`;
     everything below needs a config.
   - Otherwise read `{{CLOUD_ID}}` and `{{CONF_SPACE}}` from it, then `searchConfluenceUsingCql` with
     `label = "seamlex-portal-config" AND space = "{{CONF_SPACE}}"` and `getConfluencePage` on the hit.
     Extract the config from the four-backtick code block (see *Page format* in
     [`/seamlex-setup`](${CLAUDE_PLUGIN_ROOT}/commands/seamlex-setup.md)) and write it to
     `seamlex/config.md`, backing up a differing local file to `seamlex/config.md.local.bak` first.
   - **Atlassian unavailable, or no page found** — do not fail. Fall back to the local config, say clearly
     that the recorded step could not be read and that what follows is inferred from local files only, and
     carry on from step 2 with the local signals.

2. **Determine the step.** Take §6 `{{STAGE}}` from the page as the answer. Then sanity-check it against
   what you can actually see, because a stale §6 is the common failure:
   - Any `<...>` left outside §4 → `setup`, whatever §6 says.
   - `{{STAGE}}` is `discovery` but the Discovery Brief page is complete → propose `requirement`.
   - `{{STAGE}}` is `requirement` and the board shows work in flight → propose `status`.
   - §6 is missing entirely (a config published before this section existed) → infer the step from the
     signals in step 3, tell the customer §6 is absent, and offer to record it in step 5.
   Where the recorded step and the evidence disagree, show both and ask with `AskUserQuestion` — never
   silently pick.

3. **Load the context that step needs — and only that step's.** Pulling everything is slow and buries the
   thing they came for.

   | Step | What to load |
   |---|---|
   | `setup` | Nothing further. The config *is* the context. |
   | `discovery` | `seamlex/discovery/discovery-brief.md`, plus the published brief in `{{CONF_SPACE}}` if there is one. Note which of the nine sections are complete and which are open. |
   | `requirement` | The Discovery Brief (the pains and actors every story anchors to), local drafts in `seamlex/requests/`, and open `{{TYPE_EPIC}}` issues in `{{JIRA_PROJECT}}` carrying `{{LABEL_REQUEST}}` — so a new requirement can be checked against them for duplicates. |
   | `status` | Everything in `{{JIRA_PROJECT}}` labelled `{{LABEL_REQUEST}}` via `searchJiraIssuesUsingJql`, ordered by updated, with assignee and status. Do not summarize it here — that is the liaison's job. |

   Every read is read-only. Nothing in this command writes to Jira, and the only Confluence write is the
   optional one in step 5.

4. **Report, then hand off.** Short and concrete: which step, what it was read from (the page, with its URL
   and who last updated it, or "local files only"), what context you loaded, and the one command to run
   next — `/seamlex-setup`, `/seamlex-discovery`, `/seamlex-request` or `/seamlex-status`. If the customer
   already said what they came to do, run that command's agent now rather than making them type it.

5. **Record a step change, if one happened.** Only when the step you settled on differs from §6 — and only
   with explicit approval, since `{{CONFIRM_WRITES}}` defaults to `always`. Update §6 in
   `seamlex/config.md` (`{{STAGE}}`, `{{STAGE_UPDATED}}` = today, `{{STAGE_NOTE}}` = one line on why), then
   `updateConfluencePage` on that same page id — never create a second page. If they decline, leave both
   alone and say the recorded step stays where it was.

> Advancing the step is a company-wide statement — "discovery is done, we are taking requirements now" —
> so make what it means clear before asking. Moving *back* a step is fine and sometimes right; a second
> discovery round after a reorg is a real thing, not a mistake.
