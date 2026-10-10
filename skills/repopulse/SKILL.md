---
name: repopulse
description: Answer questions about a codebase using git — pulse it for a quick health read, or trace a specific change or question.
disable-model-invocation: true
---

# repopulse

Pulse a repository. Two branches:

- **pulse** — quick health snapshot: churn, hotspots, rhythm, contributor shape. Take a pulse before any unfamiliar repo.
- **trace** — dig into a specific question: who changed what, when something broke, why a module churns.

A pulse is not an audit. Spend 3–5 git commands; report the shape, not the details. Read `commands.md` for exact flag reference when you need it.

---

## Pulse

Take a reading. Answer: _Does this repo look healthy or sick, and where should I look next?_

**1. Read the shape.**

```
git log --since="30 days ago" --format="%H|%an|%ai|%s" --numstat
git shortlog -sn --since="30 days ago"
git log --since="30 days ago" --format="" --name-only | sort | uniq -c | sort -rn | head -20
```

Commit count, contributor shape, file churn (hotspots).

**2. Read the open work.**

```
gh pr list --state open --limit 20 2>/dev/null || echo "no gh"
```

How much is in flight.

**3. Synthesize a pulse report.**

- _Rhythm_: commits/day, active vs dormant, weekend/odd-hour work.
- _Shape_: contributor distribution (bus factor signal), churn concentration.
- _Gaps_: anything missing from the data (no review info, no deploy data).
- _Verdict_: **steady** / **frayed** / **hot** (one word, then 1–2 sentences why).

**Completion criterion**: A pulse verdict is written. If you cannot reach git (no repo in scope), say so and stop — no fallback.

---

## Trace

Follow a question backward through the commit graph.

| If the question is about... | Start here |
|---|---|
| a file or module | `git log --since="90 days ago" -- <path>` |
| a renamed file | `git log --follow --since="90 days ago" -- <path>` |
| a bug or regression | `git log --all --oneline --grep="fix" --since="90 days ago"` |
| who changed X | `git shortlog -sn --since="90 days ago" -- <path>` |
| a time window | `git log --after="date" --before="date" --format="%h %an %s"` |
| a PR | `gh pr view <number> --json commits,files,reviews` |

**1. Pick the starting point** from the table above. Read `commands.md` for exact flag combinations.

**2. Find and verify the relevant changes.** Start with 2–4 git commands. When explaining how or why code changed, inspect the patch for the relevant commit(s) with `git show --format=fuller --find-renames <commit> -- <path>`. A commit message records what someone wrote; it does not prove the underlying cause. Read only the diffs needed to support the answer, and widen the search only when the evidence is insufficient.

**3. Report.** Lead with the answer, then give evidence. For each historical claim, include the commit SHA and relevant path (or the author/date metadata for an authorship question). Separate:

- **Evidence** — what the commit metadata or inspected diff directly shows.
- **Inference** — an interpretation supported by that evidence, labeled as an inference.
- **Unknowns** — causes, intent, or history the available Git data cannot establish.

Do not claim causality, author intent, or a root cause from a commit message or timing alone. If the repository is shallow or the relevant history is missing, state that limit and avoid "first" or "original" claims.

**Completion criterion**: The user's question is answered with evidence, or an explicit data gap is stated. Historical explanations must be supported by the relevant diff or clearly labeled as inference/unknown.

---

## General rules

- **Lead with the answer.** One sentence, then evidence.
- **Prefer git over gh.** Git works offline and is faster; use gh only when you need PR/issue data and `gh` is configured.
- **Every number has context.** Not "5 bugs" but "5 bugs, 3 in payment module (60%)".
- **Data gaps are honest.** "No deploy data — lead time unavailable." Not silence.
- **Messages are not causes.** Verify explanations against diffs; label unsupported causes and intent as unknown.
- **One branch per call.** Run pulse or trace, not both.

---

## Pulse verdict definitions

| Verdict | Signal |
|---|---|
| **steady** | Regular commits, distributed contributors, moderate churn |
| **frayed** | Single-owner modules, declining commit cadence, PR bottleneck |
| **hot** | Heavy churn concentrated in one area, spike in activity, many open PRs |
