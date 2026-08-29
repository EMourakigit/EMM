# Momentum — Data Model

Single source of truth: one JSON object, persisted in `localStorage` under
key **`momentum:v2`**, read fully into memory on load and written back on
every mutation. No IndexedDB, no partial persistence.

`v2` is a deliberate new key, not a migration of the first design's
`momentum:v1` shape — that version had no categories, no tasks-with-schedule,
no goals/identity, and a different status model (target/floor/skipped vs.
this version's done/todo/not_done/na). The two are structurally
incompatible, so there is no migration path between them; `momentum:v1` is
simply left alone in `localStorage` (harmless, unread) if it exists from an
earlier install.

## Per-device UI preferences (not inside `momentum:v2`)

A few small, separate `localStorage` keys hold pure UI state — which
sidebar tab is open, whether the sidebar is collapsed, the chosen theme,
and the Dashboard time filter. None live inside the `momentum:v2` object:
they're per-device presentation state, not product data, so bundling them
with categories/tasks/goals would be the wrong place for them (a future
export/import, if one gets built, should not carry "which tab was open" or
"dark mode" between devices).

- `momentum:activeTab` — one of `"dashboard" | "calendar" | "tracker" |
  "goals"`. Read once on load; if missing or not one of those four exact
  strings, defaults to `"dashboard"` rather than trusting a possibly-stale
  or corrupted value. Written every time the sidebar's active tab changes,
  so a refresh reopens on the same screen instead of always landing back
  on Dashboard.
- `momentum:sidebarCollapsed` — `"1"` or `"0"`. If unset (first-ever
  load), defaults from `window.innerWidth < 880` rather than a fixed
  value, so it starts sensibly narrow on a laptop-sized window without
  requiring a stored preference yet.
- `momentum:theme` — one of `"auto" | "light" | "dark"`. If missing or not
  one of those three exact strings, defaults to `"auto"`. A footer button
  in the sidebar cycles through the three in that order and writes the new
  value immediately; `applyTheme()` mirrors it onto the `<html>` element as
  a `data-theme` attribute (`"light"` or `"dark"`), or removes the
  attribute entirely for `"auto"` so the OS-level `prefers-color-scheme`
  media query takes over. See Theme, below, for the CSS side of this.
- `momentum:dashFilter` — JSON `{ mode, start, end }`, the Dashboard time
  filter. `mode` ∈ `today | week | month | year | custom` (defaults to
  `today` if missing/corrupt); `start`/`end` are `YYYY-MM-DD` and only
  meaningful for `custom`. Written on every filter change so the Dashboard
  reopens on the same range. See "Dashboard time filter + range stats".

## Theme

Three CSS custom-property definition sites, all keyed to the same set of
token names (`--bg`, `--surface`, `--border`, `--text`, etc.), so the rest
of the stylesheet never branches on theme — it just reads the tokens:

1. Base `:root { ... }` — the light values, always the fallback.
2. `@media (prefers-color-scheme: dark) { :root:not([data-theme="light"])
   { ... } }` — dark values, applied automatically when the OS is in dark
   mode, *unless* the user has explicitly forced light (see below).
3. `:root[data-theme="dark"] { ... }` — the same dark values again, outside
   any media query, so an explicit dark choice applies even on a
   light-system OS.

`data-theme` is set on `document.documentElement` by `applyTheme(theme)`:
`"light"`/`"dark"` set the attribute to that value; `"auto"` removes it,
letting rule 2 (or the light default) decide from the OS setting alone.
This is why `"auto"` isn't itself a token set — it's the *absence* of an
override, not a third palette.

The sidebar footer's theme button cycles `auto → light → dark → auto` on
each click, immediately persisting to `momentum:theme` and swapping its
own icon/label to match the state it just entered.

## Top-level shape

```jsonc
{
  "schemaVersion": 1,
  "createdAt": "2026-08-26T09:00:00.000Z",

  "settings": {},   // reserved for future global settings; empty today (flexDayOfWeek was removed, see below)

  "categories": [
    {
      "id": "cat_a1b2",
      "name": "Health & Well-being",
      "color": "#3a7ca5",
      "order": 0,
      "archived": false,
      "createdAt": "2026-08-26T09:00:00.000Z",
      "archivedAt": null
    }
  ],

  "tasks": [
    {
      "id": "tsk_c3d4",
      "categoryId": "cat_a1b2",
      "name": "Gym",
      "scheduledDays": [0, 1, 2, 3, 4, 5, 6],  // days of week this task applies to; e.g. [1,2,3,4,5] = Mon-Fri only
      "nonNegotiable": false,   // shows in the Dashboard Today card's Non-negotiable group instead of Other; no effect on accomplishment logic
      "order": 0,
      "archived": false,
      "createdAt": "2026-08-26T09:00:00.000Z",
      "archivedAt": null
    }
  ],

  "statusesByTask": {
    "tsk_c3d4": {
      "2026-08-26": "done"      // "done" | "not_done"
      // no key for a date = "todo" if that date is within the task's
      // schedule and not in the future, "na" if outside the schedule.
      // Never store "todo" or "na" explicitly — always derived, see below.
    }
  },

  "scheduleBlocks": [
    {
      "id": "blk_e5f6",
      "label": "Gym",            // free text, the block's only name
      "dayOfWeek": 1,            // 0=Sunday .. 6=Saturday
      "startTime": "07:00",      // "HH:MM", 24h, local time
      "endTime": "07:45",
      "color": "#3a7ca5"         // hex or null — null renders as the default accent color
    }
  ],

  "goals": [
    {
      "id": "gol_g7h8",
      "name": "Get a new job",
      "horizon": "long",          // "short" | "mid" | "long" — chosen in the New goal popup, editable after
      "deadline": null,           // "YYYY-MM-DD" or null — optional, no reminders, just an overdue badge
      "percent": 0,               // 0-100 — MANUAL fallback progress (used only when the goal isn't linked)
      "parentGoalId": null,       // another goal one horizon up; this goal's live progress rolls into that parent
      "linkedTaskIds": [],        // short-term only: Input Tracker task ids feeding live progress
      "linkType": null,           // short-term only: null (manual) | "rate" | "count"
      "countTarget": null,        // short-term "count" only: numeric target for cumulative "done" entries
      "order": 0,
      "archived": false,
      "createdAt": "2026-08-26T09:00:00.000Z",
      "updatedAt": "2026-08-26T09:00:00.000Z"
    },
    {
      "id": "gol_h1i2",
      "name": "Read 500 pages",
      "horizon": "short",
      "deadline": "2026-10-15",
      "percent": 0,
      "parentGoalId": "gol_g7h8",
      "linkedTaskIds": ["tsk_c3d4"],
      "linkType": "count",
      "countTarget": 500,
      "order": 0,
      "archived": false,
      "createdAt": "2026-08-27T09:00:00.000Z",
      "updatedAt": "2026-08-27T09:00:00.000Z"
    }
  ],

  "identityStatement": "I'm becoming the kind of person who shows up daily on their goals."
}
```

## Entities

### Category
- `id`: stable, generated once (`cat_` + random base36), never reused.
- `name`: short free text, user-defined (no hardcoded category list).
- `color`: a hex color, either user-picked or auto-assigned from a small
  fixed palette when the category is created (so every category is visually
  distinct from the start with zero required decisions). Used consistently
  for that category's group header in Input Tracker, its KPI card on the
  Dashboard, and its blocks on the Calendar.
- `order`: integer, controls display order.
- `archived` / `archivedAt`: never hard-deleted, so historical data stays
  meaningful. An archived category (and its tasks) drops out of Input
  Tracker, Dashboard, and Calendar going forward; past statuses referencing
  its tasks are untouched.

### Task
- `id`: stable, generated once (`tsk_` + random base36), never reused.
- `categoryId`: the one category this task belongs to. A task always
  belongs to exactly one category — no multi-category tagging.
- `name`: short free text.
- `scheduledDays`: array of day-of-week ints (`0`=Sunday..`6`=Saturday) this
  task applies to. `[0,1,2,3,4,5,6]` (every day) is the default for a new
  task. A day outside this array is always N/A for this task, regardless of
  anything else. **Must be non-empty:** a task with no scheduled days is
  never "applicable" anywhere, so its %, streak, and dashboard stats all
  silently read "—". `toggleTaskScheduledDay` refuses to remove the last
  remaining day, and `loadState()` restores any task found with an empty
  (or missing/non-array) `scheduledDays` to every day.
- `nonNegotiable`: boolean, default `false`. Toggled in the task's edit
  strip. The only thing it drives is which of the two groups a task falls
  into on the Dashboard's Today card (see Task status, and
  `docs/PRODUCT.md`) — it has no effect on `scheduledDays`, applicability,
  accomplishment %, or streaks.
