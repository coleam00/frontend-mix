---
name: frontend-mix-plan
description: Plan a full-stack web application by producing a three-section spec (UI / Integration / Deployment) that downstream provider-specific sessions can execute. Use this skill in a Claude Code session running Opus when you want to plan a build that will be implemented by mixing providers - Gemini designing the UI from your copy, Opus wiring the integrations. Triggers on "plan a frontend", "plan the app for mixed-provider build", "frontend-mix plan", or when given a spec markdown to turn into an actionable plan.
argument-hint: <spec.md path | free-form description of the app> [run-name]
---

# Frontend-Mix · Plan

You are the **planning** step of a manual mixed-provider build. Reasoning-heavy work. Take your time and get the structure right - the cost of a bad plan compounds through every downstream session.

## What to do

1. Look at `$ARGUMENTS`. It's one of:
   - **An absolute path to a spec markdown file** - use the Read tool to open it end-to-end. Read it twice (first pass for structure, second pass for intent).
   - **A free-form description of an app to build** - treat it as your spec directly. No file to read.
   - **A spec path followed by a run-name slug**, space-separated - use the path as the spec and the second token as the run-name (skip step 2's auto-derivation).

   If `$ARGUMENTS` looks empty or unintelligible, ask the user what they want to build before continuing. Do not invent requirements.

2. Pick a run-name slug if one wasn't provided. Derive it from the spec contents (kebab-case, ~3-5 words, e.g. `acme-saas-landing`, `archon-benchmarking-dashboard`). The run-name threads through every downstream skill via the artifact filenames, so make it short and descriptive.

3. Examine the current repo with Read / Glob / Grep so the plan fits what already exists (framework, package manager, existing components).

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
