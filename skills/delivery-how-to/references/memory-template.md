# Root index — template

> `README.md` plus `config.yml` together are the root index of the customer's GitHub documentation
> repo, split into a human-readable map and a machine-readable settings file. Files are found by their
> fixed path under the tree `delivery-how-to/SKILL.md` defines — never by a label.

---

## `README.md`

```markdown
# <Company> — Seamlex delivery docs

Maintained by Seamlex. Do not edit `config.yml` without checking with your Seamlex contact.

## Purpose

<One paragraph. What this repo indexes — the settings the Seamlex commands read and the files that
hold the project's knowledge for {{PROGRAM}} at {{COMPANY}}.>

## Map

| Path | What it holds | Read when |
|---|---|---|
| `config.yml` | Settings the commands read `{{PLACEHOLDER}}`s from | every session |
| `scope.md` | Epics and items in scope, exclusions, assumptions, phases | every session |
| `discovery/discovery-brief.md` | §3 areas and roles, §4 processes, §6 actores, §7 dolores, §9 alcance y restricciones | every session |
| `relevamientos/` | One `<KEY>.md` + `<KEY>-transcript.md` per relevamiento; the two or three most recent are read in full | every session |
| `glosario.md` | The customer's vocabulary; used instead of the platform's | every session |
| `features-no-identificados.md` | What is already parked outside the signed scope, so it is not repeated | every session |

## Maintenance

| | |
|---|---|
| **Maintained by** | <name, Seamlex> |
| **Last updated** | <YYYY-MM-DD> — <what changed> |
```

## `config.yml`

```yaml
# Settings the Seamlex plugin resolves its {{PLACEHOLDER}} tokens from.
# A setting that does not apply to this engagement is a missing key, never a blank value.

JIRA_PROJECT: ABC              # Jira project key of the engagement
TYPE_TASK: Subtarea            # the project's sub-task type, used for pending items
LABEL_REQUEST: cliente         # Jira label on every issue the plugin creates
LABELS_EXTRA: fase-1           # extra Jira labels on every issue the plugin creates (optional)
CONFIRM_WRITES: always         # show and approve every write before it happens
DETAIL: business               # level of language for the customer: business / technical
DRAFTS_DIR: seamlex            # local workspace folder for discovery notes
PROGRAM: <program name>        # the program's name as the customer says it
COMPANY: <company name>        # the customer
```

No plugin release is needed to change either file — an edit to the customer's own repo is enough.
Seamlex maintains both; the plugin never edits `config.yml` and edits `README.md`'s Map table only to
flag it as stale, never to rewrite it silently.

---

## Section folder — e.g. `relevamientos/`

No separate index file is needed for a folder: GitHub's own directory listing (`get_file_contents` on
`relevamientos/`) already shows every file in it, sorted by name — which, because filenames are Jira
keys, sorts usefully on its own. `README.md`'s Map table says what the folder is *for*; the folder
itself says what is *in* it.
