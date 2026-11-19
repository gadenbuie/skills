---
name: writing-go-code
description: Rules for writing and testing Go code. Use for any task that creates, edits, fixes, refactors or tests Go (.go files, go.mod, *_test.go), including new packages, new commands, bug fixes, small features and test work. Read it before writing the first line of Go.
---

# Writing Go

Read all of this file before you write Go; it is short on purpose. Priority: (1) your task prompt and its acceptance criteria, (2) the repository's instructions (`AGENTS.md`, `CONTRIBUTING.md`), (3) this skill. If a higher one disagrees with this skill, follow it and say so in your hand-off. When nothing covers a case, write the simplest code that does the task, in the style of the package you are in.

## Before you write code

- [ ] Read the package you will change and one of its test files. Match their style.
- [ ] Search for an existing helper before writing a new one.
- [ ] Do only what the task asks: no options, hooks, interfaces or settings "for later".
- [ ] Add no dependency the task does not name.

## Before you write a test

- [ ] Say in one sentence what must be true, without looking at what the code prints. That sentence is the test's *oracle*. No oracle, no test.
- [ ] Fixing a bug: write the test first and watch it fail. Put it in the lowest layer that shows the bug: a unit test of the function that makes the decision.
- [ ] Fixing a wording-only finding (a message, help text): if the fix changes a fact (which exit code, which field), assert that fact. Otherwise write no test and put "wording; not under test" in the commit body.
- [ ] Test decisions, refusals, exit codes, machine-readable output and state written to disk. Fewer, sharper tests beat many overlapping ones.
- [ ] Assert facts, not wording: error identity (`errors.Is`), exit codes, returned values, parsed JSON, files written or not written. Never assert error messages, help text, log lines or layout, and never use golden files.
- [ ] Every assertion runs on every path. No assertion inside an `if` that can skip it.
- [ ] Isolate: temp directories, a private HOME, no real services, no real user files.
- [ ] Once per new test: make one small edit to the code under test that still compiles (invert the condition, delete the call). Run only that test (`go test -run '^TestName$' ./pkg`) and see it fail. Undo the edit and check `git diff` shows no trace of it. For a bug fix, the failing run before the fix counts.

## Before you hand off

- [ ] Renamed something: grep for the old name and update every use. Removed something: delete what only it used (fields nothing reads, helpers nothing calls, flags, comments about it). Keep code that still has other callers. `go build ./...` must pass.
- [ ] Reread the comments in and above each function you changed. Fix or delete any that describe old behavior.
- [ ] Every error is handled. Check each `return x, f()` and each `_ =`.
- [ ] Each `if` or `switch` case you added that writes, deletes, refuses or kills something has a test.
- [ ] `gofmt`, `go vet ./...` and the linter pass, test files included. For each `//go:build` platform you touched, also run `GOOS=<that os> go vet ./...` (for example `GOOS=linux` on a Mac).
- [ ] Imports changed: run `go mod tidy` and include `go.mod` and `go.sum` in your change.
- [ ] `go test -race` passes on the packages you changed. Then run the checks, commit and hand off as your task and the repository's instructions say (priorities 1 and 2 at the top of this file).

## Rules

### Packages and types

- One package per concept. `main` and command code stay thin: parse flags, call a library package, print.
- Library packages return values and errors. They never print, call `os.Exit`, or read flags.
- Short noun package names, not repeated in identifiers: `store.Client`, not `store.StoreClient`. No `util`, `common` or `helpers` packages.
- No `init()`, no import cycles, no package variable that changes after start. Unexported read-only tables are fine. Flag variables bound in `main` are fine. For a test seam, pass the dependency in; never swap a package variable.
- Plain structs with exported fields. Make the zero value usable. Use `*T` only when "absent" and "zero" differ.
- Define an interface in the package that uses it, only when two implementations exist or a test must swap one. Keep it small. Return concrete types.
- No generics unless the alternative is copying a function for several types.
- Initialisms in one case: `ID`, `URL`, `HTTP`, `JSON`, `PID`, `CWD`. No `Get` prefix on getters.
- Enumerations are a named string type with constants, so they print readably. `iota` only for values that never leave the process.

### Errors

