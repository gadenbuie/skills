# Decompose

Answer: how do I break this into pieces, and what depends on what?

## 1. Pick a shape per sub-problem

One body of work often needs different shapes in different parts. Choose per sub-problem.

| Shape | Fits | How |
| --- | --- | --- |
| Vertical slice | Features touching UI, logic and data | Thinnest end-to-end path first, then widen; every slice is usable |
| Horizontal layer (avoid) | Almost never; only when a vertical slice is impossible and the reason is written down | Late integration is the risk, and it is worse with agents: each layer passes its own checks while the seams stay untested. If unavoidable, land a thin end-to-end path through the layers first |
| Expand, migrate, contract | Changing anything others depend on (schema, API, format) | Add new beside old, move callers, remove old; each step lands whole |
| Spike then build | Feasibility is uncertain | Time-boxed, read-only or fake experiment; turn its result into tasks afterwards |
| Strangler | Replacing a legacy component | Route piece by piece to the new one |
| Divide by data | Batch work, migrations, content edits | Same operation per slice; exemplar first, then fan out (`dispatch.md`) |
| Question tree | Research and analysis | Main question, sub-questions, evidence each needs |
| Draft, review, revise | Writing, design | Outline, rough draft, structural review, polish |

Parallel-computing names, when the question is how units relate: **domain** (same operation, different data), **functional** (different operations), **recursive** (same-type subproblems, then combine), **exploratory** (search space unknown: branch, then prune), **speculative** (assume an outcome, do the work, roll back if wrong). Speculation is only safe when rollback is cheap and nothing irreversible runs.

## 2. Apply the 100% rule

Children must sum to exactly the parent's scope.

- **Gap:** something the parent needs that no child owns. Test: if every child passed, would the parent's goal be met?
- **Overlap:** two children both own something. Pick one owner and state it in the other's out-of-scope line.
- Shared files nobody owns (changelog, registries, index files) are the usual gap and overlap. Give them to one unit, or leave them to the integrator.

## 3. Build a dependency graph

Nodes are **one verifiable outcome**, not a role or persona. An edge means "cannot begin until an upstream artifact, decision or resource exists".

- Add an edge only for a real need (the code, decision or data must exist). Never add an edge to force an order; order is a sequencing choice (`sequence.md`).
- Write the reason beside each edge. An edge with no reason is a guess; remove it.
- Reject cycles. A cycle means two units are really one, or a shared piece should be extracted and done first.
- Nodes with no unmet edges are ready. Shared writes and irreversible actions are never parallel with each other.

## 4. Check the size of each node

A unit should be completable by one worker in one session without questions, and be one review unit. Too big: split at a seam. Too small: merge into its neighbor. Units that must land together are one unit with several steps, or a group landed whole.

**Plans routinely undersize the split.** A step reads as "one worker can do this" when it is really several pieces of work. A leaf in a plan is a claim about size, not a guarantee.

Signs a unit is several units:

- It spans more than one subsystem or module.
- It needs more than one independent group of new tests, or its **Verify:** lists outcomes that do not depend on each other.
- It touches many files for different reasons, or a reviewer would want to read it as separate changes.
- Its description uses "and" between pieces that could land alone.

**When planning:** lean smaller. When in doubt, split. Independent pieces then run in parallel and each stays a clean review unit; a unit that is too big costs a stuck or overfull session. Do not merge pieces back together to keep the tree small.

**When routing:** do not accept each leaf as correctly sized. Before dispatch, skim each task against the signs above. If it looks like several units, split it (or run it as several workers) and record the split on the tracker, rather than handing it to one worker. After the worker returns, check how full its session got; a near-full session is evidence the unit was too big, so split the remainder.

## Avoid these pitfalls

- Slicing by layer when a vertical slice would prove the seams earlier.
- Splitting a schema or wire-format change across units; it is an atom.
- A unit whose output nobody can check mechanically; rewrite it until there is a **Verify:**.
- Planning every detail of work that depends on what earlier work will discover; plan the near wave in full and the later wave as scope plus a planning step.
