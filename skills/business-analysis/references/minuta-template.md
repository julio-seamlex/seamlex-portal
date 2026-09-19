# Minuta body — <task summary as it reads in Jira>

> The body of a `Minuta <KEY> — <task>` Confluence page, placed under the header table
> `seamlex-refinar.md` §4a builds (**Tarea Jira**, **Sprint**, **Épica**, **Relevado con**,
> **Fecha(s)**, **Estado**). Adapted to the task, never copied section for section — a small task will
> leave several sections short or explicitly marked not applicable. The **Requirements** table is the
> heart of the document; every other section exists to support it.

> Anything not answered is `⚠️ TBD — <the open question>` with an owner. Gaps are findings, not blanks.

---

## What this task delivers

<Three to six sentences a Seamlex architect could read alone: what becomes true for the business, for
whom, why it matters, and what goes wrong today without it (the pain, from discovery §7 — quote the
customer where it lands). No Salesforce terms.>

| Measure | Baseline today | Target | How we'll know | Owner |
|---|---|---|---|---|

> If the success-measure table cannot be filled, say so rather than inventing a number.

## Actors

| Actor | Area / role (discovery §3) | What they do in this capability |
|---|---|---|

## Requirements

> Every discrete thing this task asks for, one row each — the single source an architect or developer
> reads to make design and build decisions without coming back to ask. A requirement with no clear actor
> or no testable statement is not ready; push back on it in conversation rather than writing it vague.

| REQ-ID | Capability | Actor | Statement | Source | MoSCoW | Complexity (1–10) | In/out of scope | Open questions |
|---|---|---|---|---|---|---|---|---|

- **REQ-ID** — `<KEY>-R<n>`, numbered in the order raised. Stable once written; never renumbered on a
  resumed session, only appended to.
- **Capability** — the short business capability this belongs to (e.g. "Devolución — registro",
  "Devolución — aprobación"). Groups requirements that share a process or actor; becomes a natural
  boundary for design later.
- **Actor** — a real role from the **Actors** table above, never "a user".
- **Statement** — one testable sentence stating the need, not the mechanism: "<actor> must be able to
  …" / "The business needs …". If it reads like a solution ("add a field for region"), ask what it is
  for and write the need instead.
- **Source** — `<KEY> §<transcript reference> — "<short verbatim quote>"`, so the statement traces back
  to what was actually said. Paraphrase only when nothing was said verbatim, and mark it as paraphrased.
- **MoSCoW** — Must / Should / Could / Won't (this phase), as the customer prioritized it — ask
  directly when it is not obvious.
- **Complexity (1–10)** — a rough, honest gut sense from the conversation (1 = trivial configuration,
  10 = major unknowns or heavy integration), not a committed estimate. Mark `⚠️` when there isn't enough
  information to guess; the architect revises every number at high-level design.
- **In/out of scope** — `In`, or `Out — <why, e.g. "phase 2", "excluded in signed scope">`.
- **Open questions** — `—` when settled, otherwise `⚠️ TBD — <the question> — <owner>`. Once a pending
  sub-task exists for it, append the key: `⚠️ TBD — … — <owner> (KEY-123)`.

## Process context

> Only for a process whose shape actually changes — today versus after delivery, in enough detail that
> the requirements above make sense in context. Skip entirely, or say "no process changes, only new
> capability," when nothing here would add information beyond the Requirements table.

### <process name>
- **Today:** <how it runs now>
- **After delivery:** <what changes, referencing the REQ-IDs that drive it>
- **Volume and frequency:**
- **Exceptions:**

## Cross-cutting details

> Whatever applies across several requirements rather than to one row — mark a subsection "Not
> applicable" rather than deleting it, so a reader knows it was considered.

- **Data:** what is created, read or changed; where it comes from today; required vs. optional and
  validation rules; retention, history and audit.
- **Visibility and permissions:** who must see this, who must not, who can edit versus only read.
- **Reporting:** what someone will want to measure about this later, and at what grain.
- **Integrations and dependencies:** systems, other tasks, decisions or third parties this waits on —
  the direction and trigger of each, not the mechanism.

## Scope & open questions

**Assumptions**
-

**Other open questions** — session-level items that don't belong to a single requirement row.

| ⚠️ | Question | Who can answer | Needed by | Jira key |
|---|---|---|---|---|

## Raised here, outside this task

<Anything the customer raised during this session that belongs to another task, or to no task at all.
Cross-reference the entry in "Features no identificados". Do not resolve it here.>

## Para el equipo de delivery

<The only section where platform vocabulary may appear — a Salesforce term the customer used, a hint
the architect will want, a constraint the PO noticed but doesn't decide. Empty is fine; invented content
is not.>

---

## Ready for design

`seamlex-refinar.md`'s closing step walks this list from a fresh read of the page before marking the
relevamiento `Finalizado`. Any line not true keeps the task `En progreso`.

- [ ] The title is `Minuta <KEY> — <task summary>`, and the Jira key in the header links to the issue.
- [ ] What it delivers, the pain and the success measure are filled — no invented numbers.
- [ ] Every actor is a real role from discovery §3, not "a user".
- [ ] Every requirement has a REQ-ID, a real actor, a testable statement, a source, a MoSCoW and an
      in/out-of-scope call — a missing complexity number is fine only when marked `⚠️` with a reason.
- [ ] Cross-cutting details are answered or explicitly marked not applicable.
- [ ] No `⚠️ TBD` remains — in the Requirements table or in Scope & open questions — that would block
      design; any that stay have an owner and a date.
- [ ] The customer has seen the summary and approved it.
