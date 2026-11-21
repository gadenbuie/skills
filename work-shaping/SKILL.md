---
name: work-shaping
description: "Load when you are about to write a plan for non-trivial work (splitting it into tasks, issues or a dependency graph), or about to interpret and schedule an existing plan (deciding what runs in parallel, what stays serial, in what order, with how many workers). Especially when the work has repeated near-identical units (migrations, batch edits, N similar modules or reviews), hidden dependencies or a shared resource, or is long-running or has side effects. Load alongside kata-plan or route-work sessions. Do not load for a single atomic task."
metadata:
  author: Garrick Aden-Buie (@gadenbuie)
  version: "1.1"
---

# Work shaping

How work is split, ordered and dispatched changes its cost, speed and correctness more than who executes it. This skill supplies the patterns; it owns no tracker or workflow. It informs the skill that does.

**The one thing to get right:** two opposite errors cost the same, *serializing independent work* (seven identical units handed to one worker) and *parallelizing dependent work* (B needs A's output). Verify independence before choosing fan-out width. When units are independent and similar, do one first as a verified exemplar, then fan out the rest against it.

## Read the references before you plan or schedule

**This file is an index, not the method.** The patterns that change outcomes (the independence test, the exemplar brief, sizing a leaf, width and pacing, resuming a failed run) are only in `references/`. Skimming this file and planning from memory is the common failure: the plan looks reasonable and misses exactly what the references catch.

1. Before you write, outline or propose any plan, or decide any order, width or parallelism, open the reference files for your situation (table below, or the session list) with your own file reads. Do it now, not "before the final plan": the first outline is already the plan.
2. A reference you intended to read later is a reference you did not read. If you catch yourself thinking "I'll check that before finalizing", read it now instead.
3. After reading, say in one line which files you read and the one change they made to your plan. If they changed nothing, re-read the one that matches your riskiest unit.

## If you are in a kata-plan or route-work session

If the user invoked `/kata-plan` or `/route-work`, that skill is already loaded and is the primary workflow; this one supplies reading for it. Read the references named below before planning or scheduling. Those skills already set the tree limits, tiers, sizing, landing and review rules. Do not override them; use this skill inside them, for the parts below.

### In kata-plan (step 2 Shape, step 4 Check the tree)

- **Most relevant:** `references/decompose.md` (shape per sub-problem, 100% rule, edges with reasons, and §4 on undersized leaves: plan smaller) and `references/sequence.md` §1-2 (risk first, critical path).
- **Repeated similar units:** plan the first as the exemplar leaf. Each fan-out leaf is `blocked-by` it, with the reason ("needs the exemplar's pattern landed") in the body and the exemplar linked as the reference implementation. Its **Verify:** is the exemplar's acceptance criteria. See `references/dispatch.md` §1.
- **Blockers:** use the independence test in `references/dispatch.md` §2. Units that only share files get a note in both bodies; units that must land together become a container.
- **Checking the tree:** look for needless blockers on the longest chain (`sequence.md` §2) and edges with no reason (`decompose.md` §3).
- **Skip:** `flow.md` beyond keeping leaf steps safe to re-run.

### In route-work (steps 3 Dispatch, 4 Collect, 5 Land)

- **Most relevant:** `references/dispatch.md`. Dispatch the exemplar alone, check it against its **Verify:**, then dispatch the dependents with its branch head or landed commit as the reference. If the exemplar shows a defect, fix it before any fan-out worker starts.
- **Width and pacing:** `references/flow.md` §1-2 and `references/sequence.md` §3. Landing is one-at-a-time, so it is usually the constraint; do not release more workers than landing and review can absorb.
- **Failures:** `references/flow.md` §5-6 and the recovery questions: isolate per-task failures, resume from the branch and hand-off, never restart.
- **Leaf size:** do not trust that each leaf is sized for one worker. Check each against `references/decompose.md` §4 before dispatch and split any that are several units.
- **Plan gaps:** if tasks look wrongly split or ordered, fix small gaps in the tracker with `decompose.md` and `sequence.md`; take changes to what gets built to the user.

### Neither running

Apply the patterns directly and keep the shape note (below) with the plan.

## Which reference answers your question

If you are asking any of these, read the file before answering from memory. When unsure which applies, read `decompose.md` and `dispatch.md`.

| Question | Read |
| --- | --- |
| How should I split this? Is each piece the right size for one worker? Did I leave a gap or overlap? What really depends on what? | `references/decompose.md` |
| What goes first? What is risky or uncertain? Which chain sets the pace, and what is the bottleneck? Which ready unit should run next? | `references/sequence.md` |
| Can these units run in parallel or must they stay serial? Should one go first as an example? What does each worker's brief need? | `references/dispatch.md` |
| How many should run at once? How big a batch? What happens if a run fails halfway, and can it resume safely? | `references/flow.md` |

## Quick triage

1. **More than one unit?** No: nothing to shape.
2. **Same operation on different data?** Exemplar then fan-out (`dispatch.md`).
3. **Does any unit need another's output, or write the same file, record or resource?** That is a blocker; the pair stays serial.
4. **Anything uncertain or expensive to get wrong?** It goes first, as a spike or exemplar (`sequence.md`).
5. **Long chain or one bottleneck step?** Find the critical path and constraint; spend the strongest worker only there.
6. **Could failure halfway leave a mess?** Idempotent steps, checkpoints, isolated failures (`flow.md`).

## Output

A short shape note in the plan or as a comment on the root item, not a separate document:

- **Units** and what each proves.
- **Edges** with the reason for each; everything else is parallel.
- **Critical path** and the constraint.
- **Exemplar**, its acceptance criteria, and the units that fan out after it.
- **Serial points:** shared writes, irreversible actions, schema atoms.
- **Width:** how many run at once, and why that number.
