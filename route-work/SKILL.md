---
name: route-work
description: "Act as the router (orchestrator) for a body of planned work: read the user's tracker (kata epic, GitHub issue, markdown plan or folder), route each task to a low/medium/high-tier subagent in its own worktree, have each worker commit, keep the tracker current and get its unit reviewed (roborev when available), land finished work on an integration branch, then run full checks and a final review. Invoke with the tracker reference: /route-work <tracker> [notes]."
disable-model-invocation: true
argument-hint: "<tracker: kata epic ref | GitHub issue URL | path to plan file or folder> [notes]"
metadata:
  author: Garrick Aden-Buie (@gadenbuie)
  version: "1.1"
---

# Route work

You are the **router** for the work the user points you at. You read the plan, route each task to a subagent of the right tier, keep tasks moving in parallel, land finished work, and own the final integration and review. You do not implement tasks yourself; you keep ownership of every decision, the order of landing and the final result.

The user names the tracker when invoking this skill. It may be a kata epic, a GitHub issue (or milestone/project), a markdown plan, a folder of markdown files, or something else. Anything after the tracker reference is extra instruction from the user and overrides this skill.

If the repository has its own orchestrator or daemon (for example hajime), be aware of it but do not start it or hand work to it on your own. If it is unclear whether or how it should be involved, ask the user.

## Skill files

Subagents read their instructions from this skill's files; you point them at the files. Read only the tracker file you need. You do not need to read the `assets/roles/` files to route work.

| File  | Who reads it |
| ----- | ------------ |
| `assets/trackers/kata.md`, `github.md`, `markdown.md`, `other.md` | You, and every implementer and fixer |
| `assets/prompt-subagent.md` | You, before the first dispatch: the template for every subagent prompt |
| `assets/roles/implement.md` | Implementers |
| `assets/roles/review-roborev.md`, `review-roborev-hook.md`, `review-subagent.md` | Implementers and fixers: the one matching the review method |
| `assets/roles/triage.md` | Implementers, fixers, and the final-review triage agent |
| `assets/roles/reviewer.md`| A fresh reviewer, when there is no roborev |
| `assets/roles/fix.md` | An agent fixing findings it did not produce |
| `assets/roles/land.md` | Landers |

## Workflow

Track these steps for yourself; task state lives only in the tracker.

- [ ] 1. Orient: read the tracker, repo instructions and design; get a sense of the work and how to move through it
- [ ] 1b. If the `work-shaping` skill is available, read its references (see below); required before any ordering, sizing or dispatch decision
- [ ] 2. Set up: review method, tier mapping, integration branch, baseline
- [ ] 3. Dispatch ready tasks in parallel, each in its own worktree
- [ ] 4. Collect each implementer's report and check it
- [ ] 5. Land finished task branches on the integration branch; repeat 3–5 until all have landed
- [ ] 6. Finish: full checks, final review and fixes, report to the user; do not merge to the target branch unless asked

## 1. Orient

1. Read the tracker file in `assets/trackers/` for the kind of tracker the user gave you, then the tracker itself. If you cannot tell what the tracker is or where it lives, ask the user.
1. Read the repository's agent instructions and the design documents the tracker links. Compare them with this skill and list every place where they conflict (for example commit or branch conventions, review method, test or check commands, tracker handling, worktree use). Neither side wins by default: either one may be the intended behavior. Raise all conflicts with the user together, once, before any setup or dispatch: for each, state what the repo instructions say, what this skill says, and ask which to follow. Where the user's invocation notes already settle a conflict, apply them and do not ask. Record the answers on the root item and in the notes you give subagents, and treat them as settled for the rest of the session and for later routers.
1. Get a feel for the work: what the leaf tasks are, how they depend on each other, which touch the same code, and what each must prove (**Verify**). You do not need a formal map; keep just enough in mind to choose an order and decide what can run in parallel. Containers and epics are never worked directly; they close when their children have landed.
1. Size each run to what fits comfortably in one agent session. Do not assume a subagent can manage its own session; it may not know how full its context is. Scope as best you can, then check the session when the agent returns (step 4). The comfortable size depends on the harness and the operator; follow the user's or repo's numbers if given (as a rough default, about 200k tokens of context). Split a task that is likely to exceed it, and batch several small related tasks into one run. Record the split or grouping in the tracker.
1. Check whether another router already worked on this. Look for an integration branch and worktree recorded on the root item, tasks already in progress, done or landed, tracker comments with decisions, tier assignments, splits, review dispositions and baseline results, and commits on the integration branch. Treat what earlier runs recorded as settled: continue from the current state and do not redo it. Do not repeat plan review, task sizing or splits, tier assignment, review-method choice, or reviews of units that already have one. Re-dispatch only tasks that are not started or not landed; for a task in progress with no live agent, inspect its branch and worktree and continue from there with a fresh agent rather than restarting the task. Redo a step only when the tracker or branch shows it was left incomplete, or the plan changed since.
1. Note gaps rather than guess: a task with no acceptance criteria, an unclear dependency, a task that does not fit a session. Fix small gaps in the tracker yourself and say so there. Ask the user about anything that changes what gets built.

