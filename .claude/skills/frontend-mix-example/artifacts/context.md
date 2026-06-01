# Context Dump — Cosmic Explorer

## Spec Summary

Cosmic Explorer ("Cosmos") is a cinematic, no-auth interactive web app for exploring 12 canonical cosmic objects (planets, moons, stars, nebulae, galaxies, one black hole, one exoplanet system). Core capabilities: landing/featured object, full catalog with category filter + name search, per-object detail pages, deterministic "object of the day" (by date), and a server-side anonymous "chills" reaction counter backed by SQLite. Visual style: dark-sky, rendered via CSS/SVG/gradients — no real photo assets. Local-only build (`npm run dev`). No auth, no accounts.

---

## Repo State

**Type:** Bare Archon workspace scaffold — **greenfield build, no application code exists.**

### Present in worktree
```
C:/Users/colem/.archon/workspaces/colem/dynamous-engine/worktrees/archon/task-feat-cosmic-explorer-smoke2/
├── CLAUDE.md              # Dynamous second-brain documentation (not app-related)
├── docker-compose.yml     # Workspace-level compose (not the app)
├── master.env.example     # All env vars for the dynamous-engine workspace
├── README.md              # Dynamous second-brain README
├── setup_workspace.py     # Workspace provisioning script
└── templates/             # tone-of-voice.md + memory/ subfolder
```

**No package.json, no lockfile, no framework, no src/, no app/ directory.**

### Missing (must be scaffolded from scratch)
- Application directory with framework scaffold
- package.json / package-lock.json or pnpm-lock.yaml
- tsconfig.json
- Tailwind / shadcn config (if adopted)
- .env / .env.local for app
- SQLite database file + seed script
- Lint/format config (ESLint, Prettier, Biome)

---

## Framework Recommendation

**Next.js 14+ (App Router) + TypeScript** — matches `npm run dev` deployment target, supports both server components and API routes in one process, ships SQLite via `better-sqlite3` in route handlers. Alternative: Vite + React + Express, but Next.js avoids a separate backend process.

**Package manager:** npm (default, matches spec's `npm run dev` instruction). pnpm is fine too.

**Database:** SQLite via `better-sqlite3` — file-based, no daemon, no env vars needed. Seed the 12 objects from a TypeScript/JS module at startup. Reaction counts in a separate `reactions` table keyed by object slug.

---

## Design Tokens / Brand Assets

**None defined yet.** The spec prescribes:
- Dark-sky palette (deep space blacks, navy, midnight blue)
- Strong typographic contrast (likely white/light on near-black)
- CSS/SVG/gradient visuals — no photos; procedural starfields encouraged
- Loose brand constraint: deep-space palette only; wordmark/name/tagline open for design

No logo files, no font files, no Tailwind preset found in the worktree. The plan session should define a color palette and choose a display typeface (e.g., Space Grotesk, Orbitron, or Inter with weight contrast).

---

## Environment Variables

**App-level .env:** None defined. The app needs none for MVP (SQLite is file-local, no external APIs). If a dev `.env.local` is created, only one variable is needed:
```
DATABASE_PATH=./cosmic.db    # optional override; default to ./cosmic.db
```

**Workspace master.env.example** lists dynamous-engine keys (ANTHROPIC_API_KEY, OPENAI_API_KEY, Slack, Asana, etc.) — irrelevant to this app.

---

## Data Layer

- **Catalog:** 12 objects with fixed canonical facts (see spec §"Content & data"). Can be seeded from a TypeScript constant module (`data/catalog.ts`) — no LLM calls, no external API.
- **Reactions table:** `CREATE TABLE reactions (slug TEXT PRIMARY KEY, count INTEGER NOT NULL DEFAULT 0)` — initialized on first read.
- **Object of the day:** Deterministic — `dayOfYear % 12` index into sorted catalog array. No DB lookup needed.
- **Rate limiting:** Spec says "casual" — session cookie or IP check is fine; no strict quota needed.

---

## API Shape (from spec)

| Method | Path | Notes |
|--------|------|-------|
| GET | `/api/objects` | `?category=&q=` optional filters |
| GET | `/api/objects/[slug]` | Single object |
| GET | `/api/objects/today` | Object of the day |
| GET | `/api/categories` | Category list with counts |
| GET | `/api/reactions/[slug]` | Read chills count |
| POST | `/api/reactions/[slug]` | Increment chills count |

---

## Constraints Summary

- No auth, no user accounts, no payments
- No real photo/image assets — render with CSS/SVG/generated visuals
- Exactly 12 seed objects — facts are canonical, prose can be rewritten
- Local dev only (`npm run dev`); Vercel deploy is optional stretch
- No third-party services, no external API keys

---

## Open Design Decisions (for plan node)

1. Exact page/route structure (home, catalog, detail, or SPA with modal overlays?)
2. Tailwind CSS vs vanilla CSS-in-JS vs CSS modules
3. shadcn/ui adoption vs fully custom components
4. Procedural starfield implementation (canvas, CSS, SVG?)
5. SQLite initialization strategy (migration file vs inline CREATE IF NOT EXISTS)
6. Casual rate-limit approach (cookie vs IP header)
