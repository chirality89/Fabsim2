# ROADMAP.md — FabSim roguelite

Work top to bottom, one checkbox per session. Plan in `tasks/todo.md` first (see `PROJECT.md`). Tick the box here when done.

Current milestone: **M1 — Seeded RNG and test harness**

## M0 — Move in (week 1)
- [x] Put `index.html` and the kit in a new GitHub repo; turn on GitHub Pages; confirm the live URL loads and plays on a phone
- [x] Play 2–3 shifts by hand and write down anything that feels wrong or unclear in `tasks/todo.md`
- [x] Run a first headless check: load the page, run one simulated shift with a simple bot, confirm no errors

## M1 — Seeded RNG and test harness (week 2)
- [x] Add an `RNG` section (mulberry32); replace all 5 `Math.random()` calls; keep the seed in `s.seed`; show it on screen
- [ ] Move the `gauss()` calls inside `PHYSICS.ETCH` and `PHYSICS.CMP` onto the seeded generator
- [ ] Playwright set up (`@playwright/test`, `tests/`); first physics test with one textbook value from David
- [ ] Physics checks for each model with David's textbook values
- [ ] Bot: ~100 seeded runs; report win rate, errors and NaNs by seed
- [ ] Screenshots at desktop and 375 px phone widths

## M2 — Run structure (weeks 3–4)
- [ ] Settle the open decisions in `CLAUDE.md` §1b in a written plan
- [ ] Run state: shift counter (5 per run), deadline, `customerTrust`
- [ ] Lose conditions (`cash < 0`, `customerTrust <= 0`) and a run-over screen; a win screen
- [ ] Shift report becomes the run's quarter report; add a "Free play" option
- [ ] Tune so a run takes about 30 real minutes

## M3 — Branching map (weeks 5–6)
- [ ] Map screen between shifts: 2–3 nodes (Standard, Pull-in, Maintenance, Event)
- [ ] Fault selection and hot-lot rate weighted by node type (weights in CONFIG)
- [ ] 3 event cards with a tradeoff each

## M4 — Telegraphed excursions (weeks 7–8)
- [ ] Move faults out of `TOOLS` into a CONTENT table (data-only; effects as input modifiers)
- [ ] Explicit telegraph lead time and visible metric per fault; early / late / missed outcomes
- [ ] 10 more fault cards drafted by `content-drafter`, each with a red herring; reviewed by David
- [ ] `physics-reviewer` pass on all of them

## M5 — In-run automation (weeks 9–10)
- [ ] Pick-1-of-3 screen after each shift
- [ ] Dispatch rules, auto-PM, alert filter, auto-hold — each with its "dumb" failure mode
- [ ] Curses with cash payouts
- [ ] Automation resets each run

## M6 — Boss and meta (weeks 11–12)
- [ ] Qual lot boss (Cpk target) and ramp-to-volume finish
- [ ] Engineering notebook persists between runs (its own localStorage key)
- [ ] Daily seed and a shareable result card

## M7 — Balance and share (week 13)
- [ ] 100-run bot balance pass (win-rate band, run length)
- [ ] Playtest with 3 people; fix the top issues
- [ ] Publish and share the link

## Later
- Fab realism fixes from the 2026-10-22 playtest (David agreed; details in `tasks/todo.md`): litho focus should drive CD and process window, not overlay; allow strip-and-regrow rework for the first oxide; Deal-Grove A and B constants (with M1 item 4); SPC control limits and a spread chart; a "barrier/seed assumed" note before copper plating; analyzer drift should show as beam current or species, not energy
- BEOL and MEMS starting-fab unlocks; QubitFab (superconducting flip-chip route, aluminum base metal)
- Ascension levels
- Split the script into `game.js` if the file gets unwieldy
- Port to React + Vite only if David decides it's worth it
