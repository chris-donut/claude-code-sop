# SOP Plugin - Minimal, Finish-What-You-Start Workflow

A Claude Code plugin that enforces the "Minimal, Finish-What-You-Start" SOP for consistent, high-quality code delivery.

## Philosophy

- **Base branch always runnable** - Never break main/dev
- **One session = one slice** - Smallest shippable unit
- **Verification-first** - Proven by `./scripts/verify.sh`
- **Diff-sized asks only** - Minimal patches, no sweeping refactors
- **Clean staging** - Always `git add -p`, never `git add -A`
- **Token stop-loss** - Halve scope after 2 failed attempts

## Commands

| Command | Description |
|---------|-------------|
| `/sop:kickoff` | Start a session - asks 3 questions, names slice, creates branch |
| `/sop:bootstrap` | First-time setup - creates `verify.sh` and GitHub Actions workflow |
| `/sop:ask` | Generate diff-sized ask prompt, tracks micro-blocks |
| `/sop:commit` | Pre-commit gate - verify → diff → `git add -p` → commit |
| `/sop:pr` | Create/update PR with SOP hygiene |

## Quick Start

### First Time (New Project)

```
/sop:bootstrap
```

This creates `./scripts/verify.sh` and `.github/workflows/verify.yml`.

### Starting a Session

```
/sop:kickoff
```

Answer the 3 questions:
1. **Goal** - What are you building?
2. **Non-goals** - What are you NOT doing?
3. **Done means** - How do you know it's done?

### Working in Micro-blocks

```
/sop:ask add error handling to the login endpoint
```

After 3 micro-blocks, commit:

```
/sop:commit
```

### Creating a PR

```
/sop:pr
```

## Session State

The plugin tracks session state in `.sop-session.json` (gitignored):

```json
{
  "sliceName": "add-login-validation",
  "branchName": "feat/add-login-validation",
  "baseBranch": "main",
  "goal": "Add input validation to login endpoint",
  "nonGoals": ["Refactor auth system", "Add new auth methods"],
  "doneMeans": "verify.sh passes, invalid inputs return 400",
  "microBlockCount": 2,
  "failedAttempts": 0,
  "startedAt": "2024-01-15T10:00:00Z"
}
```

## Hard Rules Enforced

1. **No commits without verification** - `./scripts/verify.sh` must pass
2. **No `git add -A`** - Hook warns and asks before allowing
3. **Token stop-loss** - After 2 failed attempts, must halve scope
4. **Commit after 3 micro-blocks** - Reminds you to commit regularly

## Slice Selection Criteria

Good slices:
- Verify in ≤30 minutes (target ≤10)
- Touch ≤2 subsystems
- Measurable improvement

Bad slices:
- "Build the whole system"
- "Refactor everything"
- "Touch many modules at once"

## Requirements

- Git
- GitHub CLI (`gh`) for PR operations
- `jq` (optional, for session state management)

## Installation

The plugin is installed at `~/.claude/plugins/sop/`.

Add `.sop-session.json` to your project's `.gitignore`.
