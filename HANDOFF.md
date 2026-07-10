# Cube Club — Handoff

A quick orientation for picking this project up in Claude Code.

## Where things stand
Working v1 is done: a single `index.html` with four features — Speed Timer, Learn to Solve (9 beginner steps with cube diagrams), Watch Videos (curated), and My Cubes (collection). It runs offline by double-clicking the file and is ready to host on GitHub Pages. It was smoke-tested in a headless browser (jsdom): timer math, scrambles, solver steps, cube saving, and video rendering all pass with no runtime errors.

## Files in this handoff
- `index.html` — the entire app. This is the project.
- `CLAUDE.md` — architecture, data model, conventions, and roadmap. Claude Code auto-reads this.
- `PUT-IT-ONLINE.md` — non-technical GitHub Pages deploy guide (for the parent).
- `README.md` — short project intro.
- `HANDOFF.md` — this file.

## How to get going in Claude Code
1. Make a folder, drop these files in it, and `git init` (or clone your GitHub Pages repo into it).
2. Open the folder in Claude Code. It will read `CLAUDE.md` automatically.
3. To preview: just open `index.html` in a browser — no server or install needed.

## Key decisions already made (and why)
- **Single vanilla-JS file, no framework, no build step.** The owner is a non-developer who wants to edit the video list and host it for free. A buildless single file keeps that possible. Preserve this unless there's a strong reason not to.
- **Free static hosting on GitHub Pages**, not Streamlit — chosen because Streamlit's free apps sleep and always need a server, and this app benefits from instant load + offline use. (Streamlit stays in use for a separate, unrelated project of the owner's.)
- **localStorage for all state** — no backend, per-device. Keys: `cc_times`, `cc_cubes`.
- **Closed video hub** — the child must never reach the open YouTube feed. Videos come only from the `VIDEOS[]` list.
- **Design for ~6–9 year old** — big buttons, pictures over text, confirm() on anything destructive.

## The one open to-do carried over
The `VIDEOS[]` entries are currently placeholder YouTube *search* links. Replace them with real, parent-approved video/playlist URLs (SoupTimmy and others). Optionally upgrade the hub to embed videos inline by YouTube ID rather than linking out.

## Suggested first tasks in Claude Code
- Fill in real video URLs.
- Add badges + a practice-streak counter (nice, self-contained wins).
- Add a JSON export/import so times and collection can move between devices.
- Later: a draggable three.js 3D cube (the one case where adding a CDN script is worth it).

See the Roadmap section of `CLAUDE.md` for the fuller list and guardrails.
