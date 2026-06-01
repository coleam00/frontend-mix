## SECTION A - Site Content & Intent

### 1. What it is

Cosmic Explorer (working title "Cosmos") is a cinematic, no-login interactive web toy for exploring twelve canonical objects in the universe — planets, moons, stars, nebulae, galaxies, a supermassive black hole, and one exoplanet system. The audience is curious laypeople: space enthusiasts, students, late-night Wikipedia-hole readers. The single most important thing a visitor should walk away with is **a felt sense of awe and scale** — they arrive, they're pulled in, they want to keep clicking from one object to the next. It is an educational toy, not a database; the facts are real, but the experience must feel like a planetarium, not a spreadsheet.

### 2. Voice & mood

- **Mood:** dark-sky, cinematic, vast, a little reverent. Planetarium energy. Quiet confidence rather than excited carnival.
- **Voice:** awe-leaning but factually grounded. Prose can have rhythm and weight; it should never sound like a textbook caption or marketing copy. Taglines and descriptions can be rewritten freely for voice — the canonical facts (numbers, names, dates) must be preserved verbatim.
- **Brand constraints (loose, on purpose):**
  - Deep-space palette only — deep blacks, navy, midnight blues, with high-contrast typography. Accent hues from the cosmos (warm star yellows, nebula magentas/cyans) are fine where they earn it.
  - All cosmic imagery must be rendered with CSS / SVG / gradients / procedural starfields. **No real photo assets.** If imagery is referenced, use clearly-marked placeholders. This constraint is deliberate: it pushes the design to *render* the universe instead of pasting Hubble photos.
  - Wordmark, product name, and tagline are open — the design session may rebrand from "Cosmos."

### 3. Content & messaging

The substance the site must convey:

- **The catalog is exactly twelve objects, spanning seven categories** (Planet, Moon, Star, Nebula, Galaxy, Black Hole, Exoplanet System). Each object carries: name, category, a one-line tagline, a longer description, a small set of key stats, and 2–4 "did you know" facts.
- **Canonical facts (must remain accurate; do not invent numbers):**
  1. **The Sun** — Star. G-type main-sequence (G2V), ~4.6 billion years old; ~150 million km (1 AU) from Earth; core temperature ~15 million °C; contains ~99.86% of the Solar System's mass.
  2. **Jupiter** — Planet. Largest planet in the Solar System; Great Red Spot is a storm wider than Earth, raging for centuries; ~778 million km from the Sun; 95 officially recognized moons; a day lasts ~10 hours.
  3. **Saturn** — Planet. Famous ring system of ice and rock; ~1.4 billion km (9.5 AU) from the Sun; gas giant less dense than water; largest moon is Titan.
  4. **Mars** — Planet. "Red Planet," colored by iron oxide; Olympus Mons is the tallest volcano in the Solar System at ~22 km; two small moons, Phobos and Deimos; ~228 million km from the Sun.
  5. **Europa** — Moon. Moon of Jupiter; icy crust over a subsurface saltwater ocean; one of the most promising places to search for life beyond Earth; slightly smaller than Earth's Moon.
  6. **Titan** — Moon. Saturn's largest moon; only moon with a dense atmosphere (thick nitrogen); has lakes and rivers of liquid methane and ethane on its surface.
  7. **Betelgeuse** — Star. Red supergiant in Orion, ~550–650 light years away; among the largest stars visible to the naked eye; expected to end in a supernova; visibly dimmed in 2019–2020 (the "Great Dimming").
  8. **Sagittarius A\*** — Black Hole. Supermassive black hole at the center of the Milky Way; ~4.3 million times the mass of the Sun; ~26,000 light years away; first directly imaged by the Event Horizon Telescope in 2022.
  9. **Orion Nebula (M42)** — Nebula. Stellar nursery ~1,344 light years away where new stars are being born; visible to the naked eye as the middle "star" of Orion's sword.
  10. **Pillars of Creation** — Nebula. Towering columns of gas and dust in the Eagle Nebula (M16), ~5,700 light years away; famously imaged by Hubble in 1995 and by the James Webb Space Telescope in 2022.
  11. **Andromeda Galaxy (M31)** — Galaxy. Nearest large spiral galaxy to ours, ~2.5 million light years away; roughly one trillion stars; on a collision course to merge with the Milky Way in ~4.5 billion years.
  12. **TRAPPIST-1** — Exoplanet System. Ultracool dwarf star ~40 light years away orbited by seven Earth-sized planets, several in the habitable zone; one of the best-known systems for studying potentially habitable worlds.

