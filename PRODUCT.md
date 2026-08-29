# Momentum — Product Concept

## The problem this solves

Full calendar/task apps optimize for capturing *everything*. That's the
wrong tool for tracking a working set of daily commitments organized around
a few life categories, alongside the handful of outcome goals and the
identity behind them. Momentum is a personal dashboard for exactly that —
not a to-do list, not a project manager, not a general habit tracker.

## Core philosophy

1. **No partial credit, no lesser version of a win.** The app must never
   present a day as partial credit, a lower score, or anything other than
   "you did it" or "you didn't." Earlier drafts of this app expressed that
   through a separate floor/target effort level per task; that concept has
   since been removed entirely (not just hidden) in favor of a single
   undifferentiated Done — one less thing to weigh before tapping, and an
   even more literal reading of "no partial credit" than floor/target ever
   was. See Task status, below.
2. **Honesty over motivation.** No streak-saving gimmicks, no points, no
   badges, no encouraging copy that papers over a missed day. Every number
   the dashboard shows reflects what actually happened. The user is
   expected to be intrinsically motivated; the app's job is just to make
   tracking effortless and truthful.
3. **Minimize taps, minimize decisions.** The daily check-in should be
   answerable in seconds. If a feature adds a decision the user has to
   think about before tapping, it's cut or simplified — see Task status,
   below, for how this shaped the tracker.
4. **The app must never become a chore.** If maintaining Momentum starts to
   feel like its own project — more settings, more views, more data entry —
   that's a regression, not a feature.

## Structure: sidebar sections

A persistent left sidebar with four sections (see `docs/ROADMAP.md` for
what's built in which phase). On narrow screens it collapses to icons and,
on phones, moves to a fixed bottom navigation bar with a matching top app
bar above the content — centered logo, light/dark/auto toggle on the
right. The layout is responsive from phone through desktop. There's no
separate Setup section this time —
category and task management (add/rename/reorder/archive) happens directly
inline within Input Tracker, since categories and tasks are that screen's
content, not standalone configuration:

1. **Dashboard** — the app's home screen (what's open first, every time).
   The at-a-glance overview. A compact **time filter** at the top — a
   calendar-icon button that opens a small menu of Today / Week / Month /
   Year / Custom range (Custom adds a date-range picker) — scopes
   *everything* below it; it defaults to Today and is remembered across
   sessions.
   Under it: a one-line completion headline for the range, then a
   **Non-negotiable / Other** card. On the Today filter each group lists
   only the tasks scheduled for today, one tap from done — tasks not due
   today are simply left off, not shown greyed. On any other range each
   group lists every task that applied during it, with its completion
   stats and a per-task % list. **Both groups are boxed cards**: the
   non-negotiable one is a ★-marked green panel with a done/total tally so
   the must-dos read as commitments; "Other tasks" is a lighter
   orange-accented card one step down in emphasis. On the Today filter,
   each task in either card is one tap from done — tapping fills its
   checkbox and strikes the name through ("scratched off"), tapping again
   undoes it. Completion is stored per calendar day, so the list starts
   fresh the next day on its own. Then two charts — a
   **donut** of Done vs Not Done, taken over *every* scheduled task-entry
   in the range (not just the ones logged): with nothing done yet it reads
   100% Not Done and fills toward Done as habits get checked off. "Not
   Done" folds in both explicit Not-Done marks and still-To-Do entries;
   N/A days never count. Beside it a **per-category bar** of done-vs-not —
   and a per-category grid showing
   that category's completion % for the range plus its current streak.
   The streak is the only figure that always reads "now", not the range.
   Aside from the Today list's taps, the Dashboard is read-only.
2. **Calendar** — a visual weekly schedule showing planned blocks, for
   reference alongside the tracker. This is a repeating weekly template,
   not a real calendar app: no specific dates, no reminders, no syncing to
   anything. The week is shown **Monday-first**. Two ways in: a
   **"＋ Add block"** button at the top of the screen opens a popup (name,
   color, one or more days — selecting several adds an identical block to
   each — and start/end time), and **tapping any block on the grid** opens
   an edit popup with the same fields plus Delete. There's no per-block
   link back to a category or task; a block is just whatever you name it.
   A "Clear calendar" action below the grid removes every block at once,
   with a confirmation first — for starting the template over rather than
   deleting blocks one at a time.

   Times are picked as an **hour + minute pair** (short dropdowns, not the
   native spinner and not one long list), with **duration shortcut chips**
   below — 30 min, 1h, 1h 30m, 2h — that set the end relative to the
   start. The grid still has **no drag** — no drawing,
   moving, or resizing by gesture; a plain click on a block is the only
   thing it responds to. (Earlier designs tried drag-to-draw and
   edit-in-place and dropped them for not holding up across
   browser/input combinations; a single click that opens an ordinary form
   popup is the reliable middle ground.)
