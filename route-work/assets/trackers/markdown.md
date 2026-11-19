# Tracker: markdown file or folder

- **Read:** a single file has one task per heading or checklist item; its body is the scope and acceptance criteria. A folder has an index (order, dependencies, status) and one file per task.
- **Tier, status, branch:** a line under the task heading, e.g. `Tier: medium · Status: in progress · Branch: task/3-parser`.
- **Progress and decisions:** a `Notes` list under your task (or in your task file), appended with dated entries. Record a decision as the question, the options, the choice and the reasoning.
- **Backlog item:** an entry in a `Backlog` section (single file) or a new file marked backlog (folder), linking the root.
- **Commit trailer:** `Refs: <file>#<task id or heading>`.
- **Cannot finish:** a note under your task saying where you stopped and what remains.

**If the tracker is tracked in git:** edit it in your worktree, only under your own task heading or in your own task file, and commit it with your work. Notes reach the integration branch when the task lands. The landing agent resolves conflicts in it; two tasks recording different decisions for the same thing is reported to the router, not resolved.

**If the tracker is outside git:** do not edit it. List the exact updates (where, what text) in your report; the router applies them.

Router only:

- **Dispatch / landed:** change the task's `Status:` line (and a folder's index) on the integration branch, or in place for a tracker outside git.
- **Workers' notes before landing:** read them in the worker's worktree.
- **Integration branch:** record it at the top of the file or index.
