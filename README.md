# frontend-mix

A mixed-provider workflow for building beautiful, full-stack web apps by routing each phase of the build to the model that earns its tokens at that step.

> [TIP]
> **Don't want to set this up by hand? Ask your coding agent to do it.** Point Claude Code (or any agent) at this repo and say *"set up frontend-mix for me."* It can install the skills, copy the design skill into Pi, and walk you through the OpenRouter + Pi config below. The whole thing is just files and CLI commands an agent can run.

- **Opus 4.8** plans the architecture, page copy, integrations, and deploy target.
- **Gemini 3.5 Flash** (via Pi + OpenRouter) designs the UI from the plan's copy.
- **Opus 4.8** wires the integrations (auth, APIs, SDKs) into the UI scaffold.
- **Sonnet** validates the build (install, typecheck, lint, build, tests).
- **Opus 4.8** addresses anything validation flagged.
- **Sonnet** smoke-tests the *running* app with agent-browser - the step that catches "compiles green, renders broken."
- **Opus 4.8** runs the deploy.

The handoff between sessions is a single markdown document on disk. Each downstream session reads the prior artifact and writes the next. The file is the contract.

## What's in this repo

- `.claude/skills/frontend-mix-*` (seven skills, one per phase) for running the chain by hand.
- `.claude/skills/archon` for asking your coding agent to drive the Archon workflow.
- `.archon/workflows/archon-build-frontend-mixed.yaml` for the automated 8-node DAG.
- `.claude/skills/frontend-mix-example/` a complete worked example (the Cosmos build) with the real artifacts each step produced.

## Installing

Drop the skills into your global skills directory.

For Claude Code (all seven frontend-mix skills + the archon skill):

```bash
cp -r .claude/skills/* ~/.claude/skills/
```

For Pi (just the design skill, which is the one that runs there):

```bash
cp -r .claude/skills/frontend-mix-design ~/.pi/agent/skills/
```

<details>
<summary><b>Set up Gemini 3.5 Flash via OpenRouter + Pi</b></summary>

The design step runs in **Pi** (`@mariozechner/pi-coding-agent`) pointed at OpenRouter. One-time setup:

