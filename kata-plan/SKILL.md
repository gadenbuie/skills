---
name: kata-plan
description: "Plan work as kata issues a router (route-work) can execute: the smallest tree, leaves a fresh low or medium agent can finish in one session, agent.profile, semantic blockers, priority and target branch. Plans only; never implements. /kata-plan <design doc | kata ref | description> [notes]."
disable-model-invocation: true
argument-hint: "<design doc path | kata ref | description of the work> [notes]"
metadata:
  author: Garrick Aden-Buie (@gadenbuie)
  version: "1.1"
---

# Kata plan

You are the **planner**. Turn what the user names (a design document, a kata issue to re-plan, or a description) into kata issues that a router such as `route-work` can hand to agents, each in its own worktree and session, without coming back to ask. You plan; you do not edit code, start workers or land anything. The operator approves the plan. Text after the reference is user instruction and overrides this skill.

- **You make the design and planning calls.** If your harness has subagents, you may delegate exploration and research to the cheapest ones; never the planning itself.
- **Plan once, accurately, for the cheapest tier that can do the work.** Detail written here is cheaper than a stronger worker, or one that comes back with questions.
- **Before asking the operator,** check the design, reference implementation, landed code and recorded decisions. Then ask in terms of user-visible behavior or developer intent, with options and a recommendation in plain language, naming tasks by what they do, not bare refs. Use the Plannotator Inbox when available.

**What a router expects.** Structure, never the title, decides kind: an epic has children and no parent, a container has both, a leaf has neither. Only leaves are worked; containers and epics close once their children land. Each leaf has `agent.profile`, **Verify:**, `blocked-by` edges for real dependencies, and fits one session. Decisions live on tasks. A leaf closes when it lands on the root's integration branch, so dependents become ready before the target branch has the code.

## Workflow

- [ ] 1. Orient: design, conventions, existing issues, what has landed
- [ ] 1b. If the `work-shaping` skill is available, read its references (see below); required before step 2
- [ ] 2. Shape: propose the smallest tree; the operator confirms it before you create anything
- [ ] 3. Write: create the issues and their bodies
- [ ] 4. Check the tree
- [ ] 5. Hand off

## 1. Orient

1. Find the design (the requirements source of truth), any plan, the reference implementation for a port, and earlier decisions. Read the repository's agent instructions for kata conventions: labels, metadata keys, branch attachment, tier names. Raise every conflict with this skill with the user together, once, before creating anything.
1. `kata search` and reuse existing issues. When re-planning, read the epic's decision comments and the closed leaves' hand-offs; plan against what landed, not the old bodies.
1. A design gap that changes *what* gets built is the operator's: amend the design with them first. If documents conflict, the design wins; amend the wrong one rather than writing a new one.
1. Record any baseline you verify (checks green at the branch point, a behavior confirmed) on the first task that depends on it, so its implementer need not rerun it.

## 1b. Read the work-shaping references

This skill works on its own. If the `work-shaping` skill is installed, it is optional supporting material for this workflow and replaces no step; skip this section when it is not. **Before you outline, sketch or propose any tree,** load it and read, in full, with your own file reads in this session:

- `references/decompose.md` and `references/sequence.md` always;
- `references/dispatch.md` when any units are similar or could run in parallel.

Loading the skill is not reading its references. Do not defer this to "before the final plan": the outline you show the operator is already the plan. If you have not read these files yet, stop and read them now. State in one line which you read, then continue.

## 2. Shape

Choose the smallest tree that carries the work:

- **One task** when one session can carry it; never an epic for one coherent change.
- **Standalone top-level tasks** for small independent items, so they do not wait behind the rest.
- **An epic** only for work spanning several sessions with real dependencies. At most three levels: epic, containers, leaves.
- **A container** groups two or more leaves that must land together: one shared worktree, landed whole on the target branch. A container with one child is a leaf.

Order the work:

