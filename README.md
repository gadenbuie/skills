# Skills

My personal agent skills.

| Skill | Use |
|---|---|
| [writing-go-code](writing-go-code/SKILL.md) | Rules for writing and testing Go. Checklists come first and are written for smaller implementer models. The `references/` folder holds the testing rationale, concurrency notes and a review checklist for reviewers. |
| [route-work](route-work/SKILL.md) | Acts as the router for a body of planned work (kata epic, GitHub issue or markdown plan). It sends each task to a tiered subagent in its own worktree, gets it reviewed, lands it, and runs the final checks. Invoke with `/route-work <tracker>`. |
| [kata-plan](kata-plan/SKILL.md) | Plans a body of work as kata issues that `route-work` can execute: the smallest tree, leaves a fresh low or medium agent can finish in one session, tiers, semantic blockers and the target branch. Plans only. Invoke with `/kata-plan <design doc \| kata ref \| description>`. |
| [orient-me](orient-me/SKILL.md) | Principles for explaining something so a human can act on it: ground the mechanism in who it serves, and shape the account around what the reader must decide. Use before asking the operator for a decision, when they lack context, ask to be caught up or walked through, or when a plan surfaces an issue they must adjudicate. Also for design artifacts that record why software works the way it does. |
| [work-shaping](work-shaping/SKILL.md) | Patterns for shaping a body of work before it is dispatched: decompose into units with a real dependency graph, sequence by risk and critical path, dispatch an exemplar first and then fan out similar units, and keep flow bounded and resumable. Loaded automatically while planning or routing non-trivial work, alongside `kata-plan` and `route-work`. The `references/` folder has one file per pattern. |
