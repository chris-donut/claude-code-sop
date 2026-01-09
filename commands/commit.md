---
name: commit
description: Pre-commit gate - runs verify, shows diff, guides interactive staging with git add -p, commits cleanly
allowed-tools: ["Bash", "Read", "Write"]
argument-hint: "[optional: commit message]"
---

# SOP Commit - Pre-commit Gate

You are guiding the user through the SOP pre-commit process. Follow these steps strictly in order.

## Step 1: Read Session State

```bash
cat .sop-session.json 2>/dev/null || echo "NO_SESSION"
```

If no session, warn but continue (user might be doing a quick fix).

## Step 2: Run Verification (MANDATORY)

This step is non-negotiable. Run:
```bash
./scripts/verify.sh
```

**If verification FAILS:**
- Show the error output
- Tell the user: "Verification failed. Fix the issues or revert. No WIP commits allowed."
- Do NOT proceed to commit
- Offer to help fix the issues

**If verification PASSES:**
- Show: "Verification PASSED"
- Proceed to next step

## Step 3: Show Git Status

```bash
git status
```

Review what's changed:
- Untracked files
- Modified files
- Staged files

If nothing to commit, tell the user and stop.

## Step 4: Show Diff for Review

```bash
git diff
```

If there are staged changes already:
```bash
git diff --cached
```

Let the user review. Ask: "Does this diff look correct? Any changes that shouldn't be included?"

## Step 5: Interactive Staging with git add -p

**CRITICAL: Never use `git add -A` or `git add .`**

Guide the user through `git add -p`:

```bash
git add -p
```

Explain the interactive options:
- `y` - stage this hunk
- `n` - don't stage this hunk
- `s` - split into smaller hunks
- `q` - quit (done staging)
- `?` - help

For each hunk, help the user decide:
- Does this hunk relate to the slice goal?
- Is this a necessary change or accidental?
- Should this be in a separate commit?

For new files that need to be added entirely:
```bash
git add <specific-file>
```

## Step 6: Review Staged Changes

After staging:
```bash
git diff --cached
```

Confirm with user: "These are the changes that will be committed. Look correct?"

## Step 7: Compose Commit Message

The commit message must fit the pattern:
> "This commit adds/fixes X, and verification still passes."

Based on the session goal and staged changes, propose a commit message:

Format: `type: brief description`

Types:
- `feat:` - new feature
- `fix:` - bug fix
- `chore:` - maintenance
- `refactor:` - code restructuring
- `docs:` - documentation
- `test:` - test changes

Example: `feat: add user validation endpoint`

If user provided a message argument, use that (but validate it fits the pattern).

Ask user to confirm or modify the message.

## Step 8: Commit

```bash
git commit -m "<approved message>"
```

Show the commit result.

## Step 9: Update Session State

Reset micro-block count after successful commit:

```bash
if command -v jq &> /dev/null; then
  cat .sop-session.json | jq '.microBlockCount = 0' > .sop-session.json.tmp && mv .sop-session.json.tmp .sop-session.json
fi
```

## Step 10: Push (Optional)

Ask user: "Push to remote? (y/n)"

If yes:
```bash
git push -u origin <branch-name>
```

If this is the first push for the branch, remind about creating a PR with `/sop:pr`.

## Step 11: Summary

```
Commit Complete
---------------
Branch: <branch>
Commit: <short hash> <message>
Files: <N> files changed

Micro-block count reset to 0.
Next: Continue with /sop:ask or create PR with /sop:pr
```

## Error Handling

### If user tries to skip verification
- Refuse. Say: "SOP rule: No verified run = no commit. Let's fix the verification issues first."

### If user wants to commit unrelated changes
- Suggest creating a separate commit or stashing those changes
- Help them stage only relevant changes

### If commit message is too vague
- Push back: "Can you be more specific? The message should describe what this commit adds/fixes."