- Slice vertically: a thin end-to-end path first, then widen; land the core flow before hardening it.
- Shared foundations first: a change many later tasks touch (a shared module, a design split, a schema) lands before the tasks built on it.
- Order by stranded value: if the effort stopped after any unit landed, the target branch should hold something worth keeping.
- Schema, wire-format and store-format changes are atoms: each lands whole in one leaf, never in parallel with another task touching the same resource.
- When the design is uncertain or expensive, plan a spike first (read-only or fake processes, no live or paid model calls) and turn its result into tasks afterwards.

Plan the whole tree up front and write every leaf in full. Only when a very large or complicated project's later work depends on what earlier work finds out, leave later waves as a title and short scope, each with a **planning leaf** blocked by the wave before (template in `assets/templates.md`).

Once-per-unit steps go in the container or epic body, not in leaves, and only when needed: **Before starting work under this task** (e.g. review the plan against what has landed since) and **Before closing this work** (e.g. review and verify the whole diff).

If the work overturns or changes existing tasks, add a closing note or leaf that updates them, and say what the plan does to the backlog.

Show the operator the outline: what each leaf does, its tier, what is parallel or sequential, what is deferred. Create nothing until they confirm.

## 3. Write

Create issues with `kata create --body-file <path> --idempotency-key <stable-key> --agent`, using `--parent` for containment and `--blocked-by` for dependencies. Write bodies from `assets/templates.md`.

### Kata mechanics

Add `--agent` to every kata command. Search first and reuse a look-alike. Create each issue in one call, for example:

`kata create "<title>" --body-file <path> --parent <ref> --blocked-by <ref> --priority <0-4> --label <label> --meta agent.profile=<tier> --idempotency-key <stable-key> --agent`

- Leave every created issue unclaimed, with no `work.*` keys; the worker's claim is the first signal. Do not claim leaves or start work on them.
- Retire issues that a re-plan replaces with `kata close <ref> --superseded-by <ref> -m "<why>" --agent` (or `--duplicate-of`, or `--wontfix` for dropped work), never by deleting. Never run `kata delete` or `kata purge`.
- Re-planning an existing issue: claim it, set `work.attention ok` with a one-line `work.attention_msg`, and set `needs-human` while waiting for the operator's confirmation at step 2. At hand-off release it (`kata unassign <ref> --agent`) so the router can claim it.

### Size

- Target work that one agent handles comfortably in a modest context, roughly 200k tokens; the operator's or repository's numbers win. In a harness without subagents one agent may work several leaves in turn; size them the same, since each leaf is one review unit.
- Too big: spans several subsystems, needs several independent groups of new tests, or is more than one coherent review unit. Split at the seams into a container, one child per piece.
- Too small: a trivial step next to a related one. Merge it.

### Tier (`agent.profile`)

- `low`: fully specified work, including multi-file: moving code, renames, small UI changes, pure functions with a table of behavior and tests, ports with the reference in hand, landing branches.
- `medium`: the design is settled but details must be found while working: unfamiliar code paths, concurrency, recovery.
- `high`: the main output is a decision: an open design call, a plan review, a triage. Rare for implementation; only for a named tricky part.

Start at `low`; you must be able to state why a leaf needs `medium`. On a borderline leaf, write more detail instead of raising the tier: low agents are competent when the body spells the work out. A plausible silent mistake (a lost write, a race) calls for a precise **Verify:**, not a `high` implementer. Use the project's tier names if it has its own.

### Bodies

A body is the final plan for one task, self-contained for a fresh agent with no chat:

- Link the parent, the design section, the plan step and, for a port, the reference implementation (it settles questions).
- Name files and symbols; mark new paths. Refer to code by symbol or heading, never line number, so the body survives upstream changes.
- Record mechanism decisions already made. A `low` or `medium` leaf leaves no design choice open.
- Say what is out of scope, and which task owns it.
- Leave out investigation history, chat and superseded options; rewrite a body that has grown a history down to the final plan. Long bodies distract workers.

