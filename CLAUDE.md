# Subroutine

Zach's planner and notebook, served at https://zjaylewis94.github.io/SUBROUTINE/.
It's the hub of the library: the Archive (`zjaylewis94.github.io`) shows it as the plate under the title, above every shelf.
Public repo. The data lives in the private `subroutine-data` repo.

## Privacy rule
This repo and its Pages site are public. **Never put planner data, names, or personal details in `index.html` or anything
else here.** New content belongs in Zach's saved data, which syncs to the private repo, not in `SEED`.
- `SEED` already holds personal starter content from before the data moved (the Army tracker, recommendation names,
  client names, the manifesto fallback in `hydrateState()`). Don't add more, and don't copy it into other repos.
- Planner data was committed here as `subroutine_data.json` until Sep 27, 2026 (`0f41352` removed it). It's gone from
  `main`, but it's still in git history. Clearing that needs a history rewrite on `main`, which is Zach's call.

## The file
- `index.html` is the whole app: a single self-contained file (~9,100 lines), vanilla JS, no build step. Keep it that way.
  - CSS: lines ~15–1423.
  - App script: ~1460–8674.
  - GitHub sync: a second `<script>` at the end, ~8677–9089.
- One global state object `S`. `save()` stamps `S._ts`, writes localStorage, and hands the JSON to `window._fbSave`
  (the name is left over from Firebase; it's GitHub sync now). `renderApp()` rebuilds the sidebar, tab bar, right sidebar and canvas.
- Navigation is `curTab` (a box id, `home`, or `lists`) plus `curSec`. `renderCanvasInner()` dispatches on them.
  Re-renders keep scroll position (`canvasScrollSnap` / `canvasScrollRestore`), keyed on `curTab|curSec|curChapter|curPage|curFitnessPage`.

## What's in it
- **Home** tabs (`S.homeTabOrder`, drag to reorder):
  - **Production Board** (`overview`): `S.projects` → books → chapters → tasks, as a map or a list (`S.dtaskView`).
  - **Brain Dump** (`dump`): `S.dump` and the `S.feed` notes. AI sort calls the Anthropic API from the browser with the key
    stored in `subroutine_ai_key`. The model id is hard-coded in the `fetch` in that section.
  - **Monthly** (`S.calEvents`).
  - **Weekly** (`S.weeklyData`).
  - **Daily** (`S.dailyData`). It holds Today's Targets, the Game Plan (`S.gamePlan`), the schedule (with a live red now-line), and **Tasks by Box**.
- **Boxes** (`BOXES` / `BOX_META`): teaching, coaching, art, house, personal, finance, fitness, cookbook, plus `S.customBoxes`.
  An `army` box exists in `SEED` and in `TASK_BOX_RULES`. Each box has Tasks, Projects (text, one per line with `[status]`) and Notes,
  plus chapters with pages. Extra sections:
  - Teaching has Curriculum (`S.curriculum`, seeded from `CURRICULUM`).
  - Fitness has Program (`S.fitness`).
  - There's also an optional `tracker` (nested checklist).
- **Lists** (`curTab==='lists'`): chores (reset on Sunday by `doChoreReset()`), shopping (fixed sections, saved view in
  `sub_groc_view`), manifesto, quotes, media.
- **Schedule:** `S.weekSchedule` (bell schedule by weekday, from `S.scheduleStart`), `S.timeBlocks` (one-off blocks; `startMin` counts from `DSTART`), `S.scheduleSkips`.
- **Tasks by Box** (Daily):
  - Tasks are `{t, d, children}`.
  - Tags in the text: `!high`/`!!!`, `!med`/`!!`, `!low`/`!` and due dates `@10/31` or `@oct 31` (`taskTags()`).
  - Auto-sort (on unless `S.ui.taskAutoSort === false`) orders by done, then priority, then due date.
  - Checking a parent checks its subtasks.
  - "File Unsorted" moves `S.unsortedTasks` into boxes by keyword (`TASK_BOX_RULES`), and it can be undone.

## Saved data and seeding
- localStorage keys (shared origin with every book, ~5 MB total, so keep keys `sub_`-prefixed):
  - `sub_v5`: the state.
  - `sub_v5_unpushed`: there are edits GitHub hasn't seen.
  - `sub_v5_backups`: up to 3 local copies taken before data is replaced.
  - `subroutine_gh_cfg`: sync settings and token.
  - `subroutine_ai_key`.
  - `sub_groc_view`.
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
