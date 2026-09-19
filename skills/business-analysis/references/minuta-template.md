# Minuta body — <task summary as it reads in Jira>

> The body of a `Minuta <KEY> — <task>` Confluence page, placed under the header table
> `seamlex-refinar.md` §4a builds (**Tarea Jira**, **Sprint**, **Épica**, **Relevado con**,
> **Fecha(s)**, **Estado**). Adapted to the task, never copied section for section — a small task will
> leave several sections short or explicitly marked not applicable.

> Anything not answered is `⚠️ TBD — <the open question>` with an owner. Gaps are findings, not blanks.

---

## What this task delivers

<Three to six sentences a Seamlex architect could read alone: what becomes true for the business, for
whom, and why it matters. No Salesforce terms.>

## The pain it resolves

<From discovery §7: what goes wrong today, who feels it, what it costs. Quote the customer where it lands.>

## Success measure

| Measure | Baseline today | Target | How we'll know | Owner |
|---|---|---|---|---|

> If this table cannot be filled, say so rather than inventing a number.

## Actors

| Actor | Area / role (discovery §3) | What they do in this capability |
|---|---|---|

## Processes in scope

> The processes from discovery §4 this task touches, and the delta being asked for.

### <process name>
- **Today:** <how it runs now>
- **After delivery:** <what changes>
- **Volume and frequency:**
- **Exceptions:**

## Functional detail

> The heart of the refinement: what must be possible, and under which rules. Enough that an architect
> makes design decisions and a developer makes build decisions without coming back to ask.

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

## Visibility and permissions

<Who must see this, who must not, who can edit versus only read. Cheap to decide now, expensive to
retrofit.>

## Reporting

<What someone will want to measure about this later, and at what grain.>

## Integrations and dependencies

<Systems, other tasks, decisions or third parties this waits on. Name the direction and the trigger of
each integration, not the mechanism — the architect chooses that.>

## Scope boundaries

**In scope**
-

**Explicitly out of scope**
-

**Assumptions**
-

## Open questions

| ⚠️ | Question | Who can answer | Needed by |
|---|---|---|---|

## Raised here, outside this task

<Anything the customer raised during this session that belongs to another task, or to no task at all.
Cross-reference the entry in "Features no identificados". Do not resolve it here.>

---

## Ready for design

`seamlex-refinar.md`'s closing step walks this list from a fresh read of the page before marking the
relevamiento `Finalizado`. Any line not true keeps the task `En progreso`.

- [ ] The title is `Minuta <KEY> — <task summary>`, and the Jira key in the header links to the issue.
- [ ] What it delivers, the pain and the success measure are filled — no invented numbers.
- [ ] Every actor is a real role from discovery §3, not "a user".
- [ ] Every process in scope states today, after delivery, volume and exceptions.
- [ ] Functional detail covers rules, statuses, notifications and the failure path.
- [ ] Data, visibility, reporting and integrations are answered or explicitly marked out of scope.
- [ ] In scope and out of scope are both written down.
- [ ] No `⚠️ TBD` remains that would block design — any that stay have an owner and a date.
- [ ] The customer has seen the summary and approved it.
