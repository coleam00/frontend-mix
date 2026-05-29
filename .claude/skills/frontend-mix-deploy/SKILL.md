---
name: frontend-mix-deploy
description: Deploy a validated build per SECTION C of a planning document. Run this skill in a Claude Code session driving Opus - the deploy step is reasoning-light most of the time but needs judgment when the dry-run output looks off. Skill is generic: it does whatever SECTION C says (Vercel, Fly, Railway, clerk deploy, local-only). Refuses to deploy unless validation is confirmed clean.
argument-hint: <plan.md path> <validation-summary.md OR resolution-summary.md path>
---

# Frontend-Mix · Deploy

You are the **deploy** step of a manual mixed-provider build.

## What to do

1. Read `$1` (the plan markdown) with the Read tool. Extract everything under `## SECTION C - Deployment Plan`. Ignore SECTION A and SECTION B.

2. Read `$2` (the validation status file) with the Read tool. It's either:
   - `<run-name>-validation-summary.md` (clean run, came straight from validate) → proceed.
   - `<run-name>-resolution-summary.md` (after fix-validation) → check the final status line. If it ends with "NOT READY: <reason>", **do not deploy.** Write a deploy-summary noting the blocker and exit cleanly.

3. The filename in `$1` carries your run-name. Strip the directory and the `-plan.md` suffix. You'll use it to name your output file.

4. If `$1` or `$2` is empty or doesn't resolve, ask the user for the missing path. Do not deploy a build you cannot confirm is clean.

## What SECTION C might say

- **Local only** - no deploy. Write a deploy-summary noting why and exit cleanly.
- **A platform CLI** - `vercel deploy --prod`, `flyctl deploy`, `railway up`, `wrangler deploy`, etc.
- **An auth provider's deploy step** - e.g. `clerk deploy` (once it ships publicly) to take a Clerk instance from dev to prod.
- **Custom commands** the plan describes.

## Your job

1. Identify the deploy target and pre-deploy steps from SECTION C.
2. Run pre-deploy steps in order (env var promotion, migrations, etc.).
3. **Dry-run first wherever the tool supports it** (`vercel deploy --prebuilt`, `clerk deploy --dry-run`, `flyctl deploy --dry-run`). Show the diff or planned changes.
4. If the dry-run looks correct, run the real deploy.
5. Run the post-deploy verification SECTION C describes (curl a health endpoint, hit the deployed URL, check the auth flow end-to-end).

## Output

Write to `.claude/artifacts/<run-name>-deploy-summary.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

Contents:
- Target (vercel / fly / local / etc.)
- Every command run, with output
- Dry-run diff (if any)
- Deployed URL (if any)
- Verification results
- Rollback instructions if something looks off

If SECTION C said local-only or validation was NOT READY, the file is one or two lines explaining why no deploy happened.

## After deploying

Confirm the absolute path to `<run-name>-deploy-summary.md` and surface any post-deploy follow-ups (DNS, custom domain config, monitoring) that the plan flagged.
