---
name: frontend-mix-integrate
description: Wire up auth, backend API, and third-party SDK integrations into a UI scaffold by executing SECTION B of a planning document. Run this skill in a Claude Code session driving Opus - integration is reasoning-heavy. The skill is plan-driven, not vendor-specific: it uses whatever auth/SDKs/APIs SECTION B says. Use when the design session has finished and `ui-summary.md` plus `plan.md` are ready.
argument-hint: <plan.md path> <ui-summary.md path>
---

# Frontend-Mix · Integrate

You are the **integration** step of a manual mixed-provider build. Opus does the judgment here.

## What to do

1. Read `$1` (the plan markdown) and `$2` (the ui-summary markdown) end-to-end with the Read tool.

   From the plan, extract everything under the `## SECTION B - Integration Scope` header. Ignore SECTION A and SECTION C.

   From the ui-summary, take every `// INTEGRATION:` stub as one of your work items.

2. The filename in `$1` carries your run-name. Strip the directory and the `-plan.md` suffix to get the slug. You'll use it to name your output file.

3. If either `$1` or `$2` is empty or doesn't resolve, ask the user for the missing path. Do not work from context summaries; the artifacts are the files on disk.

The `clerk-cli` skill is loaded so you CAN drive the `clerk` binary directly if SECTION B picked Clerk for auth. If SECTION B picked a different auth provider, ignore it and use whatever the plan says (Supabase, NextAuth, Auth.js, etc.).

## Your job

1. **Auth** - implement whatever auth provider SECTION B chose. If Clerk: drive the `clerk` CLI directly (`clerk init`, `clerk link`, `clerk env pull`, `clerk config patch --dry-run` then apply, `clerk doctor --json` at the end). Once `clerk deploy` ships publicly, that becomes the final step. If Supabase / NextAuth / Auth.js / something else: do that instead.
2. **Wire every `// INTEGRATION:` stub** the design session left.
3. **Build the backend API** per SECTION B. Every protected handler authenticates against the chosen auth provider's session.
4. **External services** - wire up any SDKs/APIs SECTION B listed (Stripe, Supabase, OpenAI, etc.).
5. Run whatever health checks the chosen stack provides. Fix every failure before signaling complete.

## Important constraints

- Do NOT redesign the UI. The design session owned look-and-feel; you own behavior.
- Do NOT change copy that's already in the rendered components. If the copy looks wrong, raise it to the user, do not unilaterally rewrite it.
- Do NOT skip the auth health check. The cost of shipping broken auth far exceeds the cost of one extra `clerk doctor` call.

## Output

Write to `.claude/artifacts/<run-name>-integration-summary.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

The file lists:
- Every file touched (path + what changed)
- Every external command run (CLI invocations, migrations, etc.) with truncated output
- Any open issues you couldn't resolve (e.g. "Clerk org switcher needs a paid plan; left as a TODO")

## After integrating

Tell the user the absolute path to `<run-name>-integration-summary.md` and the next step:

```
Wrote .claude/artifacts/<run-name>-integration-summary.md

Next: switch to Sonnet and invoke /frontend-mix-validate with the integration summary path.
```
