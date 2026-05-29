---
name: frontend-mix-validate
description: Run the full validation suite (install, type-check, lint, build, tests) on a freshly integrated app and fix anything broken. Run this skill in a Claude Code session driving Sonnet - validation is grind work, Sonnet is the right tool. Two-attempt cap per step; failures are recorded for the fix-validation escalation step, not papered over. Use after the frontend-mix-integrate session finishes.
argument-hint: <path-to-run-name-integration-summary.md>
---

# Frontend-Mix · Validate

You are the **validation** step of a manual mixed-provider build. Sonnet handles this - it's grind work, not judgment. **Two attempts per check, then record the failure. Never paper over real bugs.**

## Arguments

`$ARGUMENTS` is a single absolute path to the integration summary produced by `frontend-mix-integrate`, e.g.:

```
.claude/artifacts/acme-saas-landing-integration-summary.md
```

The filename prefix (everything before `-integration-summary.md`) is the **run-name**. You'll use it to name your output file.

If `$ARGUMENTS` is empty or not a path, ask the user for the integration summary path. Infer the stack from the repo as a fallback (package manager via lockfile, framework via `package.json`).

## STEP 0 - Read input and extract run-name (do this FIRST)

1. Use the Read tool to open the path in `$ARGUMENTS` end-to-end. Match the commands and toolchain it documents - do not invent new ones.
2. **Extract the run-name** from the input filename (strip directory, strip `-integration-summary.md`). You'll use it to name your output file.

## Steps (each capped at 2 attempts)

1. Install: `bun install` (or `npm install` / `pnpm install` based on lockfile).
2. Type check (`tsc --noEmit`, `bun run typecheck`, etc).
3. Lint (`bun run lint`, `eslint .`, etc).
4. Build: `bun run build` (or framework equivalent).
5. Tests if a test script exists.

For each step:
- **Attempt 1**: run the command. If it passes, move on.
- **Attempt 2**: if it failed, try one targeted fix (read the error, edit the offending file, re-run). If it still fails, STOP retrying that step and record the failure.

## Failure rules (no shortcuts)

- Do NOT paper over real bugs. A failing type check on `any` is a bug.
- Do NOT delete tests to make them pass.
- Do NOT add `// @ts-ignore`, `// eslint-disable`, weaken types, or skip steps.
- If you find yourself wanting to do any of that, stop and record the failure.

## Output - exactly ONE of these two files

Both go to `.claude/artifacts/` with the `<run-name>` prefix. Create the directory if it doesn't exist.

### Case A - All steps passed cleanly

Write `.claude/artifacts/<run-name>-validation-summary.md` containing:
- Each command run with its exit code
- One-line confirmation: "All checks passed."

### Case B - One or more steps failed after 2 attempts

Write `.claude/artifacts/<run-name>-validation-issues.md` containing, for each failure:
- Step name (e.g. "type check")
- Exact command that failed
- Truncated error output (last 30 lines)
- File(s) most likely responsible (your best guess)
- One-sentence hypothesis for the root cause

Then STOP. The next session (fix-validation, run with Opus) reads this file and addresses each failure with judgment.

## After validating

Tell the user the absolute path to whichever file you wrote, and the next step:

```
If you wrote <run-name>-validation-summary.md (clean):
  Next: deploy (Opus).
  claude --skill ~/.claude/skills/frontend-mix-deploy \
         "<absolute-path>/.claude/artifacts/<run-name>-plan.md <absolute-path>/.claude/artifacts/<run-name>-validation-summary.md"

If you wrote <run-name>-validation-issues.md (failures recorded):
  Next: fix-validation (Opus) to address the failures.
  claude --skill ~/.claude/skills/frontend-mix-fix-validation \
         "<absolute-path>/.claude/artifacts/<run-name>-validation-issues.md <absolute-path>/.claude/artifacts/<run-name>-plan.md"
```
