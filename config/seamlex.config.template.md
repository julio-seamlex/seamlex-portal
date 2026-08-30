# Seamlex Portal — Configuration

> **This file parameterizes the whole plugin.** Every Seamlex agent reads it first and resolves the
> `{{PLACEHOLDER}}` tokens in its instructions from the **Value** column below.
>
> You do not edit the agents — they are generic. You only fill this file in.
> Run `/seamlex-setup` and Claude will fill it in with you; you can also edit it by hand at any time.
>
> Lives at `seamlex/config.md` in your workspace. Safe to commit — it holds no secrets.
> Authentication to Jira and Confluence happens through the Atlassian MCP server's own browser login,
> never through this file. **Never put an API token or password in here.**

## 1. Who you are

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Company name | `{{COMPANY}}` | <e.g. Northwind Logistics> | Your company, as it should appear on documents. |
| Industry | `{{INDUSTRY}}` | <e.g. Third-party logistics> | Used to tune discovery questions and examples. |
| Program name | `{{PROGRAM}}` | <e.g. Northwind CRM Transformation> | The Salesforce initiative Seamlex is delivering. |
| Locale | `{{LOCALE}}` | <e.g. en_US> | Language for the artifacts written for you. |
| Your role | `{{MY_ROLE}}` | <e.g. Head of Operations> | Helps agents pitch detail at the right level. |

## 2. Atlassian workspace

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Atlassian site | `{{SITE_URL}}` | <e.g. https://northwind.atlassian.net> | The Jira/Confluence site Seamlex shares with you. |
| Cloud ID | `{{CLOUD_ID}}` | <filled by /seamlex-setup> | Resolved once via `getAccessibleAtlassianResources`; cached here so agents skip the lookup. |
| Jira project key | `{{JIRA_PROJECT}}` | <e.g. NWCRM> | Where your epics, stories, questions and requests live. |
| Confluence space key | `{{CONF_SPACE}}` | <e.g. NWCRM> | Where discovery briefs and reference docs are published. |
| Discovery parent page | `{{CONF_PARENT}}` | <page id or title, or "space root"> | Discovery briefs are created under this page. |

## 3. Issue types and fields

> Names must match your Jira project exactly — `/seamlex-setup` reads them from the project metadata
> rather than guessing. If your project does not have a given type, write `n/a` and the agents will fall
> back to the nearest available type and say so.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Epic type | `{{TYPE_EPIC}}` | <e.g. Epic> | Container for a business capability. |
| Story type | `{{TYPE_STORY}}` | <e.g. Story> | User stories raised from your requirements. |
| Question type | `{{TYPE_QUESTION}}` | <e.g. Task> | How a question to Seamlex is raised. |
| Request label | `{{LABEL_REQUEST}}` | <e.g. customer-request> | Label stamped on everything raised through this plugin. |
| Extra labels | `{{LABELS_EXTRA}}` | <comma-separated, or "none"> | Anything your team wants added, e.g. a release or workstream tag. |

## 4. Working with Seamlex

> **Leave this section blank — Seamlex fills it in.** These are delivery-side facts, not yours to
> look up, and `/seamlex-setup` will not ask you for them. Everything works without them: questions
> and requests are simply raised unassigned, and status answers skip the board link and the cadence.

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Seamlex contact | `{{SEAMLEX_CONTACT}}` | <filled by Seamlex — leave blank> | Default assignee for questions and new requests. Blank: they are raised unassigned. |
| Escalation contact | `{{ESCALATION}}` | <filled by Seamlex — leave blank> | For anything urgent or blocked. Blank: no escalation route is offered. |
| Status board | `{{BOARD_URL}}` | <filled by Seamlex — leave blank> | Linked when reporting status. Blank: no link is shown. |
| Sprint cadence | `{{CADENCE}}` | <filled by Seamlex — leave blank> | Sets expectations in status answers. Blank: no delivery timing is quoted. |

## 5. How the agents should behave

| Key | Placeholder | Value | Notes |
|---|---|---|---|
| Write confirmation | `{{CONFIRM_WRITES}}` | `always` | `always` (recommended) or `summary`. Whether every Jira/Confluence write is shown for approval first. |
| Detail level | `{{DETAIL}}` | <e.g. business> | `business` (plain language) or `technical` (Salesforce terms welcome). |
| Local drafts | `{{DRAFTS_DIR}}` | `seamlex` | Folder in this workspace holding config, discovery notes and request drafts. |

### What lives in your workspace

```
seamlex/
├── config.md                   # this file
├── discovery/
│   └── discovery-brief.md      # your discovery notes — resumable across sessions
└── requests/
    └── <slug>.md               # requirement drafts, before they become Jira issues
```

Nothing is written to Jira or Confluence until you approve it. Drafts stay local until then.
