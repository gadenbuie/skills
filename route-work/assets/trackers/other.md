# Tracker: anything else

For Linear, Jira, a spreadsheet, a TODO list or anything without its own file here, work out these answers from the tool's CLI or API and write them into a short note that you give workers in place of a tracker file:

- **Read:** how a worker reads its task, its parent and its dependencies.
- **Tier:** where a task's tier is recorded, if anywhere.
- **Progress and decisions:** where a worker records them.
- **Backlog item:** how to file an out-of-scope finding linked to the root.
- **Commit trailer:** the line that links a commit to its task.
- **Cannot finish:** how a worker marks a task as stopped.
- **Router only:** what marks a task in progress and done, and where the integration branch is recorded.

If you cannot read or update the tracker from the shell, ask the user how they want progress recorded before dispatching anything.