**1. Get an OpenRouter key.** Create one at [openrouter.ai/keys](https://openrouter.ai/keys) and add a few dollars of credit (Gemini 3.5 Flash runs about $1.50 / $9 per M tokens in/out).

**2. Give Pi the key.** Add an `openrouter` block to `~/.pi/agent/auth.json`:

```json
{
  "openrouter": { "type": "api_key", "key": "sk-or-..." }
}
```

**3. Register the model.** Newer standalone Pi binaries already list `google/gemini-3.5-flash`; if yours doesn't (the npm `pi-ai` catalog lags), add it to `~/.pi/agent/models.json`. Note `models` must be an **array**, not an object:

```json
{
  "providers": {
    "openrouter": {
      "baseUrl": "https://openrouter.ai/api/v1",
      "api": "openai-completions",
      "models": [
        {
          "id": "google/gemini-3.5-flash",
          "name": "Google Gemini 3.5 Flash",
          "api": "openai-completions",
          "provider": "openrouter",
          "baseUrl": "https://openrouter.ai/api/v1",
          "input": ["text", "image"],
          "contextWindow": 1000000,
          "maxTokens": 65536,
          "reasoning": true,
          "cost": { "input": 1.5, "output": 9, "cacheRead": 0, "cacheWrite": 0 }
        }
      ]
    }
  }
}
```

**4. Verify.** This should print `OK`:

```bash
pi --provider openrouter --model "google/gemini-3.5-flash" -p "Respond OK." --no-tools --no-skills --no-extensions --no-prompt-templates
```

Once that works, the design step (`/skill:frontend-mix-design`) runs against Gemini 3.5 Flash. Driving the **automated Archon workflow** with 3.5 Flash needs a couple of extra patches; see Requirements at the bottom.

</details>

Claude Code makes skills user-invocable as slash commands by default. Pi exposes skills as `/skill:<name>`. Nothing to enable.

## How to run the chain by hand

Each step is a slash command in the right tool. The output of each step is a markdown file in `.claude/artifacts/` with the run-name as the filename prefix. The next step just needs that path.

### Quick chain

1. `/frontend-mix-plan <spec.md path or "description">` (Claude Code, Opus)
2. `/skill:frontend-mix-design <plan path>` (Pi, Gemini 3.5 Flash)
3. `/frontend-mix-integrate <plan path> <ui-summary path>` (Claude Code, Opus)
4. `/frontend-mix-validate <integration-summary path>` (Claude Code, Sonnet)
5. `/frontend-mix-fix-validation <validation-issues path> <plan path>` (Claude Code, Opus, only if step 4 flagged failures)
6. `/frontend-mix-smoke <integration-summary path> <validation-summary or resolution-summary path>` (Claude Code, Sonnet)
7. `/frontend-mix-deploy <plan path> <validation-summary or resolution-summary path>` (Claude Code, Opus)

<details>
<summary><b>Detailed walkthrough</b> (click to expand)</summary>

### 1. Plan - Claude Code with Opus 4.8

The plan step accepts **either** an absolute path to a spec markdown **or** a free-form description of what you want to build. Both work; whichever is more convenient.

With a spec file:

```
/frontend-mix-plan C:/path/to/spec.md
```

With a plain-text description:

```
/frontend-mix-plan "A landing page + sign-in + dashboard for an AI image-gen SaaS, Clerk for auth"
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

The summary ends with `READY TO DEPLOY` or `NOT READY: <reason>`. Do not advance to smoke or deploy if it's NOT READY.

### 6. Smoke - Claude Code with Sonnet

Drive the *running* app and confirm it actually works - the bugs static checks can't see. Pass the integration-summary path AND the validation summary (clean run) OR the resolution summary (after fix-validation):

```
/frontend-mix-smoke C:/.../acme-saas-landing-integration-summary.md C:/.../acme-saas-landing-validation-summary.md
```

It starts the app on a free port, drives `agent-browser` through every route, exercises the primary interaction, checks a bogus route returns a real 404, and screenshots each page. A page that loads 200 but renders blank/unstyled is a DEFECT. Output:

```
.claude/artifacts/acme-saas-landing-smoke-summary.md
```

The summary ends with `SMOKE: PASS` or `SMOKE: DEFECTS`. Fix any defects (Opus) before deploying.

> Requires `agent-browser` (`npm install -g agent-browser && agent-browser install`). Without it, the step falls back to curl-only route/API checks.

### 7. Deploy - Claude Code with Opus 4.8

Pass the plan path AND either the validation summary (clean run) OR the resolution summary (after fix-validation):

```
/frontend-mix-deploy C:/.../acme-saas-landing-plan.md C:/.../acme-saas-landing-resolution-summary.md
```

Output:

```
.claude/artifacts/acme-saas-landing-deploy-summary.md
```

App is live.

</details>

## How to run the chain automatically (via Archon)

The Archon workflow at `.archon/workflows/archon-build-frontend-mixed.yaml` runs all 8 nodes end-to-end inside an isolated git worktree. You don't invoke `archon` directly from your shell. You ask your coding agent to do it via the **Archon skill**, which is what knows the right invocation and worktree setup.

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

The skill handles the dispatch, the worktree, and the env vars. A worktree spins up, the 8 nodes execute in sequence (with the same artifact handoff chain as the manual flow), and the final node commits the result.

## Requirements (for the Archon workflow only)

The automated workflow needs:

- **Archon** built from source with the pi-adapter patch applied (see [Archon](https://github.com/coleam00/Archon))
- `@mariozechner/pi-ai` and `@mariozechner/pi-coding-agent` bumped to `^0.73.1`
- `~/.pi/agent/models.json` registering `google/gemini-3.5-flash` as a custom OpenRouter model
- An `OPENROUTER_API_KEY` in `~/.pi/agent/auth.json`
- `agent-browser` installed globally (`npm i -g agent-browser && agent-browser install`) for the smoke node

The manual chain only needs Claude Code + Pi + an OpenRouter key (plus `agent-browser` for the smoke step). No Archon required.
