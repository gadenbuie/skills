# Triage review findings

1. **Disposition every finding before fixing any.** Record one comment on the task listing each finding and its disposition:
   - _fix_: a real defect against the task or design.
   - _decline_: give the reason. Behavior another task owns is that task's work: cite it.
   - _defer_: real, but outside this task: file a backlog item linked to the root.
   - _escalate_: needs a decision you cannot make: stop and report it.
1. **Weigh findings against the design**, not just the diff. A reviewer checks a change against its predecessor, not against the whole feature.
1. **A bug caught by review and not by the tests is two bugs.** The fix commit includes the missing test, at the lowest layer that reproduces the bug.
1. **Do not chase a clean review.** Fix medium and low findings and move on. Get a follow-up review, of the fixing commits only, after fixing a high, critical or blocking finding. Where fix commits are reviewed automatically anyway, read those reviews once the fixes are done, triage new high, critical or blocking findings, and disposition the rest without another round.
1. **Close a review once every finding has a disposition and the accepted fixes are committed.** For roborev: `roborev job comment --job <id> "<what you did>"`, then `roborev job close <id>`. Do not wait for a follow-up review to confirm. Leave a review with an escalated finding open.
1. **Valve.** Three findings against the same mechanism: stop and escalate instead of patching again. Someone with the whole picture decides "patch, or delete and replace?" first.