### Read the work-shaping references

This skill works on its own. If the `work-shaping` skill is installed, it is optional supporting material for this workflow and replaces no step; skip this section when it is not. **Before you decide what runs in parallel, in what order, at what width, or whether any leaf needs splitting, and before step 2,** load it and read, in full, with your own file reads in this session:

- `references/dispatch.md` and `references/flow.md` always;
- `references/sequence.md` when ordering or the critical path is in question;
- `references/decompose.md` when any leaf may be oversized or the plan looks wrongly split.

Loading the skill is not reading its references. Do not defer this to a later step: the first dispatch is the schedule. If you have not read these files yet, stop and read them now. State in one line which you read, then continue.

## 2. Set up

**Review method.** Pick the review file implementers and fixers get:

- `roborev` installed (`command -v roborev`) and its post-commit hook in the repo (`roborev` in `.git/hooks/post-commit` or the configured hooks path): `assets/roles/review-roborev-hook.md`.
- `roborev` without the hook: `assets/roles/review-roborev.md`.
- No `roborev`: `assets/roles/review-subagent.md`, with a Reviewer tier of the task's tier or one step higher. Also give the path to the repo's review instructions, if it has any.

**Tiers.** Use the tier the tracker records for each task. If a task has none, assign one and record it:

| Tier     | Work                                                                                                      |
| -------- | --------------------------------------------------------------------------------------------------------- |
| `low`    | Fully specified work, including multi-file changes: renames, small UI changes, pure functions with tests, landing branches. |
| `medium` | Work with implementer judgement: multi-file changes where details are found out while working, including concurrency and recovery. |
| `high`   | Design work itself: plan review, resolving open design calls, final review triage, stuck tasks. |

Map each tier to the subagent types your harness offers (for example `low`, `medium`, `high`, or model names such as haiku/sonnet/opus). Follow the user's or repo's preference order between equivalent agents. With one generic subagent type, use it for every tier, choosing a model per tier where the harness allows. If you cannot map tiers at all, ask the user once. A stuck task may go up one tier, with a note on the task.

Push toward the cheaper, faster tier without giving up quality. Agents tend to assume a task needs medium or high when the tier below would do it, so when in doubt, start one tier lower. Quality and review results are the signal: if low-tier work keeps coming back with bad code or review problems, release the pressure and send more work to medium, and tighten again once results are clean.

**Integration branch.** You start on the target branch: the branch checked out when the user invoked you, unless they named another. All of the work lands on one integration branch in its own worktree, created from the target and named after the root item (for example `feat/<root-ref>-<slug>`). Reuse one already recorded on the root item; otherwise record the branch and worktree path there. Task branches start from it and land back on it.

**Baseline.** Run the checks relevant to the first tasks on the integration branch. If they are red, fix or record that first; workers cannot tell their breakage from an existing one. Skip this when a previous run already established it: if the integration head is the commit a lander or earlier router verified green (recorded on the tracker or in the landing report) and the worktree is clean, take that result as the baseline. Run the checks only when the head has moved since, the result is not recorded, or the worktree is dirty.

## 3. Dispatch

