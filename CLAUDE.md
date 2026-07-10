# CLAUDE.md — Cube Club

Project context for Claude Code. Read this first.

## What this is
Cube Club is a kid-friendly (ages ~6–9) Rubik's cube web app. It's a **single, self-contained `index.html`** — all HTML, CSS, and JavaScript in one file, no build step, no dependencies, no server. It's meant to be hosted for free on GitHub Pages or opened directly from disk (works offline).

Target user: a young beginner cuber. Design priorities, in order: big tap targets, pictures over words, short friendly wording, nothing can break or be deleted by accident, safe (no open YouTube, no accounts, no data collection).

## Architecture
Everything lives in `index.html`:
- `<style>` — CSS. Cube colors are CSS variables in `:root` (`--white`, `--yellow`, `--red`, `--orange`, `--blue`, `--green`).
- Markup — one `<section class="screen">` per screen: `#home`, `#timer`, `#solve`, `#videos`, `#cubes`, `#games`, `#fixer`. Only the one with class `active` is shown.
- `<script>` — plain vanilla JS, no framework. Organized in labeled comment blocks by feature.

Navigation: `go(screenId)` toggles the `.active` class and shows/hides the Back button.

### Data / persistence
Uses `localStorage` via two helpers: `load(key, default)` and `save(key, value)`. Keys:
- `cc_times` — array of solve times in milliseconds (max 50 kept).
- `cc_mem_best` / `cc_pop_best` — high scores for the two Cube Games.

