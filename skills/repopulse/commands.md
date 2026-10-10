---
---

# repopulse — git command reference

Exact combinations of git flags for common `pulse` and `trace` operations. Read this file when you need exact flag syntax.

---

## Pulse commands

```bash
# Recent commits with stats (author, date, message, files changed)
git log --since="30 days ago" --format="%H|%an|%ai|%s" --numstat

# Contributor counts
git shortlog -sn --since="30 days ago"

# File churn (hotspots)
git log --since="30 days ago" --format="" --name-only | sort | uniq -c | sort -rn | head -20
```

## Trace commands

```bash
# File history, including renames
git log --follow --since="90 days ago" --format="%h %an %cI %s" -- <path>

# Module or path history
git log --since="90 days ago" -- <path>
git shortlog -sn --since="90 days ago" -- <path>

# Inspect the actual patch before explaining how or why it changed
git show --format=fuller --find-renames <commit> -- <path>

# Check whether the available history is shallow before making "first" or "original" claims
git rev-parse --is-shallow-repository

# Search for fix commits
git log --all --oneline --grep="fix" --since="90 days ago"

# Time window
git log --after="YYYY-MM-DD" --before="YYYY-MM-DD" --format="%h %an %s"

# Author activity
git shortlog -sn --since="90 days ago"
git log --author="<name>" --since="90 days ago" --format="%h %s"

# PR info (requires gh CLI + auth)
gh pr view <number> --json commits,files,reviews
gh pr list --state open --limit 20

# Branch info
git branch -a --sort=-committerdate
git log <branch> --not <base> --format="%h %an %s"

# Find introducing commit (bisect)
git bisect start
git bisect bad HEAD
git bisect good <last-known-good>
# Then mark each step good/bad until the first bad commit is found
git bisect reset

# Commit ancestry
git log --ancestry-path <commit>..<commit>
```
