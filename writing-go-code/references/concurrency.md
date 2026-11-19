# Contexts, goroutines, signals and child processes

For reviewers and for work that starts goroutines, waits on deadlines, handles signals or supervises child processes. The short rules are in [SKILL.md](../SKILL.md).

## Context lifetime

The rule "take `ctx` first" fixes the signature. Most review findings are about the *lifetime* of the context instead.

- **One deadline, one wait.** Create the deadline where the wait starts. A deadline created once and reused across two waits gives the second wait whatever time is left over. A deadline set on a connection and never cleared fails later, unrelated reads.
- **Cleanup outlives cancellation.** After Ctrl-C or a cancelled request, the original ctx is already done, so cleanup that uses it does nothing. Derive a fresh one:

  ```go
  cleanupCtx, cancel := context.WithTimeout(context.WithoutCancel(ctx), 5*time.Second)
  defer cancel()
  ```

- **Long-lived work does not inherit a short-lived ctx.** A mirror, watcher or server that must outlive the call that started it gets its own context and its own stop path. Don't hand it the request's ctx.
- **Non-cancellable calls.** Some writes and syscalls ignore ctx. Don't pass a ctx in and assume it bounds the call; bound it another way (a deadline on the connection, a separate goroutine with an owner) and say so in a comment.
- **Children that must outlive ctx.** `exec.CommandContext` kills the child when ctx ends. For a child that must keep running (a daemon you launch and detach), use `exec.Command`, with a comment on the line saying why.

## Goroutines

- **Every goroutine has an owner that waits for it and a way to stop it.** Use `errgroup` when errors matter; otherwise use a `sync.WaitGroup` and a ctx.
- **Publish state before starting the goroutine that reads or writes it.** A `wg.Go` or `go` statement that runs before a field is assigned races with the assignment.
- **A blocking send needs a reader that is certain to exist,** or a `select` on `ctx.Done()`.
- **Shared values:** a `bytes.Buffer`, `io.Writer`, map or slice touched from two goroutines needs a mutex, or exactly one owning goroutine. This includes writers you pass into a library that writes from its own goroutine (terminal renderers, loggers).
- **Locks:** take them in one order and write that order in a comment, or don't nest them. Don't hold a lock (or a file lock) across a call that can block on another process.

## Signals

- **Register narrowly.** Call `signal.Notify` in the part of the program that handles the signal, for as long as it handles it, and `signal.Stop` when done. Registering for a whole command catches signals meant for other phases.
- **Buffer the channel** (`make(chan os.Signal, 1)` at least). Decide what a second signal means (for example, "force"), and read the channel in a loop, not once.
- **Signal state is process-global.** In tests, reset it, and don't run signal tests in parallel.

## Child processes

- **The program is looked up when the command is created.** `exec.Command(name, ...)` resolves `name` on the *current* process's PATH. Setting `PATH` in `cmd.Env` afterwards doesn't change which binary runs. If `name` isn't found, `Run` returns the stored error. When the child's PATH differs from yours, resolve the binary against that PATH yourself and pass the full path. Test the "tool not installed" case.
- **Process groups:** put a child in its own group (`SysProcAttr.Setpgid`) only when you mean to signal the group. Then kill the group (`-pid`), not just the leader, and keep going past the first error when stopping several groups.
- **Inherited files:** only `Stdin`, `Stdout`, `Stderr` and `ExtraFiles` reach the child. `os.Open` and `os.Pipe` set close-on-exec, but a descriptor from a raw `syscall` call without `O_CLOEXEC` leaks into every child. A grandchild that inherits the child's stdout pipe keeps `cmd.Wait` blocked; set `cmd.WaitDelay`.
- **Hermetic git:**
  - `git` reads the user's config, for example `log.showSignature`, which changes `git log` output. When you parse git output, set `GIT_CONFIG_GLOBAL` and `GIT_CONFIG_SYSTEM` to `/dev/null` (or pass the flags that turn the behavior off).
  - Remove `GIT_DIR` and `GIT_WORK_TREE` from a child's environment unless you set them.
- **stderr:** capture it and include its tail (bounded) in the error. A child's `cmd.Stderr` left nil throws away the reason it failed.
- **Process identity:** a PID alone is reused. Pair it with something the process can't share, such as its start time or a token you passed it.

## Retries and polling

A retry is a poll with a deadline and a backoff; a fixed sleep in a loop is neither. Decide which errors are fatal and which are worth retrying, and only repeat operations that are safe to repeat.
