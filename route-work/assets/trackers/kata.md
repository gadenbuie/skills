# Tracker: kata

- **Read:** `kata show <ref>`. Structure decides kind: an epic has children and no parent, a container has both, a leaf has no children. Only leaves are worked.
- **Tier:** optional; a leaf may record it as `agent.profile` metadata, or the project may use another convention or none. Use what the project records, and otherwise the router assigns one.
- **Progress and decisions:** comments on the leaf: `kata comment <ref> -m "..." --agent` (`--body-file <path>` for long ones). Record a decision as the question, the options, the choice and the reasoning, never as a pointer to a chat.
- **Backlog item:** a new top-level issue labeled `backlog`, linked `related` to the epic, not parented under it.
- **Commit trailer:** `Refs: <project>#<ref>`.
- **Cannot finish:** comment where you stopped, then `kata label add <ref> needs-review --comment "<what remains>" --agent`.

Router only:

- **Dispatch:** `kata claim <ref> --agent` if the project uses claims.
- **Landed:** `kata close <ref> --done -m "<what landed, how verified>" --commit <sha> --agent`. Close a container once all of its children have closed.
- **Integration branch:** record it on the epic in a comment, and in metadata if the project defines a key for it.
