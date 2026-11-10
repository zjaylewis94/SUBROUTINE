# Subroutine

Zach's planner and notebook, served at https://zjaylewis94.github.io/SUBROUTINE/.
It's the hub of the library: the Archive (`zjaylewis94.github.io`) shows it as the plate under the title, above every shelf.
Public repo. The data lives in the private `subroutine-data` repo.

## Privacy rule
This repo and its Pages site are public. **Never put planner data, names, or personal details in `index.html` or anything
else here.** New content belongs in Zach's saved data, which syncs to the private repo, not in `SEED`.
- `SEED` is neutral placeholder content: a new device shows it only until Sync loads the real planner.
  The personal starter content, the manifesto fallback and a hard-coded April 2026 save were removed on Oct 30, 2026.
  They're still in git history.
- Planner data was committed here as `subroutine_data.json` until Sep 27, 2026 (`0f41352` removed it). It's gone from
  `main`, but it's still in git history. Clearing that needs a history rewrite on `main`, which is Zach's call.

## The file
- `index.html` is the whole app: a single self-contained file (~8,900 lines), vanilla JS, no build step. Keep it that way.
  - CSS: lines ~15–1427.
  - App script: ~1464–8463.
  - GitHub sync: a second `<script>` at the end, ~8466–8878.
- One global state object `S`. `save()` stamps `S._ts`, writes localStorage, and hands the JSON to `window._fbSave`
  (the name is left over from Firebase; it's GitHub sync now). `renderApp()` rebuilds the sidebar, tab bar, right sidebar and canvas.
- Navigation is `curTab` (a box id, `home`, or `lists`) plus `curSec`. `renderCanvasInner()` dispatches on them.
  Re-renders keep scroll position (`canvasScrollSnap` / `canvasScrollRestore`), keyed on `curTab|curSec|curChapter|curPage|curFitnessPage`.

## What's in it
- **Home** tabs (`S.homeTabOrder`, drag to reorder):
  - **Production Board** (`overview`): `S.projects` → books → chapters → tasks, as a map or a list (`S.dtaskView`).
  - **Brain Dump** (`dump`): `S.dump` and the `S.feed` notes. AI sort calls the Anthropic API from the browser with the key
    stored in `subroutine_ai_key`. The model id is hard-coded in the `fetch` in that section.
  - **Monthly** (`S.calEvents`). Event names show without clock times (`calTitleNoTime()`); the saved title keeps them.
- **Color rules** (Zach's, Nov 10, 2026): `COLOR_RULES` / `colorCategory()` pick the color of a new calendar event
  (`guessCalColor()`) or schedule block (`guessDomainColor()`) from its title, first match wins: Shamrock's = green (`work`),
  Army = yellow, holidays = gray, fitness = dark blue, school and coaching = light blue (`teaching`), art = pink, everything
  else = personal red. A color picked by hand sticks. `applyColorRules()` ran once per data set (`S._colorRules_1110`) to recolor
  events and blocks from that day on and the bell schedule (Lunch red, every other period light blue).
  - **Weekly** (`S.weeklyData`), the Sunday plan (Nov 8, 2026). Top to bottom it follows Zach's Sunday routine:
    last week's slipped goals and targets (each one Carry → or Drop, kept in `handled`, plus a "what slipped, and why" `review`),
    Top 3 goals, then each day's calendar events, **Lesson** and **Dinner** (stored on `S.dailyData[date].lesson` / `.meal`,
    so Daily shows them under "From Your Week Plan") and targets (the same as Daily's Today's Targets), then week `notes`.
    On Sundays a banner jumps to next week. A scheduled task opens a planning session with Claude every Sunday afternoon.
    Extras on the page (Nov 9, 2026):
    - US holidays on each day (`usHolidays()`, computed for any year). A holiday hides itself when you logged your own event
      for it that day (`HOL_MATCH`, e.g. "THANX GIVING").
    - **✨ Ideas from your Production Board** under the goals while a slot is empty (`boardSuggestions()`). It ranks
      overdue / due-this-week / `!high` / next-up tasks, at most 2 per project, and skips SHELVED/HELD books; tags are
      stripped by `stripTaskTags()`. Tap one to fill the next empty slot.
    - A blank Dinner offers the Cook Book's "Last Meal" for that day (`cookbookDinnerOn()`). That reads the Cook Book's
      `cb2_plan` in this device's localStorage, whose keys are weeks from the current Sunday (`wk0_` = this week) + day (0 = Sun) + meal.
  - **Daily** (`S.dailyData`). It holds Today's Targets, the Game Plan (`S.gamePlan`), the schedule (with a live red now-line), and **Tasks by Box**.
- **Boxes** (`BOXES` / `BOX_META`): teaching, coaching, art, house, personal, finance, fitness, cookbook, plus `S.customBoxes`.
  An `army` box exists in `SEED` and in `TASK_BOX_RULES`. Each box has Tasks, Projects (text, one per line with `[status]`) and Notes,
  plus chapters with pages. Extra sections:
  - Teaching has Curriculum (`S.curriculum`, seeded from `CURRICULUM`).
  - Fitness has Program (`S.fitness`).
  - There's also an optional `tracker` (nested checklist).
- **Lists** (`curTab==='lists'`): chores, shopping, manifesto, quotes, media.
- **Chores** (`renderChores()`), two tabs (view saved in `sub_chore_view`):
  - **Weekly**: `S.lists.chores` as tap-to-check tiles (`doneAt` shows the day). Reset on Sunday by `doChoreReset()`.
    "Edit list" brings back the rows for reorder, rename and delete.
  - **Monthly upkeep**: `S.lists.upkeep` = `{id, t, area, every (days), log:[timestamps]}`, shown as "last serviced" cards
    colored by status (overdue, due within 7 days, OK, never logged). Seeded once from `UPKEEP_SEED` (`S.lists._upkeepSeed1`).
- **Shopping list** (`renderShopping()`, `S.lists.grocery`, fixed `GROCERY_SECTIONS`). Zach's names for the two views:
  - **The Pantry**: the master list, every item ever added, A–Z. Tap an item to put it on or take it off the Shopping List.
  - **Shopping List**: what to get on the next trip. Checked items go to "In the cart".
  - **Reset list** empties the Shopping List and leaves The Pantry alone.
  - The view is saved in `sub_groc_view` (`need` = Shopping List, `all` = The Pantry).
  - An item is on the list when `!d`, in the cart when `d` and `gotAt`, and Pantry-only when `d` with no `gotAt`.
    It reuses the old fields, so older cached copies still read it sensibly.
- **Schedule:** `S.weekSchedule` (bell schedule by weekday, from `S.scheduleStart`), `S.timeBlocks` (one-off blocks), `S.scheduleSkips`.
  - Saved `startMin` counts from `DSTART` (5 AM). Never change `DSTART`: every saved block would move.
  - The widget shows `DVIEW0`–`DVIEW1` (12 AM to 12 AM), so a block before 5 AM has a negative `startMin` (down to `DMIN_LO`).
  - On phones the widget is a 55vh window that scrolls on its own and opens at the current time.
  - Block titles wrap onto as many lines as the block's height fits (`_fitBlockText`); short blocks drop the time row, and the tooltip has both.
- **Tasks by Box** (Daily):
  - Tasks are `{t, d, children}`.
  - Tags in the text: `!high`/`!!!`, `!med`/`!!`, `!low`/`!` and due dates `@10/31` or `@oct 31` (`taskTags()`).
  - Auto-sort (on unless `S.ui.taskAutoSort === false`) orders by done, then priority, then due date.
  - Checking a parent checks its subtasks.
  - The list is flat rows with depths (`buildTaskList()`). A row's block is the row plus every row under it:
    - Dragging moves the whole block, and only among its siblings, so a subtask stays in its parent.
    - Tab indents the block under the task above it at the same level.
    - Shift+Tab moves the block out to just below its parent's group.
    - Phones get ⇤ ⇥ buttons while a task is being edited.
  - "File Unsorted" moves `S.unsortedTasks` into boxes by keyword (`TASK_BOX_RULES`), and it can be undone.

## Look (glass theme, Nov 2026)
- An obsidian background, a fixed `body:before` layer (dark gradients, faint sheen, light grain). iPhone ignores
  `background-attachment:fixed`, so don't move it onto `body`.
- Frosted glass panels: the `GLASS THEME` block at the end of the `<style>` turns `--s1/--s2/--s3/--br/--br2` translucent
  and blurs the main surfaces (top bar, sidebars, tab bar, dialogs, Daily columns).
  - Phones skip the blur on cards to keep scrolling smooth.
  - Deleting that one block brings back the old flat look.
- Use the CSS variables for new UI so it picks up the glass. A hard-coded dark hex stays opaque.
- **Sky** (`Sky` module, near the end of the app script): the background follows the time of day and the weather.
  - Five palettes in `PAL` (night, morning, noon, afternoon, dusk) blend along keyframes set from this device's
    sunrise, solar noon and sunset (NOAA math, no library).
  - Weather comes from Open-Meteo (free, no key, every 30 min) and sets the mood: rain and fog mute the colors and the
    `#sky-fx` layer draws rain; a clear sky is glossier.
  - Everything feeds CSS variables (`--sky0`–`--sky4`, `--sky-sat/bright/gloss/haze`). Their defaults are the plain obsidian look.
  - Every color is clamped dark (`MAX_BASE_L`, `MAX_BLOB_L`): the background stays under about 7% luminance, so white text keeps about 9:1 contrast.
  - The 🌙 chip in the top bar opens the settings: on/off, weather on/off, use my location, and a preview-the-day slider.
  - Per device in localStorage: `sub_sky`, `sub_sky_loc`, `sub_sky_wx`. The location is never synced or committed.
    Without one, the city comes from the time zone (`TZ` map).

## Saved data and seeding
- localStorage keys (shared origin with every book, ~5 MB total, so keep keys `sub_`-prefixed):
  - `sub_v5`: the state.
  - `sub_v5_unpushed`: there are edits GitHub hasn't seen.
  - `sub_v5_backups`: up to 3 local copies taken before data is replaced.
  - `subroutine_gh_cfg`: sync settings and token.
  - `subroutine_ai_key`.
  - `sub_groc_view`.
  - `sub_chore_view`, `sub_upkeep_filter`, `sub_sky`, `sub_sky_loc`, `sub_sky_wx` (per-device view settings).
  - sessionStorage `sub_adopt_at` / `sub_sync_note`.
- **A saved planner never re-reads `SEED`.** `hydrateState()` only fills in missing fields. To ship new seeded content to Zach's
  devices, add a one-time step guarded by a flag on `S`, like the existing `S._pdSeed_0806`, `S._schoolSeed_2627b` and
  `S._librarySeed_0930` (`mergeLibraryGoals()` from `LIBRARY_GOALS`). The step should add only what's missing, so anything Zach
  deleted stays deleted.
- Automatic saves at startup (chore reset, migrations) must not count as user edits. Only `_userInteracted` saves set
  `sub_v5_unpushed`, so a stale device never blocks newer data.
- Changing the shape of the data means bumping the key (`sub_v5` → `sub_v6`) and copying the old data forward.

## GitHub sync (rewritten Oct 4, 2026)
The comment at the top of the sync script is the spec. Read it before touching sync. In short:
- Saves `subroutine_data.json` to the **private `zjaylewis94/subroutine-data`** repo. Any device still pointed at the
  public `SUBROUTINE` repo is switched automatically in `getCfg()`. Never default sync to a public repo.
- Each device tracks the GitHub blob it came from (`baseSha`), not timestamps.
- It checks on open, on return to the app, and every 90 s.
- Uploads send `baseSha`. A 409 means re-check, never overwrite.
- When both sides changed, the user picks, and the other side stays in **Version history** (every save is a commit).
- It syncs the whole file, with no per-piece merge, so a real conflict is resolved by that prompt. The Teacher Book has
  per-piece merging (`teacher_book_sync_1`) if this ever needs it.

## How we work
- Work on a branch and open a PR. Zach merges from his phone. Don't push to `main`.
- Before opening a PR, check the page in headless Chromium (Playwright is installed) at 390px and 1280px wide, with
  no page errors. Most use is on a phone: narrow screens get a slide-in menu and a pinned top bar.
- Keep commits small and named for what changed for Zach ("Tasks by Box: checking a parent checks its subtasks").
- Say "ready to merge" only when a PR is finished.
