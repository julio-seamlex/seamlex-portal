# Seamlex Portal — Configuration

> **This file parameterizes the whole plugin.** Every Seamlex command and agent reads it first and
> resolves the `{{PLACEHOLDER}}` tokens in its instructions from the **Value** column below.
>
> **It is fixed and ships with the plugin.** Nothing generates it, nothing overwrites it, and no copy is
> made in the customer's workspace — the agents read it in place at
> `${CLAUDE_PLUGIN_ROOT}/config/seamlex.config.md`. To change how the plugin behaves for a customer, edit
> this file in the plugin and ship it; `/seamlex-setup` only checks these values against the live
> Atlassian site and reports any mismatch.
>
> The only state kept per-workspace is the lifecycle step, in `seamlex/state.md`. Discovery notes and
> request drafts live under `seamlex/` too, and never come back into this file.
>
> Safe to commit — it holds no secrets. Authentication to Jira and Confluence happens through the
> Atlassian MCP server's own browser login, never through this file.
> **Never put an API token or password in here.**

## 1. Who you are

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Company name | `{{COMPANY}}` | Northwind Logistics | Demo customer. |
| Industry | `{{INDUSTRY}}` | Third-party logistics (3PL), B2B | |
| Program name | `{{PROGRAM}}` | Northwind CRM Transformation | Sales + Service Cloud, phase 1. |
| Locale | `{{LOCALE}}` | en_US | |
| Your role | `{{MY_ROLE}}` | Head of Commercial Operations | |

## 2. Atlassian workspace

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Atlassian site | `{{SITE_URL}}` | https://seamlex.atlassian.net | |
| Cloud ID | `{{CLOUD_ID}}` | 87285af3-bdbc-4c75-bc0c-2ec7805668fb | Resolved once via `getAccessibleAtlassianResources`; cached here so agents skip the lookup. |
| Jira project key | `{{JIRA_PROJECT}}` | SCRUM | My Software Team. |
| Confluence space key | `{{CONF_SPACE}}` | MST | |
| Discovery parent page | `{{CONF_PARENT}}` | 327858 | My Software Team home. |

## 3. Issue types and fields

> Names must match the Jira project exactly. `/seamlex-setup` reads the project metadata and reports any
> row that does not match, so a wrong value is caught before the first issue is raised. If the project does
> not have a given type, write `n/a` and the agents will fall back to the nearest available type and say so.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Epic type | `{{TYPE_EPIC}}` | Epic | |
| Story type | `{{TYPE_STORY}}` | Historia | This project's issue types are in Spanish. |
| Question type | `{{TYPE_QUESTION}}` | Tarea | No Question type in this project — questions are raised as Tarea. |
| Request label | `{{LABEL_REQUEST}}` | customer-request | |
| Extra labels | `{{LABELS_EXTRA}}` | phase-1 | |

## 4. Working with Seamlex

> **Seamlex owns this section.** These are delivery-side facts, filled in when the plugin is shipped to a
> customer. Everything works without them: questions and requests are simply raised unassigned, and status
> answers skip the board link and the cadence.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Seamlex contact | `{{SEAMLEX_CONTACT}}` | Ana Rivas (Delivery Lead) | Default assignee for questions and new requests. |
| Escalation contact | `{{ESCALATION}}` | delivery@seamlex.com | For anything urgent or blocked. |
| Status board | `{{BOARD_URL}}` | https://seamlex.atlassian.net/jira/software/projects/SCRUM/boards/1 | Linked when reporting status. |
| Sprint cadence | `{{CADENCE}}` | 2-week sprints, starting Wednesdays | Sets expectations in status answers. |

## 5. How the agents should behave

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Write confirmation | `{{CONFIRM_WRITES}}` | `always` | `always` (recommended) or `summary`. Whether every Jira/Confluence write is shown for approval first. |
| Detail level | `{{DETAIL}}` | business | `business` (plain language) or `technical` (Salesforce terms welcome). |
| Local drafts | `{{DRAFTS_DIR}}` | `seamlex` | Folder in the customer's workspace holding lifecycle state, discovery notes and request drafts. |

### What lives in your workspace

```
seamlex/
├── state.md                    # which lifecycle step you are on — the only local state
├── discovery/
│   └── discovery-brief.md      # your discovery notes — resumable across sessions
└── requests/
    └── <slug>.md               # requirement drafts, before they become Jira issues
```

This file is not among them: it stays in the plugin. Nothing is written to Jira or Confluence until you
approve it, and drafts stay local until then.