- `order`: integer, controls display order within its category.
- `archived` / `archivedAt`: same non-destructive archive pattern as
  Category.

**Removed: `floor` / `target`.** Earlier versions of this task had a
`floor` and a `target` — free-text minimum/ideal effort descriptions,
never a second thing to log (Task status was always a single tap either
way), just reference text. First their inline grid display was removed,
then the fields themselves: both are gone from the schema entirely now,
not merely hidden. `loadState()` deletes `floor`/`target` from any older
saved task on load — the same ad-hoc migration pattern as flex day's
removal, below.

### Task status (`statusesByTask`)
- Keyed first by task id, then by local date key `YYYY-MM-DD`.
- Stored value is `"done"` or `"not_done"` — **only ever written when the
  user explicitly taps one of those two.** Nothing else is ever persisted
  here; there is no stored `"todo"` or `"na"` value.
- **Derived status** for a given task + date, computed at render/calc time,
  never stored:
  1. If the date is outside `task.scheduledDays` for that weekday → **N/A**.
  2. Else if `statusesByTask[taskId][date]` is `"done"` or `"not_done"` →
     that value.
  3. Else → **To-Do** (covers both "not yet logged, still upcoming" and
     "was never logged" — see accomplishment-% handling of past-due To-Do
     below).
- This keeps storage sparse (only real taps are written) rather than
  writing an entry for every applicable day of every task forever.
