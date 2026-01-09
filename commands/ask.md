---
name: ask
description: Generate a diff-sized ask prompt and track micro-blocks. Enforces token stop-loss.
allowed-tools: ["Bash", "Read", "Write"]
argument-hint: "<what you want to achieve>"
---

# SOP Diff-sized Ask

You are helping the user create a minimal, focused code change request.

## Step 1: Read Session State

Read `.sop-session.json` from project root:
```bash
cat .sop-session.json 2>/dev/null || echo "NO_SESSION"
```

If no session exists, tell the user to run `/sop:kickoff` first.

## Step 2: Check Token Stop-loss

From the session file, check `failedAttempts`:
- If `failedAttempts >= 2`: **STOP**
  - Tell the user: "Stop-loss triggered. You've had 2 failed attempts."
  - Offer options:
    1. Halve the scope (reduce what you're trying to do by 50%)
    2. Write/trim `@SPEC.md` to clarify requirements
    3. Reset failed attempts counter (only if user confirms they've addressed the issue)

## Step 3: Check Micro-block Count

From the session file, check `microBlockCount`:
- If `microBlockCount >= 3`:
  - Tell the user: "You've completed 3 micro-blocks. Time to commit!"
  - Suggest running `/sop:commit`
  - Ask if they want to continue anyway or commit first

## Step 4: Generate Diff-sized Ask

Based on the user's input argument, generate a structured ask:

```
Change only the minimum files needed to achieve: [USER'S GOAL]

Constraints:
- No refactors unrelated to this change
- No unrelated renames
- No repo-wide formatting changes
- Keep the patch as small as possible

After making the change, tell me:
1. Which files were modified
2. How to verify using ./scripts/verify.sh
3. If the change is bigger than expected, propose a smaller slice instead

Current session context:
- Slice: [slice name from session]
- Goal: [goal from session]
- Non-goals: [non-goals from session]
```

## Step 5: Present the Ask

Show the generated ask to the user and tell them to paste it (or you can use it directly if they confirm).

## Step 6: Update Session State

After the user confirms they're starting a micro-block, increment `microBlockCount`:

```bash
# Read current session
SESSION=$(cat .sop-session.json)

# Update micro-block count (using jq if available, otherwise manual)
if command -v jq &> /dev/null; then
  echo "$SESSION" | jq '.microBlockCount += 1' > .sop-session.json
else
  # Manual update - read, modify, write
  # (implement basic JSON update)
fi
```

## Step 7: Track Completion

After the micro-block is complete, ask the user:
- "Did this micro-block succeed?" (working local state)
- If YES: Keep the changes, micro-block count stays incremented
- If NO:
  - Suggest reverting or stashing
  - Increment `failedAttempts` in session file
  - Check if stop-loss is now triggered

## Micro-block Reminders

At the end of each ask interaction, remind:
- Current micro-block count: X/3
- Failed attempts: Y/2
- "After 3 successful micro-blocks, run `/sop:commit`"

## If User Describes Something Too Big

If the user's goal seems to require:
- More than ~30 minutes of work
- Touching more than 2-3 files significantly
- Multiple unrelated changes

Push back and suggest:
1. Breaking it into smaller pieces
2. Focusing on just one aspect first
3. Creating a spec if the scope is unclear

Example response:
> "This sounds like it might touch several areas. Let's focus on just [smallest piece]. We can do the rest in subsequent micro-blocks or a new slice."
