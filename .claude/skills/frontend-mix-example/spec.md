# Spec — Cosmic Explorer

> This is a **content & intent brief**, not a wireframe. It says what the site is,
> what it must communicate, the facts that must be accurate, and what a visitor
> must be able to do. It deliberately does **not** prescribe pages, layout,
> components, or copy — those are the design session's (Gemini's) to invent.

## Overview / intent

An interactive explorer for the most remarkable objects in the universe — planets,
moons, stars, nebulae, galaxies, and a black hole. The point is **awe**: a visitor
should land, feel the scale and strangeness of the cosmos, and want to keep clicking
from one object to the next. It's an educational toy, not a database — the facts are
real, but the experience should feel cinematic, not like a spreadsheet.

Working title is "Cosmos" — the design session may rebrand it (wordmark, name, tagline
are open).

**No authentication. No accounts. No login of any kind.** Anyone can use everything.

## Audience & mood

- For the curious — space enthusiasts, students, late-night Wikipedia-hole people.
- Mood: dark-sky, cinematic, vast, a little reverent. Think planetarium, not textbook.
- Express the cosmos through **CSS / SVG / gradients / procedural starfields and
  generated visuals** — do **not** assume real photographic assets are available. If
  imagery is referenced, use clearly-marked placeholders. (This constraint is on
  purpose: it pushes the design to *render* the universe rather than paste photos.)
- Any genuine brand constraint is loose: deep-space palette, strong typographic
  contrast. Beyond that, the look is the design session's call.

## Content & data (these facts must stay accurate)

The catalog is a fixed seed set of ~12 objects. Each object has: a name, a category,
a one-line tagline, a longer description, a small set of key stats, and 2–4 "did you
know" facts. The **facts below are canonical — preserve them; do not invent numbers.**
Taglines and descriptive prose can be (re)written for voice.

Categories: Planet, Moon, Star, Nebula, Galaxy, Black Hole, Exoplanet System.

1. **The Sun** — Star. A G-type main-sequence star (G2V), ~4.6 billion years old; ~150
   million km (1 AU) from Earth; core temperature ~15 million °C; contains ~99.86% of
   the Solar System's mass.
2. **Jupiter** — Planet. Largest planet in the Solar System; the Great Red Spot is a
   storm wider than Earth that has raged for centuries; ~778 million km from the Sun;
   95 officially recognized moons; a day lasts only ~10 hours.
3. **Saturn** — Planet. Famous for its ring system of ice and rock; ~1.4 billion km
   (9.5 AU) from the Sun; a gas giant less dense than water; largest moon is Titan.
4. **Mars** — Planet. The "Red Planet," colored by iron oxide; home to Olympus Mons,
   the tallest volcano in the Solar System at ~22 km; two small moons, Phobos and
   Deimos; ~228 million km from the Sun.
5. **Europa** — Moon. A moon of Jupiter with an icy crust over a subsurface saltwater
   ocean; one of the most promising places to search for life beyond Earth; slightly
   smaller than Earth's Moon.
6. **Titan** — Moon. Saturn's largest moon and the only moon with a dense atmosphere
   (thick nitrogen); has lakes and rivers of liquid methane and ethane on its surface.
7. **Betelgeuse** — Star. A red supergiant in the constellation Orion, ~550–650 light
   years away; one of the largest stars visible to the naked eye; expected to end in a
   supernova; it visibly dimmed in 2019–2020 (the "Great Dimming").
8. **Sagittarius A\*** — Black Hole. The supermassive black hole at the center of the
   Milky Way; about 4.3 million times the mass of the Sun; ~26,000 light years away;
   first directly imaged by the Event Horizon Telescope in 2022.
9. **Orion Nebula (M42)** — Nebula. A stellar nursery ~1,344 light years away where new
   stars are being born; visible to the naked eye as the middle "star" of Orion's sword.
10. **Pillars of Creation** — Nebula. Towering columns of gas and dust in the Eagle
    Nebula (M16), ~5,700 light years away; famously imaged by Hubble in 1995 and by the
    James Webb Space Telescope in 2022.
11. **Andromeda Galaxy (M31)** — Galaxy. The nearest large spiral galaxy to ours, ~2.5
    million light years away; contains roughly one trillion stars; on a collision course
    to merge with the Milky Way in ~4.5 billion years.
12. **TRAPPIST-1** — Exoplanet System. An ultracool dwarf star ~40 light years away
    orbited by seven Earth-sized planets, several of which sit in the habitable zone —
    one of the best-known systems for studying potentially habitable worlds.

## Capabilities (jobs to be done)

A visitor must be able to:

- **Arrive and be pulled in** — the entry experience should establish the mood and
  surface a featured object ("today's object").
- **Browse the whole catalog** and **narrow it** — by category, and/or search by name.
- **Open any object** and read its full detail: description, key stats, and facts.
- **Discover the "object of the day"** — one object highlighted per calendar day,
  chosen deterministically (same object for everyone on a given date).
- **React to an object** — a lightweight, no-login "this gave me chills" reaction that
  increments a count on the backend and reflects the running total. This is the one
  piece of genuine write-interaction; it must persist server-side, not just in the
  browser.

How many pages this takes, how browsing and filtering are laid out, how the detail
view is composed, and how the reaction control looks — all of that is the design
session's decision.

## Backend / integration scope

Keep it self-contained — no third-party services, no external API keys.

- **Data store:** a local SQLite database (file-based) holding the seed catalog and a
  reaction-counts table. Seed it from the canonical content above. (A seeded data module
  is acceptable for the read-only catalog if simpler, but the reaction count must live
  in a real server-side store.)
- **API endpoints** (shape is a guide; the integration session finalizes exact paths):
  - List objects, with optional category filter and name search.
  - Get one object by slug.
  - Get the "object of the day" (deterministic by current date).
  - List categories with counts (if useful for filtering).
  - Increment and read an object's "chills" reaction count (no auth; rate-limit
    casually by session/IP if convenient, but logins are out of scope).
- **No auth, no user accounts, no protected routes.** Every endpoint is public.

## Deployment

**Local only for this build** — `npm run dev`, no cloud deploy required. (Optional
stretch: a single `vercel deploy --prod`, but it is not needed and should not block the
build.)

## Constraints / non-goals

- No authentication, no payments, no user-generated content beyond the anonymous
  reaction count.
- No real photographic/image assets required — render the cosmos with CSS/SVG/generated
  visuals; placeholders only if images are referenced.
- Keep the data to the 12 seed objects; the facts above are the source of truth.
- Multiple distinct pages/views are expected so the design range is visible, but the
  exact page set is the design session's call.