- The Dashboard's Today card reads and writes this exact same structure,
  scoped to today's date key — it is not a separate list of "today's
  tasks" that has to be kept in sync; it's a second view onto
  `statusesByTask`, so a tap in either place is immediately visible in the
  other on next render. The card's tap is simpler than the grid's
  three-way cycle, though: it only toggles between `"done"` and cleared
  (back to whatever the derived default is), it doesn't cycle through
  `"not_done"` — marking a task "not done" for the day is still only done
  from the Input Tracker grid.

### Schedule block (Calendar)
- A **repeating weekly template**, not date-specific instances — one
  `dayOfWeek` + time range per block, applying to every week the same way.
  No exceptions, no specific-date overrides, no reminders — this is a
  reference schedule, not a real scheduling system (see `docs/PRODUCT.md`).
- `dayOfWeek` is stored `0`–`6` = Sun–Sat (JS `getDay()`), unchanged. The
  calendar *renders* Monday-first via `CAL_DAY_ORDER = [1,2,3,4,5,6,0]` —
  a display-order array only, no storage change. (`getWeekInfo` — used by
  the Input Tracker week view and the Dashboard "Week" filter — is also
  Monday-start now, so the whole app agrees on where a week begins.)
- **Editing is two popups, no drag.** The grid has one gesture: a plain
  click on a rendered block (`<button class="cal-block">`) sets
  `editingBlockId` and opens `renderBlockEditModal` — name, color, day
  (single-select, Mon-first), start/end time, Delete, Done. Every field
  autosaves via `updateBlockField` as it changes (Done just closes).
  Creating is separate: a "＋ Add block" button at the top of the view
  sets `creatingBlock` and opens `renderBlockCreateModal` — name, color,
  days (multi-select), start/end time. Its draft lives only in that
  render call's closure until "＋ Add block" is pressed; picking N days
  calls `addScheduleBlock` N times, producing N independent
  single-`dayOfWeek` blocks identical apart from `dayOfWeek`/`id` (there
  is no multi-day block — it's purely a create-time convenience). See
  Post-launch revisions in `docs/ROADMAP.md` for the history: drag/draw
  and edit-in-place were tried and dropped for cross-browser
  unreliability, then a below-grid form + list, now the click-to-popup.
- Time is picked with `renderTimeControls` — a **Start** row and an **End**
  row, each an hour `<select>` (limited to the calendar's visible hours,
  6–23) plus a minute `<select>` (`:00 / :15 / :30 / :45`) — plus a row of
  **duration chips** (30 min / 1h / 1h 30m / 2h) that set End = Start + N.
  No `<input type="time">`, and no scrolling all-day megadropdown. Values
  are still `"HH:MM"` 24h; an existing off-grid hour or minute is kept as
  its own extra option. Ordering is enforced inside the control (moving
  Start past End bumps End to Start + 1h; setting End at/before Start snaps
  it to Start + 15m), so `onChange(start, end)` always hands the caller a
  valid pair — the create popup does no further time validation.
- `label`: free text, the block's only name. Not linked to a task or
  category (an earlier version could optionally point at a task via a
  `taskId` field and derive its label/color from there; that link was
  removed entirely — every block is now independently named and colored,
  with nothing to keep in sync when a task is renamed or archived).
- `color`: optional hex color, or `null`. With no task link to fall back
  on, a block with `color: null` just renders in the default accent color
  (`--floor`) rather than inheriting anything.
- Overlap between blocks is allowed and unvalidated — neither popup checks
  a new or changed time range against any other block.
- "Clear calendar" (a button below the grid, shown only when at least one
  block exists) sets `scheduleBlocks` to `[]` in one action, after a
  native `confirm()` — the only bulk/destructive action on this screen.

### Goal
- `id`: stable, generated once (`gol_` + random base36).
- `name`: short free text, an outcome (e.g. "Save 100K DH").
- `horizon`: one of `"short" | "mid" | "long"` — which of the three
  sections (Short-term / Mid-term / Long-term, displayed in that order —
  nearest-term first) the goal is grouped under. Chosen in the New goal
  popup (defaults to `"short"`) and editable any time via the Horizon
  field in the Edit popup.
- `deadline`: local date key `YYYY-MM-DD`, or `null` (default, most goals
  won't set one). Purely informational — there is no reminder, notification,
  or scheduling behavior tied to it anywhere in the app (see
  `docs/PRODUCT.md`, Explicit non-goals). Its only computed effect is
  `isGoalOverdue(goal[, pct])`: true when `deadline` is in the past *and*
  the goal's **live** progress (`goalEffectiveProgress`, below) is `< 100`.
  Rendered as a small badge next to the date field; not stored anywhere,
  recomputed on every render (and re-checked in the Edit popup after any
  change that could move progress — slider, linked-task toggle, target).
- `percent`: integer 0–100 — the **manual fallback** progress. It is the
  displayed progress *only* when the goal is not linked (see below); once a
  goal is linked its `percent` is left untouched and ignored, so unlinking
  restores whatever it last was. Moved on a slider in the Edit popup.
- `parentGoalId`: id of another goal one horizon up (`short → mid`,
  `mid → long`; `long` has no parent), or `null`. Set via the "Ladders up
  to" picker in the create/edit popups. Its effect is on the **parent**:
  a mid/long goal's live progress is the average of the goals that ladder
  into it. (This field had been removed in an earlier pass and is now
  back — see below.)
- `linkedTaskIds` / `linkType` / `countTarget` — **short-term goals only**,
  the Input Tracker link:
  - `linkType`: `null` → manual (`percent`); `"rate"` → live progress is
    the average of each linked task's accomplishment `%`
    (`computeTaskAccomplishment(...).pct`, or `0` for a task with no
    applicable days yet); `"count"` → live progress is
    `min(100, round(Σ all-time "done" entries across linked tasks /
    countTarget * 100))`.
  - `linkedTaskIds`: task ids; archived tasks are ignored. Empty (or
    `countTarget` unset for `"count"`) makes the goal fall back to
    `percent` even with a `linkType` set — linking is never forced.
- `order`, `archived`/`archivedAt`: same pattern as Category/Task, scoped
  to siblings within the same `horizon`. An archived goal drops off the
  visible list but isn't deleted.

**`goalEffectiveProgress(goal)` — derived, never stored.** Returns
`{ pct, source }` where `source` is `"linked"` (short-term via
tasks), `"rollup"` (mid/long via child goals) or `"manual"` (`percent`).
mid/long roll-up recurses (`long` averages `mid` which averages `short`);
a hand-edited `parentGoalId` cycle is broken by a `seen` guard. Every goal
card and the overdue check read this, so a linked goal reflects current
Input Tracker / child-goal state on the next render with no manual
refresh. Nothing writes the computed value back into `percent`.

**`progressMode` / `status` removed (unchanged from before).** A
status-mode goal's `status` is folded onto `percent` (`done → 100`,
`in_progress → 50`, else `0`) by `loadState()`, then both fields deleted.
**`parentGoalId` reinstated:** it had been dropped when goal-to-goal
linking was removed; the roll-up feature needs it, so `loadState()` now
defaults it (and `linkedTaskIds: []`, `linkType: null`,
`countTarget: null`) instead of deleting it.

Each horizon has a fixed accent color (short/mid/long), set once as
`--hz-color` inline on the `.horizon-section` container and inherited by
every `.goal-card` inside it via ordinary CSS custom-property inheritance —
no per-card color is stored or computed in JS. Both the section background
and each card's background/left-border are `color-mix()` tints of that one
color, at different mix percentages so the card reads as a level "above"
its section rather than blending into it.

**Creation and editing are both popups.** One "＋ New goal" button at the
top of the Goals screen (`renderGoals`, next to the heading) sets the
module flag `creatingGoal = true`; `render()` then appends
`renderGoalCreateModal()`. Its body rebuilds on horizon change: always
Name / Horizon / Deadline, a "Ladders up to" parent `<select>` for
short/mid, and for a short-term goal a `Manual | Rate-based | Count-based`
selector plus (when rate/count) the task checklist and a target-count
field. Draft state lives only in that render call's closure until
"＋ Add goal" is pressed (`addGoal(s, draftHorizon, {name, deadline,
parentGoalId, linkType, linkedTaskIds, countTarget})`); Cancel / ✕ /
backdrop / Escape just clear `creatingGoal`.

`editingGoalId` holds the goal whose Edit popup is open — set by clicking
anywhere on that goal's card (`renderGoalCard` is itself a `<button>`;
there is no separate "✎ Edit" affordance). `render()` appends
`renderGoalEditModal(goal)` for it. It holds Name, Horizon, Deadline,
"Ladders up to", and a **Progress** area that depends on the goal:
- short-term: a `Manual | Rate-based | Count-based` row. Manual shows the
  `percent` slider; rate/count show `renderTaskLinkPicker` (a scrollable
  "Category · Task" checklist) plus, for count, a target field, and a
  read-only live-progress read-out.
