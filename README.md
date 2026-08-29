# Momentum

A small, self-contained personal dashboard for sticking to a daily practice
organized around a few life categories — not a calendar app, not a project
manager, not a general habit tracker.

A persistent sidebar holds four sections: **Dashboard** (the home screen —
a Today card of tasks one tap from done, per-category accomplishment %,
streaks, and completion scores, at a glance), **Calendar** (a weekly
template of when things happen, directly editable — click/drag right on
the grid), **Input Tracker** (the daily work — tasks grouped under
user-defined, color-coded categories, each with a single-tap daily
status), and **Goals** (short/mid/long-term outcome goals that can ladder
into each other, plus a short identity statement — who this practice is
building, separate from what it's aiming at).

Every task is either done or it isn't — no partial credit, no effort
level underneath to weigh before tapping. No points, no badges, no
gamification.

See `docs/PRODUCT.md` for the full concept and design principles.

## Status

**All four build phases are done.** This is the second design of Momentum —
see `CLAUDE.md` for how it relates to the first (category-less anchors +
weekly/monthly review) design. See `docs/ROADMAP.md` for details on each:

- Phase 1: sidebar shell + Input Tracker (core daily tracking) ✅
- Phase 2: Dashboard (KPIs derived from tracker data) ✅
- Phase 3: Calendar view ✅
- Phase 4: Goals + Identity section ✅

## Tech

Plain HTML/CSS/JS, `localStorage` only, no backend, no login, no external
APIs, no build step, no dependencies. The whole app is a single `index.html`
file you can open directly in a browser — offline by construction. See
`CLAUDE.md` for full conventions.

## Docs

- [`docs/PRODUCT.md`](docs/PRODUCT.md) — concept, philosophy, non-goals.
- [`docs/DATA-MODEL.md`](docs/DATA-MODEL.md) — storage schema,
  accomplishment %/streak/completion-score calculations.
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — phased build plan.
- [`CLAUDE.md`](CLAUDE.md) — project conventions for working in this repo.

## Running it

Open `index.html` directly in a browser (double-click it, or `file://` it).
That's the entire "install" step, on any device — nothing to build, nothing
to serve. Data lives only in that browser's `localStorage` under the key
`momentum:v2`; there's no export/import in this version (see
`docs/ROADMAP.md`, "Explicitly not planned"), so switching browsers or
clearing site data currently means starting over.
