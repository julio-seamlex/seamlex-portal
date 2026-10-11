---
description: Show where the project stands as a live Artifact — the open Jira sprint and its remaining tasks, the completeness of the GitHub documentation repo, and the key missing tasks, warnings and dependencies. Read-only; renders the project-status template in the Seamlex look.
allowed-tools: Read, Write, Skill, Artifact, mcp__atlassian__searchJiraIssuesUsingJql, mcp__atlassian__getJiraIssue, mcp__atlassian__getAccessibleAtlassianResources, mcp__github__get_file_contents, mcp__github__list_commits, mcp__github__search_code
---

# Project status

**Before Step 1, load the `seamlex-portal:project-status` skill with the `Skill` tool** (if it is not
available, read `../skills/project-status/SKILL.md`). The skill decides **what counts** as a missing
task, a warning or a dependency and **how the Artifact looks**; this command decides the order of work.
The house rules for Jira and GitHub are `seamlex-portal:delivery-how-to`, loaded by `/hi-seamlex`.

This command **only reads**. It never creates, edits, comments on or transitions anything in Jira or GitHub.

Settings come from the **`config.yml` and `README.md` of the customer's GitHub repo** that `/hi-seamlex`
loaded. If they are not in context, stop and ask the customer to run `/hi-seamlex`. Resolve
`{{JIRA_PROJECT}}`, `{{PROGRAM}}`, `{{COMPANY}}`; `{{CLOUD_ID}}`, `{{GITHUB_ORG}}` and `{{GITHUB_REPO}}`
are what `/hi-seamlex` settled on. If the Atlassian or GitHub tools are unavailable, say which and stop.

## Step 1 — Sprint

`searchJiraIssuesUsingJql` on `{{CLOUD_ID}}`:
```
project = {{JIRA_PROJECT}} AND sprint in openSprints() ORDER BY Rank ASC
```
Ask for summary, status, issuetype, parent, assignee, labels, duedate, updated, sprint, issuelinks.
Count by status category; list what is not Done. No open sprint → say so, and go on to Step 2.

## Step 2 — Documentation repo

Read the files the skill lists (paths from `config.yml`, else the default tree) with `get_file_contents`;
list `relevamientos/`; `list_commits` for the last activity. A file that is not there is a finding, not an
error.

## Step 3 — Derive

Apply the skill's rules for key missing tasks, warnings, dependencies and the overall status. Every row
cites its Jira key or GitHub path.

## Step 4 — Show it

Fill `skills/project-status/references/project-status-template.md`, render the page as the skill says and
publish it with `Artifact` (`open` it on the first publish; on a re-run, the same `file_path`). Then reply
in chat with at most three lines: overall status, sprint progress, and the single most important warning,
plus the link. `$ARGUMENTS` may be a sprint name or board to report on instead of the open sprint.