- **Points to land throughout the site:**
  - Scale (distances, sizes, durations) is the through-line — every object should make the visitor feel how absurdly big, hot, far, or old the cosmos is.
  - The "chills" reaction is the one social signal — a quiet collective note that many other anonymous visitors found this object moving too. It should feel like leaving a candle, not like upvoting a post.
  - "Object of the day" is the same for everyone on a given calendar date — that universality is part of the point.
  - This is built without photos on purpose — the visual interpretation of each object is part of the art.

- **Final headlines, button labels, taglines, navigation labels, empty states, and microcopy are the design session's to author.** Do not lock copy here.

### 4. Capabilities (jobs to be done)

A visitor must be able to:

- **Arrive and be pulled in** — the entry experience surfaces the mood and a featured object (the "object of the day").
- **Browse the full catalog** of twelve objects.
- **Narrow the catalog** by category, and/or search by name.
- **Open any object** and read its full detail: description, key stats, and "did you know" facts.
- **Discover the object of the day** — one object highlighted per calendar day, chosen deterministically (same object for all visitors on a given date).
- **React with "chills"** to an object — a lightweight, no-login reaction that persists server-side and reflects a running total across all visitors. This is the only write interaction.

How many pages this takes, how filtering/search are exposed, how detail is composed, and how the reaction control surfaces — all design-session decisions.

---

## SECTION B - Integration Scope

### Framework & runtime

