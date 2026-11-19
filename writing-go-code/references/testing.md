# Testing: the reasoning behind the rules

For reviewers, orchestrators and anyone deciding which tests a change needs. The rules themselves are in [SKILL.md](../SKILL.md); this file explains them and covers the harder calls.

## Tests check correctness, not current behavior

Early in a tool's life the cost of change outweighs the cost of defects. A test that taxes change is a net loss even when it adds nominal safety.

The deciding question is whether the test has an oracle: does it know what "right" means independent of the code?
- **A correctness test has one:** an invariant, a refusal, a parsed structure, a contract another tool relies on.
- **A capture test copies its expected value from the program's own output.** It uses the code as its own oracle, so it answers "did this change?", not "is this right?", and it fails on every intentional improvement.

Two quick checks:
- If you can name the invariant in one sentence, the test has an oracle.
- If you would update the expectation when the output gets better, it is a capture test. Delete it or rewrite it around the invariant.

What is worth testing:
- decisions and derived state (what gets admitted, merged, retried, counted, owned);
- safety properties and refusals (never writes, merges, closes or kills when it must not; retries and crash recovery do not act twice);
- contracts with other tools (markers and metadata they read, resulting git state, config parsing and validation);
- machine-facing output (exit codes, `--json`, NDJSON);
- wiring that can break silently (signals, child cleanup, environment isolation).

What is not worth testing:
- **Presentation and text:** golden files, human-readable wording, layout, help text, prompt or doc content, exact error strings.
- **Output anyone can look at.** Run the command and look. An assertion on it is a second, worse copy.

A test is a commitment. Don't freeze a decision that isn't finished; add the test when the behavior is a settled contract. A test that fails for benign reasons trains everyone to update or ignore it, and the suite stops being a signal.

## Proving a test can fail

A test that passes with the feature removed is worse than no test: it reports safety that isn't there. Reviews repeatedly find:

- an assertion inside an `if` that only runs when the output already looks right;
- a file written in setup and never read;
- a test placed in a package that can't observe the behavior (it checks a helper's result, not the effect);
- a fixture that never seeds the state the code under test must not write, so "nothing was written" passes trivially;
- a bound loosened until it can't fail (`launches <= 1` when the contract is exactly one).

The cure is a mutation. Break the code on purpose (invert the condition, delete the call, return early), run the test and watch it fail, then restore the code. For a bug fix, the unfixed code is the mutation: the test must fail before the fix.

## Regression tests

A bug fix starts with a test that fails without the fix, when the bug is a correctness bug. Put it at the lowest layer that reproduces the bug, preferably a unit test in the package that makes the decision.

**Review findings about wording.** Examples: a message that names the wrong command, or a refusal that doesn't say where the reason is. These are fixed and recorded, never pinned with a text assertion.
- If the fix changes a structured fact (which reason, which exit code, which field is named), assert that fact.
- If the fix is visible only in the text, the regression evidence is the recorded disposition ("fixed in <commit>; wording is not under test").

**When a task, a design doc or an orchestrator asks for a test this file advises against** (a golden file, a pinned message): do what the task says, and note the conflict in the commit body and the hand-off, so someone can resolve it in the doc.

## End-to-end scenarios and suite time

Suite time is a design constraint. Every test must earn its seconds, especially one that builds a binary and drives real processes.

- **One end-to-end scenario per wiring path.** A wiring path is behavior observable only across a process or tool boundary. Add a scenario only for a new path, never for a new variant of an existing one; prove the variant with a unit test.
- **Replace slow scenarios.** When a scenario is slow and its oracle can be checked lower down, replace it with a lower-layer test. When in doubt, replace. Keeping an expensive scenario needs a stated contract that no smaller test can check.
- **Record each removal or conversion in one line:** the contract, where it is now covered, and any wiring that is no longer tested. Coverage count is not the target.
- **Cached runs prove nothing about time.** Measure uncached.
- **Watch for a test that multiplies expensive calls.** Hundreds of git commits or process launches in one test belong at a cheaper layer.

## Isolation and process-global state

- **Child environments:** give each test its own temp directories and HOME, and build child environments explicitly.
  - Never pass `os.Environ()` to a child `git`. A git hook exports `GIT_DIR`, and the child then acts on the outer repository.
  - Clear inherited variables that select a user identity or a server.
  - When you override HOME, keep the build and module caches (`GOCACHE`, `GOMODCACHE`) pointing at the real ones, or every child `go` build starts cold.
- **Test helpers and fakes follow the same rules.** Never build a fake binary into a fixed or shared path, and never let a helper inherit the host environment wholesale.
- **Process-global state:** environment, working directory, signal handlers and `umask` belong to the whole test binary.
  - `t.Setenv` and `t.Chdir` panic in a test that is parallel or has a parallel parent. `os.Setenv` and `os.Chdir` enforce nothing and race; never call them in a test.
  - `signal.Ignore` or `signal.Notify` in one subtest leaks into the next unless it is reset.
  - Prefer passing environment and directories in explicitly, so the test can run in parallel. A test that must touch global state stays serial, with a one-line comment saying why.
- **Don't parallelize subtests that share a fixture.**
- **Sibling processes:** several agents or CI jobs may run tests at once on one machine. A test that scans the process table can see their children; match on something your test owns (a temp path or a tag), never on a program name alone.
- **Waiting for absence:** a check that nothing happens may poll for a short, bounded time. Name the bound in a constant and say why it is long enough.
