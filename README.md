<p align="center">
  <img src="./assets/readme/hero.svg" width="100%"
       alt="repopulse — Codebase intelligence from git. A terminal-themed hero showing the two commands: pulse for health snapshot, trace for question answering.">
</p>

---

## What is repopulse

**repopulse** is a lightweight, git-first [Codex skill](https://docs.codex.com) that answers questions about a codebase. Give it a repository, and it produces a readable health snapshot — or follows a specific question backward through the commit graph.

It replaces the earlier six-skill Pipeline with a single user-invoked skill and one reference file. No dashboards, no databases, no external tokens required.

---

## From EIA to repopulse

The previous version (Engineering Intelligence Agent, or EIA) was a six-skill pipeline with rigid JSON contracts, bilingual rules, and a full orchestrator. **repopulse** is the same idea halved twice — half the scope, a fraction of the surface area.

| | EIA (old) | repopulse (new) |
|---|---|---|
| **Skills** | 6 files, pipeline orchestration | 1 `SKILL.md` + 1 `commands.md` |
| **Invocation** | Model-invoked, always in context | User-invoked, zero context load |
| **Data sources** | git + GitHub + external APIs | git first, `gh` when available |
| **Language** | Chinese + English bilingual | English only |
| **Output** | Rigid JSON schemas per step | Natural language report |
| **Analytics depth** | Risk scoring, RCA, trend, planning | Pulse (health) + Trace (question) |
| **Size** | ~30 KB across 6 files | ~4.7 KB across 2 files |

### What was removed and why

| Removed | Rationale |
|---|---|
| Intent classifier table (debug / investigate / monitor / predict / report / optimize) | The model already distinguishes question types — the table was a **no-op** |
| Risk scoring formulas with score thresholds | A useful heuristic, but too specific to be general — churn and review depth are better signalled directly |
| Knowledge graph (entity-relationship model) | The commit graph is the source of truth; maintaining a second graph alongside git is unnecessary |
| JSON output schemas | An artefact of the pipeline architecture, no longer needed with a single file |
| Bilingual rules (every table in Chinese and English) | **Duplication** — single language is more precise and maintainable |
| `~/.eia/memory.md` persistence | The skill is stateless by design; history lives in git |

### What was kept and sharpened

| Kept | How it improved |
|---|---|
| `git log` with `--numstat` and `--name-only` | Consolidated into `commands.md` as external reference |
| Churn and hotspot detection | Anchored on the leading word **pulse** — the same analysis, clearer framing |
| PR review via `gh` CLI | Same command, lighter wrapper |
| Data-gap honesty | Made explicit in the completion criteria |

---

## Two branches

### Pulse

A quick health snapshot. Three steps:

1. **Read the shape** — `git log` for commit cadence, contributor distribution, and file churn.
2. **Read the open work** — `gh pr list` for in-flight pull requests.
3. **Synthesise a verdict** — **steady**, **frayed**, or **hot**, with supporting evidence.

Use pulse before any unfamiliar repository. It answers: _is this repo healthy, and where should I look next?_

Completion criterion: a pulse verdict is written.

### Trace

Follow a question backward through the commit graph. Given a file, a time window, a PR number, or a symptom, it runs 2–4 git commands and reports the causal chain.

| When you ask about | It starts with |
|---|---|
| A file or module | `git log --since="90 days ago" -- <path>` |
| A bug or regression | `git log --all --oneline --grep="fix" --since="90 days ago"` |
| Who changed what | `git shortlog -sn --since="90 days ago" -- <path>` |
| A time window | `git log --after / --before` with `--format` |
| A PR | `gh pr view <number> --json commits,files,reviews` |

Completion criterion: the question is answered with evidence, or an explicit data gap is stated.

---

## Quick start

**Prerequisites:** git (required), GitHub CLI `gh` with auth (optional, for PR data).

The skill is user-invoked (`disable-model-invocation: true`). In any Codex session pointed at a git repository:

```bash
# Health snapshot
repopulse pulse

# Specific question
repopulse trace "why is src/payment/test_handler.go changing every week?"
```

That is the full API. Two commands.

---

## Implementation

```
skills/repopulse/
├── SKILL.md         — Core behaviour (user-invoked, 2 branches, completion criteria)
└── commands.md      — External reference (exact git flag combinations)
```

### Design decisions

**User-invoked.** `disable-model-invocation: true` removes the description from the agent's context. Zero context load — the skill costs nothing until you call it by name.

**Leading word "pulse".** The model already understands _pulse_ as a quick check — its pretrained meaning costs no tokens to install. The verdicts (steady / frayed / hot) extend the metaphor naturally.

**Progressive disclosure.** Basic git commands are inlined in `SKILL.md`. Exact flag combinations with edge cases live in `commands.md`, loaded only when the agent needs them.

**Single pass per call.** The skill runs exactly one branch. No orchestration, no pipeline state, no JSON handoff.

---

## Why this matters

The original EIA pipeline was comprehensive but heavy — six skills, bilingual duplication, rigid schema contracts, and a full orchestrator. **repopulse** preserves the core insight (git holds the answers) while cutting the surface area to a fraction.

**For users:** faster answers, no configuration, works on any git repository.

**For maintainers:** one file to read, one file to change, `writing-great-skills` principles applied throughout.

**For the ecosystem:** a demonstration that developer-tool skills can be thin, predictable, and zero-cost when they are not being used.

---

## Development

**License:** MIT

The skill is a single Markdown file plus a reference companion. To modify it:

1. Edit `skills/repopulse/SKILL.md`.
2. Run the skill against a test repository.
3. If behaviour diverges from the completion criteria, sharpen the wording in `SKILL.md` or move edge-case commands to `commands.md`.

To install it locally:

```bash
# Copy the skill directory into your Codex skills path
cp -r skills/repopulse ~/.codex/skills/repopulse
```

Contributions that add a new branch should justify the cut by the **granularity** principle: a new branch earns its place only when it serves a genuinely distinct invocation pattern that neither pulse nor trace covers.
