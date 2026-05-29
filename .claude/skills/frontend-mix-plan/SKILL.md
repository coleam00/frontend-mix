---
name: frontend-mix-plan
description: Plan a full-stack web application by producing a three-section spec (UI / Integration / Deployment) that downstream provider-specific sessions can execute. Use this skill in a Claude Code session running Opus when you want to plan a build that will be implemented by mixing providers - Gemini designing the UI from your copy, Opus wiring the integrations. Triggers on "plan a frontend", "plan the app for mixed-provider build", "frontend-mix plan", or when given a spec markdown to turn into an actionable plan.
argument-hint: <path-to-spec.md> [run-name]
---

# Frontend-Mix · Plan

You are the **planning** step of a manual mixed-provider build. Reasoning-heavy work. Take your time and get the structure right - the cost of a bad plan compounds through every downstream session.

## Arguments

`$ARGUMENTS` is one of:

- `<absolute-path-to-spec.md>` - the spec file you'll plan from (preferred form)
- `<absolute-path-to-spec.md> <run-name>` - same, plus an explicit run-name slug for the artifact filename
- A free-form description of the app, if no spec file exists

If no path is present and no inline description is given, ask the user for one. Do not invent requirements.

## STEP 0 - Resolve inputs (do this FIRST)

1. **Read the spec.** If `$ARGUMENTS` includes a `.md` path, use the Read tool to open it end-to-end. Read it twice - first pass pulls structure, second pulls intent.
2. **Pick a run-name slug.** Either use the explicit slug from `$ARGUMENTS` (the second arg), or derive one from the spec contents (kebab-case, ~3-5 words, e.g. `acme-saas-landing`, `archon-benchmarking-dashboard`). The run-name threads through every downstream skill via the artifact filenames, so make it short and descriptive.
3. **Examine the current repo** with Read / Glob / Grep so the plan fits what already exists (framework, package manager, existing components).

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

Tell the user the absolute path to `<run-name>-plan.md` and the next manual step. The run-name is the prefix of the file you just wrote - downstream skills read it back from that filename, so it threads through the whole chain.

```
Wrote .claude/artifacts/<run-name>-plan.md

Next: open Pi with Gemini 3.5 Flash and run the frontend-mix-design skill.
pi --provider openrouter --model "google/gemini-3.5-flash" \
   --skill ~/.pi/agent/skills/frontend-mix-design \
   "<absolute-path-to>/.claude/artifacts/<run-name>-plan.md"
```

## Reasoning tips

- Re-read the spec twice before writing. The first read pulls structure; the second pulls intent.
- If two sections start blurring (an integration concern leaking into UI copy, or vice versa), split it. Down-line sessions can only execute what's in their section.
- Show the work - when you pick a deploy target or an auth provider, write one short sentence on WHY in the plan. The integrate session needs that context.
- Use the canonical headers (`## SECTION A - UI Scope` etc) verbatim. Downstream skills extract sections by header match.
