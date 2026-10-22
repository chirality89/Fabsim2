# ROADMAP.md — FabSim

Current milestone: **M0 — Move into Claude Code**

Work top to bottom. One checkbox per session. Tick it when done (see CLAUDE.md §11).

## M0 — Move into Claude Code
- [ ] Put the current single-file build in a git repo with CLAUDE.md, ROADMAP.md, CHANGELOG.md, .claude/agents/
- [ ] Reorganize index.html into the six labeled sections (no behavior change)
- [ ] Replace every Math.random() with one seeded generator; show the seed on screen
- [ ] Deploy to GitHub Pages (public repo, personal account) and confirm the live URL loads on a phone

## M1 — Test harness
- [ ] Physics checks with David's textbook values (Deal-Grove, Arrhenius, etch rate, CD vs dose)
- [ ] Playwright screenshots, desktop + phone
- [ ] Simple seeded bot that can play a quarter

## M2 — Core run loop
- [ ] Run state: cash, customer trust, deadline, 5 quarters
- [ ] Lose conditions and a run-over screen
- [ ] Streamline dispatch and recipe flow so a quarter takes ~5 minutes

## M3 — Branching map
- [ ] Map screen with 2–3 nodes per quarter (Standard, Pull-in, Maintenance, Event)
- [ ] Excursion deck weighted by node type
- [ ] 3 map events

## M4 — Telegraphed excursions
- [ ] Telegraph field + drift on inputs + early/late/missed responses
- [ ] 10 excursion cards drafted (content-drafter), reviewed by David
- [ ] physics-reviewer pass on all 10

## M5 — Automation upgrades
- [ ] Pick-1-of-3 screen between quarters
- [ ] Dispatch rules, auto-PM, alert filter, auto-hold (each with its dumb failure mode)
- [ ] Curses with cash payouts

## M6 — Boss and meta
- [ ] Qual lot boss (Cpk target) and ramp-to-volume win screen
- [ ] Engineering notebook persists between runs
- [ ] Daily seed + shareable result card

## M7 — Balance and share
- [ ] 100-run bot balance pass (win rate band, run length)
- [ ] Playtest with 3 people, fix top issues
- [ ] Publish the playable build

## Later
- BEOL fab unlock (CMP, thin film), MEMS, QubitFab
- React + Vite port
