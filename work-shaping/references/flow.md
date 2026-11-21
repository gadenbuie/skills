# Flow

Answer: how much is in flight at once, and how cheap is failure?

## 1. Cap work in progress

**Cycle time = WIP / throughput.** If capacity fixes throughput, the only lever on cycle time is WIP. Even where that is not guaranteed (high-variance work), lower WIP still hides fewer defects, keeps state legible and reduces rework.

- Cap in-flight units at what the constraint can absorb and the machine can run well, not at the number of units available.
- Work piling up in front of one stage means upstream is feeding faster than the bottleneck; slow the feed, do not add upstream workers.
- Pace from demand and the constraint's capacity (takt), not from the fastest worker.

## 2. Small batches

Small batches catch a mistake on the first unit rather than after the full batch, cut waiting and shrink the cost of a defect. Prefer one-piece flow; when not possible, the smallest feasible batch. This is the same reason as the exemplar: learn on one before committing many.

Batch several small related units into one run only when each is too small to repay a worker's startup cost, and keep each a separate review unit.

## 3. Make every step idempotent

Re-running a step must be harmless.

- Key external side effects (publish, send, delete, pay) by `(run_id, step_id)`, never by timestamp or random value.
- Upsert instead of insert; check before act.
- Read state from the source of truth, not from memory of a previous attempt.

## 4. Checkpoint at boundaries

Persist state after a phase, after an expensive step, and before a risky action. On restart, skip completed steps and resume at the first incomplete one.

A useful checkpoint stores **state, not transcript**: the goal restated, the working set (facts, not raw tool output) and completed steps with their outputs. Put it where the next worker will look (the tracker item, the branch, a hand-off note).

## 5. Isolate failures

One unit's failure must not stop the rest. Collect per-unit errors and report them; do not crash the batch. This is what keeps a fan-out robust: one failed worker leaves the other N-1 results intact.

For work that must not be done twice, use a **lease** with heartbeat: one owner at a time, reclaimable if it goes quiet past a timeout. This prevents double execution and silent hangs.

## 6. Retry with a budget

Retry transient errors with backoff and jitter, but with a finite budget. After it is spent, escalate to a human or park the unit; never loop. Distinguish **retry** (repeat the failed step) from **resume** (skip completed steps, continue from the first incomplete).

## The four recovery questions

For any failed step, in order:

1. Can it be **repeated safely**? Retry.
2. Can it **resume from saved state**? Restore the checkpoint.
3. Could it have **already changed the outside world**? Verify before replaying.
4. Did it **fail again after the budget**? Escalate.

## Mistake-proofing

Design so errors are impossible or caught immediately: constrained output formats, required fields, validation gates that block the next step on malformed output. Stop at the first defect and fix it at the source before more units start.

## Avoid these pitfalls

- Releasing every ready unit at once because they are ready.
- Checkpoints that save transcripts instead of state.
- Retrying a non-idempotent step without verifying what already happened.
- Unbounded retries.
- One unit's failure aborting the whole batch.
