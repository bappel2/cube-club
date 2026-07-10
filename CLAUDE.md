# CLAUDE.md — Cube Club

Project context for Claude Code. Read this first.

## What this is
Cube Club is a kid-friendly (ages ~6–9) Rubik's cube web app. It's a **single, self-contained `index.html`** — all HTML, CSS, and JavaScript in one file, no build step, no dependencies, no server. It's meant to be hosted for free on GitHub Pages or opened directly from disk (works offline).

Target user: a young beginner cuber. Design priorities, in order: big tap targets, pictures over words, short friendly wording, nothing can break or be deleted by accident, safe (no open YouTube, no accounts, no data collection).

## Architecture
Everything lives in `index.html`:
- `<style>` — CSS. Cube colors are CSS variables in `:root` (`--white`, `--yellow`, `--red`, `--orange`, `--blue`, `--green`).
- Markup — one `<section class="screen">` per screen: `#home`, `#timer`, `#solve`, `#videos`, `#trophy`, `#games`. Only the one with class `active` is shown.
- `<script>` — plain vanilla JS, no framework. Organized in labeled comment blocks by feature.

Navigation: `go(screenId)` toggles the `.active` class and shows/hides the Back button.

### Data / persistence
Uses `localStorage` via two helpers: `load(key, default)` and `save(key, value)`. Keys:
- `cc_times` — array of solve times in milliseconds (max 50 kept).
- `cc_streak` — `{count, best, last}` practice-streak state (`last` = `YYYY-MM-DD`).
- `cc_badges` — array of earned badge ids (monotonic; badges never un-earn).
- `cc_maxstep` — furthest Solve-Helper step index reached (drives the Cube Master badge).
- `cc_mem_best` / `cc_pop_best` — high scores for the two Cube Games.

There is no backend. All state is per-device, in the browser.

### Feature modules (in `<script>`, in order)
1. **Navigation** — `go()`.
2. **Storage helpers** — `load()`, `save()`.
3. **Timer** — `newScramble()`, `pressStart()`/`pressEnd()` (pointer + touch), spacebar `keydown`/`keyup` listeners, state machine `tState` (`idle → ready → set → running`), `recordTime()`, `renderTimes()`, `clearTimes()`. Hold ~350ms to arm (clock turns green), release to start, tap to stop.
4. **Solve Helper** — `STEPS[]` array (9 beginner-method steps). Each step is `{t, face, say, algo, legend}` where `face` is 9 hex colors drawn by `drawFace()` as an SVG 3×3 grid. `renderStep()` / `stepMove(dir)` drive a one-step-per-screen flow with a progress bar. Color letter constants: `W Y R O B G X` (X = neutral gray placeholder).
5. **Videos** — `VIDEOS[]` array of `{group, title, url}`, rendered by `renderVideos()` grouped by `group`. This is the parent-editable curated list. Currently the entries are placeholder YouTube *search* links — real approved video URLs still need to be filled in.
6. **Celebration popup** — a queued full-screen overlay reused by both new-best-times and new-badge unlocks. `celebrate(icon, title, sub)` enqueues; `closePop()` advances the queue.
7. **Trophy Room** — badges + practice streak. `BADGES[]` (`{id, icon, name, how, check}`), `badgeStats()` derives stats from stored data, `checkBadges(silent)` unlocks + celebrates new ones, `renderTrophy()` draws the streak banner and badge grid. `markPracticedToday()` / `streakCurrent()` manage the streak (date-based). Called from `recordTime()` and `stepMove()`.
8. **Cube Games** — a menu (`showGamesMenu()`) into two games rendered in `#gameStage`: **Color Memory** (Simon-style: `startMemory`, `flashSeq`, `memTap`, `memOver`) and **Color Pop** (speed matching: `startPop`, `newPopRound`, `popTap`, `popDone`). `gameToken` cancels stale async/timers when leaving a game; `go()` calls `stopGame()` on exit. Both use the shared `GCOLORS[]` cube palette.

`init` at the bottom calls the render functions once on load and seeds badges silently with `checkBadges(true)`.

## Conventions
- Vanilla JS only. **Do not** add a framework or a build step — the whole value here is a zero-dependency single file that a non-developer can host and edit.
- Keep it one file unless there's a strong reason to split. If you must split, keep it buildless (plain `<script src>` / `<link>`), not a bundler.
- Destructive actions (e.g. clear times) must stay behind a `confirm()`.
- Keep wording short and kid-readable. Test copy against a 7-year-old reading level.
- Escape user input when injecting into HTML if you ever add free-text fields again.

## How to run / test
- Run: open `index.html` in any browser. No server needed.
- Quick syntax check: extract the script and `node --check`.
- Runtime smoke test used during development (needs `npm i jsdom`): load the file in jsdom with a `url:` set (localStorage needs a non-opaque origin), then call `go()`, `newScramble()`, `recordTime()`, `stepMove()`, `addCube()` and assert on the rendered DOM. Note jsdom doesn't implement `window.scrollTo` — stub it: `w.scrollTo = () => {}`.

## Deploy
GitHub Pages: push `index.html` to a public repo, Settings → Pages → deploy from `main` / root. Live at `https://<user>.github.io/<repo>/`. See `PUT-IT-ONLINE.md`.

## Roadmap / good next tasks
- Fill `VIDEOS[]` with real, parent-approved video URLs (and optionally support inline YouTube embeds by video ID instead of links out).
- More badges / games, or tie games into badges (e.g. a Color Memory high-score badge).
- Draggable 3D cube (three.js) — this WOULD justify adding a CDN script.
- More cube types in Solve Helper (2×2, Pyraminx).
- Optional sound effects and a light/dark or theme picker.
- Export/import progress — times, streak, badges (JSON) — so data can move between devices.

## Guardrails
- No accounts, no analytics, no ads, no collection of personal data. Keep it that way.
- Keep the video experience closed — the child should never reach the open YouTube feed from inside the app.
