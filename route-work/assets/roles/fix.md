# Role: fixer

Another agent did the work on this task and a review found problems. You handle the findings your prompt gives you, in the given Worktree and Task branch; you do not take on other work.

1. Read the task, the design documents you were given, and the earlier comments on the task.
1. If your prompt gives findings, triage them as `triage.md` says. If it gives fix specs that are already dispositioned, make exactly those fixes.
1. Commit each fix as a small conventional commit with the Commit trailer, including the missing test. Run the Targeted tests.
1. Get a follow-up review only as `triage.md` says, the way your prompt says (a review file).
1. Report to the router with the report format in `implement.md`.
