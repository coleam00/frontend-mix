# frontend-mix

A mixed-provider workflow for building beautiful, full-stack web apps by routing each phase of the build to the model that earns its tokens at that step.

- **Opus 4.8** plans the architecture, page copy, integrations, and deploy target.
- **Gemini 3.5 Flash** (via Pi + OpenRouter) designs the UI from the plan's copy.
- **Opus 4.8** wires the integrations (auth, APIs, SDKs) into the UI scaffold.
- **Sonnet** validates the build (install, typecheck, lint, build, tests).
- **Opus 4.8** addresses anything validation flagged + runs the deploy.

The handoff between sessions is a single markdown document on disk. Each downstream session reads the prior artifact and writes the next. The file is the contract.

## What's in this repo

- `.claude/skills/frontend-mix-*` (six skills, one per phase) for running the chain by hand.
- `.claude/skills/archon` for asking your coding agent to drive the Archon workflow.
- `.archon/workflows/archon-build-frontend-mixed.yaml` for the automated 7-node DAG.

## Installing

Drop the skills into your global skills directory.

For Claude Code (all six frontend-mix skills + the archon skill):

```bash
cp -r .claude/skills/* ~/.claude/skills/
```

For Pi (just the design skill, which is the one that runs there):

```bash
cp -r .claude/skills/frontend-mix-design ~/.pi/agent/skills/
```

Claude Code makes skills user-invocable as slash commands by default. Pi exposes skills as `/skill:<name>`. Nothing to enable.

## How to run the chain by hand

Each step is a slash command in the right tool. The output of each step is a markdown file in `.claude/artifacts/` with the run-name as the filename prefix. The next step just needs that path.

### 1. Plan - Claude Code with Opus 4.8

The plan step accepts **either** an absolute path to a spec markdown **or** a free-form description of what you want to build. Both work; whichever is more convenient.

With a spec file:

```
/frontend-mix-plan C:/path/to/spec.md
```

With a plain-text description:

```
/frontend-mix-plan a landing page + sign-in + dashboard for an AI image-gen SaaS, Clerk for auth
```

The skill picks a run-name slug from your input (e.g. `acme-saas-landing`) and writes:

```
.claude/artifacts/acme-saas-landing-plan.md
```

Note that path. The next step needs it.

### 2. Design - Pi with Gemini 3.5 Flash

Start Pi pointed at OpenRouter:

```bash
pi --provider openrouter --model "google/gemini-3.5-flash"
```

Then invoke the skill from inside the Pi prompt:

```
/skill:frontend-mix-design C:/.../acme-saas-landing-plan.md
```

Output:

```
.claude/artifacts/acme-saas-landing-ui-summary.md
```

### 3. Integrate - Claude Code with Opus 4.8

Back in Claude Code with Opus. Pass BOTH the plan path AND the ui-summary path:

```
/frontend-mix-integrate C:/.../acme-saas-landing-plan.md C:/.../acme-saas-landing-ui-summary.md
```

Output:

```
.claude/artifacts/acme-saas-landing-integration-summary.md
```

### 4. Validate - Claude Code with Sonnet

Switch to Sonnet:

```
/frontend-mix-validate C:/.../acme-saas-landing-integration-summary.md
```

Output is one of two files. If the build is clean:

```
.claude/artifacts/acme-saas-landing-validation-summary.md
```

If validation flagged failures after two repair attempts:

```
.claude/artifacts/acme-saas-landing-validation-issues.md
```

### 5. Fix validation - Claude Code with Opus 4.8 (only if step 4 wrote `validation-issues.md`)

```
/frontend-mix-fix-validation C:/.../acme-saas-landing-validation-issues.md C:/.../acme-saas-landing-plan.md
```

Output:

```
.claude/artifacts/acme-saas-landing-resolution-summary.md
```

The summary ends with `READY TO DEPLOY` or `NOT READY: <reason>`. Do not advance to step 6 if it's NOT READY.

### 6. Deploy - Claude Code with Opus 4.8

Pass the plan path AND either the validation summary (clean run) OR the resolution summary (after fix-validation):

```
/frontend-mix-deploy C:/.../acme-saas-landing-plan.md C:/.../acme-saas-landing-resolution-summary.md
```

Output:

```
.claude/artifacts/acme-saas-landing-deploy-summary.md
```

App is live.

## How to run the chain automatically (via Archon)

The Archon workflow at `.archon/workflows/archon-build-frontend-mixed.yaml` runs all 7 nodes end-to-end inside an isolated git worktree. You don't invoke `archon` directly from your shell. You ask your coding agent to do it via the **Archon skill**, which is what knows the right invocation and worktree setup.

### Install the Archon skill

This repo includes a copy at `.claude/skills/archon`. The latest is also tracked alongside the engine itself at the [Archon repo](https://github.com/coleam00/Archon). Either way, drop it into `~/.claude/skills/`.

### Install the workflow into your target project

Copy the workflow YAML into the project the build should land in:

```bash
mkdir -p <target-repo>/.archon/workflows
cp .archon/workflows/archon-build-frontend-mixed.yaml <target-repo>/.archon/workflows/
```

### Dispatch from your coding agent

Open Claude Code in the target repo and ask the Archon skill to run the workflow:

```
Use the archon skill to run the archon-build-frontend-mixed workflow.
Build the AI image-gen SaaS landing page + sign-in + dashboard.
Full spec at C:/path/to/spec.md.
```

The skill handles the dispatch, the worktree, and the env vars. A worktree spins up, the 7 nodes execute in sequence (with the same artifact handoff chain as the manual flow), and the final node commits the result.

## Requirements (for the Archon workflow only)

The automated workflow needs:

- **Archon** built from source with the pi-adapter patch applied (see [Archon](https://github.com/coleam00/Archon))
- `@mariozechner/pi-ai` and `@mariozechner/pi-coding-agent` bumped to `^0.73.1`
- `~/.pi/agent/models.json` registering `google/gemini-3.5-flash` as a custom OpenRouter model
- An `OPENROUTER_API_KEY` in `~/.pi/agent/auth.json`

The manual chain only needs Claude Code + Pi + an OpenRouter key. No Archon required.
