# Cube Club — Project Plan

A kid-friendly Rubik's cube website for a young beginner (ages ~6–9). One place to learn to solve, time solves, watch trusted cubers, and track the cubes he owns.

---

## 1. The idea in one sentence

A simple, colorful, mostly-picture website where a kid can learn to solve a cube step by step, time his solves, watch SoupTimmy and other cubers, and keep a "binder" of the cubes he owns — with no logins, no ads, and nothing scary to get lost in.

## 2. Who it's for

A young beginner, roughly 6 to 9 years old. Design decisions all flow from this:

- Big buttons and big text. A young kid taps, he doesn't read menus.
- Pictures and colors over words. Show a cube face, not a paragraph.
- Short words, friendly tone. "Great job!" not "Solve completed."
- Forgiving. Nothing breaks, nothing gets deleted by accident, no dead ends.
- Safe. Curated videos only — he never falls into the open YouTube feed.

## 3. The four features

### A. Solve Helper (the heart of it)

A step-by-step guide to solving a 3x3 using the beginner method, one screen per step.

- Each step shows a big picture of the cube and the moves as **arrows**, not just letters — a 6-year-old can't yet read "R U R' U'".
- A "Next" and "Back" button, and a progress bar (Step 3 of 7) so he sees how far he's come.
- The classic beginner stages: make a white cross, finish the white corners, do the middle row, make the yellow cross, finish the yellow face, position the last corners, position the last edges.
- A tiny "What do the arrows mean?" helper that teaches notation gently once he's ready.
- Start with the 3x3 only. Other cube types can be added later.

### B. Speed Timer

A real speedcubing timer he can use over and over.

- Hold the screen (or spacebar), let go, and it counts up. Tap to stop.
- Shows a random scramble to set the cube up before each solve.
- Saves his times and shows his **best time** and **average** — watching the number drop is the single biggest motivator for a kid.
- Big friendly celebration when he beats his best.

### C. Video Hub

A safe, curated shelf of videos instead of open YouTube.

- Hand-picked playlists from **SoupTimmy** and a few other kid-appropriate cubers.
- Organized by what he needs: "Learn to solve," "Get faster," "Cool tricks," "Different cubes."
- You (the parent) approve every video that goes on the shelf. He can only watch what's on it.

### D. My Cubes Collection

A digital binder of the cubes he owns — like collecting trading cards.

- Add each cube with a name, type (3x3, 2x2, Pyraminx, Megaminx, etc.), and a photo or emoji.
- He can mark favorites and see his collection grow.
- A gentle motivator that ties the whole site to *his* stuff.

## 4. Bonus ideas for later (don't build these first)

- **Badges** — earn one for first solve, first sub-2-minute, learning all the steps.
- **Practice streak** — "You've practiced 3 days in a row!"
- **Scramble-only mode** — random scrambles to practice from.
- **Kid glossary** — friendly definitions of cubie, edge, corner, scramble.
- **A draggable 3D cube** on screen (more advanced to build).
- **Algorithm cards** — a searchable deck once he's older and reading notation.

## 5. How to build it (options, simplest first)

**Option 1 — Single web page (recommended start).**
One HTML file that opens in any browser, works offline, saves his data on the device. No server, no accounts, no monthly cost. Perfect for a first version and easy to hand to him. This is what I'd build first.

**Option 2 — A "real" hosted site.**
Put it online at its own web address (free hosting exists) so he can open it on a tablet or Chromebook anywhere. Same code as Option 1, just published.

**Option 3 — An installable app.**
A version he can add to a tablet home screen like a real app. Nice later; not needed to start.

I'd strongly suggest starting with Option 1 — you get something real and usable fast, then grow it.

## 6. A sensible build order

1. Get the shell working: four big buttons on a home screen (Solve, Timer, Videos, My Cubes).
2. Build the **Timer** first — it's the most self-contained and instantly fun.
3. Build the **Solve Helper** steps with pictures and arrows.
4. Add the **Video Hub** with your approved playlist.
5. Add **My Cubes** so he can start his collection.
6. Polish: colors, celebration animations, sounds.
7. Add bonus items (badges, streaks) once he's using it.

## 7. Design and safety notes

- **No logins, no personal info, no ads.** Everything stays on your device.
- **You curate the videos.** He never touches the open YouTube app.
- **Nothing can be permanently deleted** by a mis-tap — add a "are you sure?" on anything that removes data.
- **Works offline** so it's usable in the car or anywhere.
- Big tap targets, high contrast, cube colors as the theme.

## 8. What it could cost

Built as a single web page (Option 1), the cost is **$0** — no hosting, no subscriptions, no accounts. If you later publish it online, free hosting covers it. The only "cost" is the time to build and add videos.

## 9. Suggested next step

Say the word and I'll build a **working single-file prototype** with the Timer and My Cubes working end-to-end, plus a starter Solve Helper and a Video Hub you can drop your approved links into. You'd be able to open it in a browser the same day and try it with him.
