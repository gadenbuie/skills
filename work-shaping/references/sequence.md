# Sequence

Answer: in what order, and where is the constraint?

## 1. Risk first

Put risky and uncertain work first. It front-loads the parts most likely to invalidate the plan, so the cost of being wrong is paid early, while little depends on it.

- Feasibility questions become spikes before anything is built on them.
- Define milestones that each leave the system working, so stopping after any one still leaves something worth keeping.
- Cost of delay explains why: every unit of time spent building on an unproven assumption is exposed to rework.

## 2. Find the critical path

The critical path is the longest chain of dependent units. It sets the minimum duration, and a delay on it delays everything. Units off it have **float**: they can be deferred, parallelized or given to cheaper workers freely.

For a router: the critical path is what must be sequenced; float is what can fan out. Shorten the path by splitting its long units, removing edges that lack a real reason, or extracting a shared piece so dependents stop waiting on a whole unit.

## 3. Find the constraint

Every system has one constraint limiting throughput; improving anything else is wasted. Its signature is the longest queue: work piling up in front of it.

Five steps:

1. **Identify** it (what has the longest queue: review, a shared file, one slow unit, worker count?).
2. **Exploit** it: never let it idle or do low-value work. Spend the strongest worker and the best briefing here.
3. **Subordinate** everything else to it: other units run only as fast as the constraint absorbs.
4. **Elevate** it: add capacity only after exploiting it.
5. **Repeat**: once relieved, the constraint moves.

Decide which kind it is. If it is *worker count*, parallelize more. If it is *sequencing*, reorder or remove edges. If it is a *serial resource* (one reviewer, one integration branch), adding workers only builds a queue in front of it.

### Drum, buffer, rope

Pace work to the constraint (drum), keep a small buffer ahead of it so it never starves, and release new work only as fast as it is absorbed (rope). This is the direct fix for dumping every unit on the system at once.

## 4. Order ready work by value per size

When more is ready than capacity allows, **weighted shortest job first**: cost of delay (value + time criticality + risk reduction) divided by size. Among equal value, smaller goes first because it frees capacity sooner. A unit that unblocks many others carries their delay cost too.

## 5. Estimate under uncertainty

If a unit cannot be sized with confidence, use three points, optimistic O, likely M, pessimistic P, and expect `(O + 4M + P) / 6`. A wide spread (P much bigger than O) is itself a signal: spike it or split it before fixing the fan-out width.

## Avoid these pitfalls

- Sequencing by what is easy or interesting rather than by risk.
- Blocking units just to impose an order; use priority for order and edges only for real dependencies.
- Optimizing a unit off the critical path.
- Adding workers in front of a serial resource.
