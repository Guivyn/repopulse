<!-- Project identity and workflow are composed in the SVG below. -->
<p align="center">
  <img src="./assets/readme/hero.svg" width="100%"
       alt="RepoPulse maps Git history into two paths: pulse reports repository rhythm and hotspots; trace follows commits through inspected diffs to evidence.">
</p>

RepoPulse is a user-invoked Codex skill for understanding unfamiliar Git repositories. Run `repopulse pulse` for a health snapshot, or `repopulse trace "..."` to investigate a question through Git history. Git is required; GitHub CLI (`gh`) is optional for pull-request context.

## Quick start

### Install

Clone the repository once:

```bash
git clone https://github.com/Guivyn/repopulse.git
cd repopulse
```

Install the skill:

**Windows PowerShell**

```powershell
$skillsDir = if ($env:CODEX_HOME) { Join-Path $env:CODEX_HOME "skills" } else { Join-Path $HOME ".codex\skills" }
$skillDir = Join-Path $skillsDir "repopulse"
New-Item -ItemType Directory -Force $skillDir | Out-Null
Copy-Item .\skills\repopulse\*.md -Destination $skillDir -Force
```

**macOS / Linux**

```bash
skill_dir="${CODEX_HOME:-$HOME/.codex}/skills/repopulse"
mkdir -p "$skill_dir"
cp skills/repopulse/*.md "$skill_dir/"
```

### Run

In a Codex session pointed at a Git repository:

```text
repopulse pulse
repopulse trace "Why does src/payment/handler.go keep changing?"
```

## Example output

Examples are illustrative; repository details and commit IDs are fictional.

**Pulse**

```text
Pulse: frayed

- Rhythm: 8 commits in the last 30 days, down from 18 in the previous 30.
- Shape: 7 of 8 commits came from one contributor; most churn is in src/payment/.
- Open work: 2 PRs, reported by gh.
- Gap: Git history has no deployment data, so production health is unknown.
```

**Trace**

```text
Question: Why does src/payment/handler.go keep changing?

Answer: Two inspected patches change retry handling. That describes the code changes, not their cause.
Evidence:
- a1b2c3d — diff changes retry handling in src/payment/handler.go.
- d4e5f6a — diff adds timeout coverage in src/payment/handler_test.go.
Inference: the changes appear focused on retry behavior.
Unknown: Git history does not establish the production incident or original motivation.
```

## How it works

**Pulse** reads recent commit rhythm, contributor shape, and file churn. If `gh` is available, it can also include open pull requests. The verdict—**steady**, **frayed**, or **hot**—is a quick signal, not a production audit.

**Trace** starts with targeted history, then inspects relevant patches with `git show`. Historical claims include a commit SHA and path; interpretations are labeled as inference, and causes or intent that Git cannot establish are stated as unknown. A commit title alone is not evidence of why code changed.

## Project files

- `skills/repopulse/SKILL.md` — the two workflows and evidence rules.
- `skills/repopulse/commands.md` — exact Git commands for common questions.

Edit these files to change the skill. The repository is MIT licensed; see [LICENSE](./LICENSE).

