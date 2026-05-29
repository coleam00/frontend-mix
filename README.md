# frontend-mix

A mixed-provider workflow for building beautiful, full-stack web apps by routing each phase of the build to the model that earns its tokens at that step.

- **Opus 4.8** plans the architecture, page copy, integrations, and deploy target.
- **Gemini 3.5 Flash** (via Pi + OpenRouter) designs the UI from the plan's copy.
- **Opus 4.8** wires the integrations (auth, APIs, SDKs) into the UI scaffold.
- **Sonnet** validates the build (install, typecheck, lint, build, tests).
- **Opus 4.8** addresses anything validation flagged + runs the deploy.

The handoff between sessions is a single markdown document on disk. Each downstream session reads the prior artifact and writes the next. The file is the contract.

## The skills (manual chain)

Six Claude Code skills under `.claude/skills/frontend-mix-*` make the chain runnable by hand. Each skill is one fresh session in the right tool.

| Skill | Run in | Model | Input | Output |
|-------|--------|-------|-------|--------|
| `frontend-mix-plan` | Claude Code | Opus 4.8 | a spec.md | `<run>-plan.md` |
| `frontend-mix-design` | Pi | Gemini 3.5 Flash | `<run>-plan.md` | `<run>-ui-summary.md` |
| `frontend-mix-integrate` | Claude Code | Opus 4.8 | plan + ui-summary | `<run>-integration-summary.md` |
| `frontend-mix-validate` | Claude Code | Sonnet | integration-summary | `<run>-validation-summary.md` OR `<run>-validation-issues.md` |
| `frontend-mix-fix-validation` | Claude Code | Opus 4.8 | validation-issues + plan | `<run>-resolution-summary.md` |
| `frontend-mix-deploy` | Claude Code | Opus 4.8 | plan + validation/resolution | `<run>-deploy-summary.md` |

The first skill (`plan`) picks a run-name slug from the spec. Every downstream skill extracts the same slug from the input filename and writes its output with the same prefix to `.claude/artifacts/`. That's how context threads through a chain of independent sessions without a shared chat history.

## Installing

Copy `.claude/skills/frontend-mix-*` into `~/.claude/skills/` (or your project's `.claude/skills/`). The Gemini design step also runs in Pi, so symlink or copy `frontend-mix-design` into `~/.pi/agent/skills/` as well.

## The Archon workflow (automated chain)

`.archon/workflows/archon-build-frontend-mixed.yaml` is the same 7-node chain wired up for [Archon](https://github.com/coleam00/Archon) so you can dispatch it once, walk away, and come back to a deployed app.

```bash
IS_SANDBOX=1 archon workflow run archon-build-frontend-mixed \
  --branch feat/<your-branch> \
  "Build <your app description>. Full spec at <path-to-spec.md>"
```

The workflow needs Archon source with the pi-adapter patch, pi-ai bumped to ^0.73.1, and `~/.pi/agent/models.json` registering `google/gemini-3.5-flash` as a custom OpenRouter model. Walk through the setup recipe in your local Archon clone before dispatching.
