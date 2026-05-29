---
name: frontend-mix-design
description: Design and scaffold a beautiful frontend from SECTION A of a planning document. Run this skill in a Pi session driving Gemini 3.5 Flash, because Gemini builds the most beautiful frontends right now. The skill only builds the UI surface - auth, API calls, and SDK integrations belong to the next session. Use when the user has a plan.md from the frontend-mix-plan skill and is ready to build the UI.
argument-hint: <path-to-run-name-plan.md>
---

# Frontend-Mix · Design

You are the **UI design** step of a manual mixed-provider build. This step is best handled by Gemini 3.5 Flash. **Only build the UI surface - auth, API calls, third-party SDKs, and deployment belong to later sessions. Do not touch them.**

## Arguments

The path to the plan markdown (produced by `frontend-mix-plan`) comes in via `$ARGUMENTS`. Example value of `$ARGUMENTS`:

```
.claude/artifacts/acme-saas-landing-plan.md
```

The filename prefix (everything before `-plan.md`) is the **run-name**. You'll use it to name your output file so the next skill in the chain can find it.

If `$ARGUMENTS` is empty, ask the user for the plan path. Do not proceed without it.

## STEP 0 - Read inputs and extract run-name (do this FIRST)

1. Use the Read tool to open the path in `$ARGUMENTS` end-to-end.
2. Extract everything under the `## SECTION A - UI Scope` header. Ignore SECTION B and SECTION C - they are not your scope.
3. **Extract the run-name** from the input filename: strip the directory, strip the `-plan.md` suffix. Example: `.claude/artifacts/acme-saas-landing-plan.md` → run-name = `acme-saas-landing`.

## How to build

1. Treat SECTION A's copy as canonical - do not invent or paraphrase headlines.
2. Scaffold pages and components in the framework already in the repo (Next.js App Router by default; honor whatever the plan says).
3. Use Tailwind + shadcn/ui. Lean into beautiful layout, generous spacing, subtle animation, strong accessible contrast. Mobile-first.
4. **Leave seams for the integration session.** Every place that needs auth state, every API call site, every protected route - leave a clearly named stub and a `// INTEGRATION: ...` comment describing what the integration session should fill in.
5. Do NOT install auth SDKs (no `clerk init`, no Supabase wiring, etc).
6. Do NOT call third-party APIs.
7. Do NOT create server routes - those are the integration session's job.

## Output

Write to `.claude/artifacts/<run-name>-ui-summary.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

The file lists:
- Every file created (path + one-line purpose)
- Every `// INTEGRATION:` stub left behind (file + line + what's expected)

## After scaffolding

Tell the user the absolute path to `<run-name>-ui-summary.md` and the next manual step:

```
Wrote .claude/artifacts/<run-name>-ui-summary.md

Next: open Claude Code with Opus and run the frontend-mix-integrate skill.
Pass BOTH paths - plan first, ui-summary second.

claude --skill ~/.claude/skills/frontend-mix-integrate \
       "<absolute-path>/.claude/artifacts/<run-name>-plan.md <absolute-path>/.claude/artifacts/<run-name>-ui-summary.md"
```

## Design tips

- The hero/landing page is where Gemini's edge shows. Spend disproportionate effort here.
- Subtle motion is not heavy motion. A 200ms fade on card hover beats a parallax scroll.
- One accent color, used sparingly. Two if the brand demands it.
- White space is not wasted space. Don't fill it.
