---
description: Start an SOP session - asks the 3 kickoff questions, names the slice, and creates the branch
---

# SOP Session Kickoff

You are starting an SOP workflow session. Follow this process strictly.

## Step 1: Detect Base Branch

Run this command to detect the base branch:

```bash
git branch -a 2>/dev/null | grep -E '(^|\s)(dev|main|master)(\s|$)' | head -1 | xargs || echo "main"
```

Use `dev` if it exists, otherwise `main`.

## Step 2: Check for Existing Session

Check if `.sop-session.json` exists in the project root. If it does, read it and show the user:
- Current slice name
- Branch name
- Micro-block count
- Failed attempt count

Ask: "Continue this session or start fresh?"

## Step 3: Ask the 3 Kickoff Questions

If starting fresh (or no existing session), ask the user these questions and wait for answers:

**Question 1: Session goal** (one sentence)
> What is the single goal for this session?

**Question 2: Non-goals** (up to 3 bullets)
> What are you explicitly NOT doing? (helps prevent scope creep)

**Question 3: Done means** (1-2 verifiable checks)
> How will you know it's done? Must include expected `./scripts/verify.sh` behavior.

If the user can't answer these in under 2 minutes of thinking, tell them: "The slice might be too big. Consider breaking it down."

## Step 4: Validate Slice Size

Check the answers against slice criteria:
- Can verify in ≤30 minutes? (target ≤10)
- Touches ≤2 subsystems?

If it seems too big, suggest breaking it down.

## Step 5: Name the Slice and Branch

Based on the goal, propose:
- **Slice name**: kebab-case, descriptive (e.g., `add-user-endpoint`)
- **Branch name**: `feat/<slice-name>`, `fix/<slice-name>`, or `chore/<slice-name>`

Get user confirmation.

## Step 6: Create Branch and Session File

1. Create the branch from base:
```bash
git checkout <base-branch>
git pull
git checkout -b <branch-name>
```

2. Create `.sop-session.json` in project root:
```json
{
  "sliceName": "<slice-name>",
  "branchName": "<branch-name>",
  "baseBranch": "<base-branch>",
  "goal": "<session goal>",
  "nonGoals": ["<non-goal-1>", "<non-goal-2>"],
  "doneMeans": "<done criteria>",
  "microBlockCount": 0,
  "failedAttempts": 0,
  "startedAt": "<ISO timestamp>"
}
```

3. Remind user to add `.sop-session.json` to `.gitignore` if not already there.

## Step 7: Check Verification Harness

Check if `./scripts/verify.sh` exists:
- If YES: Run it to confirm it works
- If NO: Tell user to run `/sop:bootstrap` first

## Step 8: Confirm Ready

Print a summary:
```
SOP Session Started
-------------------
Slice: <slice-name>
Branch: <branch-name>
Base: <base-branch>
Goal: <goal>
Non-goals: <non-goals>
Done means: <done-criteria>

Ready for micro-blocks. Use /sop:ask to generate diff-sized prompts.
After 3 micro-blocks, commit with /sop:commit.
```