(The Cube Guide is static content and stores nothing. Old keys `cc_streak`, `cc_badges`, `cc_maxstep` were retired with the Trophy Room; any leftover values in a returning user's browser are simply ignored.)

There is no backend. All state is per-device, in the browser.

### Feature modules (in `<script>`, in order)
1. **Navigation** — `go()`.
2. **Storage helpers** — `load()`, `save()`.
3. **Timer** — `newScramble()`, `pressStart()`/`pressEnd()` (pointer + touch), spacebar `keydown`/`keyup` listeners, state machine `tState` (`idle → ready → set → running`), `recordTime()`, `renderTimes()`, `clearTimes()`. Hold ~350ms to arm (clock turns green), release to start, tap to stop.
4. **Solve Helper** — `STEPS[]` array (9 beginner-method steps). Each step is `{t, face, say, algo, legend}` where `face` is 9 hex colors drawn by `drawFace()` as an SVG 3×3 grid. `renderStep()` / `stepMove(dir)` drive a one-step-per-screen flow with a progress bar. Color letter constants: `W Y R O B G X` (X = neutral gray placeholder).
5. **Videos** — `VIDEOS[]` array of `{group, title, by, id}` (`id` = YouTube video id), rendered by `renderVideos()` as grouped thumbnail cards. Tapping a card calls `playVideo()`, which embeds a locked-down `youtube-nocookie.com/embed/` player **inline** (`rel=0`, autoplay) so the child never reaches the open YouTube feed; `closeVideo()` (also called from `go()` on leaving) drops the iframe to stop playback. This is the parent-editable curated list — entries are real, parent-approved videos.
6. **Celebration popup** — a queued full-screen overlay reused by new-best-times (Timer) and the solved-cube finish (Solve My Cube). `celebrate(icon, title, sub)` enqueues; `closePop()` advances the queue.
7. **Cube Guide** (`#cubes`) — an illustrated, tap-to-learn index of puzzle-cube types (replaces the old Trophy Room). `CUBES[]` entries (`{id, group, name, nick, stars, tag, icon, facts, cool}`) are grouped like the video list; `renderCubes()` builds the card grid and `openCube(id)`/`closeCubeDetail()` toggle a detail panel (same show/hide pattern as the video player). Every icon is **drawn as SVG** so there are no photos and it stays offline: `isoFace()`/`isoCube()` tile an isometric cube from fraction "cut" arrays (`cubeSVG(n)` = 2×2–7×7; `mirrorSVG`, `voidSVG`, `gearSVG`, `ghostSVG`, `skewbSVG` are variants), plus `triSVG` (pyramids), `megaSVG` (pentagon), `clockSVG`. `CUBEVB` is the shared cube viewBox and `ISO_A/B/UR/UL/D` are the isometric axis vectors — note the **side faces use `UR/UL`** (up-and-out, y = −0.5); using the top-face `A/B` there instead makes the cube droop below its front vertex. `starHTML(n)` draws the difficulty stars.
8. **Cube Games** — a menu (`showGamesMenu()`) into two games rendered in `#gameStage`: **Color Memory** (Simon-style: `startMemory`, `flashSeq`, `memTap`, `memOver`) and **Color Pop** (speed matching: `startPop`, `newPopRound`, `popTap`, `popDone`). `gameToken` cancels stale async/timers when leaving a game; `go()` calls `stopGame()` on exit. Both use the shared `GCOLORS[]` cube palette.
9. **Solve My Cube** (`#fixer`) — a real cube solver. The kid paints in all six sides (`scEntry`, per-face 3×3 with neighbor-color hints so orientation is unambiguous), it's validated, then solved one of two ways — **✨ Solve it!** (the teachable beginner method, grouped into the same 7 steps) or **⚡ Quick fix** (the fewest-moves solver) — and shown move-by-move. Two parts:
   - **Engine + solver** (all `cb*`/`qs*`-prefixed): a canonical Kociemba cubie model (`MOVES`, `cbMult`, `cbApply`, `cbFromFacelet` which also validates: color counts + orientation/permutation parity; `cbToFacelet` is the inverse — cubie→facelet, used to render the 3D cube at each step of the guided solve — verified round-trip). `cbSolve()` is layer-by-layer via a curated bounded-search (`cbSearch`) over the beginner algorithms (auto-rotated to all sides by `cbVariants`), grouped into the 7 `PH` phases, then `cbSimplify`'d. **It self-verifies**: it simulates its own moves and returns `solved:false` rather than ever showing a solution that doesn't reach solved. Stress-tested to 100% on thousands of random cubes. `cbQuickSolve()` is a separate **near-optimal** solver for lightly-scrambled cubes: IDA* over the 18 half-turn moves guided by small BFS pattern-database heuristics (`qsBFS` builds corner-twist / corner-perm / edge-flip + three edge-quartet position tables, lazily on first use via `qsBuild`, ~200ms; each coordinate transforms independently under `cbMult`, so decode→apply→encode is valid). It returns the **shortest** solution it finds (self-verified too), with a **cumulative** node budget (~1.2M) so genuinely-hard cubes cleanly return `solved:false` and the UI falls back to `cbSolve`. Verified 100% correct + optimal on hundreds of shallow scrambles; near-instant to ~depth 10, graceful fallback (~1.4s) beyond. The heuristic tops out ~depth 10 by design — small tables keep it buildless/offline; don't ship the big (88M) full-corner PDB.
   - **UI** (`sc*`-prefixed): `scRender()` dispatches on `scMode` (`intro`/`entry`/`preview`/`solving`/`error`/`guide`). Frame is **white on the DOWN face, green FRONT** (`SCOLOR`/`SNAME` map colors↔face letters; the kid holds it white-down/green-front). Friendly validation errors (impossible / wrong count / already solved). `MOVEHELP` gives each move a kid-readable description.
   - **Mid-solve recovery** (so a wrong turn never means starting over): from the guide, **"😕 My cube looks different"** (`scFixHere`) re-opens the color editor **pre-filled with the cube's *current* state** — `scFacelet2Entry(cbToFacelet(cbApply(scState0, movesSoFar)))` — with `scEditing=true`, so the kid corrects only the squares that don't match and taps **"✅ Fixed it — solve again!"** (`scSolveAgain` → `scStartSolve(scLastMode)`, re-solving in whichever mode they were using). An incomplete or invalid edit **keeps them in the editor** with a gentle `scFixMsg` note (set via the `scEditing` branch in `scStartSolve` instead of the error screen; cleared on the next `scPaint`/`scClearFace`); **"✕ Keep steps"** (`scCancelFix`) bails back to the guide unchanged. The first-time scan (`scEditing=false`) is untouched. `scStartSolve(mode)` is the shared solve entry for **both** buttons and remembers `scLastMode`.
   - **3D cube** (`sc3d*`/`C3D`/`C3D_VIEW`, styled `.sc3d-*`): a **draggable pure-CSS 3D cube**, shown in three places. **During entry** (small 140px, above the flat grid, built from `scEntry`): unpainted stickers render as `.empty` slots and fill in as the kid paints, the current side gets a `.cur` glow ring, and the cube auto-turns to foreground that side (`scSetFaceView` sets a target from `C3D_VIEW`, eased in `sc3dTick`). In the **preview step** (240px, reached by "See my cube →" on the last face, `preview` mode). In the **guided solve** (178px, `guide` mode): the cube shows the cube's **current state** — `scFacelet2Entry(cbToFacelet(cbApply(scState0, movesSoFar)))`, so it stays in sync with the child's real cube as they hit Next — auto-turns face-on to the moving side (`C3D_GVIEW`/`scSetGuideView`), lights it, and overlays a **turn arrow** (`scArrowSVG` in `.sc3d-arrow`: ↻ for an unprimed move, ↺ for prime `'`, ½ for a `2`/double — verified every unprimed move is CW on the face-on view, so the mapping is uniform). `scCube3DHTML(size,curFace,state,arrow)` builds it (`state` defaults to `scEntry`) — sizes and `translateZ` are size-derived and set inline; the six panels take the painted arrays **directly** (identity — verified face-by-face that each side lands on the correct edge, so don't reorder without re-verifying). `scInitCube3D` wires pointer-drag + flick momentum + a preview-only idle turntable through `sc3dTick`/rAF, which **sleeps when settled** and self-stops when the cube is gone or `#fixer` isn't active. NOTE: rAF is paused in a hidden/background tab, so the eased turn, momentum, and idle spin only animate when the tab is visible (dragging still works, since it sets the transform directly). Preview buttons: "✨ Solve it!" → `scDoSolve` and "⚡ Quick fix" → `scQuickSolve` (both funnel through `scStartSolve(mode)`; quick silently falls back to the learn method if the cube is too twisted); "← Fix a side" → `scBackToEntry`. In `guide` mode the header is the 7 step-chips + "Step N: name" for the learn method, or a "⚡ Quick fix — just N moves!" banner when `scQuick` is set (then `scGroups` is `null`, so `cur.gi`/`cur.name` are only read inside the learn branch). `window.SC3D_DEBUG=true` labels each sticker with its index.

`init` at the bottom calls the render functions once on load (`renderTimes`, `renderStep`, `renderVideos`, `renderCubes`, `newScramble`, `renderHomeArt`). `renderHomeArt` injects the home-screen cube **mascot** (`#heroCube`) and the faint **background cubes** (`#bgDecor`, `.bg-decor`), both reusing the Cube Guide's `cubeSVG(3)`. The header title is centered on every screen (`.backbtn` is absolutely positioned at the left so the centered `.logo` isn't pushed off-center); tiles have an inset top-glow.

### Solver frame note
The color scheme is fixed so the first layer (white) is the solver's D face and the last layer (yellow) is U — so the 7 phase labels line up with Learn-to-Solve, and the kid holds the cube the same way for entry and solving. Do not "simplify" by swapping colors without re-checking the whole `cbFromFacelet` → `cbSolve` pipeline against a real-cube color arrangement.

## Conventions
- Vanilla JS only. **Do not** add a framework or a build step — the whole value here is a zero-dependency single file that a non-developer can host and edit.
- Keep it one file unless there's a strong reason to split. If you must split, keep it buildless (plain `<script src>` / `<link>`), not a bundler.
- Destructive actions (e.g. clear times) must stay behind a `confirm()`.
- Keep wording short and kid-readable. Test copy against a 7-year-old reading level.
- Escape user input when injecting into HTML if you ever add free-text fields again.

## How to run / test
- Run: open `index.html` in any browser. No server needed.
- Quick syntax check: extract the script and `node --check`.
- Runtime smoke test used during development (needs `npm i jsdom`): load the file in jsdom with a `url:` set (localStorage needs a non-opaque origin), then call `go()`, `newScramble()`, `recordTime()`, `stepMove()`, `renderCubes()`/`openCube()` and assert on the rendered DOM. Note jsdom doesn't implement `window.scrollTo` — stub it: `w.scrollTo = () => {}`.

## Deploy
GitHub Pages: push `index.html` to a public repo, Settings → Pages → deploy from `main` / root. Live at `https://<user>.github.io/<repo>/`. See `PUT-IT-ONLINE.md`.

## Roadmap / good next tasks
- Fill `VIDEOS[]` with real, parent-approved video URLs (and optionally support inline YouTube embeds by video ID instead of links out).
- More Cube Games, or more entries / deeper facts in the Cube Guide (`CUBES[]`).
- Draggable 3D cube — DONE for Solve My Cube as a pure-CSS 3D preview (no CDN, stays offline). Next stretch: animate it through the solution moves during the guided solve (that's where three.js might finally be worth vendoring locally).
- More cube types in Solve Helper (2×2, Pyraminx).
- **Quick fix / shortest-solve — DONE** (`cbQuickSolve`): a near-optimal IDA* solver offered alongside the teachable method on Solve My Cube (fewest moves for a lightly-mixed cube; falls back to the beginner solve when the cube is deeply scrambled). Stretch: a full two-phase (Kociemba) solver with larger tables would push the instant range past ~depth 10 and always return ≤~22 moves — but that costs build time/memory and only helps the rare deeply-scrambled "quick fix", so it's likely not worth breaking the buildless/instant-load feel.
- Optional sound effects and a light/dark or theme picker.
- Export/import progress — times and game high scores (JSON) — so data can move between devices.

## Guardrails
- No accounts, no analytics, no ads, no collection of personal data. Keep it that way.
- Keep the video experience closed — the child should never reach the open YouTube feed from inside the app.