3. **Input Tracker** — the daily work. Tasks grouped under user-editable
   category headers, each with a per-day status and a rolling
   accomplishment %. Category blocks are visually separated (a blank band
   between them, an accent-coloured header per category) so it's obvious
   at a glance where one ends and the next begins. This is the screen
   opened daily; everything else is downstream of what happens here.
4. **Goals** — a single identity statement, always shown first, plus outcome
   goals grouped into three time horizons (short-term first). All
   editable. See Goals & identity, below.

Below the four sections, the sidebar footer holds one more control: a
light/dark/auto theme toggle (on phones it moves to the right side of the
top app bar instead). "Auto" (the default) follows the OS/browser's
own light-or-dark setting; the other two states are an explicit override
that sticks regardless of what the OS is set to, remembered per device.
One button, cycling through the three states on each click — not a
settings panel, in keeping with the "must never become a chore" principle
above.

## Categories

A category is a user-defined grouping for tasks — e.g. "Productivity
Improvement," "Career Development," "Health & Well-being." Categories are
not hardcoded; the user creates, renames, reorders, and archives them.

Each category has a name and a color, used consistently everywhere the
category shows up (Input Tracker's group headers, the Dashboard's
per-category KPIs) so a category is visually recognizable at a glance
without reading the label every time. The Calendar is the one exception —
its blocks aren't linked to a category or task at all, and pick their own
color independently; see Calendar, above.

## Tasks

A task lives inside exactly one category and is the thing actually tracked
day to day — e.g. "Gym," "1 outreach message," "Read 10 pages." Each task
has:

- **Name.**
- **Schedule** — which days of the week the task applies to (e.g. every
  day, or Mon–Fri only). Days outside the schedule show as N/A rather than
  as a gap the user has to explain.
- **Non-negotiable** (optional flag) — whether this task belongs in the
  Dashboard Today card's Non-negotiable group rather than Other. Purely a
  display grouping; it has no effect on accomplishment logic.

Earlier versions of this task also carried a floor and a target — a
minimum-effort and ideal-effort description in the user's own words. Both
have been removed entirely: not merged, not hidden, not still stored and
just unshown. A task is either done or it isn't; there's no effort level
underneath that to describe. See Task status, below.

## Task status (the daily tap)

Each task has one status per applicable day, and marking it is a single
tap:

- **Done** — the task happened. One undifferentiated state — there is no
  effort level or "how well" to weigh before tapping, which is what makes
  this genuinely a single tap rather than a decision.
- **To-Do** — not yet acted on. The default state for any applicable day
  that hasn't been marked — today included, and any day still in the
  future. Nothing is ever silently blank; every applicable cell has an
  explicit status.
- **Not Done** — actively did not happen. Neutral language on purpose — no
  "missed," no "failed."
- **N/A** — not applicable. Automatic, not chosen: any day outside the
  task's schedule renders N/A and never enters the accomplishment math.

## Accomplishment %, streaks, and completion scores

- **Per-task week %** (the "Week %" column on every Input Tracker row):
  Done days ÷ days the task is scheduled *in the week currently shown* —
  the whole Mon–Sun span, including weekdays still to come, so the figure
  reads as progress through that week rather than jumping to 100% off the
  one day logged so far. N/A days never count. Navigating to another week
  recomputes it for that week; it shows "—" for a week the task isn't
  scheduled on any day (or predates its creation). (A separate all-time
  rolling accomplishment % is still computed internally for rate-linked
  goals — see Goals — but is no longer shown in the tracker.)
- **Per-category streak** (shown on the Dashboard): consecutive days, most
  recent backward, on which every applicable task in that category was
  Done. N/A days are transparent — they neither extend nor break the
  streak, they're simply skipped over.
- **Weekly / overall completion score** (shown on the Dashboard): the same
  Done ÷ applicable ratio, aggregated across every task in every category —
  weekly scoped to the current week so far, overall scoped to all time.

Full detail and exact rules are in `docs/DATA-MODEL.md`.

