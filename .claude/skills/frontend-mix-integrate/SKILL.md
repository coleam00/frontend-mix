---
name: frontend-mix-integrate
description: Wire up auth, backend API, and third-party SDK integrations into a UI scaffold by executing SECTION B of a planning document. Run this skill in a Claude Code session driving Opus - integration is reasoning-heavy. The skill is plan-driven, not vendor-specific: it uses whatever auth/SDKs/APIs SECTION B says. Use when the design session has finished and `ui-summary.md` plus `plan.md` are ready.
argument-hint: <path-to-run-name-plan.md> <path-to-run-name-ui-summary.md>
---

# Frontend-Mix · Integrate

You are the **integration** step of a manual mixed-provider build. Opus does the judgment here.

## Arguments

`$ARGUMENTS` contains TWO absolute paths, space-separated:

1. Path to `<run-name>-plan.md` (from `frontend-mix-plan`)
2. Path to `<run-name>-ui-summary.md` (from `frontend-mix-design`)

Example:

```
.claude/artifacts/acme-saas-landing-plan.md .claude/artifacts/acme-saas-landing-ui-summary.md
```

If either path is missing, ask the user for it. Do not work from context summaries - the artifacts are the files on disk.

## STEP 0 - Read inputs and extract run-name (do this FIRST)

1. Use the Read tool to open the plan path end-to-end. Extract everything under `## SECTION B - Integration Scope`. Ignore SECTION A and SECTION C.
2. Use the Read tool to open the ui-summary path end-to-end. Every `// INTEGRATION:` stub it lists is one of your work items.
3. **Extract the run-name** from the plan filename (strip directory, strip `-plan.md`). You'll use it to name your output file.

## Your job

1. **Auth** - implement whatever auth provider SECTION B chose.
   - If Clerk: drive the `clerk` CLI directly (`clerk init`, `clerk link`, `clerk env pull`, `clerk config patch --dry-run` then apply, `clerk doctor --json` at the end). Once `clerk deploy` ships publicly, that becomes the final step. (See [[clerk-cli]] skill if available.)
   - If Supabase / NextAuth / Auth.js / something else: do that instead. The skill loaded above is a capability, not a mandate.
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

Tell the user the absolute path to `<run-name>-integration-summary.md` and the next manual step:

```
Wrote .claude/artifacts/<run-name>-integration-summary.md

Next: validate (Sonnet). If it flags failures, then fix-validation (Opus).
claude --skill ~/.claude/skills/frontend-mix-validate \
       "<absolute-path>/.claude/artifacts/<run-name>-integration-summary.md"
```
