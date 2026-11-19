# Subagent prompt template

Point the subagent at files; do not paste their contents or restate their rules. Use absolute paths: `<skill>` is this skill's directory. Fill in only the values the role uses, and add notes only for what is specific to this task.

```markdown
You are the <implementer | fixer | reviewer | lander> for task <ref>: <title>.

Read these files first and follow them:
- <skill>/assets/roles/<implement.md | fix.md | reviewer.md | land.md>
- <skill>/assets/roles/<review-roborev.md | review-roborev-hook.md | review-subagent.md>  (implementer, fixer)
- <skill>/assets/roles/triage.md  (implementer, fixer)
- <skill>/assets/trackers/<kata.md | github.md | markdown.md>, or <your tracker note>  (implementer, fixer)

Then read the task (<`kata show <ref>` | `gh issue view <n> --json title,body,comments` | path#heading>)
and: <design docs and sections, the repo's agent instructions, other paths>.

Values:
- Worktree: <absolute path>
- Task branch: <branch>
- Base: <sha>   (implementer, fixer, reviewer: the review unit is <base>..HEAD)
- Main checkout: <absolute path>
- Setup: <command | none>
- Targeted tests: <commands>
- Commit trailer: <e.g. Refs: project#ref>
- Root: <root ref>
- Reviewer tier: <tier>   (review-subagent.md only)
- Integration worktree / branch: <path> / <branch>   (lander only)

<Findings or fix specs (fixer only), and task-specific notes, if any.>
```
