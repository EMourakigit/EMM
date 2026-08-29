# CLAUDE.md — Momentum project conventions

## What this is

A small, self-contained, offline personal dashboard (see `docs/PRODUCT.md`
for the product concept). It is not a product for other users — optimize
for "stays out of the way," not for extensibility, configurability, or
scale.

This is the second design of "Momentum." The first was a flat, category-less
anchors + weekly/monthly review app (single scrolling page, then later a
dashboard shell). This version restructures around user-defined categories,
a richer per-task tracker, a KPI dashboard, a reference calendar, and a
goals/identity section — see `docs/ROADMAP.md` for the phased rebuild. Two
things carried over from the first design's philosophy did *not* survive
this version's post-launch revision pass: floor/target (a per-task
minimum/ideal effort description) and the weekly flex day were both
removed entirely — see `docs/DATA-MODEL.md`'s "Removed:" notes and
`docs/ROADMAP.md`'s "Post-launch revisions" for why. Don't reintroduce
either without being asked.

## Hard technical constraints

- **No backend, no login, no network calls of any kind.** Everything runs
  client-side against `localStorage`. Do not add fetch/XHR/WebSocket calls,
  analytics, telemetry, or third-party embeds — not even for something
  innocuous like a CDN font.
- **No build step, no bundler, no package manager, no framework.** Plain
  HTML/CSS/JS only. No React/Vue/npm/webpack/Vite. If a "we need a library
  for this" impulse comes up, prefer writing the ~20 lines by hand.
- **The entire app is one file: `index.html`**, with `<style>` and
  `<script>` inline in it. This isn't a build-time concatenation step —
  write directly into that one file from Phase 1 onward. This keeps the
  "self-contained single HTML file" deliverable true at every commit, not
  just at some future packaging step.
- **Offline-capable by construction** — since there are no network calls and
  no external assets, this is automatic. Don't undermine it by adding a
  `<link>` to an external stylesheet/font/icon set.
- Target modern evergreen browsers (desktop + mobile Safari/Chrome). No
  polyfills, no transpilation, no need to support old browsers.

## File layout

```
momentum/
  index.html        # the entire app — markup, styles, logic (Phase 1+)
  CLAUDE.md
  README.md
  docs/
    PRODUCT.md       # product concept and philosophy
    DATA-MODEL.md    # localStorage schema, accomplishment/streak math, export/import
    ROADMAP.md       # phased plan
```

Nothing else gets added to the repo root without a good reason. If a file
count is growing, that's a signal to reread `docs/PRODUCT.md`'s "minimize"
principle before continuing.

## Code conventions (for when Phase 1 starts)

- Vanilla JS, no classes-as-framework ceremony — plain functions and a
  single in-memory state object mirroring `docs/DATA-MODEL.md`, re-rendered
  after each mutation. A simple "mutate state → save to localStorage →
  re-render affected DOM" loop is enough; no virtual DOM, no reactivity
  library.
- One `localStorage` key: `momentum:v2` (see `docs/DATA-MODEL.md` for why
  this is a new key rather than a migration of the old `momentum:v1` shape
  — the two data models are structurally incompatible). All reads/writes to
  it go through a couple of small helper functions (`loadState()` /
  `saveState(state)`), never scattered `localStorage.*` calls.
- Keep functions small and named after what they do to the data
  (`setTaskStatus`, `computeAccomplishmentPercent`, `computeCategoryStreak`,
  `saveGoal`) — this is a personal tool, but it should still read clearly
  six months from now.
- Dates: always use local-time `YYYY-MM-DD` string keys, never raw `Date`
  objects as map keys or in storage (see `docs/DATA-MODEL.md`). Be careful
  with timezone bugs around midnight — derive the date key from local
  `getFullYear`/`getMonth`/`getDate`, not from `toISOString()` (which is
  UTC and will shift the date near midnight).
- Build every screen inside the persistent sidebar shell from Phase 1
  onward (see `docs/ROADMAP.md`) — don't build Input Tracker as a standalone
  page and retrofit the shell later.
- No console warnings/errors left in on load. No dead code left "just in
  case" — this is a small app, delete instead of commenting out.

## Testing approach

No test framework. Manual verification against each phase's "Definition of
done" in `docs/ROADMAP.md`. If an accomplishment-%, streak, or date-handling
edge case is subtle enough to worry about, add a tiny inline
`console.assert(...)`-based sanity check near the function rather than
standing up a test runner — remove it once confidence is established, or
leave it if cheap and non-intrusive.

## Workflow

- Follow `docs/ROADMAP.md` phase by phase. Don't pull work forward from a
  later phase "while we're in there" — ask first if something in the
  current phase seems to require it.
- Changes to product behavior (not just implementation) should be reflected
  back into `docs/PRODUCT.md` and/or `docs/DATA-MODEL.md` in the same pass,
  not left to drift out of sync with the code.
- Commit only when asked. When committing, keep messages short and
  descriptive of the actual change (e.g. "Add Input Tracker screen"), not
  generic ("update files").
