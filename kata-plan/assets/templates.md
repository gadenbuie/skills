# Kata body templates

Replace `<…>` and drop lines that do not apply.

## Leaf

```markdown
Part of <project>#<epic>[ / <project>#<container>]. Design: <path>#<section>. Plan step: <name>.
Reference: <path or URL to the reference implementation, for a port>.
Branch: work lands on <integration or target branch>[; commit to <slice branch> while the container is open].

<What this task changes and why, in user-visible terms.>

## Scope
- <file>: <symbol> — <change>
- <new file> (new): <what it holds>
- Decisions already made: <mechanism decisions>

Out of scope: <item> (owned by <project>#<ref>); <item> (not planned).
Shared files: <files another open task also edits> (<ref>).
Blocked by <project>#<ref> because <reason>.

## Verify
- <observable outcome: command and expected result, exact output, scenario>
- <expected output changes>
- Targeted tests: `<command>`
- Baseline: <checks verified green at <sha>; no need to rerun | none>

## Done when
<End state: what exists, what passes, what is recorded on this task.>

Do not stop to ask about anything you can work out yourself from this body, the design or the
reference. If you are blocked, first try <fallback 1>, <fallback 2>, <fallback 3>. Report back
only after concrete progress or after exhausting reasonable options.
```

## Container

```markdown
Part of <project>#<epic>. Design: <path>#<section>.
Lands as one unit on <target branch>; children share one integration worktree.

<What the unit delivers once landed.>

Children, in order (their bodies own their scope):
- <project>#<ref> <title> — <tier>
- <project>#<ref> <title> — <tier>, blocked by <ref> because <reason>

Verify for the unit: <end-to-end outcome after all children land>.

## Before starting work under this task
Once, when the first child starts; record the result here:
- <e.g. review the plan and children's bodies against what has landed since; adjust and comment what changed>

## Before closing this work
Once, after every child has landed:
- <e.g. review and verify the whole unit's diff against <target branch>; run <full checks>>
```

The two sections fit an epic body too; include them only when needed.

## Planning leaf (rolling waves)

Very large or complicated projects only. Blocked by the wave it follows; tier `medium` unless the repository says otherwise.

```markdown
Planning only; do not implement or land code. After <project>#<previous wave> lands, reconcile
<project>#<next wave> against the landed code, the design and <reference>.
Focus: <questions this wave must settle>.

Read the hand-offs and decisions on <previous wave>'s closed leaves, then plan the wave
following ~/.agents/skills/kata-plan/SKILL.md (steps 2-4; ask the operator only if the wave
changes what gets built). Update later tasks the landed work overturned. No child may start before this leaf closes.

Done when the wave's leaves are written and blocked correctly and this leaf closes with their
refs and a short outline. Leave the implementation undispatched.
```

## Hand-off comment on the root

```markdown
Plan (kata-plan): <one-line goal>. Target branch: <branch>.

Tasks:
- <ref> <title> — <tier>; parallel with <ref> | after <ref> (<reason>)
Once-per-unit: <container ref> before starting: <step>; before closing: <step>.
[Later waves: <ref> <title>, planned by <planning leaf ref>.]

Deferred: <item> → <ref or backlog ref>.
Decisions: <question> — <choice> (<reason>).
Next: /route-work <root ref>
```
