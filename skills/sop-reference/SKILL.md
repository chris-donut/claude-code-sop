# SOP Reference - Minimal, Finish-What-You-Start Workflow

This skill provides reference knowledge for the SOP workflow. It should be used when the user is working on a project and needs guidance on slice selection, verification practices, commit hygiene, or the overall workflow rules.

---

## Hard Rules (Non-negotiable)

1. **Base branch always runnable** - Base = `dev` (if exists) else `main`. Never break base.

2. **One session = one slice** - A slice is the smallest shippable unit with a verifiable "done".

3. **Verification-first** - "Done" is proven by `./scripts/verify.sh`. If it can't be verified quickly, the slice is too big.

4. **Diff-sized asks only** - Every AI request must produce a minimal patch. No sweeping refactors.

5. **No verified run = no commit** - If verification fails, fix or revert. No "WIP" commits.

6. **Clean staging only** - Never `git add -A`. Use `git add -p` or explicit files.

7. **Token stop-loss** - If you fail twice to land a minimal patch, halve scope or write/trim spec. No thrashing.

---

## Slice Selection Criteria

**Pick only slices that pass this filter:**

| Criterion | Requirement |
|-----------|-------------|
| Proof speed | Verify in ≤30 minutes (target: ≤10) |
| Blast radius | Touches ≤2 subsystems |
| Value | User/dev experience improves measurably |
| Learning | Builds confidence without deep rewrites |

**Score and choose:** proof speed ≥4 AND blast radius ≤2

### Good Slice Examples
- Add one endpoint (plus one test)
- Add one UI component (plus one verify step)
- Add a metric/log for one path
- Fix one flaky test / bug
- Improve bootstrap scripts / env checks
- Add CI job running `verify.sh`

### Bad Slice Examples
- "Build the whole system"
- "Refactor everything then add features"
- "Touch many modules at once"

---

## Spec Gate (Phase 0)

**Spec is optional for tiny changes. Mandatory if any trigger hits:**

### Triggers
- Touches >2 files OR >1 subsystem
- Any UI/UX ambiguity
- External dependency / auth / payments / persistence
- You can't define "done" precisely
- You expect token burn / scope creep

### Output: `@SPEC.md`
- Scope + non-goals
- Acceptance checks (verifiable)
- Contracts (API/data/state) if relevant
- Risks + rollback
- Verification contract (what `verify.sh` proves)

---

## Session Kickoff (3 Required Questions)

Before any coding, answer these:

1. **Session goal** (one sentence)
2. **Non-goals** (up to 3 bullets)
3. **Done means** (1-2 verifiable checks, must include expected `verify.sh` behavior)

If you can't answer in <2 minutes, the slice is too big.

---

## Branch Naming Convention

- `feat/<slice-name>` - New features
- `fix/<slice-name>` - Bug fixes
- `chore/<slice-name>` - Maintenance tasks

---

## Micro-block Rules

Each micro-block must be:
- One minimal patch
- Small intent (10-30 minutes)
- Ends in either:
  - Working local state, OR
  - Revert/stash (no messy half-state)

### Integration Checkpoint (Hard Rule)
After every **3 micro-blocks**:
1. Run `./scripts/verify.sh`
2. If green → make one clean commit
3. Push to PR

---

## Diff-sized Ask Template

```
Change only the minimum files needed to achieve: <X>.
No refactors. No unrelated renames. No repo-wide formatting.
Return the minimal patch and tell me how to verify using ./scripts/verify.sh.
If the change is bigger than a small patch, propose a smaller slice instead.
```

---

## Commit Rules

### One-sentence Commit Rule
Commit message must fit:
> "This commit adds/fixes X, and verification still passes."

If you can't write it, split the change.

### Pre-commit Gate
1. `./scripts/verify.sh`
2. `git status`
3. `git diff` (sanity)
4. Stage cleanly: `git add -p` or explicit files
5. `git commit -m "type: <one sentence>"`
6. Push

---

## Token Budget / Stop-loss

1. **Patch definition limit:** If you can't describe the patch in ≤5 acceptance bullets, stop and shrink slice.

2. **Stop-loss:** After 2 failed attempts (sprawl, contradictions, repeated rewrites):
   - Freeze scope
   - Reduce slice by 50%
   - Write/trim `@SPEC.md` if needed

3. **No "ambitious prompts."** You are buying diffs, not dreams.

---

## PR Hygiene

- PR should contain a few clean commits
- Description includes: goal, non-goals, how to verify
- CI must be green (verify job)
