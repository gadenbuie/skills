# Tracker: kata

Add `--agent` to every kata command. Refs are `<project>#<id>`, or a short id inside the bound project. Do not read `kata quickstart`; this file is the contract for this workflow.

- **Read:** `kata show <ref>`. Structure decides kind: an epic has children and no parent, a container has both, a leaf has no children. Only leaves are worked. Parent links group work and do not gate readiness; `--blocked-by` / `--blocks` edges do.
- **Tier:** optional; a leaf may record it as `agent.profile` metadata, or the project may use another convention or none. Use what the project records, and otherwise the router assigns one.
- **Progress and decisions:** comments on the issue: `kata comment <ref> -m "..." --agent` (`--body-file <path>` for long ones). Record a decision as the question, the options, the choice and the reasoning, never as a pointer to a chat.
- **Backlog item:** search first (`kata search "<terms>" --agent`) and reuse a look-alike. Otherwise create a new top-level issue labeled `backlog`, linked `related` to the epic, not parented under it. This is the only kind of issue you create unless the router tells you otherwise.
- **Commit trailer:** `Refs: <project>#<ref>`.

## Working signals

Every issue being worked carries a signal: it is claimed, `work.attention` says whether it is moving, and `work.attention_msg` says why in one line. Only the issue's owner writes its `work.*` keys; everyone else reads them. Never write `work.*` on a closed issue.

- `work.attention ok`: unblocked and moving.
- `work.attention stuck`: blocked; `work.attention_msg` says on what.
- `work.attention needs-human`: waiting on a decision or review; `work.attention_msg` says which.

Set the pair with `kata meta set <ref> work.attention <value> --agent` and `kata meta set <ref> work.attention_msg "<one line>" --agent`. Change them the moment the state changes, not at the end. Before you stop for any reason, the issue is closed or the pair describes the hand-off.

### Implementers and fixers (the task leaf)

- **Start:** `kata claim <ref> --agent`, `kata meta set <ref> work.attention ok --agent`, `kata meta set <ref> work.attention_msg "<what you are doing>" --agent`, `kata meta set <ref> work.branch <task branch> --agent`.
- **Blocked or waiting:** set `stuck` or `needs-human` with the message, then stop and report as `blocked` or `escalated`.
- **Done:** leave `work.attention ok`, set `work.attention_msg "returned: committed and reviewed, ready to land"`, then release the claim with `kata unassign <ref> --agent`, in that order (you may not write `work.*` after releasing). Leave the task open; the router closes it after landing.
- **`needs-human`** is reserved for when a human must step in. Finishing is not one of those.
- **Cannot finish:** comment where you stopped, set the pair, and run `kata label add <ref> needs-review --comment "<what remains>" --agent`.
- **Do not:** close or reopen any issue, claim or start other tasks (`kata next`, `kata ready`), create issues other than backlog items, `kata notify`, schedule or park, or touch daemon or project setup. Report what you want done instead.

### Router (the root item, and tasks you take over)

You own the signal on the root item for the whole run, and on any leaf you work yourself or take back from a worker that is gone.

- **Start of run:** claim the root item and set `work.attention ok`, `work.attention_msg "<phase: dispatching | collecting | landing | final review>"`, and `work.branch <integration branch>`. If an earlier router left the signal, take it over with a comment and fix it.
- **Each phase change:** update `work.attention_msg` (for example `landing 3 of 7`).
- **Waiting on the user** (a conflict question, a scope decision, the merge decision): set `needs-human` with the question in `work.attention_msg`, and put the full question in a comment.
- **Blocked** (red baseline, a failure you cannot route): set `stuck` with the reason.
- **Back to work:** set `ok` again as soon as you resume.
- **End of session:** if the root is not closed, the pair must describe the hand-off for the next router. Never leave `ok` on a run that has stopped.
- **Workers' signals:** read them with `kata show <ref>`. Do not write a worker's `work.*` while it is live.
- **On collecting a task:** `kata claim <ref> --agent` (if the worker did not release it, unassign it first), set `work.attention ok` and `work.attention_msg "landing"`, then close it after the lander merges. A task a worker returned unowned with `ok` is yours to take over this way.
- **Identity:** give each subagent its own `KATA_TEAMMATE=<name>` and `KATA_INBOX_USER=<actor>/<name>`, keeping the router's actor.
- **Waiting on workers:** `kata wait <refs> --until attention --any --timeout <seconds> --agent` returns when a worker goes `needs-human` or `stuck`, or closes.

## Closing (router only)

- **Landed leaf:** `kata close <ref> --done -m "<what landed and how it was verified, 40+ characters>" --commit <sha> --agent`. Do not write `work.*` after the close.
- **Container:** close once all its children have closed. A parent cannot close with open children.
- **Root item:** close with the final outcome (what landed, what was deferred, the integration branch and head) only if the user asked you to finish the epic; otherwise leave it open with `needs-human` and `work.attention_msg "ready for merge decision"`.
- **Retire:** `--duplicate-of <ref>` or `--superseded-by <ref>` for duplicate or superseded work, `--wontfix` for dropped work, each with a message of 20 or more characters (60 or more for `--wontfix`). Never run `kata delete` or `kata purge`.
- **Integration branch:** record it on the root item in a comment, and in `work.branch`.
