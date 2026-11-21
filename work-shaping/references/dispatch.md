# Dispatch

Answer: how do I hand the pieces out, in parallel or serial?

## 1. Exemplar, then fan out

When a body of work is N similar, independent units, do not give all N to one worker, and do not launch all N at once blind.

1. **One worker completes unit 1** against explicit acceptance criteria.
2. **Verify it.** The exemplar is not done when it is written; it is done when it passes its checks (the first-good-piece rule).
3. **Fan out units 2..N in parallel**, each told to follow unit 1 as the canonical example.

Why: the exemplar validates the approach, produces a reference artifact workers can imitate, and surfaces edge cases before a mistake is multiplied N times. If a defect appears, stop and fix it at the source (jidoka) on unit 1, not unit 7.

Choosing the exemplar: pick the most representative unit, or the hardest one if the point is to flush out edge cases. Record settings, decisions and acceptance results so the fan-out reproduces them.

When not to: N is small and units are trivial (the exemplar round trip costs more than a mistake would), or units are not similar enough for one to serve as a reference.

## 2. Is it safe to parallelize? The independence test

Parallel is safe when each unit:

- reads the same snapshot without modifying it, or
- writes to its own namespace (own files, own branch or worktree), or
- produces output that a deterministic reducer can combine.

Serial is required for: writes to the same record or overlapping code, decisions that depend on a prior judgment, irreversible external actions (publish, delete, send), and schema or format atoms.

Enforce this **outside the prompt**: separate branches or worktrees, file isolation, version checks, leases, transactions. "Please don't touch X" is not a control.

Three pre-flight questions before fanning out:

1. Can each unit get a **self-contained input**?
2. Can it return a **machine-checkable artifact**?
3. Is **merging cheaper than waiting** for serial?

Any "no" means serial, or reshape the units until each is "yes".

## 3. Worker contract

Treat each brief as a service call with a minimum contract. A fresh worker has no memory of the plan.

| Field | Decide |
| --- | --- |
| Objective | The single result in scope, and what is excluded (and which unit owns it) |
| Inputs | Authoritative files, data versions, upstream artifacts; for fan-out, the exemplar |
| Permissions | Tools, network, write targets allowed |
| Output | Schema, artifact type, evidence to return |
| Acceptance | The automated and human checks that define success |
| Failure | What status to report for timeout, missing evidence or conflict |

Workers do not talk to each other; all coordination goes through the dispatcher. Point at files instead of restating them; give absolute paths.

## 4. Match the pattern to the dependency

| Shape of work | Pattern | Caveat |
| --- | --- | --- |
| Independent similar units | Exemplar, then fan-out | Units must pass the independence test |
| Linear dependencies | Sequential pipeline | One failed stage halts everything; checkpoint between stages |
| Distinct jobs, one synthesis | Orchestrator and workers | Orchestrator never does the work; workers get narrow briefs |
| Mixed | Serial spine, parallel branches off it | Edges decide which is which |

Scale effort to complexity: one worker for simple work, more only when units are independent and large enough to repay the overhead. Parallel workers on dependent work is just expensive serial execution.

## Avoid these pitfalls

- Handing N independent units to one worker.
- Fanning out before the exemplar passes its checks.
- Brief says "follow the plan" with no pointers.
- Concurrency control that lives only in the prompt.
