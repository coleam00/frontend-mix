---
name: frontend-mix-plan
description: Plan a full-stack web application by producing a three-section spec (UI / Integration / Deployment) that downstream provider-specific sessions can execute. Run this skill in a Claude Code session on Opus, AFTER frontend-mix-explore has produced a context.md. Pure planning - it reads the context handoff (and the spec it points to) and turns it into an actionable plan. Triggers on "plan a frontend", "plan the app for mixed-provider build", "frontend-mix plan", or when given a context.md to turn into a plan.
argument-hint: <context.md path>
---

# Frontend-Mix · Plan

You are the **planning** step of a manual mixed-provider build. Reasoning-heavy work. Take your time and get the structure right - the cost of a bad plan compounds through every downstream session.

The exploration is already done. The `frontend-mix-explore` step ran first and wrote a `context.md` for you. **Do not re-explore the repo from scratch** - read the context handoff and the spec it points to, then plan.

## What to do

1. Look at `$ARGUMENTS`. It is the path to the `<run-name>-context.md` that the explore step wrote. Use the Read tool to open it end-to-end.

   If `$ARGUMENTS` is empty or the path doesn't resolve, ask the user for the context.md path (or to run `/frontend-mix-explore` first). Do not invent requirements.

2. The filename in `$ARGUMENTS` carries your run-name. Strip the directory and the `-context.md` suffix. Example: `.claude/artifacts/acme-saas-landing-context.md` → run-name = `acme-saas-landing`. You'll reuse it to name your output file so the next skill in the chain can find it.

3. Read the spec the context points to. The context.md has a `## Spec Path` section:
   - If it names a spec file, use the Read tool to open that spec end-to-end. Read it twice (first pass for structure, second pass for intent).
   - If it says "none - description only", the full requirements live in the context's `## Spec Summary` - treat that as your spec.

4. Lean on the context's `## Repo State`, `## Framework Recommendation`, and `## Open Decisions` so the plan fits what already exists (framework, package manager, existing components). Only spot-check the repo with Read / Glob if the context left something genuinely ambiguous - don't redo the full exploration.

## Output

Write the plan to `.claude/artifacts/<run-name>-plan.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

Use **exactly these three section headers** so downstream skills can grep for them:

```
## SECTION A - UI Scope
## SECTION B - Integration Scope
## SECTION C - Deployment Plan
```

### SECTION A - UI Scope (the design session reads this)

For every page in the app: route, purpose, what it contains.
For every component: name, what it shows, where it appears.

**Write the actual user-facing copy.** Every headline, sub-headline, button label, card title, empty-state message, error message. The design session will build the UI directly from this text; if the copy is vague, the UI is vague.

### SECTION B - Integration Scope (the integrate session reads this)

List every non-UI concern. Be specific about which tools/services and why:
- Authentication: which provider, which flows (sign-in, sign-up, protected routes, org switcher), and the exact CLI / SDK calls
- Backend API: every endpoint - method, path, input, output, auth check
- External APIs / SDKs and where they're called
- Data model: tables, fields, indexes; which database
- Background jobs / async workflows, if any

### SECTION C - Deployment Plan (the deploy session reads this)

State the deployment target explicitly. Examples:
- "Local only. No deployment needed this run"
- "Vercel via `vercel deploy --prod`"
- "Fly.io via `flyctl deploy`"
- "Auth provider's deploy command (e.g. `clerk deploy` once it ships)"

Include pre-deploy steps (env var promotion, migrations) and success criteria.

## After writing the plan

Tell the user the absolute path to `<run-name>-plan.md` and the next step. The run-name is the prefix of the file you just wrote; downstream skills read it back from that filename.

```
Wrote .claude/artifacts/<run-name>-plan.md

Next: start Pi pointed at OpenRouter + Gemini 3.5 Flash, then invoke /skill:frontend-mix-design with the plan path.
```

## Reasoning tips

- Re-read the spec twice before writing. The first read pulls structure; the second pulls intent.
- If two sections start blurring (an integration concern leaking into UI copy, or vice versa), split it. Downstream sessions can only execute what's in their section.
- When you pick a deploy target or an auth provider, write one short sentence on WHY in the plan. The integrate session needs that context.
- Use the canonical headers verbatim. Downstream skills extract sections by exact header match.