- mid/long: if any goals ladder into it, a read-only roll-up read-out
  (with each child's name + %); otherwise the manual `percent` slider.

Every field calls `updateGoalField` immediately (no draft). A `linkType`
change (or horizon change) rebuilds the Progress area's controls; a
target/task/slider change only refreshes the read-out and the Overdue
badge in place — the "update the DOM directly, skip a full `render()`"
pattern. Horizon change does call `render()` so the whole popup rebuilds
for the new horizon's field set.

### Identity
`identityStatement` is a single free-text string at the top level of the
state object (not its own array of records) — one continuously-edited
block, not a list of separately added/removed entries. No status, no
progress, no archiving — it isn't a thing that gets "done."

An earlier version of this phase modeled identity as a list of short
statements (`identityStatements: [{id, text, order, createdAt}]`, each
`idn_`-prefixed). On first load after the change, `loadState()` folds any
existing entries into the new single field (joined with a blank line
between each, in their original order) and removes the old array — a
one-time, automatic migration; nothing the user has to do or notice.

## Accomplishment %, streaks, and completion scores

Shared building block — **is date `D` "applicable" for task `T`?**
1. `D` must be on/after `T.createdAt` (by calendar date) and, if `T` is
   archived, before `T.archivedAt`.
2. `D`'s weekday must be in `T.scheduledDays`.
3. `D` must not be in the future (after today).

If all three hold, `D` counts toward that task's applicable-days
denominator. Every scheduled day is treated identically here — there is no
day-of-week that gets excluded from this math (an earlier version had a
weekly "flex day" that did; see Removed: flex day, below).

### Per-task week % (Input Tracker "Week %" column)
`computeTaskWeekAccomplishment(s, task, weekDates, todayKey)`:
`(done days in the displayed week) / (days the task is scheduled in that
week)`. The denominator is **every day of the whole Mon–Sun week whose
weekday is in `scheduledDays`** — days already past, today, and weekdays
still to come alike. It is *not* clipped to today and there is *no*
per-day `createdAt` / `archivedAt` gate: within a week the task is active
for, all of that week's scheduled slots count (the same cells the grid
lets you tick), so one day ticked on a 5-day-a-week habit reads 1 / 5 =
20%, never 100%. N/A days (weekday not in `scheduledDays`) never count.
Returns `null` (rendered "—") only when the task is scheduled on no day of
that week, or the **entire** week lies before `createdAt` / on-or-after
`archivedAt`. Recomputed live per displayed week; navigating weeks re-runs
it for the new span.

