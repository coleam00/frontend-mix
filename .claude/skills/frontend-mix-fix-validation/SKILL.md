---
name: frontend-mix-fix-validation
description: Address validation failures recorded by the frontend-mix-validate session. Run this skill in a Claude Code session driving Opus - failures that survived two Sonnet repair attempts need real judgment, not more grinding. Reads validation-issues.md, fixes each failure at the source, re-runs the validation suite, writes resolution-summary.md. Use when frontend-mix-validate wrote validation-issues.md instead of validation-summary.md.
argument-hint: <validation-issues.md path> <plan.md path>
---

# Frontend-Mix · Fix Validation

You are the **escalation** step for validation failures the prior Sonnet session couldn't fix in two attempts. Opus reasoning is what's needed here.

## What to do

1. Read `$1` (the validation-issues markdown - your work list) end-to-end with the Read tool.

2. Read `$2` (the plan markdown) and extract everything under `## SECTION B - Integration Scope`. This is what was supposed to be wired up.

3. The filename in `$1` carries your run-name. Strip the directory and the `-validation-issues.md` suffix. You'll use it to name your output file.

4. For each failure listed in the issues file, Read the source file(s) it blames before forming a fix. Do not assume - verify.

5. If `$1` or `$2` is empty or doesn't resolve, ask the user for the missing path. Do not work from memory.

## Address each failure

For every failure in the issues file:

1. **Diagnose at the source.** Read the actual offending file end-to-end. If the error says "X has no property Y", figure out whether the right fix is adding Y to X's type or stopping the access to Y, based on what the plan says should be true.
2. **Apply a real fix.** Edit the source code, not the validation config.
3. **Re-run the specific failing command** to confirm the fix worked. If a fix breaks a different check, address that too.

### Banned shortcuts (these are bugs, not solutions)

- Adding `// @ts-ignore`, `// eslint-disable`, or `any` to silence checks
- Deleting failing tests
- Removing strict mode or loosening tsconfig / eslint config
- Skipping validation steps you can't fix
- Pretending a failure is "flaky"

### Genuine blockers

If a failure truly cannot be fixed without changing the plan (e.g. a missing third-party API key, a Clerk feature requiring a paid plan, a breaking change in an upstream library), record it as an **OPEN ISSUE** in the resolution file with the exact blocker. Do not silently skip.

## Re-run the full validation suite once

After all individual fixes, run install → typecheck → lint → build → tests once end-to-end. Record the final state of each step.

## Output

Write to `.claude/artifacts/<run-name>-resolution-summary.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

Contents:
- Each original failure from the issues file
- The fix applied (file:line + what changed) OR "OPEN ISSUE: <blocker>"
- Final state of `bun run build` and tests
- A status line at the end: **"READY TO DEPLOY"** if everything is now clean, else **"NOT READY: <reason>"**

## After fixing

Tell the user the absolute path to `<run-name>-resolution-summary.md` and the next step:

```
If status is READY TO DEPLOY:
  Next: invoke /frontend-mix-smoke with the integration-summary path and the resolution-summary path.

If status is NOT READY:
  Surface the open issues to the user and stop. Do not deploy a broken build.
```
