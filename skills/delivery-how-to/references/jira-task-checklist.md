# A well-completed Jira task — checklist

> Walk the block for the task's kind **before** calling it done, from a fresh `getJiraIssue` (with
> comments, sub-tasks, links) and a fresh `get_file_contents` on the minuta file — never from memory of
> what was written. Every line is either true or named as not true in the close of the command; a task
> is never closed over a line that is not true. Field rules are in `SKILL.md` → *A well-completed Jira
> task*; this file is the walk.

---

## Relevamiento task (label `relevamiento`)

**Summary**
- [ ] Reads as the task was written in Jira (it is also the second half of the minuta file's header table) — not renamed during the session.

**GitHub file**
- [ ] `relevamientos/<KEY>.md` exists — and only that path, no duplicate for the same key.
- [ ] Front matter opens with `jira_key: <KEY>`; the header table's Jira key is a link to the issue; *Estado* is `En progreso` or `Finalizado`.
- [ ] No section is blank — every gap is `⚠️ TBD — <question> — <owner>`, either in a requirement's own open-questions cell or in the *Scope & open questions* table.
- [ ] Every open item carries the Jira key of the sub-task created for it, in whichever table it came from.
- [ ] If this is the first file under a top-level path `README.md`'s Map table does not mention, the close names it for Seamlex to add.

**Sub-tasks**
- [ ] Every `⚠️ TBD`, deferred decision, promised document and Seamlex decision is a sub-task — `Pendiente: <one line>`, type `{{TYPE_TASK}}`, `parent` = this task.
- [ ] No pending item created twice on a resumed session; every pending item the session answered is closed with a comment holding the answer.

**Comments**
- [ ] Result comment: who ran it (Seamlex Product Owner, Claude, with `{{USER_NAME}}`), date, three-line summary, minuta file URL, sub-task keys, state left.
- [ ] Transcript comment `Transcript del relevamiento — <date>` with the URL of `relevamientos/<KEY>-transcript.md` — the comment never holds the transcript text itself, only the link.

**Links**
- [ ] Minuta file URL in the result comment.
- [ ] The file's front matter carries `jira_url` pointing back at the issue — GitHub has no mechanism to list the task under the file, so the comment is the only live Jira-side link and the close says so.

**Status**
- [ ] Moved to *in progress* when the session started.
- [ ] `Done` only if: the customer said yes to the closing summary, the minuta says `Finalizado`, the *Ready for design* lines at the foot of the minuta template were walked and named, and no open sub-task **blocks** design. Otherwise left *in progress* with what has to happen next listed in the result comment.
- [ ] The transition to done was its own explicit yes when `{{CONFIRM_WRITES}}` is `always`.

---

## Pending item (`Pendiente:` sub-task, `{{TYPE_TASK}}`)

**Summary**
- [ ] `Pendiente: <the open point in one line>`, in `{{LOCALE}}`, business words.

**Type, parent, sprint**
- [ ] Sub-task type; `parent` = the relevamiento it came from; sprint inherited, not set.

**Description**
- [ ] The shape of `../../business-analysis/references/pendiente-template.md` — where it was raised, owner, needed by, whether it blocks design, what is open, why it matters, what is already believed, what was offered, when it is done.
- [ ] Link to the minuta and to the section it comes from.
- [ ] The owner named in the first line when the assignee could not be resolved.

**Labels and assignee**
- [ ] `{{LABEL_REQUEST}}`, `pendiente`, `{{LABELS_EXTRA}}` when set.
- [ ] Assigned to the owner when `lookupJiraAccountId` resolved them; otherwise unassigned.

**Done when**
- [ ] The answer is written on the minuta (the `⚠️ TBD` replaced) **and** on this sub-task as a closing comment.
- [ ] Closed by transition, not by deletion.