- **Next.js 14+ with the App Router, TypeScript, React Server Components.** One process serves both the UI and the API route handlers, so `npm run dev` is the entire stack.
- **Package manager:** npm (matches the spec's `npm run dev` instruction; lockfile committed).
- **Node:** 20.x LTS.
- **No auth provider, no third-party SDKs, no external API keys.** Everything is self-contained.

### Project layout (initial scaffold)

```
cosmic-explorer/
├── app/                       # Next.js App Router (pages are Gemini's call)
│   └── api/                   # Route handlers below
├── lib/
│   ├── db.ts                  # better-sqlite3 connection + init
│   ├── catalog.ts             # 12-object seed (canonical facts)
│   ├── objects.ts             # query helpers (list, by-slug, today, categories)
│   └── reactions.ts           # read/increment + casual rate-limit helper
├── data/
│   └── cosmic.db              # SQLite file (gitignored; auto-created on first run)
├── public/                    # only fonts/icons; no photo assets
├── package.json
├── tsconfig.json
└── next.config.mjs
```

### Data model (SQLite via `better-sqlite3`)

The catalog is **seeded from `lib/catalog.ts` (a typed constant module)** rather than a DB table — the twelve objects are canonical and read-only, so a code-defined source of truth is simpler and version-controlled. The DB exists exclusively for the one piece of mutable state: reactions.

```sql
-- Auto-created on app boot via CREATE TABLE IF NOT EXISTS
CREATE TABLE reactions (
  slug  TEXT PRIMARY KEY,
  count INTEGER NOT NULL DEFAULT 0
);

-- Optional: light-touch rate-limit log (only if cookie approach insufficient)
CREATE TABLE IF NOT EXISTS reaction_log (
  id         INTEGER PRIMARY KEY AUTOINCREMENT,
  slug       TEXT NOT NULL,
  ip_hash    TEXT,
  created_at INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_reaction_log_ip_slug_time
  ON reaction_log (ip_hash, slug, created_at);
```

**Catalog TypeScript shape (in `lib/catalog.ts`):**

```ts
type Category =
  | "planet" | "moon" | "star" | "nebula"
  | "galaxy" | "black-hole" | "exoplanet-system";

interface CosmicObject {
  slug: string;          // e.g. "the-sun", "jupiter", "sagittarius-a-star"
  name: string;          // display name
  category: Category;
  tagline: string;       // one-line; Gemini may rewrite
  description: string;   // longer prose; Gemini may rewrite
  stats: { label: string; value: string }[];  // canonical facts
  didYouKnow: string[];  // 2-4 entries
}
```

Slugs are kebab-case derived from the name (`the-sun`, `andromeda-galaxy`, `pillars-of-creation`, `sagittarius-a-star`, `trappist-1`). Order of the array is the canonical sort order used by "object of the day."

### API surface (Next.js route handlers under `app/api/`)

All endpoints are public, JSON, no auth check, no CORS work (same-origin).

| Method | Path | Input | Output | Notes |
|---|---|---|---|---|
| GET | `/api/objects` | query: `category?` (slug), `q?` (case-insensitive name substring) | `CosmicObject[]` | Server reads from `lib/catalog.ts`, filters in memory. Merges current `chills` count from SQLite. |
| GET | `/api/objects/today` | — | `CosmicObject` | Deterministic: `dayOfYearUTC % 12` indexes into the sorted catalog. Same result for everyone on the same UTC date. |
| GET | `/api/objects/[slug]` | path: `slug` | `CosmicObject` or 404 | Full detail with current chills count. |
| GET | `/api/categories` | — | `{ slug, label, count }[]` | Derived from `lib/catalog.ts` at request time. |
| GET | `/api/reactions/[slug]` | path: `slug` | `{ slug, count }` | Reads from `reactions` table; returns `0` if row absent. |
| POST | `/api/reactions/[slug]` | path: `slug`; no body | `{ slug, count }` (new total) | Upsert + increment. Returns 404 if slug not in catalog. Returns 429 if rate-limited. |

### Reaction write path & casual rate-limit

- **Validation:** POST `/api/reactions/[slug]` first checks the slug exists in `lib/catalog.ts`. Unknown slug → 404.
- **Upsert pattern:**
  ```sql
  INSERT INTO reactions (slug, count) VALUES (?, 1)
  ON CONFLICT(slug) DO UPDATE SET count = count + 1
  RETURNING count;
  ```
- **Casual rate-limit (cookie-first, IP-fallback):**
  - On POST, set a signed HTTP-only cookie `chills_seen` (JSON: `{ slug: lastPostMs }`). Reject the next POST for the same slug within a short cooldown (e.g. 10 seconds) with 429.
  - Additionally hash the client IP (SHA-256 of `x-forwarded-for` or `req.socket.remoteAddress` + a per-install salt) and reject if more than ~30 reactions/minute across all slugs from the same hash. The salt lives in `.env.local` as `RATE_LIMIT_SALT` (auto-generated on first dev boot if missing).
  - This is deliberately leaky — the spec says "casual." No CAPTCHA, no auth.

### Database initialization

- `lib/db.ts` opens (or creates) `./data/cosmic.db` once at module load and runs `CREATE TABLE IF NOT EXISTS` for both tables.
- No migrations framework — the schema is two tiny tables and we own the only writer.
- `data/` is gitignored; the DB file is regenerated on first boot. Reactions are intentionally ephemeral across fresh clones (acceptable for an MVP toy).

### Environment variables

`.env.local` (gitignored; example in `.env.example`):

```
DATABASE_PATH=./data/cosmic.db   # optional override
RATE_LIMIT_SALT=                 # auto-filled on first boot if blank
```

No `ANTHROPIC_API_KEY`, no Slack/Asana/etc. — the workspace `master.env.example` is irrelevant to this app.

### Background jobs / async work

**None.** All writes are synchronous SQLite calls inside the POST handler. No queues, no cron, no webhooks.

### Out of scope (explicit non-goals)

- No authentication, no accounts, no protected routes.
- No payments.
- No user-generated content beyond the anonymous chills counter.
- No real photographic assets — visuals are CSS/SVG/gradient/procedural.
- No multi-user reaction history, no per-user state, no "you reacted" UI affordance beyond the cookie's local memory.

---

## SECTION C - Deployment Plan

**Target: Local only — no cloud deploy this run.**

### Run commands

```bash
cd cosmic-explorer
npm install
npm run dev          # serves on http://localhost:3000
```

### Pre-run steps

1. `npm install` (resolves `next`, `react`, `react-dom`, `typescript`, `@types/*`, `better-sqlite3`, `@types/better-sqlite3`).
2. First `npm run dev` boot will:
   - Create `./data/` if absent.
   - Create `./data/cosmic.db` and run `CREATE TABLE IF NOT EXISTS` for `reactions` (and `reaction_log` if used).
   - If `.env.local` exists with a blank `RATE_LIMIT_SALT`, fill it with a freshly generated value; otherwise skip.
3. No migrations to run — schema is bootstrapped in-process.
4. No seed step for the catalog — it lives in `lib/catalog.ts` and is read directly.

### Success criteria for "deploy"

The build is considered successfully deployed when, from a fresh clone:

- `npm install && npm run dev` exits to a running server on `localhost:3000` with no errors.
- `GET /api/objects` returns 12 entries.
- `GET /api/objects/today` returns a single object and returns the **same** object on repeated calls within the same UTC day.
- `POST /api/reactions/jupiter` returns `{ slug: "jupiter", count: 1 }` on first call and `{ slug: "jupiter", count: 2 }` on the second (after the cooldown).
- The home route renders the cinematic landing without console errors.

### Optional stretch (not required, must not block the build)

A single-command Vercel deploy is feasible later:

```bash
npx vercel deploy --prod
```

Caveats if pursued:
- `better-sqlite3` requires native bindings — Vercel's serverless runtime needs the appropriate Node build target, and the SQLite file is **ephemeral** on serverless (each invocation gets a fresh filesystem). For a real cloud deploy the reaction store would need to move to Vercel KV, Turso (libSQL), or Neon Postgres. That migration is **out of scope for this run** and noted only so a future deploy node doesn't get surprised.

No CI, no preview environments, no env-promotion pipeline for this build.
