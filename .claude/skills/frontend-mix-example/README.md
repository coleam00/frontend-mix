# Frontend-Mix Example: the Cosmos Build

This is a **worked example** of the mixed-provider frontend workflow. The `frontend-mix-*` skills
next to this folder were run end-to-end to build a real app, and the actual documents each step
produced are captured in [`artifacts/`](./artifacts) so you can read exactly what gets handed from
one model to the next.

The app built here is **Cosmos**, a cinematic, no-login deep-sky planetarium: 12 real celestial
objects, a SQLite-backed API, a deterministic "object of the day", and an anonymous "chills"
reaction counter. The whole thing is rendered with CSS/SVG/gradients, no photos.

## The core idea

You don't pick one model. You route each phase to the model that's best at it, and the **handoff
between sessions is a markdown file on disk, not a shared chat history.** Two models that have
never seen each other's context hand off cleanly because they both read the same artifact.

```
spec.md                                          ← the input brief (Section A/B/C intent)
  │  frontend-mix-explore     (Claude Code, Sonnet)
  ▼
artifacts/context.md                             ← repo state · framework rec · spec path · open decisions
  │  frontend-mix-plan        (Claude Code, Opus)    reads context.md + the spec it points to
  ▼
artifacts/plan.md                                ← SECTION A content brief · B integration scope · C deploy
  │  frontend-mix-design      (Pi, Gemini 3.5 Flash)   reads SECTION A only
  ▼
artifacts/ui-summary.md                          ← Gemini designs the UI, leaves // INTEGRATION: stubs
  │  frontend-mix-integrate   (Claude Code, Opus)      reads SECTION B + ui-summary
  ▼
artifacts/integration-summary.md                 ← Opus wires the API/DB, strips the stubs
  │  frontend-mix-validate    (Claude Code, Sonnet)
  ▼
artifacts/validation-summary.md                  ← install / typecheck / lint / build
  │  frontend-mix-fix-validation (Claude Code, Opus)   no-ops if validate was clean
  ▼
artifacts/resolution-summary.md
  │  frontend-mix-smoke       (Claude Code + agent-browser)
  ▼
artifacts/smoke-summary.md  (+ smoke-shots/)     ← drives the running app: routes, the chills click, real 404s
  │  frontend-mix-deploy      (Claude Code, Opus)      follows SECTION C
  ▼
artifacts/deploy-summary.md                      ← local-only for this build
```

## Skill → artifact map

| Step | Skill | Model | Artifact |
|------|-------|-------|----------|
| 0 | `frontend-mix-explore` | Sonnet | `artifacts/context.md` |
| 1 | `frontend-mix-plan` | Opus | `artifacts/plan.md` |
| 2 | `frontend-mix-design` | Gemini 3.5 Flash (via Pi) | `artifacts/ui-summary.md` |
| 3 | `frontend-mix-integrate` | Opus | `artifacts/integration-summary.md` |
| 4 | `frontend-mix-validate` | Sonnet | `artifacts/validation-summary.md` |
| 5 | `frontend-mix-fix-validation` | Opus | `artifacts/resolution-summary.md` |
| 6 | `frontend-mix-smoke` | Sonnet + agent-browser | `artifacts/smoke-summary.md` + `artifacts/smoke-shots/` |
| 7 | `frontend-mix-deploy` | Opus | `artifacts/deploy-summary.md` |

> The smoke step (6) catches the bug below. It runs the `frontend-mix-smoke` skill next to this
> folder; the example's `smoke-summary.md` came from running it.

## The one bug this pattern reliably produces

Gemini put components under `lib/`, but the Tailwind `content` globs only scanned `app`/`pages`/
`components`, so those classes generated **no CSS**. The page compiled, typechecked, linted, and
built clean, and loaded HTTP 200, but rendered as a flat unstyled list. Static checks all pass;
only the **smoke** step (which looks at the actual rendered page) catches it. That's the whole
argument for the smoke step: "compiles green, renders broken."

## Notes

- `spec.md` is the input brief that started the run.
- `artifacts/` mirrors exactly where the skills write during a live run (`./artifacts/`).
- The Archon workflow that automates this chain as a DAG lives in `.archon/`.
- Provenance: these are real artifacts from the Cosmos build runs. `ui-summary.md` is the genuine
  design-step output (it lists the `// INTEGRATION:` stubs); the rest are from the complete
  end-to-end run that also produced the validate/smoke/deploy docs.
