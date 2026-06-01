Deploy method: Local only (per SECTION C)
SECTION C explicitly states "Target: Local only — no cloud deploy this run." — the app is a self-contained Next.js dev server (`npm install && npm run dev` on http://localhost:3000) with a local SQLite reactions store; no cloud target, no env promotion, no migrations.