**Verify:** is the contract: observable outcomes (commands, exact outputs, scenarios) this task alone can pass, without later tasks. Ask for the smallest tests that cover the behavior; the suite's cost matters, and asking for more (repeated full-suite runs, broad new scenarios) costs the worker its session. Replacing a slow test with a faster one is fine; "replace" never means deleting coverage. Name the targeted test commands; the router runs the full checks. Say when output is expected to change.

**Done when** gives a strong sense of done, so the worker finishes without encouragement. Keep the template's standing instruction: work out what you can yourself, try the named fallbacks when blocked, report only after concrete progress or exhausting reasonable options.

### Links, metadata and labels

- `agent.profile` on every leaf.
- Record the target branch in the root's body, and attach every task to it with the project's label convention so the work filters out of the general backlog. Never write `work.*` keys: `work.branch` and the attention keys belong to whoever claims the work (the router on the root, a worker on its leaf), not to the planner.
- **Blockers are semantic:** `blocked-by` only when the work needs the other task's code or decision landed, with the reason in the body. Never block just to force an order; preferred order is priority. Tasks that only share files get a note in both bodies naming the files; the router serializes them.
- **Children never block their parent,** and no task is blocked by its own container (deadlock). Containment does not gate readiness. A downstream task needing a whole unit may be blocked by the container.
- `kata ready` lists containers and epics, since parent links do not block. Routers never work them; do not add blockers to hide them.
- **Priority** (0–4, 0 highest) orders ready work, never dependency. Repairing or unblocking a stuck flow comes before new work. Within an epic, priority ranks siblings.
- Follow the project's labels. `backlog` marks real findings that are out of scope.

### Scope

- Every mechanism traces to a requirement in the design. Before planning a guard, timer, lock, queue, flag, retry or any state that orders steps, name its requirement; with none, leave it out. Accept a rare, recoverable gap over designing for a negligible window. When the same kind of problem keeps coming back, change the design so the class disappears.
- New state names where it is recorded and how it is found again.
- Prefer a small piece of user-written code over a designed configuration API.
- Before planning a fix, check whether a later planned task covers it; if so, defer to it with a note there. A feature that still needs design gets a note on the task that tracks it, not a leaf here.
- Backlog is not a parking lot. If an item belongs at a point in this plan, put it there (e.g. a top-level task blocked by the wave it follows). Real work out of scope becomes a top-level issue labeled `backlog`, linked `related` to the root, not a child.
- No process gate without a source in the design or from the operator. Plan and final review are once-per-unit sections (step 2), not leaves, unless the operator or repository asks for leaves.

### Decisions

- Domain decisions go in the design document. Mechanism decisions go on the task that needs them as a comment: question, options, choice, reasoning. A decision made in the Inbox or chat is recorded on the task in full, never as a pointer.
- Design documents stay durable and high-level: requirements and principles, not task status, sequencing or investigation context.
- When a decision supersedes part of a body, edit the body; do not leave a contradicting comment.

## 4. Check the tree

- Every open leaf: `agent.profile`, **Verify:**, **Done when**, attached branch.
- Every container: two or more children; once-per-unit sections where needed.
- No blocking cycles, no child blocking its parent, no issue blocked by its ancestor or descendant; every blocker has a reason.
- Rolling waves only: each later wave has its planning leaf, blocked by the wave before.
- Read each leaf as a fresh agent at its tier: could it finish without asking, in a modest context? Is a design choice open? Fix the body, not the tier.
- Read the chains: parallel work, shared files, needless blockers on the longest chain.

## 5. Hand off

Comment on the root (epic or single task): outline, tier per leaf, sequencing and parallel groups, what was deferred and where, decisions taken, target branch. Leave the root and all leaves unclaimed. Tell the operator the root ref with that outline; next, hand it to a router such as `/route-work <root ref>`.
