# Momentum — Roadmap

Phased so each phase is independently useful. No phase depends on
speculative future work — if development stopped after Phase 1, the app
would still be a complete, usable daily tracker.

This is the roadmap for the second design of Momentum (categories, tasks,
Dashboard, Calendar, Goals/identity) — see `CLAUDE.md` and
`docs/PRODUCT.md` for how this relates to the first design.

## Phase 0 — Architecture ✅ done

- `CLAUDE.md`, `README.md`, `docs/PRODUCT.md`, `docs/DATA-MODEL.md`,
  `docs/ROADMAP.md`.

All four phases below were built in one pass rather than incrementally —
the sidebar shipped with all four real sections from the start (no
placeholder stubs), since Phases 1–4 were approved together.

## Phase 1 — Sidebar shell + Input Tracker ✅ done

- Persistent left sidebar (collapsible/narrow on smaller screens, reusing
  the first design's dashboard shell pattern) with all four sections:
  **Dashboard, Calendar, Input Tracker, Goals**.
- Input Tracker: category management (add/rename/recolor/archive, inline —
  no separate settings screen) and task management within each category
  (add/rename/edit scheduled days/archive — floor/target were part of the
  initial build here too, later removed entirely, see Post-launch
  revisions below). Reordering categories/tasks was scoped out, same call
  as the first design's anchors — order is set at creation, not currently
  editable, not a real need yet at this scale.
- Categories render as color-coded group headers (a fixed 8-color palette,
  auto-assigned on creation, overridable via swatches or a native color
  picker).
- Weekly grid layout (Monday–Sunday columns; originally Sunday-start,
  flipped to match the Calendar — see Post-launch revisions), with a week
  navigator clamped so you can't log a future week. Day columns are white,
  separated by thin 1px rules; today's column is marked by green header
  text only. Tapping a day cell cycles To-Do → Done → Not Done → To-Do;
  N/A cells (outside a task's schedule) and future-day cells aren't
  clickable.
- Per-task % shown on every row (a "Week %" column: done ÷ scheduled days
  for the displayed week — see Post-launch revisions; originally an
  all-time rolling %). The grid shows only a task's name and its per-day
  status.
- Each task can be flagged non-negotiable in its edit strip, driving which
  group it falls into on the Dashboard's Today card (added post-launch,
  see below) — no effect on tracking logic itself.
- Persistence to `localStorage` under `momentum:v2`, per
  `docs/DATA-MODEL.md`.

## Phase 2 — Dashboard ✅ done

- A Today card up top splits today's applicable tasks into Non-negotiable
  and Other, each one tap from done (struck through once done), reading
  and writing the same `statusesByTask` data Input Tracker uses — added
  post-launch, see below.
- Per-category KPI cards: pooled accomplishment % and current streak,
  color-dot matched to the category — all computed live from Input
  Tracker data, nothing entered directly on this part of the screen.
- A dashboard-wide **time filter** (Today / Week / Month / Year / Custom),
  a completion headline, a **Done vs Not Done donut** and a **per-category
  bar chart** — all added post-launch, see below. The original fixed
  "This week" / "Overall" stat tiles were removed then.

## Phase 3 — Calendar ✅ done

- Weekly grid (7 day columns × a 6am–11pm time gutter) rendering
  `scheduleBlocks` as absolutely-positioned colored blocks. Originally
  fully read-only after several direct-manipulation versions; a later
  revision made a **plain click on a block** open an edit popup (still no
  drag/draw/resize). Week displays Monday-first. See Post-launch revisions.
- Block **creation and editing are popups** — a "＋ Add block" button at
  the top of the view, and click-a-block to edit. Both cover name, color,
  day(s), and start/end time (hour + minute dropdowns with 30m/1h/1h30/2h
  duration shortcut chips, not the native time input); the edit popup adds
  Delete. (Originally a below-grid "Add a block" form plus a separate
  Blocks list — see Post-launch revisions.)
- A "Clear calendar" action below the grid (shown only once a block
  exists) removes every block in one confirmed action.
- No scheduling logic (conflict detection, specific dates, reminders,
  external calendar sync) — confirmed not attempted, per the Phase 3 scope.

## Phase 4 — Goals + Identity ✅ done

- Identity: one always-visible free-text block at the top of the tab
  ("who am I becoming") — revised from an initial editable-list design to
  a single continuously-edited block, since that fit the concept better;
  old entries migrate into it automatically, see `docs/DATA-MODEL.md`.
- Goals grouped into three visually distinct horizon sections, displayed
  nearest-term first — Short-term, Mid-term, Long-term (revised from an
  initial long-to-short order; see Post-launch revisions, below) — each
  with its own accent color. Creation is a single "＋ New goal" button at
  the top of the screen opening a popup (name, horizon, deadline); the
  chosen horizon decides the section (see Post-launch revisions).
- Each goal card shows just its name and a percent progress bar; an
  "✎ Edit" popup holds the editable fields — Name, Horizon (reassignable
  any time), Deadline, and Progress (a 0–100 percentage slider) — plus
  Archive. (Goal-to-goal laddering and a not-started/in-progress/done
  status mode were both part of the initial build and later removed — see
  Post-launch revisions.)
- Sidebar icons for all four tabs (not just Goals) switched from colored
  emoji to minimalist single-color SVG outline icons using `currentColor`,
  so they follow the nav item's normal/hover/active state and the
  light/dark theme automatically — done for all four rather than leaving
  three emoji next to one outline icon.
- Neither identity nor goals has a daily tap, streak, or accomplishment %
  — confirmed this stayed a periodic reference/update screen, not a
  second tracker.

## Post-launch revisions

A cross-cutting revision pass after all four phases first shipped, touching
Input Tracker, Calendar, Goals, and Dashboard together rather than a fifth
phase's worth of new features:

- **Flex day removed entirely** — not just its picker/tint hidden from
  Input Tracker, but `settings.flexDayOfWeek` and every check against it
  deleted app-wide (Dashboard's streak/score math, the Calendar's day
  tint). This reverses a principle stated as foundational when this app's
  redesign began; see `docs/DATA-MODEL.md`, "Removed: flex day," for why
  a tab-scoped removal wasn't actually possible to do cleanly.
- **Floor/target removed entirely**, in two steps: first the grid's
  floor/target subtitle line was dropped (fields stayed, just unshown in
  Input Tracker's daily grid), then — once it was clear that wasn't the
  whole ask — the `floor`/`target` fields themselves were dropped from the
  task edit strip and the schema. A task now has just a name, schedule,
  and non-negotiable flag; there's no effort-level description left to
  set at all. See `docs/DATA-MODEL.md`, "Removed: `floor` / `target`."
- **Dashboard set as the home screen** — the app now opens there by
  default (was Input Tracker) instead of requiring a tab switch to see
  the Today card and KPIs first.
- **Calendar rebuilt for direct manipulation** — replacing the add-form +
  block-list with click/drag editing on the grid itself.
- **Goals reordered and restructured** — horizons now display short-to-
  long, each goal is an explicit-fields card (including a now-editable
  Horizon field), and identity is one free-text block rather than a list
  (the identity change happened within Phase 4's own build, before this
  revision pass, but is grouped here as the same kind of "field displayed
  differently than first built" change).
- **Dashboard gained a Today card** and tasks gained a `nonNegotiable`
  flag to support it.
- **Goals gained a `deadline` field** — an optional date shown on every
  card between Horizon and Linked parent goal, with a small "Overdue"
  badge once it's past and the goal isn't finished. No reminders, no
  notifications — purely a visual flag, cleared the moment the goal is
  marked done (or hits 100% in percent mode) or the date moves.
- **Every `<select>` in the app restyled**: the OS-native dropdown arrow
  and its inconsistent padding are replaced with a custom chevron
  (`appearance: none` + an inline SVG background image) so every dropdown
  — Calendar's day/task pickers, Goals' horizon/parent/status pickers —
  looks the same and has consistent, comfortable padding regardless of
  browser. `<input type="date">` (new, for deadline) was folded into the
  same base text-input styling as the existing text/time fields.
- **Calendar's "create" interaction changed from click-to-open-a-form to
  drag-to-draw**: dragging on empty space now paints a live dashed
  "ghost" block and creates a real block matching exactly the range drawn
  on release, with a placeholder name and no task linked — no form
  involved at all for defining *when*. A plain tap still creates a
  default one-hour block the same no-form way.
- **Calendar's edit interaction changed from a below-grid panel to
  editing the block itself, in place** — clicking a block no longer opens
  a separate `.block-editor` panel under the grid; the block expands
  where it already is into a compact card (day, task-or-label (+color for
  freeform), time, Delete/Done) and collapses back on Done. Every field
  autosaves immediately, same as the rest of the app, rather than a
  Save/Cancel draft — there's nothing left to "cancel" since nothing
  outside the block itself ever changes. `calBlockDraft` was removed
  entirely in favor of a single `editingBlockId`.
- **Calendar's inline editing card simplified further, and task-linking
  removed** — the card that had grown to hold a day picker, task/freeform
  select, label, color, and start/end time was cut down to exactly three
  things: name, color, delete. Day and time are still changeable, just not
  from this card (drag the block itself); moving a block to a different
  day has no dedicated control anymore — delete and redraw it, which is
  cheap now that drawing is a single drag. The ability for a block to link
  to a task (inheriting its name and category color) was removed
  entirely, not just hidden — every block is independently named and
  colored now, with nothing to go stale if a task is later renamed or
  archived. Existing task-linked blocks migrate automatically: the linked
  task's current name (and its category's color, if the block had none of
  its own) is captured into the block's own `label`/`color` before the
  link is dropped, so nothing goes blank.
- **A "Clear calendar" action** was added below the grid (visible only
  once at least one block exists) — a native `confirm()` then
  `scheduleBlocks = []`, the only bulk/destructive action in the app.
- **Calendar drag rebuilt on plain mouse/touch events instead of the
  Pointer Events API**, after a report that a block couldn't be clicked to
  edit once created, specifically on Safari with a mouse. The first fix
  attempt (loosening the click-vs-drag movement threshold from 4px to 6px,
  and handling `pointercancel`) was reasonable hardening but didn't
  resolve it. `attachBlockDrag` and `attachDayBodyDraw` no longer use
  `pointerdown`/`pointermove`/`pointerup`/`pointercancel` or
  `setPointerCapture` at all — they use `mousedown`/`touchstart` on the
  element and `mousemove`/`mouseup`/`touchmove`/`touchend`/`touchcancel`
  on `document` (the classic pattern predating Pointer Events, and the
  reason it's used here: relying on `document` for move/up removes any
  dependency on pointer capture, whose Safari support has historically
  been less consistent than the decades-old Mouse Events API). Confirmed
  with `jsdom`'s real `MouseEvent` implementation (unlike `PointerEvent`,
  which `jsdom` doesn't implement at all, so the first fix attempt could
  only be reasoned about, never actually exercised) — dispatching
  `mouseup` on `document` rather than the original element, matching how
  a real browser delivers it, and specifically reproducing the reported
  click-a-freshly-drawn-block scenario.
- **An "Add a block" form added below the grid**, after the mouse-events
  rewrite still didn't resolve the report on Safari — rather than attempt
  a fourth fix to the drag gesture itself, a fully independent creation
  path was added that doesn't depend on drag detection working at all:
  name, color (defaulted, optional to change), a day picker, and
  start/end time inputs, submitted with one clearly primary button
  (`renderCreateBlockSection`). This sits alongside drawing rather than
  replacing it — both call the same `addScheduleBlock`, so a block made
  either way is identical and equally editable afterward.
- **Direct manipulation removed from the calendar grid entirely** —
  drawing-to-create, drag-to-move, drag-to-resize, and click-to-edit-
  in-place are all gone; `attachBlockDrag` and `attachDayBodyDraw` were
  deleted along with the CSS/state that only existed to support them
  (`.cal-block-editing`, `.cal-block-resize`, `.cal-block-ghost`). The
  grid is now purely a read-only visual summary. Editing an existing
  block moved into a new **Blocks list** below the create form, using the
  same click-to-expand-a-strip pattern as categories/tasks in Input
  Tracker (`renderBlockListSection` / `renderBlockEditStrip`,
  `editingBlockId` repurposed to track the open list row instead of an
  in-grid card) — after three rounds of drag-reliability fixes on Safari,
  this trades away the "draw it directly" feel for something built
  entirely out of ordinary, unglamorous form controls, which is what was
  asked for once it became clear direct manipulation wasn't going to be
  reliable enough on every setup.
- **The active sidebar tab now persists across a reload** — previously a
  plain in-memory variable defaulting to Dashboard on every load, now
  backed by `localStorage` (`momentum:activeTab`, kept outside
  `momentum:v2` since it's per-device UI state, not product data — see
  `docs/DATA-MODEL.md`). Refreshing the page reopens on whichever tab was
  last active instead of always jumping back to Dashboard.
- **Goal cards and their horizon sections restyled as actual cards** — the
  Short/Mid/Long-term sections and each goal inside them previously used
  only a thin colored border to signal horizon; both now carry a light
  `color-mix()` tint of that same horizon color as a background, with the
  goal card also keeping a colored left border, so the horizon color reads
  clearly without needing a darker/louder palette. Pure CSS — the color is
  set once as `--hz-color` on the section and inherited down to its cards.
- **A manual light/dark theme toggle added**, in a new sidebar footer
  button below the four nav sections. Previously the app only ever
  followed the OS's `prefers-color-scheme`; it now cycles
  auto → light → dark → auto on each click, persisting the choice to
  `momentum:theme` (outside `momentum:v2`, per-device UI state — see
  `docs/DATA-MODEL.md`, "Theme") so an explicit choice survives a reload
  and overrides the OS setting until changed back to auto.
- **Goal creation and editing split apart, mirroring the Calendar
  pattern.** Every goal field used to be always-visible and directly
  editable on its card. Now a goal is created through a small submitted
  form at the bottom of its horizon section (name required, deadline and
  parent link optional), and an existing goal's card shows only its name
  and a progress bar by default — the bar visualizes whichever progress
  representation the goal uses (percent directly, or a fixed
  not-started/in-progress/done → 0/50/100 mapping in status mode), plus
  the Overdue badge when applicable. Horizon, Deadline, Linked parent
  goal, and the Progress controls themselves all moved into an "✎ Edit"
  strip (`editingGoalId`, same click-to-expand idiom as Calendar's Blocks
  list and Input Tracker's categories/tasks) that also holds the Archive
  action, replacing the quick "✕" that used to sit directly on the card.
- **Calendar's "Add a block" form takes multiple days at once** — its day
  picker went from single-select (`draftDay`) to multi-select
  (`draftDays`, an array). Submitting with several days selected calls
  `addScheduleBlock` once per day, creating that many identical
  single-`dayOfWeek` blocks in one action. No schema change — there's
  still no multi-day block; this is a create-time shortcut for the common
  "same thing every weekday" case, replacing repeating the form per day.
- **Goal creation moved from an always-open form into a popup** — the
  per-horizon "＋ Add … goal" form at the foot of each section is now just
  a button; pressing it opens a small modal (`renderGoalModal`, gated by
  `creatingGoalHorizon`) with the same fields (name required, deadline and
  parent link optional). The app's first modal — everything else still
  uses inline strips and native `alert`/`confirm` — dismissible by Cancel,
  a ✕, a backdrop click, or Escape. Editing an existing goal is unchanged
  (still the in-card "✎ Edit" strip).
- **Goals simplified to one percent, one create button, popup editing.**
  Three changes together: (1) `parentGoalId` and all goal-to-goal
  laddering removed — the Linked-parent select, the "N goals ladder up to
  this" note, and `PARENT_HORIZON` are gone; (2) the `status` /
  `progressMode` progress-mode split removed — every goal is just a
  `percent` (0–100) on a slider, and `loadState()` folds an old
  status-mode goal's state onto that scale before deleting the dead
  fields; (3) the per-horizon create buttons replaced by one "＋ New goal"
  button at the top of the Goals screen opening `renderGoalCreateModal`
  (`creatingGoal` flag) — the popup now includes a **Horizon** select
  (default Short-term) that decides which section the goal lands in.
  Editing also became a popup (`renderGoalEditModal`, keyed by the
  existing `editingGoalId`) instead of an in-card strip; it autosaves
  every field, patching the Overdue badge in place without a full
  `render()`. `renderGoalEditStrip`, `renderGoalModal`, `STATUS_LABELS`,
  `goalProgressPercent`, and the `.goal-edit-strip` / progress-mode CSS
  were all deleted.
- **A logo mark next to the "Momentum" wordmark** in the sidebar header —
  an inline-SVG upward trend line + arrowhead (`brandMark()`) reversed out
  white on a small accent-colored rounded badge (`.brand-mark`, a CSS
  `linear-gradient` of `--target`, no SVG gradient/defs). Hidden with the
  wordmark when the sidebar is collapsed. Purely cosmetic; no new assets
  (still one self-contained file, still offline).
- **Calendar reworked: Monday-first, popup block editing, hour+minute time
  picker with duration chips.** Three changes: (1) the grid renders Mon→Sun
  (`CAL_DAY_ORDER = [1,2,3,4,5,6,0]`, display-order only — `dayOfWeek`
  storage stays 0=Sun, and Input Tracker / Dashboard weeks still start
  Sunday); (2) the below-grid "Add a block" form and Blocks list were
  removed — a `<button class="cal-block">` on the grid opens
  `renderBlockEditModal` on a plain click, and a "＋ Add block" button at
  the top of the view opens `renderBlockCreateModal`; both reuse the
  `modal-*` shell, `editingBlockId` / new `creatingBlock` flag gate them.
  `renderCreateBlockSection`, `renderBlockListSection`,
  `renderBlockEditStrip` and their `.cal-create-*` / `.block-*` CSS were
  deleted; (3) time is a `renderTimeControls` component — Start/End rows of
  an hour `<select>` (6–23 only) + a minute `<select>` (:00/:15/:30/:45),
  with duration chips (30m/1h/1h30/2h) below that set End = Start + N — no
  `<input type="time">` and no long slot list. Still stores `"HH:MM"`,
  keeps an off-grid existing value as its own option, and enforces
  End > Start internally so callers get a valid pair. (An interim version
  used a single 15-min-slot `<select>` / `buildTimeSelect`, replaced here
  for being too long a list.) Shared bits factored out: `.view-header` /
  `.header-add-btn` (was `.goals-header` / `.goals-new-btn`),
  `.modal-primary` (was `.goal-add-btn`), `modalField` (was
  `goalModalField`), `modalOverlay`.
- **Global interaction polish.** (1) `button:active { transform: scale(0.96) }`
  plus a shared `transition` on every `button` — subtle press feedback
  everywhere. (2) Modals animate in (backdrop fade + dialog fade/slide,
  `@keyframes overlayIn` / `dialogIn`, ~150–180ms) and out — every modal
  close now routes through `closeModal(commit)`, which adds `.is-closing`
  (reversed animation) and defers the state flip + `render()` until
  `animationend` (220ms fallback). (3) The Input Tracker edit strips fade
  in (`@keyframes stripIn` on `.tt-edit-form`); to stop that replaying,
  the in-strip controls (day picker, colour swatches, non-negotiable
  toggle) and the day-status grid cells now update **in place** instead of
  calling `render()` on every tap — `renderTaskRow` repaints just the
  tapped cell + that row's % cell, `renderDayPicker` toggles the clicked
  button, etc. `@media (prefers-reduced-motion: reduce)` neutralises all
  of it. No data-model change.
- **Input Tracker — clearer category separation.** A blank page-colour
  band (`.tt-cat-gap` row, `renderTracker` inserts one before each
  category after the first) plus a heavier category header (accent-colour
  top border, stronger tint, larger name). Visual only — no data or task
  fields touched. *Later tweak:* cell padding bumped to `10px 12px`, the
  gap band widened to 30px, and every divider line around it removed — the
  category header's accent `border-top`, the add-row's `border-bottom`,
  and the gap row's own borders — so category blocks sit in open space
  with no line running between one and the next; the tinted header band
  alone carries the grouping. The `.tracker-wrap`'s hard 1px outer
  `border` was then also dropped (in dark mode it stood out sharply where
  it crossed the darker gap band, reading as a vertical line joining the
  blocks) and replaced with a soft `box-shadow` (faint 1px dark rim +
  blurred drop) so the whole table reads as a floating card.
- **Goals horizons laid out as columns, not stacked rows.** The three
  `renderHorizonSection` blocks are wrapped in a `.horizon-columns` flex
  row (`renderGoals`); each `.horizon-section` is `flex: 1 1 240px`, wraps,
  then stacks below 780px, and all columns stretch to equal height.
  `.goal-grid` changed from an auto-fill grid to a single vertical `flex`
  column. Each column's title moved onto its own tinted, ruled-off strip
  (`.horizon-head` full-bleed with a `border-bottom`), the whole column
  block is ruled off from the identity block above it (`border-top` on
  `.horizon-columns`), and goal cards are now plain `--surface` with a
  soft shadow so they lift off the tinted column. The per-card "✎ Edit"
  button was dropped — `renderGoalCard` is now itself a `<button>`, so
  clicking anywhere on a card opens its Edit popup (`.goal-card-top`
  removed; card gets hover/active/focus-visible states). Layout/visual
  plus that one interaction change.
- **Goal progress can auto-track Input Tracker data.** New goal fields
  `parentGoalId` (reinstated — it had been removed), `linkedTaskIds`,
  `linkType` (`null` | `"rate"` | `"count"`), `countTarget`. `percent`
  is now the *manual fallback*, used only when a goal isn't linked. A new
  `goalEffectiveProgress(goal)` derives the shown number:
  short-term → average of linked tasks' accomplishment % (`rate`) or
  `Σ done entries / countTarget` (`count`); mid/long → average of the
  goals that ladder into them (recursive); else `percent`. Cards gained a
  `.goal-source` "Linked / Rolled up / Manual" tag; `isGoalOverdue` now
  takes live progress. The create popup rebuilds its body on horizon
  change to show the link fields; the edit popup's Progress area is a
  `Manual | Rate-based | Count-based` selector + `renderTaskLinkPicker`
  checklist (short-term) or a read-only roll-up read-out (mid/long).
  `loadState()` defaults the new fields and no longer strips
  `parentGoalId`. No stored/cached progress — always recomputed, so
  tracker edits flow to goals with no refresh. **This reverses the earlier
  "goals no longer link to each other" decision.**
- **Dashboard time filter + two charts.** A `Today / Week / Month / Year /
  Custom` filter (`dashFilter`, persisted to `momentum:dashFilter`,
  default Today) rescopes the whole Dashboard. New `computeRangeStats` does
  one pass over every task-day in the range, binning into `done /
  not_done / todo / na` entry buckets (`taskDayBucket`) with per-category
  and non-negotiable/other breakdowns. New charts: a hand-rolled inline-SVG
  **donut** (`<path>` arc slices, `renderDonut`) showing Done vs Not Done
  over the *logged* days (`done + not_done`; `todo` / `na` are not plotted)
  with a labelled legend, and a CSS-flex **stacked bar per category** (same
  pattern as the goal progress bars) — **no charting library** (the repo
  forbids deps). The non-negotiable/other card shows the tappable today
  list only when the filter is Today, per-group stats otherwise. Removed:
  the fixed "This week" / "Overall" tiles and `computeWeeklyScore` /
  `computeOverallScore` / `computeScoreOverRange` / `computeCategoryPct`
  (all superseded by `computeRangeStats`). Per-category **streak** is
  deliberately left range-independent. The filter is a single
  calendar-icon button (`.dash-filter-trigger`) that opens a small
  dropdown (`.dash-menu`, closed by a transparent full-screen backdrop) —
  the five options aren't all shown at once; Custom then reveals the two
  date inputs beside the trigger.
- **Dashboard task card: complete for wide ranges, today-only for Today.**
  Originally the Non-negotiable / Other card was filtered to tasks
  scheduled *today* on every range, so on a Week/Month/… filter individual
  tasks seemed to vanish. First fix: `todaysTasksSplit` returns all active
  tasks, the non-Today range card gained a per-task `%` list
  (`renderRangeTaskList` + `stats.byTaskId` from `computeRangeStats`), and
  `renderTodayTaskList` showed the not-due-today tasks greyed ("not
  today"). Follow-up (per request "if it's not today, don't put it on the
  dashboard"): the greyed rows were dropped — on the **Today** filter each
  group now lists only tasks scheduled today (`renderDashTaskCard`
  pre-filters with `isTaskApplicableOn`); `renderTodayTaskList` lost its
  "not today" branch and the `.today-task-off` / `.today-task-note` CSS.
  Wide ranges still list every task that applied during the range.
- **Input Tracker weeks start Monday, day columns kept plain.**
  `getWeekInfo` flipped from Sunday-start to Monday-start (so the Tracker
  week view + the Dashboard "Week" filter now match the Calendar). The
  earlier attempt at column legibility (weekend tint + accent frame down
  the today column) was pulled back to what was asked for: white cells
  with a thin 1px `border-left` rule between columns; today is just green
  header text. `tt-weekend` and the cell-level `tt-today` class are gone.
- **Empty `scheduledDays` fixed.** Deselecting every day in a task's day
  picker left `scheduledDays: []` — the task then had no applicable days
  anywhere, so its %, streak and dashboard numbers read "—" for good.
  `toggleTaskScheduledDay` now keeps at least one day, `loadState()`
  restores any already-empty task to every day, and the tracker's "%" cell
  carries a tooltip explaining a legitimate "—" (a brand-new task whose
  scheduled weekday hasn't come around yet).
- **Non-negotiable group set apart on the Dashboard.** The Non-negotiable
  half of the task card (`.today-group.nonneg`) is now a boxed panel — a
  faint `--target` tint, a 3px accent left bar, a ★ before the heading (via
  CSS `::before`, matching the tracker's non-negotiable star), the heading
  and range % in accent green, bolder task names, and a `done/total` tally
  (`.today-group-count`) pinned right. "Other tasks" stays a plain list so
  the contrast carries the hierarchy. No new palette — just existing
  `--target` tokens.
- **"Other tasks" boxed too, in orange.** Follow-up to the above: `.today-
  group.other` now gets the same card treatment as `.nonneg` — tinted
  background, 3px left accent, colored heading/`%`/`count` — but keyed to a
  new `--other-accent` token (light `#c98a2e`, dark `#d9a24a`, added to all
  three `:root` blocks) and with no ★ and a slightly softer tint, so the
  non-negotiables still out-rank it visually. The `group()` helper's
  `nonneg` boolean became a `variant` string (`"nonneg"` / `"other"`).
  Tasks in the Other card stay one-tap `<button>`s on the Today filter,
  same as before.
- **Dashboard task completion made unmistakable.** The Today-filter task
  rows in both cards were already tap-to-toggle `<button>`s writing
  `statusesByTask[id][todayKey]`, but the done state was too quiet. Now a
  completed row dims to 60% opacity, strikes the name through at 2px, and
  re-brightens on hover to signal it's still tappable to undo; the filled
  checkbox picks up the card's accent (green / orange). Added `title`
  ("Tap to mark done for today" / "Done today — tap to undo"),
  `aria-pressed`, and a `:focus-visible` outline. No logic change — because
  the status is keyed to the local date, the list re-initialises to all
  un-done at the next day's first render with nothing to reset.
- **Two "% out of what's scheduled, not what's logged" fixes.**
  (1) The Dashboard donut ("Done vs not done — overall") stopped using the
  logged-only denominator (`done + not_done`) — with one habit checked and
  the rest untouched it read 100%. It now uses `stats.applicable` as the
  total and `applicable − done` as "Not done" (folding in still-To-Do
  entries), so a fresh range is 100% Not Done and fills toward Done as
  habits are completed; the ring now agrees with the headline %. N/A still
  excluded; the separate To-Do slice stays gone.
  (2) The Input Tracker's per-row % was the all-time rolling
  `computeTaskAccomplishment`, whose denominator excludes future days — so
  early in a week the one logged day made it 100%. New
  `computeTaskWeekAccomplishment(s, task, weekDates, todayKey)` divides
  done days in the *shown* week by the task's scheduled days across that
  whole Mon–Sun week (upcoming weekdays included); the column header is now
  "Week %". `computeTaskAccomplishment` stays for rate-linked goals only.
  Follow-up: the per-day `createdAt` / `archivedAt` gate was then dropped
  from the denominator too — a habit added mid-week and ticked once was
  still showing 100% because that day was the only "counted" slot. Now
  every scheduled slot in a week the task is active for counts (matching
  the cells the grid lets you tick); only a week lying entirely outside
  the task's lifetime returns "—".
- **Made responsive for phone and tablet.** Almost entirely CSS; the only
  JS is a new `renderTopBar()` that emits a phone-only masthead (hidden by
  CSS at wider widths). Breakpoints: at `≤900px` the main padding tightens
  and the sidebar narrows to 184px (it already auto-collapses to icons
  below ~880px on first load); at `≤600px` the whole left sidebar becomes
  a **fixed bottom navigation bar** — `.shell` stacks, `.sidebar` goes
  `position: fixed; bottom: 0` as a horizontal row, the brand/collapse
  header is hidden, each `.nav-item` stacks its icon over a small label,
  and the bar holds just the four section links (evenly spaced).
  `.main` switches to normal document scrolling with bottom padding
  (`74px + env(safe-area-inset-bottom)`) so nothing hides behind the bar;
  the bar itself also respects the iOS safe-area inset. The wide grids
  (`.dash-charts`, `.today-groups`) had their `minmax()` track floors
  wrapped in `min(…, 100%)` so a single column can't force horizontal
  overflow on a ~320px screen, and `.view-header` now wraps. The tracker
  and calendar tables keep their existing `overflow-x: auto` wrappers —
  they scroll horizontally within the page, sticky first column intact.
- **Phone-only top app bar (`.topbar`).** Since the phone layout moves
  navigation to the bottom bar and hides the sidebar's brand/footer, a
  sticky masthead was added at the top on `≤600px`: a
  `grid-template-columns: 1fr auto 1fr` row with the **centered Momentum
  logo lockup** and the **light/dark/auto theme toggle pinned right**
  (a pill button — the theme control that used to sit in the sidebar
  footer, which is now `display: none` on phones). Hidden at every wider
  width, where the sidebar already carries both the brand and the theme
  toggle, so there is never a duplicate. Purely presentational: it reuses
  the existing `theme` state and `cycleTheme()`.

## Explicitly not planned

Kept out to protect the core design principle — this app should never
become another thing to manage:

- No notifications/reminders, no background sync, no accounts, no cloud.
- No gamification (points, badges, levels, streak-freeze mechanics).
- No AI features or external API calls of any kind.
- No general-purpose habit tracking beyond the category/task model above.
- No theming/customization beyond basic usability, per-category
  color-coding, and a light/dark/auto mode toggle (added post-launch — see
  Post-launch revisions, below). No user-chosen accent colors, fonts, or
  layout density beyond that.
- No real scheduling logic on the Calendar (conflict detection, specific
  dates, reminders, external calendar sync).
- No fixed weekly/monthly reflection prompts (the first design's Weekly/
  Monthly Review). Not built in Phases 1–4; if it turns out to be missed
  after real use, it's a candidate for a future phase, not an oversight to
  silently work around. **Requested post-launch:** a Dashboard "monthly
  review due" notification card (persistent until the review is completed,
  click-through to a Monthly Review section). Deferred pending a decision
  on the underlying feature — there is no monthly check-in in the current
  data model, so the notification has nothing to hang off yet. Needs:
  where reviews live in `momentum:v2` (proposed: `monthlyReviews`, an
  object keyed by `"YYYY-MM"`), what "due" means (proposed: the current
  month has no entry), what a review actually *is* (free-text? prompts?),
  and where the "Monthly Review section" lives (new nav tab vs. a
  Dashboard panel).
- No data export/import in this roadmap (the first design had it in its
  Phase 3). Worth adding later as its own phase if real usage make backup
  a real concern, but it's not one of the four phases above — ask before
  pulling it forward.
- No mobile app wrapper — the single HTML file, opened in a mobile browser
  (optionally added to the home screen), is the intended mobile experience.

If a real need for any of the above shows up after actual use, that's a
reason to explicitly scope a new phase — not to quietly add it mid-phase.
