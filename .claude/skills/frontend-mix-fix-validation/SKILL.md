---
name: frontend-mix-fix-validation
description: Address validation failures recorded by the frontend-mix-validate session. Run this skill in a Claude Code session driving Opus - failures that survived two Sonnet repair attempts need real judgment, not more grinding. Reads validation-issues.md, fixes each failure at the source, re-runs the validation suite, writes resolution-summary.md. Use when frontend-mix-validate wrote validation-issues.md instead of validation-summary.md.
argument-hint: <path-to-run-name-validation-issues.md> <path-to-run-name-plan.md>
---

# Frontend-Mix · Fix Validation

You are the **escalation** step for validation failures the prior Sonnet session couldn't fix in two attempts. Opus reasoning is what's needed here.

## Arguments

Two paths come in via `$ARGUMENTS`, space-separated:

1. The path to `<run-name>-validation-issues.md` (your work list)
2. The path to `<run-name>-plan.md` (for SECTION B context: what was supposed to be wired up)

Example value of `$ARGUMENTS`:

```
.claude/artifacts/acme-saas-landing-validation-issues.md .claude/artifacts/acme-saas-landing-plan.md
```

If either is missing, ask the user for it. Do not work from memory.

## STEP 0 - Read inputs and extract run-name (do this FIRST)

1. Use the Read tool to open `validation-issues.md` end-to-end. This is your work list.
2. Use the Read tool to open `plan.md` and extract `## SECTION B - Integration Scope`.
3. **Extract the run-name** from the first input filename (strip directory, strip `-validation-issues.md`). You'll use it to name your output file.
4. For each failure listed in validation-issues.md, Read the file(s) it blames. Do not assume - verify.

## Address each failure

For every failure in validation-issues.md:

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
- Each original failure from validation-issues.md
- The fix applied (file:line + what changed) OR "OPEN ISSUE: <blocker>"
- Final state of `bun run build` and tests
- A status line at the end: **"READY TO DEPLOY"** if everything is now clean, else **"NOT READY: <reason>"**

## After fixing

Tell the user the absolute path to `<run-name>-resolution-summary.md` and the next manual step:

```
If status is READY TO DEPLOY:
  Next: deploy (Opus).
  claude --skill ~/.claude/skills/frontend-mix-deploy \
         "<absolute-path>/.claude/artifacts/<run-name>-plan.md <absolute-path>/.claude/artifacts/<run-name>-resolution-summary.md"

If status is NOT READY:
  Surface the open issues to the user and stop. Do not deploy a broken build.
```