- Return errors. Panic only for states that cannot happen.
- Wrap with what you were doing: `fmt.Errorf("read profile %q: %w", name, err)`. Lowercase, no final period, no "failed to", no command or program name (the caller adds its own).
- `return x, f()` hides bugs: on success the caller gets `x` as it was before `f` ran.

  ```go
  // Don't: on success the caller gets "" and nil; the response is thrown away.
  return "", c.post(ctx, body, nil)

  // Do:
  var resp commentResponse
  if err := c.post(ctx, body, &resp); err != nil {
      return "", fmt.Errorf("post comment: %w", err)
  }
  return resp.ID, nil
  ```

- Ignore an error only with a reason on the same line: `_ = f.Close() // read-only; nothing to flush`.
- A helper that returns a bool or an exit code must also return the error behind it.
- Callers branch with `errors.Is` and `errors.As`. Add a sentinel or error type only when a caller branches on it.
- Decode input into typed structs, never `map[string]any`, and never type-assert without `, ok`. One `Validate()` collects every problem with its field path and returns `errors.Join(problems...)`.

### Context, goroutines and resources

- A function that does I/O, runs a process or can block takes `ctx context.Context` first.
- A deadline belongs to one wait: create it where that wait starts and never reuse it for a second wait.
- Cleanup after cancellation gets its own context: `context.WithTimeout(context.WithoutCancel(ctx), 5*time.Second)`.
- Stay synchronous unless the task needs concurrency. Every goroutine has an owner that waits for it and can stop it (`errgroup`, or a `sync.WaitGroup` and a ctx). No fire-and-forget.
- A value two goroutines touch, including a `bytes.Buffer` or `io.Writer` handed to a library, is guarded by a mutex or owned by one goroutine.
- `defer` the release on the line after the error check. Release on every path, including an error path that reassigns the variable.
- No `time.Sleep` to wait for something. Wait on the thing (a channel, `Wait()`), or poll with a deadline.
- Write a file other processes read as a temp file in the same directory, close it and check the `Close` error, then `os.Rename` it.

### External processes

- `exec.CommandContext(ctx, name, args...)`: an argument slice, never a shell string. A child that must outlive ctx (a daemon you detach) uses `exec.Command`, with a comment saying why.
- One small client package per external tool; nothing else runs that tool.
- Decode tool JSON into typed structs and ignore unknown fields. Never decide by matching human-readable text.
- Put the child's stderr in the error you return.
- Set `cmd.Env` when the child must not inherit your environment. A child `git` started from a git hook inherits `GIT_DIR` and acts on the wrong repository.
- `exec.Command` looks up the binary on *your* PATH when the command is created; setting `PATH` in `cmd.Env` does not change which binary runs. If the child's PATH differs, resolve the full path yourself. Test the case where the tool is not installed.

### Tests

- With testify: `require` for preconditions, `assert` for checks. Use the specific forms (`require.NoError`, `require.ErrorIs`, `require.ErrorAs`, `assert.Len`), not `require.True(t, errors.As(...))`.
- Table-driven when cases share a shape; name cases by behavior ("refuses a dirty worktree").
- Mark independent tests `t.Parallel()`. A test that uses `t.Setenv`, `t.Chdir` or signal handlers must not be parallel; prefer passing environment and directories explicitly, so it can be. Never call `os.Setenv` or `os.Chdir` in a test.
- Helpers call `t.Helper()`; cleanup goes in `t.Cleanup`.
- An end-to-end test (built binary, real child processes) only for *wiring*: a connection that no smaller test can see, such as signal handling, child cleanup, or a command calling its library. A new case of an existing path is a unit test.
- Tests that need the network or real external tools sit behind `//go:build integration`.
- A race report is a failing test.

### Style

- One `// Package x ...` comment, in `x.go`. Doc comments on exported identifiers: full sentences that start with the name. Other comments say why, not what.
- Small functions with one job and early returns. A closure used twice becomes a named function. Compile a regexp once, at package level; do not redo work for every loop item.

## References

These are for reviewers and harder problems; you don't need them for ordinary tasks.

- [references/testing.md](references/testing.md): why tests check correctness, slow and end-to-end tests, review findings about wording.
- [references/concurrency.md](references/concurrency.md): contexts and deadlines, signals, process groups, locks, child processes.
- [references/review.md](references/review.md): what reviews of Go changes keep finding.
