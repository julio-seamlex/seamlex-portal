# Epic — <exact title as it appears on the signed scope page>

> One page per epic of the signed contract scope. It is created when refinement starts and grows until it
> carries everything design and implementation need. The title matches the contract epic **exactly** —
> that is what makes it traceable; if the contract wording is poor, keep it anyway and add a plain-language
> subtitle below.

| | |
|---|---|
| **Program** | {{PROGRAM}} |
| **Contract epic** | <exact title on the signed scope page, and its section or item number> |
| **Signed scope page** | <link to {{CONF_SCOPE_PAGE}}> |
| **Discovery anchor** | <pain P# and/or goal from the Discovery Brief> |
| **Refined with** | <name, role, per person> |
| **Session date(s)** | <date(s)> |
| **Priority** | Must / Should / Could / Won't (this phase) |
| **Status** | Draft / Ready for design |
| **Jira key** | <filled when the epic is raised> |

> Anything not answered is `⚠️ TBD — <the open question>` with an owner. Gaps are findings, not blanks.

---

## What this epic delivers

<Three to six sentences a Seamlex architect could read alone: what becomes true for the business, for
whom, and why it is in the contract. No Salesforce terms.>

## The pain it resolves

<From discovery §7: what goes wrong today, who feels it, what it costs. Quote the customer where it lands.>

## Success measure

| Measure | Baseline today | Target | How we'll know | Owner |
|---|---|---|---|---|
| | | | | |

> If this table cannot be filled, the epic is not ready. Say so rather than inventing a number.

## Actors

| Actor | Area / role (discovery §3) | What they do in this capability |
|---|---|---|
| | | |

## Processes in scope

> The processes from discovery §4 this epic touches, and the delta being asked for.

### <process name>
- **Today:** <how it runs now>
- **After delivery:** <what changes>
- **Volume and frequency:**
- **Exceptions:**

## Functional detail

> The heart of the refinement: what must be possible, and under which rules. Enough that an architect makes
> design decisions and a developer makes build decisions without coming back to ask.

### <capability area>
- **What must be possible:**
- **Rules, thresholds and approvals:**
- **Statuses and transitions:** <the states the thing moves through, and who moves it>
- **Notifications:** <who is told, when, through what>
- **What happens when it goes wrong:**

## Data

- **What is created, read or changed:**
- **Where it comes from today:**
- **Required vs. optional, and validation rules:**
- **Retention, history and audit:**
- **Quality and migration concerns:**

## Visibility and permissions

<Who must see this, who must not, who can edit versus only read. Cheap to decide now, expensive to
retrofit.>

## Reporting

<What someone will want to measure about this later, and at what grain.>

## Integrations and dependencies

<Systems, other epics, decisions or third parties this waits on. Name the direction and the trigger of each
integration, not the mechanism — the architect chooses that.>

## Scope boundaries

**In scope**
-

**Explicitly out of scope**
-

**Assumptions**
-

## Key user stories

> The three to seven that carry the epic's value, not an exhaustive decomposition. Each is drafted with
> `user-story-template.md` and linked to this epic once raised.

| # | As a … I want … so that … | Priority | Jira key |
|---|---|---|---|
| 1 | | | |

## Open questions

| ⚠️ | Question | Who can answer | Needed by |
|---|---|---|---|
| | | | |

## Raised here, outside this epic

<Anything the customer raised during this session that belongs to another contract epic, or to no epic at
all. Cross-reference the entry in "Features no identificados". Do not resolve it here.>

---

## Ready for design

The epic moves to **Ready for design** only when every line below is true. Until then it stays `Draft` and
shows as `en progreso` on the epic list.

- [ ] The title matches the contract epic, and the contract item it covers is named in the header.
- [ ] What it delivers, the pain and the success measure are filled — no invented numbers.
- [ ] Every actor is a real role from discovery §3, not "a user".
- [ ] Every process in scope states today, after delivery, volume and exceptions.
- [ ] Functional detail covers rules, statuses, notifications and the failure path.
- [ ] Data, visibility, reporting and integrations are answered or explicitly marked out of scope.
- [ ] In scope and out of scope are both written down.
- [ ] The key user stories exist with Given/When/Then acceptance criteria.
- [ ] No `⚠️ TBD` remains that would block design — any that stay have an owner and a date.
- [ ] The customer has seen the summary and approved it.
