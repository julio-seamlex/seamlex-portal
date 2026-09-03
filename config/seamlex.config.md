# Seamlex Portal — configuration

**This file parameterizes the whole plugin.** Every Seamlex command and agent reads it first and resolves
the `{{PLACEHOLDER}}` tokens in its instructions from the **Value** column below.

**It ships blank.** Every value is empty until the first run of
[`/hi-seamlex`](../commands/hi-seamlex.md), which completes it as its very first step — asking the
customer for what only they know, and reading the rest off the live Atlassian site. Once a row has a
value, `/hi-seamlex` leaves it alone; to change one, blank it and re-run, or edit it here directly.

**Where the filled copy lives.** `/hi-seamlex` writes the values back into this file when the plugin
directory is writable. When it is not — the usual case for an installed plugin — it writes the completed
config to `seamlex/seamlex.config.md` in the customer's workspace instead. Every command and agent looks
for that workspace copy first and falls back to this file.

The **Source** column says where each value comes from: `ask` — put to the customer; `atlassian` — read
off the site with the Atlassian MCP server and confirmed with the customer; `default` — the value in
**Notes** is used unless the customer says otherwise.

Safe to commit — it holds no secrets. Authentication to Jira and Confluence happens through the Atlassian
MCP server's own browser login, never through this file.
**Never put an API token or password in here.**

## 1. Who you are

| Key | Placeholder | Value | Source | Notes |
|---|---|---|---|---|
| Company name | `{{COMPANY}}` |  | ask | The customer's company. |
| Industry | `{{INDUSTRY}}` |  | ask | Sector and B2B/B2C, e.g. "Third-party logistics (3PL), B2B". |
| Program name | `{{PROGRAM}}` |  | ask | The programme of work this engagement covers. |
| Locale | `{{LOCALE}}` |  | ask | e.g. `en_US`, `es_ES`. Default `en_US`. |
| Your role | `{{MY_ROLE}}` |  | ask | The customer's own role. |

## 2. Atlassian workspace

| Key | Placeholder | Value | Source | Notes |
|---|---|---|---|---|
| Atlassian site | `{{SITE_URL}}` |  | atlassian | From `getAccessibleAtlassianResources`. |
| Cloud ID | `{{CLOUD_ID}}` |  | atlassian | Same call; cached here so agents skip the lookup. |
| Jira project key | `{{JIRA_PROJECT}}` |  | atlassian | Chosen from `getVisibleJiraProjects`. |
| Confluence space key | `{{CONF_SPACE}}` |  | atlassian | Chosen from `getConfluenceSpaces`. |
| Discovery parent page | `{{CONF_PARENT}}` |  | atlassian | Page ID the Discovery Brief is published under. |

## 3. Issue types and fields

> Names must match the Jira project exactly, so they are read from the project metadata rather than
> guessed. If the project does not have a given type, write `n/a` and the agents will fall back to the
> nearest available type and say so.

| Key | Placeholder | Value | Source | Notes |
|---|---|---|---|---|
| Epic type | `{{TYPE_EPIC}}` |  | atlassian | From `getJiraProjectIssueTypesMetadata`. |
| Story type | `{{TYPE_STORY}}` |  | atlassian | May be localized, e.g. `Historia`. |
| Question type | `{{TYPE_QUESTION}}` |  | atlassian | Where there is no Question type, the type questions are raised as. |
| Request label | `{{LABEL_REQUEST}}` |  | ask | Default `customer-request`. |
| Extra labels | `{{LABELS_EXTRA}}` |  | ask | Optional; blank is fine. |

## 4. Working with Seamlex

> **Seamlex owns this section.** These are delivery-side facts. Everything works without them: questions
> and requests are simply raised unassigned, and status answers skip the board link and the cadence — so
> leave a row blank rather than guessing at it.

| Key | Placeholder | Value | Source | Notes |
|---|---|---|---|---|
| Seamlex contact | `{{SEAMLEX_CONTACT}}` |  | ask | Default assignee for questions and new requests. |
| Escalation contact | `{{ESCALATION}}` |  | ask | For anything urgent or blocked. |
| Status board | `{{BOARD_URL}}` |  | ask | Linked when reporting status. |
| Sprint cadence | `{{CADENCE}}` |  | ask | Sets expectations in status answers, e.g. "2-week sprints, starting Wednesdays". |

## 5. How the agents should behave

| Key | Placeholder | Value | Source | Notes |
|---|---|---|---|---|
| Write confirmation | `{{CONFIRM_WRITES}}` |  | default | `always` (the default) or `summary`. Whether every Jira/Confluence write is shown for approval first. |
| Detail level | `{{DETAIL}}` |  | ask | `business` (plain language, the default) or `technical` (Salesforce terms welcome). |
| Local drafts | `{{DRAFTS_DIR}}` |  | default | Folder in the customer's workspace holding discovery notes and request drafts. Default `seamlex`. |

## What lives in your workspace

```
seamlex/
├── seamlex.config.md           # this config, when the plugin directory is read-only
├── discovery/
│   └── discovery-brief.md      # your discovery notes — resumable across sessions
└── requests/
    └── <slug>.md               # requirement drafts, before they become Jira issues
```

Nothing is written to Jira or Confluence until you approve it, and drafts stay local until then.
