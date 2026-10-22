# FabSim — Task Tracker

## Current: M1 — Seeded RNG and test harness

### Task: M1 item 1 — one seeded random generator (DONE 2026-10-22)

David approved the plan as written, with all three proposals: seed from the clock or `?seed=`; old saves start over once; seed shown in the top bar.

**Result:**
- [x] `Math.random` appears nowhere in `index.html` (0 hits)
- [x] Same seed twice (12345) gives an identical shift: ledger, cash, fault states and lessons all match ($33,491, 12 shipped, 2 alarms). A different seed (777) differs ($22,052, 10 shipped, 4 alarms)
- [x] Seed shows in the top bar and survives a reload, generator state included
- [x] An old v3 save (under the old key, and a v3-shaped save under the new key) is ignored: fresh game, no errors
- [x] The game runs with localStorage blocked (every storage call throwing)
- [x] `?seed=banana` falls back to a clock seed; a `?seed=` link that differs from the saved game starts that seed
- [x] Screenshots: desktop 1280×800 and phone 375×812 with Playwright phone emulation. The seed fits on the top row next to the clock and cash on the phone, and the How to play box fits
- [x] No console errors; no lines inside PHYSICS changed
- Checks ran from a throwaway script outside the repo (`verify-m1-rng.js`); the real test files come in M1 item 3.

**Noticed, not fixed (out of scope):** on the phone the "Start lot (−$1,000)" button wraps between "(−" and "$1,000)". It was like this before this change (checked against the previous version). Small CSS fix, maybe in the screenshots item (M1 item 6).

**Behavior notes for David:**
- "Start over" on a `?seed=` link replays that same seed. Without `?seed=`, "Start over" gets a new clock seed.
- Same seed → same run holds exactly for the bot. In hand play, moving an etch slider still uses random numbers (fixed in item 2), and the live clock steps unevenly (a possible later item).

#### The approved plan (kept for reference)

**Goal.** Every random number in the game comes from one generator that starts from a seed. Same seed + same player actions → same run. The seed is saved with the game and shown on screen. No change to how the game plays or to any physics equation.

**Why now.** Reproducible bug reports, the bot (M1 item 5) and the daily seed (M6) all depend on it.

**Where randomness comes from today** (5 lines call `Math.random()`):

| Line | Code | What it decides |
|---|---|---|
| 513 | `gauss()` (2 calls) | Bell-curve noise: wafer spread, gauge noise, particle counts, etch RF jitter, CMP scatter |
| 515 | `expRand()` | When each tool starts drifting (mean 720 min) |
| 516 | `pick()` | Which of a tool's 3 faults appears |
| 588 | `newGame` | When the first hot lot arrives |
| 814 | `tick` | Time between later hot lots |

Lines 513, 515 and 516 are the helpers. Every other random draw in the game goes through them, so 588 and 814 are the only direct calls outside the helpers.

**Steps, by section:**

1. **CONFIG**: bump `SAVE_KEY` from `'fabsim-v3'` to `'fabsim-v4'` (the shape of `s` changes; CLAUDE.md §4).
2. **New RNG section** between CONFIG and PHYSICS, about 20 lines:
   - `mulberry32`, a small, well-known generator: a 32-bit number goes in and a number between 0 and 1 comes out. Its whole state is one integer, so it saves to localStorage as part of `s`.
   - `rand()` returns the next number and stores the new state in `s.rngState`.
   - `newSeed()` picks a seed for a new game (see question 1).
3. **HELPERS**: `gauss`, `expRand` and `pick` call `rand()` instead of `Math.random()`. Same formulas.
4. **STATE**: `newGame(seed)` stores `s.seed` and `s.rngState` and sets `v:4`. `load()` only accepts `v === 4` saves. The two direct calls (first hot lot, starting drift times) use `rand()` / `expRand()`.
5. **RULES**: hot-lot spacing in `tick` uses `rand()`.
6. **RENDER**: show `Seed 482913` in the top bar next to the clock, in the muted clock style.
7. **Check**: `grep Math.random index.html` returns nothing.

This touches 6 sections but only about 30 lines in total. CLAUDE.md asks me to check before touching more than one section, so this plan is that check.

**Not in this item** (each is its own later M1 item): moving the `gauss()` calls out of `PHYSICS.ETCH` and `PHYSICS.CMP` (item 2), Playwright and tests in the repo (item 3), the 100-run bot (item 5).

