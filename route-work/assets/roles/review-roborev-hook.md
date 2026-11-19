# Review: roborev with the post-commit hook

Every commit is reviewed automatically. Do not read reviews as they arrive and do not request a range review. When the whole unit is committed, read the reviews of all of its commits together and triage them as one review of the unit:

```sh
git log --format=%h <base>..HEAD
roborev job wait <sha>    # for each commit
roborev job show <sha>
```

A commit with no review, or a job that failed (as opposed to a review with findings), is `blocked`: say so in your report. Triage the findings as `triage.md` says. Your fix commits are reviewed automatically too; read those reviews together once the fixes are done.
