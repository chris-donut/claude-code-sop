---
name: pr
description: Create or update a PR with SOP hygiene - includes goal, non-goals, and verification instructions
allowed-tools: ["Bash", "Read", "Write"]
argument-hint: "[optional: pr title]"
---

# SOP PR - Create/Update Pull Request

You are helping create or update a PR following SOP hygiene standards.

## Step 1: Read Session State

```bash
cat .sop-session.json 2>/dev/null || echo "NO_SESSION"
```

Extract:
- `sliceName`
- `branchName`
- `baseBranch`
- `goal`
- `nonGoals`
- `doneMeans`

## Step 2: Confirm Base Branch

Verify the base branch:
```bash
git rev-parse --abbrev-ref HEAD
```

Confirm we're on the slice branch, not the base branch.

Check the remote base branch exists:
```bash
git ls-remote --heads origin <base-branch>
```

## Step 3: Check for Existing PR

Use GitHub CLI to check if a PR already exists:
```bash
gh pr list --head <branch-name> --json number,title,url
```

If PR exists:
- Show the existing PR
- Ask: "Update this PR or view it?"
- If update: proceed to update description

If no PR exists:
- Proceed to create new PR

## Step 4: Ensure Branch is Pushed

```bash
git push -u origin <branch-name>
```

## Step 5: Run Final Verification

Before creating/updating PR, run verification:
```bash
./scripts/verify.sh
```

If it fails, stop and tell user to fix issues first.

## Step 6: Generate PR Description

Create a PR description following SOP hygiene:

```markdown
## Goal

<session goal from .sop-session.json>

## Non-goals

<list non-goals as bullets>

## How to Verify

1. Check out this branch
2. Run `./scripts/verify.sh`
3. <any additional manual verification steps based on doneMeans>

## Changes

<auto-generate from git log or let user fill in>

---
*This PR follows the [SOP workflow](link-if-available).*
```

## Step 7: Create or Update PR

### Creating New PR

Generate title from slice name or user argument:
```bash
gh pr create \
  --base <base-branch> \
  --title "<type>: <slice description>" \
  --body "<generated description>"
```

### Updating Existing PR

```bash
gh pr edit <pr-number> --body "<updated description>"
```

## Step 8: Verify CI Status

After PR is created/updated, check CI status:
```bash
gh pr checks <pr-number>
```

If CI is running, tell user to wait for it.
If CI failed, help diagnose.

## Step 9: Summary

```
PR Ready
--------
Title: <title>
URL: <pr-url>
Base: <base-branch>
Status: <CI status>

Checklist:
[ ] CI is green (verify job passes)
[ ] PR description includes goal, non-goals, how to verify
[ ] Ready for review

To merge after approval: gh pr merge <number>
```

## PR Title Guidelines

Format: `type: brief description`

Good examples:
- `feat: add user validation endpoint`
- `fix: handle null response in API client`
- `chore: add verification harness`

Bad examples:
- `Update stuff` (too vague)
- `WIP` (not allowed per SOP)
- `Fix` (no description)

## If gh CLI Not Available

If `gh` command is not available:
1. Tell user to install GitHub CLI: `brew install gh` or equivalent
2. Or provide manual instructions:
   - Go to GitHub repository
   - Click "New Pull Request"
   - Select base and compare branches
   - Copy the generated description

## Session Cleanup

After PR is merged (user can confirm), offer to:
1. Delete local branch: `git branch -d <branch-name>`
2. Delete remote branch: `git push origin --delete <branch-name>`
3. Clean up `.sop-session.json`: `rm .sop-session.json`
4. Checkout base branch: `git checkout <base-branch> && git pull`