**Known limit after this item: same seed is not yet always the same run.**
- `PHYSICS.ETCH` calls `gauss()` even when there is no fault (`gauss() * 0`). The recipe panel calls `PHYSICS.ETCH` for its predicted values every time a slider moves, so moving an etch slider uses up random numbers and shifts everything that follows. Item 2 fixes this by passing noise into PHYSICS as an input, so the prediction uses none.
- The real-time loop moves the clock in uneven slices, depending on frame rate, so two people clicking "the same" way can still drift apart. The bot steps the clock exactly one minute at a time, so bot runs will repeat exactly. Making hand play repeat exactly too would need a fixed-step clock; I'd suggest that as a small later item, not now.

**Acceptance criteria:**
- `Math.random` appears nowhere in `index.html`
- The scratch bot (from M0), run twice with the same seed, gives identical end-of-shift ledgers; two different seeds give different ones
- The seed shows in the top bar on desktop (1280×800) and phone (375 px) and survives a page reload
- An old `fabsim-v3` save is ignored cleanly: the game starts fresh with no errors
- The game still runs with localStorage blocked
- No console errors; physics equations unchanged (diff shows no edits inside PHYSICS)

**Questions for David (before I start):**
1. **Where does a new game's seed come from?** Proposal: a random-looking 6-digit number from the clock (`Date.now()`), unless the address ends in `?seed=123456`, in which case that seed is used. That makes "send me the link" a full bug report.
2. **Old saves.** Bumping the save key means any game in progress today starts over once. Fine?
3. **Seed display.** Top bar, `Seed 482913`, small and muted. OK, or would you rather it lived in the shift report or the How to play box?

### Later M1 items (outline only; each gets its own plan and session)
- **Item 2:** `PHYSICS.ETCH` and `PHYSICS.CMP` take their random noise as an input instead of calling `gauss()`. Predictions pass zero noise. Equations unchanged; `physics-reviewer` pass.
- **Item 3:** add `@playwright/test` (pre-approved), a `package.json` and `tests/`. First physics test with one textbook value from David.
- **Item 4:** a physics check for each model, using David's textbook values.
- **Item 5:** bot with ~100 seeded runs; report win rate, errors and NaNs by seed.
- **Item 6:** screenshots at desktop and 375 px phone widths, using Playwright's real phone emulation (not plain headless Chrome).

## Done

### M0 — Move in (2026-10-22)
- [x] Push `index.html` and the kit to a new GitHub repo (`chirality89/Fabsim2`)
- [x] Turn on GitHub Pages (Settings → Pages → Deploy from branch `main`, folder `/ (root)`)
- [x] Confirm the live URL loads and plays on a phone (https://chirality89.github.io/Fabsim2/)
- [x] Play 2–3 shifts by hand; log what feels wrong or unclear below
- [x] Headless check: load the page, run one simulated shift with a simple bot, confirm no errors

**Playtest notes (David):**
- 2026-10-22: No issues noticed. The How to play dialog fits the phone screen (the cut-off in the headless screenshot was a test artifact).

**Progress notes:**
- Imported FabSim v2 and the kit; game code unchanged. Merged PR #1 to `main`. The first visit gave a 404 because Pages hadn't been saved yet; after saving, the build ran and the live link works. The address is case-sensitive: `Fabsim2`, capital F.
- Headless bot check, 5 one-shift runs (720 game min each) in headless Chromium at 1280×800. Bot: loads waiting lots onto free tools (hot lots first, then earliest due), accepts every metrology result, keeps one lot waiting at clean (max 8 on the line), and repairs downed tools by trying parts in order. Result: no page errors, no console errors, no failed requests, no NaN/Infinity anywhere in `s`, and the shift report opens with correct ledger totals. Spread: 10–13 lots shipped, all on time; average yield 30–79%; 0–5 alarms; cash $3,392–$56,659. No seeds yet, so these runs can't be replayed. Not a balance result: the bot never tunes recipes or pulls tools early.
- Bot-script bug found and fixed during the check (not a game bug; see `tasks/lessons.md`).
- The bot-check results above were missing from this file after PR #3 (a text-match edit silently changed nothing) and were restored with the M1 plan. Rule added to `tasks/lessons.md`.
