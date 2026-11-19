# Review: independent subagent (no roborev)

When the whole unit is committed, get an independent review.

- **If you can start subagents:** start a fresh one at the Reviewer tier from your prompt. Tell it to read `reviewer.md` (next to this file) and give it the task reference, the design documents, the repo's review instructions if any, the Worktree and the range `<base>..HEAD`. Triage its findings as `triage.md` says. A follow-up review, when one is needed, is a fresh reviewer on the fixing commits only.
- **If you cannot start subagents:** report to the router with status `unreviewed` and stop. The router arranges the review and the fixes.