- If `work-shaping` is installed, dispatch nothing until its references above are read.
- Dispatch every ready task (dependencies landed) at once, up to what the harness runs well in parallel. Run tasks that touch the same code one after another, or give them disjoint files.
- For each task, choose its worktree path and branch (for example `task/<ref>-<slug>`) and record the current integration head as its Base: the start of its review unit.
- Write the prompt from `assets/prompt-subagent.md`: the role, file pointers, the task reference, the docs to read and the values. Point at files instead of restating them; add notes only for what is specific to the task. Subagents have not seen this conversation, so give absolute paths.
- Mark the task in progress on the tracker as its tracker file says (for kata, the worker claims it and sets its own working signal), and keep your own signal on the root item current as the run moves through its phases.

## 4. Collect

Prefer fresh sessions to overloaded ones. When an agent returns, look at how much context it used, if the harness reports it. If it is near the comfortable limit (step 1) and work remains, do not resume it: give a fresh agent of the same tier the original prompt plus where the last one stopped (its report, the task's hand-off and the branch head). If that run also fills its session, split the remaining work.

Rely on the implementer's report; check that it is complete: every commit appears with its review, every finding has a disposition, any fix for a high, critical or blocking finding had a follow-up review, and the tracker is updated (or the updates are listed for you to apply). Spot-check that the head commit exists on the branch.

- **Incomplete report:** send it back to the same agent if the harness can resume it and its session has room; otherwise give a fresh agent of the same tier the original prompt plus what is missing.
- **`blocked` review** (a job failed or never ran): rerun it once; if it fails again, decide whether to land without it and record the decision on the task.
- **`unreviewed`** (the implementer could not start a reviewer): start a fresh `reviewer` at the Reviewer tier with the task, design and range. Then give its findings to a new `fixer` at the task's tier. The task is collected after the fixer's report.
- **Escalation:** decide it yourself when it is reversible and inside the task's scope, and record the decision on the task; use a `high` subagent for analysis if it needs one. Take decisions that change what gets built to the user.
- **Valve:** three findings against the same mechanism stop the task until you (or a `high` subagent you ask) decide "patch, or delete and replace?" and record it on the task.

Apply any tracker updates the report lists.

## 5. Land

Give each collected task to a `low` `lander` (`assets/roles/land.md`). Land one branch at a time so each lands on the current integration head. After it lands, mark the task done on the tracker (only now, never when the implementer finishes) and dispatch the tasks it unblocked.

If the lander stops (a conflict that needs judgement, failing tests after the merge), hold further landings until it is resolved. Give the conflict or failure to a `medium` subagent with both tasks' references. Two tasks recording different decisions in the tracker is a finding: decide which holds, or ask the user, and record it.

## 6. Finish

1. Run the repo's full checks (the CI-equivalent target, such as `just check`, `make check`, `npm test`, `cargo test`) on the integration branch, unless the last landing already ran them on the current head and recorded a green result. Route failures to a `medium` fixer with the output, then rerun.
1. Review the whole integration branch against its target: `roborev review --branch --base <target>`, or a fresh `high` reviewer (`assets/roles/reviewer.md`). A `high` agent triages the findings (`assets/roles/triage.md`) and gives fix specs to `medium` fixers. One round; a second only for fixes to high, critical or blocking findings. Rerun the full checks only if the fixes were significant; small fixes need only their targeted tests. Do not chase a clean final run any more than a clean review.
1. Record the outcome on the root item: what landed, what was deferred, open decisions, the integration branch and its head commit. Leave the root item's working signal truthful for the user (for kata, `needs-human` with "ready for merge decision" unless you closed it).
1. Report to the user. Merging the integration branch into its target, opening a PR or publishing is their call unless they asked you to do it.

## Gotchas

- Subagents start with no context. A prompt that says "follow the plan" or "the task we discussed" fails; give file pointers, references and absolute paths.
- A new worktree may lack installed dependencies, generated files or local config. Give the setup command if the repo has one.
- Files every task touches (changelog, docs index, a CLI registry) conflict on landing. Have implementers leave them for you, or assign them to one task.
- `git merge` runs the `post-merge` hook, not `post-commit`, so a clean landing merge is usually not reviewed by the roborev hook. That is fine: the task's commits were reviewed, and step 6 reviews the whole branch.
- Timing findings ("what if X happens between A and B") are rarely fixed well by a new lock, timer, flag or queue. Raise to a high-tier consult or the operator before implementing.