An older **all-time** rolling accomplishment % —
`computeTaskAccomplishment(s, task, todayKey)`, `done / applicable` from
`createdAt` through today, `null` until a scheduled day has passed — is
still computed, but only for rate-linked goals (see "Short-term goal
progress" above). It is no longer shown in the tracker.

### Per-category accomplishment %
Same **pooled** ratio (a task with more applicable days weighs more), now
scoped to the **Dashboard time filter's range** rather than all-time —
`computeRangeStats` (below) produces it in the same pass as everything
else on the Dashboard. Shown on the per-category KPI card.

### Dashboard time filter + range stats
The Dashboard has a filter — **Today / Week / Month / Year / Custom** —
that scopes every Input-Tracker-derived element on it (headline %, both
charts, the non-negotiable/other card, the KPI grid). It's per-device UI
state in its own key, **not** in `momentum:v2`:
- `momentum:dashFilter` → `{ mode, start, end }`. `mode` is one of the
  five; `start`/`end` are `YYYY-MM-DD` and only used for `"custom"`.
  Defaults to `{ mode: "today" }`. `dashRange()` resolves it to
  `{ startKey, endKey, label }` — fixed modes cap `endKey` at today;
  `"custom"` is used as picked (swapped if reversed).

`computeRangeStats(s, startKey, endKey, todayKey)` walks every task-day in
the range once (clamped to today, tasks of archived categories skipped)
and bins each into one of four **entry buckets** via `taskDayBucket`:
- `done` / `not_done` — the stored status for a scheduled day.
- `todo` — a scheduled day with nothing stored yet.
- `na` — the task existed that day but that weekday isn't in
  `scheduledDays`. Neutral: **excluded** from `applicable` and from every
  completion %.
A day before the task's `createdAt`, on/after its `archivedAt`, or in the
future is **not an entry at all** (not even `na`). It returns per-range
totals, `byCat[]` / `byCatId{}`, a `byGroup` split (non-negotiable vs
other) and `byTaskId{}`. `applicable = done + not_done + todo`; the
headline / KPI / bar **% = done / applicable** (NA and unlogged-but-due
days count against you). The **donut** ("Done vs not done — overall") now
uses that *same* denominator: `Done = done`, `Not done = applicable −
done` (i.e. `not_done + todo` folded together), `total = applicable`. So
with nothing done it is 100% Not Done and fills toward Done as entries are
checked off — it agrees with the headline % by construction. `na` is still
never shown; there is no separate `todo` slice.

### Per-category streak
The **current** streak — consecutive **applicable-and-relevant** days,
counting backward from today, on which every one of that category's active
tasks that was applicable that day was Done. This is the one KPI element
the Dashboard time filter does **not** rescope: a "streak within an
arbitrary window" isn't a meaningful number, so it always reads back from
today. Rules:
- A day where the category has zero tasks applicable (e.g. all its tasks
  are Mon–Fri and today is Saturday) is **transparent** — skipped over,
  neither extending nor breaking the streak.
- A day where at least one applicable task in the category was not Done
  breaks the streak (streak resets to 0 as of that day).
- Today is excluded from the count until every applicable task in the
  category has been explicitly marked (mirrors the first design's "pending
  today" rule) — an incomplete today never shows as a break in progress
  before the day is over.

### Completion score (replaces the old fixed Weekly / Overall tiles)
There is now a **single** headline completion % on the Dashboard —
`computeRangeStats(...).pct`, i.e. pooled `done / applicable` over the
selected filter range. The old always-on "This week" and "Overall" tiles
(and `computeWeeklyScore` / `computeOverallScore` / `computeScoreOverRange`
/ `computeCategoryPct`) were removed; pick "Week" or "Year" / a wide
custom range for those views. Weeks, where the filter uses them, are
**Monday-start** (`getWeekInfo`), consistent with the Input Tracker and
the Calendar.

### Dashboard task card
`renderDashTaskCard` renders the Non-negotiable / Other split. On the
`today` filter, each group lists **only the tasks scheduled today** — the
card pre-filters `todaysTasksSplit`'s rosters with `isTaskApplicableOn`,
and each row is a one-tap checkbox. Tasks not scheduled today are left off
(no greyed "not today" rows). On any wider range, `todaysTasksSplit`'s
unfiltered rosters are used: each group shows its pooled % + bar plus a
per-task list (`renderRangeTaskList`, reading `stats.byTaskId` from
`computeRangeStats`) covering every task that applied during the range.

### Removed: flex day
This categorized-tracker design originally carried over the first design's
weekly "flex day" — `settings.flexDayOfWeek`, a single day exempted from
every calculation above, with a rendering tint marking it distinct
everywhere it appeared (Input Tracker's toolbar and day columns, the
Calendar's day header). It has since been removed entirely — not hidden
from one screen while still running in the background on others, the
setting and every check against it are gone from the codebase. Every
scheduled day is now tracked identically, everywhere in the app.

The reasoning for the removal (documented here since it reverses an
earlier explicit design principle, not because the mechanics matter once
gone): a global setting that exempted one day from Dashboard/Calendar
calculations couldn't be cleanly hidden from just Input Tracker's display
without either leaving a confusing invisible exemption running elsewhere
with no visible control, or removing it everywhere — and "every scheduled
day is tracked the same way," which is what was asked for, only holds
together as one consistent rule if it's true throughout the app, not just
on one screen.

`loadState()` deletes `settings.flexDayOfWeek` from any older saved state
on load, the same ad-hoc-migration pattern used elsewhere — see Schema
versioning, below.

## Export / import

**Export**: serialize the entire top-level object as-is to a downloaded
`.json` file, filename `momentum-export-YYYY-MM-DD.json`. No transformation,
no partial export — full state, since the whole point is a portable backup
and a way to move to a new device.

**Import**: user selects a `.json` file; the app validates `schemaVersion`
is a known/migratable value and the top-level shape looks right
(`categories`/`tasks`/`statusesByTask`/`settings` present with the expected
types), then replaces the entire in-memory state and `localStorage`
contents with the imported object. Import is destructive to current local
data by design (a restore operation, not a merge) — the UI must confirm
before overwriting.

## Schema versioning

`schemaVersion` (inside the `momentum:v2` object) is reserved for a shape
change large enough that old code genuinely can't interpret the new one
(or vice versa) without an explicit, ordered migration step — bumped
whenever that happens, with `loadState()` running the matching migration
function before the app touches the data further, and refusing to load
data with a *newer* `schemaVersion` than the running code knows about
(e.g. an import from a future version) rather than silently corrupting it.

A field being renamed, restructured, or dropped in a way that's still
detectable and safely fixable by field presence alone — like the
`identityStatements` list becoming a single `identityStatement` string, a
goal missing `horizon`/`percent`/`deadline` or still carrying
`parentGoalId`/`progressMode`/`status` (folded onto `percent`, then
deleted), a task missing `nonNegotiable` (defaulted to `false`) or still
carrying a leftover `floor`/`target` (deleted), a schedule block still carrying `taskId`
(resolved into a plain `label`/`color` first, so a previously task-linked
block keeps its name instead of going blank, then deleted), or
`settings.flexDayOfWeek` simply being deleted — is handled as an ad-hoc,
tolerant fix-up directly in `loadState()` instead: check for the old
shape, migrate it in place, move
on. Not every shape change needs a version bump and a formal migration
entry; that machinery is for changes too structural to infer safely from
what's present. (The `momentum:v1` →
`momentum:v2` storage *key* change is a third category — no migration at
all, since the two shapes are unrelated designs; see the top of this
document.)

## ID generation

`<prefix>_` + a short random base36 string (e.g. `cat_k3j9f2`, `tsk_p8m2q1`),
generated client-side with `crypto.getRandomValues` or `Math.random`
fallback. Prefixes: `cat_` categories, `tsk_` tasks, `blk_` schedule blocks,
`gol_` goals. (Identity has no `id` — it's a single field, not a list of
records; see Identity, above.) No uniqueness registry needed beyond
"collision is astronomically unlikely for this volume of records."
