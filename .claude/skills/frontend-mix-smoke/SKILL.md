---
name: frontend-mix-smoke
description: Runtime smoke-test a freshly validated mixed-provider build by driving the RUNNING app with agent-browser - load every route, exercise the primary interaction, check status codes and console errors. Run this skill in a Claude Code session driving Sonnet. Catches the class of bug static checks (typecheck/lint/build) miss: broken routes, dead buttons, soft-404s, runtime errors, and the page that loads 200 but renders unstyled. Use after frontend-mix-validate (or fix-validation) confirms the build is clean, and before frontend-mix-deploy.
argument-hint: <integration-summary.md path> <validation-summary.md OR resolution-summary.md path>
---

# Frontend-Mix · Smoke

You are the **runtime smoke** step of a manual mixed-provider build. Sonnet handles this. Static checks already passed; your job is to confirm the **running** app actually works - the bugs typecheck/lint/build can't see.

## What to do

1. Read `$1` (the integration-summary markdown) end-to-end with the Read tool. It tells you the stack, package manager, routes, and API endpoints to exercise.
2. Read `$2` (the validation-summary or resolution-summary). If it ends "NOT READY: <reason>", do not smoke a broken build - write a smoke-summary noting it was skipped, and stop.
3. The filename in `$1` carries your run-name. Strip the directory and the `-integration-summary.md` suffix. You'll use it to name your output file.
4. If `$1` or `$2` is empty or doesn't resolve, ask the user for the missing path.

The `agent-browser` skill drives a real browser. If it isn't installed on this machine, fall back to curl-only checks (routes + status codes + API) and say so in the summary.

## Start the app

1. Pick a free port (try 3100; increment if busy). Start the app in the **background** on that port (`npm run dev -- --port <port>` or the dev command from the integration summary), redirecting output to a log file. Never foreground it - it blocks.
2. Poll until it responds (curl the root, retry up to ~60s). If it never comes up, capture the log tail as a DEFECT and stop.

## Drive the app

Exercise the surfaces the plan and integration describe. At minimum:

- **Open each main route.** Wait for network idle PLUS a few seconds - dev-mode hydration can lag before interactions work. Confirm it renders **styled** and the console is empty - a 200 is NOT enough. A blank, unstyled, or collapsed page (e.g. from a broken CSS content glob) is a DEFECT even though it loads. Screenshot each page and actually look at it.
- **Exercise the primary interaction** (the main button / form / action). Confirm the UI updates and, if it hits an API, the server state actually changes (verify with curl).
- **Hit one clearly-invalid route/resource** and confirm it fails correctly (a real 404, not a soft 200).
- **curl the key API endpoints** and confirm sane status codes.

Save a few screenshots into `.claude/artifacts/<run-name>-smoke-shots/` as evidence.

**Do not cry wolf.** Before recording a defect, rule out test confounds - hydration timing, your own rate-limiting, a wrong selector. A confound reported as a bug wastes the next session's time.

## Tear down

Kill the background server process and confirm the port is free.

## Output

Write `.claude/artifacts/<run-name>-smoke-summary.md`. Create the `.claude/artifacts/` directory if it doesn't exist. Include:

- Each route + status code
- Each interaction tested + result
- Any console errors
- A final verdict line: **`SMOKE: PASS`**, or **`SMOKE: DEFECTS`** followed by a list of each defect (what you did, expected vs actual, the file most likely responsible).

Do NOT silently pass real runtime bugs - list them under DEFECTS so a human can fix them.

## After smoking

Confirm `<run-name>-smoke-summary.md` exists on disk, then tell the user the absolute path and the next step:

```
If verdict is SMOKE: PASS:
  Next: invoke /frontend-mix-deploy with the plan path and the validation-summary or resolution-summary path.

If verdict is SMOKE: DEFECTS:
  Surface the defects to the user. Fix them (switch to Opus) before deploying - do not ship a build with known runtime defects.
```
