# What reviews of Go changes keep finding

A checklist for reviewers and for implementers who want to check their own diff. It is ordered by how often each problem appeared in about 470 automated reviews of agent-written Go. Most entries break a rule [SKILL.md](../SKILL.md) already states; the review is where they get caught.

## Tests

1. **The test cannot fail.** Signs:
   - an assertion guarded by an `if`
   - setup that writes something the test never reads
   - a test placed where it can't see the effect
   - a bound loosened until it can't fail
   - a fixture that never seeds the state the code must not write

   Ask: which mutation of the code would make this fail? See [testing.md](testing.md#proving-a-test-can-fail).
2. **A new refusal or side effect has no test.** List every new branch that writes, deletes, refuses, merges or kills; each needs a test in the package that decides it.
3. **Isolation leaks.** Signs:
   - the child gets `os.Environ()`
   - a fixture stopped passing HOME
   - git variables come in from a hook
   - a fake is built into a shared path
   - the HOME override also loses the build cache
4. **Text pinned anyway.** Help-text and wording assertions get added next to older ones. When touching such a test, replace the text check with a structured one.
5. **Slow tests.** A new test multiplies process launches or git commits, or adds an end-to-end scenario for a variant of an existing path.

## Errors

6. **Dropped values and errors in multi-value returns.** `return "", f()` returns `""` on success, even when `f` produced the value the caller needed. `return buf.String(), w.Close()` reads the buffer before `Close` flushes it. A helper that returns only a bool or exit code hides the error behind it.
7. **Discards without a reason:** `_ = x` with no comment, or a nil `cmd.Stderr`.
8. **Error text names the wrong thing:** a message copied from a sibling package that names the other tool, or a command prefix added twice (once by the callee, once by the caller).

## Context and concurrency

9. **Context lifetime:**
   - a deadline reused across waits
   - cleanup run with an already-cancelled ctx
   - a long-lived goroutine given a request's ctx
   - a ctx passed to a call that ignores it

   See [concurrency.md](concurrency.md#context-lifetime).
10. **Goroutine ownership:**
    - a goroutine that can block forever
    - a goroutine started before the state it uses is published
    - a shared writer with no lock
11. **Signals and process groups:**
    - `signal.Notify` registered too broadly
    - one read from a signal channel that can receive two
    - group kill stopping at the first error
12. **Locks:** two locks taken in opposite orders; a file lock held across a blocking call.

## Leftovers

13. **Stale comments.** A doc comment left above the wrong declaration after a deletion or move, or an inline comment describing old behavior. This is the most common low-severity finding.
14. **Dead code after a removal:**
    - a hook nothing sets
    - a field nothing reads
    - a helper with no caller
    - code "kept for a later stage" (build it then)
15. **Repeated work:**
    - a regexp compiled per call
    - a git query per loop item
    - the same closure written twice

## Portability and modules

16. **Build-constrained files not vetted** on the developer's platform. Vet with `GOOS` set for each platform the code targets.
17. **`go.mod` not tidy:** direct imports marked `// indirect`, or a stale `go.sum`.
18. **Standard-library traps:**
    - `filepath.Abs("")` returns the working directory
    - `filepath.WalkDir` on a symlinked root does not follow it
    - `exec.Command` resolves the binary on the current PATH (see [concurrency.md](concurrency.md#child-processes))
    - hand-written struct offsets for syscalls (use `golang.org/x/sys`)
19. **Parsing text that has structure:** classifying log lines by matching their words. Use the structure the writer provides.
