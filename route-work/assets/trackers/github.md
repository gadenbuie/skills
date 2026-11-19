# Tracker: GitHub issues

GitHub issues are usually public. Implementers and fixers never post to GitHub: no comments, issues, labels or edits. They work from a local copy of the issue and follow `markdown.md` for it.

- **Read (router):** `gh issue view <n> --json title,body,comments,state,labels,subIssues,blockedBy`. Plain `--comments` output drops the body when piped. Sub-issues or a task list in the root issue's body define the leaves; for a milestone, `gh issue list --milestone <m>`.
- **Tier:** a label such as `tier:medium`, or a "Tier:" line in the body.
- **Progress, decisions, backlog items, cannot finish:** in the local copy, as `markdown.md` describes. A backlog item is an entry in its `Backlog` section linking the root issue.
- **Commit trailer:** `Refs: #<n>`. Not `Fixes`, which closes the issue on merge.

Router only:

- **Local copy:** write the root issue and its leaves into a local markdown tracker (one file, or an index plus one file per task) with tier, status and branch lines per task. Put it where the repo's instructions say; if they say nothing, ask the user. With no guidance at all, keep it local and untracked (for example under `.git/info/exclude` or an ignored scratch directory) and maintain it yourself. If the repo tracks it in git, it follows the "tracked in git" rules of `markdown.md`; otherwise workers list their updates in their reports and you apply them. Give workers its path in their prompts. Record in it the integration branch.
- **Posting to GitHub:** ask the user once what, if anything, to post. Default to nothing. If they want updates, post only high-level status at major milestones (for example when the work starts, when it is ready for review, and the final outcome), using `gh issue comment <n> --body-file <path>`. Never post decision records, review findings, or worker notes unless the user asks.
- **Dispatch / landed:** update `Status:` in the local copy. Assign, label, close or check boxes on GitHub only if the user said to.
