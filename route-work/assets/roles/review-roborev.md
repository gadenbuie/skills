# Review: roborev, no post-commit hook

When the whole unit is committed, request one review of it and wait:

```sh
roborev review --since <base>      # prints the job ID
roborev job wait --job <id>
roborev job show --job <id>
```

`job wait` exits 1 both when the review fails (it has findings) and when the job itself fails; `job show` tells them apart. A job that failed is `blocked`: say so in your report. Triage the findings as `triage.md` says. A follow-up review covers only the fixing commits: `roborev review --since <last reviewed head>`.