An earlier version of this design had a weekly "flex day" — one day where
every task was optional and excluded from this math. It was removed
entirely (not just hidden from Input Tracker) once it became clear that
"optional here" but still silently shaping calculations everywhere else
was more confusing than useful; see `docs/DATA-MODEL.md` for the specific
change. Every scheduled day is now tracked identically, everywhere.

## Goals & identity

Two related but distinct things, shown together in the Goals section, both
editable, neither tied to a daily tap, streak, or accomplishment % —
updating either is a deliberate, occasional action, not a daily one.

### Identity

One free-text block, always shown first, answering "who am I becoming" —
e.g. "I'm becoming the kind of person who shows up daily on their goals."
Separate from outcome goals on purpose: an outcome goal is a destination
("get a new job," "save 100K DH"); identity is about who's doing the
showing up, and doesn't get "completed." A single block, not a list —
earlier drafts had this as several separate short entries, but one
continuous statement (edited freely, as long or short as it needs to be)
turned out to fit "who am I becoming" better than a bag of fragments.

### Goals, in three horizons

Goals are grouped into three horizons, each its own visually distinct
section, laid out as **side-by-side columns** (nearest-term leftmost),
wrapping and then stacking on narrower screens:

- **Short-term** (days to weeks)
- **Mid-term** (weeks to months)
- **Long-term** (months to years)

Goal cards stack vertically within their horizon column.

A goal is **created** through a single "＋ New goal" button at the top of
the Goals screen (next to the heading, above Identity), which opens a
popup — name (required), horizon, deadline — submitted with one button.
There is no per-section add form. The **horizon is chosen in the popup**
and decides which section the new card lands in (it defaults to
Short-term). Cancel, the ✕, a click on the backdrop, or Escape all
dismiss it without creating anything.

Once created, a goal's card shows the **name**, a **progress bar**, and a
small **"Linked" / "Rolled up" / "Manual"** tag saying whether that
progress is live-tracked or hand-entered. **Clicking anywhere on the
card** opens a popup holding everything editable — Name, Horizon,
Deadline, "Ladders up to", Progress — plus Archive. Every field autosaves;
"Done" just closes. Changing the Horizon moves the card to another
section. Each horizon carries its own accent color, used for the section
header strip and the card's left edge.

### Progress can track Input Tracker automatically

Progress is still a **simple 0–100 indicator**, but a goal can compute it
from real data instead of a slider:

- **Short-term goals** may link to one or more Input Tracker tasks, in one
  of two modes:
  - **Rate-based** — progress = the linked task's rolling accomplishment %
    (averaged if several tasks are linked).
  - **Count-based** — the goal has a numeric target (e.g. 500); progress =
    the running count of "Done" entries on the linked task(s), as a % of
    that target.
- **Mid- and long-term goals** don't touch tasks directly. Their progress
  is the **average progress of the goals that ladder up into them** (via
  "Ladders up to"). A long-term goal thus rolls up its mid-term goals,
  which roll up their short-term goals, which track tasks — so daily
  check-ins flow all the way up on their own.
- Any goal with **no link** (no tasks, or no child goals yet) keeps the
  **manual slider** as the fallback — linking is never forced. The manual
  value is remembered and reused if the goal is later unlinked.

Linked progress is always recomputed from current state, so it reflects
today's Input Tracker without any refresh step.

Deadline is optional — a plain date, not a reminder or a notification (no
notifications exist anywhere in this app; see Explicit non-goals). Its
only effect is a small "Overdue" badge once that day has passed and the
goal's (live) progress isn't yet 100% — cleared automatically when it
reaches 100% or the deadline is pushed back.

Which horizon a goal belongs to is the user's call. Goals are still not
tied to *categories*; the task/child links above are the only data
relationships.

## Explicit non-goals

- No points, badges, levels, or gamified rewards of any kind.
- No social features, sharing, or leaderboards.
- No notifications/reminders (offline, no backend — nothing to push from).
  Could be reconsidered later only as a purely local mechanism, never a
  requirement.
- No AI features, no external API calls, no network requests at all.
- No multi-user support, no login, no cloud sync.
- No real scheduling system: the Calendar is directly editable, but only as
  a repeating weekly template — no specific dates, no conflict detection,
  no reminders, no syncing to any device calendar.
- No fixed weekly/monthly reflection prompts in this version (the first
  design had them). Not ruled out forever, but not in scope for
  Phases 1–4 — see `docs/ROADMAP.md`.
- No mood tracking or journaling.
- No flex day / optional-day concept — see Accomplishment %, streaks, and
  completion scores, above, for what this app had instead and why it was
  removed.
