# UI Summary — Cosmic Explorer

Every file created in the `cosmic-explorer` Next.js frontend has been mapped along with the precise locations of the integrated code-seams/stubs below:

## File Architecture Created

- `cosmic-explorer/lib/types.ts` — TypeScript types for the application.
- `cosmic-explorer/lib/data/catalog.ts` — Canonical, fully typed static array of 12 hand-picked stellar objects.
- `cosmic-explorer/lib/components/ProceduralVessel.tsx` — Custom planetarium component rendering beautiful, unique SVG/CSS visualizers based on cosmic categories.
- `cosmic-explorer/lib/components/FeaturedSolarPick.tsx` — Dynamic component capturing "Object of the Day" with dynamic API streams and button responses.
- `cosmic-explorer/lib/components/CatalogBrowser.tsx` — Cinematic grid layout with categorical filters, tag checks, and search bars.
- `cosmic-explorer/lib/db/client.ts` — Placeholder/mock for SQLite client initialization.
- `cosmic-explorer/lib/db/schema.ts` — Schema creation tables.
- `cosmic-explorer/lib/db/seed.ts` — Initial database bootstrap check.
- `cosmic-explorer/lib/api/objects.ts` — Methods resolving summaries, detailed records, categories, and UTC day deterministic objects.
- `cosmic-explorer/lib/api/reactions.ts` — Logic managing chills counts & temporary simulation states.
- `cosmic-explorer/lib/api/rate-limit.ts` — Abstract cooling validation helper.
- `cosmic-explorer/app/layout.tsx` — Custom deep-sky layouts filled with responsive vector-rendered starfields.
- `cosmic-explorer/app/globals.css` — Core styles incorporating custom radial glow techniques and orbital scrollbars.
- `cosmic-explorer/app/page.tsx` — Landing page detailing cosmic targets.
- `cosmic-explorer/app/catalog/page.tsx` — Entry path to the Catalog database.
- `cosmic-explorer/app/catalog/[slug]/page.tsx` — Dynamic slide views capturing each unique custom render and stat workbook.
- `cosmic-explorer/app/api/categories/route.ts` — Category API handler.
- `cosmic-explorer/app/api/objects/route.ts` — Catalog lists API.
- `cosmic-explorer/app/api/objects/day/route.ts` — Deterministic UTC pick solver.
- `cosmic-explorer/app/api/objects/[slug]/route.ts` — Object detail API.
- `cosmic-explorer/app/api/objects/[slug]/react/route.ts` — Real reaction increment engine.

---

## Integration Code Seams & Stubs (`// INTEGRATION:`)

We structured the codebase to leave intuitive seams for the integration node. Below is the list of every seam file, approximate line, and instructions for database binding, rate limits, and client overrides:

### 1. Database Connection & Client Setup
- **File:** `cosmic-explorer/lib/db/client.ts`
- **Line:** 4
- **Seam Description:** The integration node should install `better-sqlite3` and create a persistent SQLite instance that matches the schema requirements.
- **Expected Action:** Replace mock client code with the actual `better-sqlite3` driver setup directed to `cosmic.db` and run schema checks.

### 2. Idempotent SQLite Bootstrapping Seeding
- **File:** `cosmic-explorer/lib/db/seed.ts`
- **Line:** 3
- **Seam Description:** Ensure that on server start/boot, if database contents are empty, table columns are mapped and seeded with the typed canonical array.
- **Expected Action:** Implement database check, and execute `INSERT INTO objects` loop.

### 3. Resolving Live Store via SQL Statements
- **File:** `cosmic-explorer/lib/api/objects.ts`
- **Line:** 4
- **Seam Description:** Convert mock/memory arrays to SQL selections containing search, category queries and left-joining tables.
- **Expected Action:** Implement SQLite prepared statements:
  - `GET /api/objects` -> `SELECT objects.*, COALESCE(reactions.chills_count, 0) FROM objects LEFT JOIN reactions ON ...`

### 4. Cooldown and Hits Validation
- **File:** `cosmic-explorer/lib/api/reactions.ts`
- **Line:** 3
- **Seam Description:** Handle atomic increment values and throttling checks inside transactional writes.
- **Expected Action:** Implement transactional safety:
  - `INSERT OR REPLACE INTO reactions ... chills_count = chills_count + 1`

### 5. IP Address extraction & Limits Block
- **File:** `cosmic-explorer/lib/api/rate-limit.ts`
- **Line:** 1
- **Seam Description:** Use SQLite `rate_limit` table to enforce cooling delays by checking incoming consumer IPs against previous epochs.
- **Expected Action:** Extract IP, fetch block delay, output boolean values or rate status.

### 6. API Route Wrappers
- **File:** `cosmic-explorer/app/api/categories/route.ts` (Line: 6)
- **File:** `cosmic-explorer/app/api/objects/route.ts` (Line: 10)
- **File:** `cosmic-explorer/app/api/objects/day/route.ts` (Line: 5)
- **File:** `cosmic-explorer/app/api/objects/[slug]/route.ts` (Line: 11)
- **File:** `cosmic-explorer/app/api/objects/[slug]/react/route.ts` (Line: 11)
- **Seam Description:** Dynamic API handlers acting as endpoint routes that forward queries to underlying sqlite queries.
- **Expected Action:** Keep structure as-is, which forwards smoothly to the underlying database client.
