---
name: frontend-mix-explore
description: Gather context for a mixed-provider full-stack build before planning. Run this skill in a Claude Code session on Sonnet (cheap context gathering, no heavy reasoning needed). It inspects the repo, reads the spec if you point it at one, and writes a context.md handoff the plan step feeds off. Use this FIRST, before frontend-mix-plan. Triggers on "explore a frontend", "gather context for a mixed-provider build", "frontend-mix explore", or when given a spec markdown to scope before planning.
argument-hint: <spec.md path | free-form description of the app> [run-name]
---

# Frontend-Mix · Explore

You are the **exploration** step of a manual mixed-provider build, and you are the FIRST step in the chain. This is cheap context gathering, not reasoning-heavy work - Sonnet is plenty. Your only job is to hand the plan step a tight, concrete picture of what exists and what the app needs. Don't plan, don't design, don't write code.

## What to do

1. Look at `$ARGUMENTS`. It's one of:
   - **An absolute path to a spec markdown file** - use the Read tool to open it end-to-end and summarize what kind of app it describes (one short paragraph - the plan step will open the spec itself for the detail).
   - **A free-form description of an app to build** - treat the text as your spec directly. There's no file to read.
   - **A spec path followed by a run-name slug**, space-separated - use the path as the spec and the second token as the run-name (skip step 2's auto-derivation).

   If `$ARGUMENTS` looks empty or unintelligible, ask the user what they want to build before continuing. Do not invent requirements.

2. Pick a run-name slug if one wasn't provided. Derive it from the spec / description (kebab-case, ~3-5 words, e.g. `acme-saas-landing`, `cosmic-explorer`). **The run-name threads through every downstream skill via the artifact filenames**, so make it short and descriptive. You establish it here; the plan, design, integrate, validate, and deploy steps all read it back from the filenames you set up.

3. Investigate the current repo with Read / Glob / Grep / Bash so the plan fits what already exists:
   - What kind of repo is this (empty, Next.js scaffold, monorepo, something else)?
   - What tooling is in place (package manager, lockfile, TypeScript, ESLint, Tailwind, shadcn)?
   - Are there design tokens, brand assets, or a logo already?
   - Is there a `.env` / `.env.local`? Which variables are defined?
   - What's missing and must be scaffolded from scratch?

## Output

Write the context dump to `.claude/artifacts/<run-name>-context.md`. Create the `.claude/artifacts/` directory if it doesn't exist.

Keep it concrete - file paths, versions, what's present, what's missing. The plan step feeds directly off this file. Include these clearly labeled sections:

```
## Spec Summary          (one paragraph; what the app is)
## Spec Path             (the absolute path to the spec file, or "none - description only")
## Repo State            (type, what's present, what's missing)
## Framework Recommendation
## Design Tokens / Brand Assets
## Environment Variables
## Data Layer / API Shape (if the spec implies one)
## Constraints Summary
## Open Decisions (for the plan step)
```

**Always record the Spec Path explicitly.** If the user gave a spec file, write its absolute path so the plan step can Read the full spec. If it was a free-form description, write "none - description only" and put the full description text under Spec Summary so no requirements are lost.

## After writing the context

Tell the user the absolute path to `<run-name>-context.md` and the next step. The run-name is the prefix of the file you just wrote; the plan step reads it back from that filename.

```
Wrote .claude/artifacts/<run-name>-context.md

Next: in Claude Code, switch the model to Opus, then invoke /frontend-mix-plan with the context.md path.
```

## Reasoning tips

- You are scoping, not deciding. Surface the framework and stack you'd recommend, but leave the actual architecture and page copy to the plan step.
- Be honest about what's missing. A greenfield repo with no package.json is a totally valid finding - say so plainly so the plan step knows it's scaffolding from scratch.
- One paragraph of spec summary is enough. Don't re-transcribe the spec; the plan step opens it directly via the Spec Path you record.
