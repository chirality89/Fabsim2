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

## Playtest findings: 3 personas (2026-10-22)

Three AI playtesters drove the real page in headless Chromium (Playwright), clicking the UI like players. Nothing in the repo was changed. Each finding below was reproduced by the playtester who reported it; none has been checked by hand yet. Their screenshots were in the session's temporary folder, which is deleted when the session ends; rerun a seed to see a finding again.

| Persona | Seed(s) | What they played |
|---|---|---|
| **P1** Integration engineer who games | 1111 | 3 full shifts through the UI, early pulls on drifting tools |
| **P2** Min-maxer, no fab knowledge | 101–108, bot on 11–88 | Tried to break it; compared 7 strategies across 8 seeds × 2 shifts |
| **P3** Redditor with game-dev hobby experience | 3333 | Arrived cold; 2 shifts on desktop, 1 pass on phone |

**No page errors, no console errors, and no NaN or Infinity** in any run (P2 checked 64 two-shift bot runs and every PHYSICS input from min to max with every fault at full strength).

### Where the personas agree (2–3 of them hit it)

1. **There is no pressure: you can't lose, and judgment doesn't pay.** (P1, P2, P3)
   - Cash goes negative with no effect: 200 lot starts took it to −$180,000 and play carried on (P2, seed 102).
   - P1 went from $20k to $218,747 in 3 shifts with no late lots; maintenance and wrong parts were "rounding errors".
   - P2's bot: **accepting every metrology result unread beat reworking or scrapping out-of-spec samples by about $7k per shift.** Starting no lots at all came within about $9k of the best plan, because the starting lots and hot lots are free.
   - Yield is flat inside spec (`metricFactor` returns 1 for any in-spec value), so centering a recipe does nothing. The shift report's "tighten recipes toward target" advice is therefore not true today (P1).
   - Planned fix: M2 (lose conditions, `customerTrust`). Shipping bad wafers and ignoring hot lots need a cost, or "accept everything" stays the best plan.
2. **The default ILD trench etch recipe predicts out of spec.** 68 s gives 228.5 nm against the 230 nm lower limit; the prediction shows red from the start (P1, P2). **David: is this a deliberate trap?** If so, say so in How to play; if not, change the default in `ROUTE`.
3. **The copper residue reading always shows yellow, even at 0.0 nm.** The guard band (0.12 × 2 × 30 = 7.2 nm) is wider than the 5 nm spec, so nothing can read green (P1, P2; `judge()`). The residue chart's 0–35 nm scale also squashes all points at the bottom (P1).
4. **Toasts cover dialog buttons.** The toast layer sits above dialogs (z-index 30 vs 20). Clicks still go through, but Accept / Rework / Scrap, the shift report's Start shift button and the side-panel charts get hidden, mostly on phone and short windows (P1, P2, P3).
5. **Phone top bar:** "Start lot (−$1,000)" wraps between "(−" and "$1,000)", and "2×" is clipped (P1, P2, P3). Already noted under M1 item 1.
6. **The clean toast reads "Recipe locked: ."** The clean tool has no recipe settings, so the text comes out empty (P2, P3).

### Bugs (seed, how to repeat it)

| # | Bug | Seed | Repeat it | Who | Where in the code |
|---|---|---|---|---|---|
| B1 | **A qual wafer can backfire.** Running a qual on a drifting tool puts it down; part prices jump from $300 to $600 (the after-alarm price), the right fix no longer counts as early, and `alarms` stays 0 | 1111 | Furnace drifting (zone heater at about 0.40, t=420), tool idle → Maintenance → Run a qual wafer → wait 15 min | P1 | `maintain` |
| B2 | **Opening the etcher panel uses up random numbers**, so the same seed plays out differently (and RENDER changes state) | 1111 | Open the etcher panel 6 times before running 600 min: the furnace gets the gas-flow fault instead of the zone heater | P1 | `PHYSICS.ETCH` `gauss()`, called from `predHtml`. **This is M1 item 2.** |
| B3 | **Space and Enter buy lots.** After clicking Start lot, the button keeps focus, so Space starts a lot instead of pausing (held Enter bought 30) | 101 | Click Start lot, press Space 10 times | P2 | `wireInput` keydown (`e.target === document.body` check) |
| B4 | **Keyboard focus escapes dialogs.** Tab reaches Start lot, the tools and the panel behind a dialog, and works while the clock is stopped | 103 | Open a metrology review, press Tab, Enter on Start lot | P2 | `openModal` (no focus trap) |
| B5 | **Help replaces an open dialog.** Over a review, Escape then stops working; over the shift report, Escape closes it without starting the next shift (it comes back on Play, so no softlock) | 103, 107 | Tab to How to play from a review, press Enter | P2 | `openModal` (one dialog at a time) |
| B6 | **Accept by key repeat.** Accept gets focus automatically, so Enter twice on a lot card opens the review and accepts it unread | 104 | Focus a lot card awaiting review, press Enter twice | P2 | `openModal` autofocus |
| B7 | **A `?seed=` link wipes your save without asking.** "Start over" has no confirmation either (and sits in red next to "Start the shift" on a new game) | 105 → 106 | Have a save, open `?seed=106` | P2, P3 | Boot code (`s.seed !== linkSeed`) |
| B8 | **Tools alarm with no warning when they've never run a lot.** Drift ramps on time, not on use, so an unused tool's SPC chart is empty when it alarms. This breaks the fairness rule ("every telegraph visible in a metric") | 3333 | Never send lots past the furnace; the etcher alarms at about t=363 | P3 | `tick` drift ramp |
| B9 | **Lot cards reorder when a lot loads**, so a quick second tap can hit a different lot | 3333 | Load the first lot; it jumps to the end of the list | P3 | lot list render order |
| B10 | **Shift-report advice ignores the real failure.** With 0 shipped and 9 lots awaiting review it still gives SPC advice | 3333 | Never tap a yellow review card for a shift | P3 | shift-report advice |
| B11 | The diagnosed-signatures list shows the same fault twice (analyzer) | 1111 | Diagnose the same implanter fault twice | P1 | `s.lessons` render |
| B12 | `CLAUDE.md` says `SAVE_KEY = 'fabsim-v3'`; the code uses `'fabsim-v4'` since M1 item 1 | — | — | P2 | CLAUDE.md §0 (docs only) |

Also noted, not bugs: the save in localStorage shows the hidden fault and true wafer values (devtools cheating only, P2). Seed hunting works: on seed 358 the first drift starts at 944 min, against a median of 222 min over seeds 1–2000 (P2). What held up: double-clicks don't double-count, same seed repeats exactly, reload mid-dialog restores safely (P2).

### Fab realism (P1). David to rule on each; nothing changed

| What the game does | What a real fab does | P1's severity | Where |
|---|---|---|---|
| Focus error drives overlay (`overlay = |focus| × 180`) | Defocus hurts CD and process window (Bossung); overlay comes from stage, alignment, lens distortion | Eye-roll | `PHYSICS.LITHO` |
| Oxidation can never be reworked | A pad/first oxide on bare silicon is routinely HF-stripped and regrown; "no rework" fits gate oxide or after patterning | Eye-roll | `reworkRule` 'OX' |
| Deal-Grove: 1000 °C, 12 min gives 54 nm; A ≈ 0.165 µm; only B has the Arrhenius term (1.23 eV, the dry-B value) | Textbook dry O₂ gives about 20–25 nm (A ≈ 0.0165 µm); B/A has its own activation energy | Misleading if labelled Deal-Grove | `PHYSICS.OX`. **Needs David's textbook values (M1 item 4)** |
| SPC charts show spec limits only, means only | X̄/R or X̄/S charts with control limits; spread faults show on the R/S chart | Misleading | `spcSvg` |
| Copper plating straight onto the dielectric | Barrier/seed deposition first | Eye-roll; a one-line "barrier/seed assumed" note would do | `ROUTE` |
| Analyzer magnet drift raises implant energy | Energy is set by the acceleration voltage; analyzer drift shows as beam current or species | Mild eye-roll | `PHYSICS.IMP` |
| Silicon trench and ILD oxide etch share one chemistry (0.8 factor) | Different chemistries | Fine as a simplification; worth a comment | `PHYSICS.ETCH` |
| Reticle particle as a slowly rising particle count | A repeating defect on the same die in every field | Fine as a simplification | litho fault |
| Faraday (0.367 nm per mA·s/cm²), Preston form, Rs ∝ 1/dose, 2 h pre-furnace Q-time, 3-slot sampling | — | Rings true | — |

**Diagnosis fairness (P1):** CMP slurry and carrier faults both raise mean dishing (slurry at 0.7 gives 24.8; carrier gives 42.9). P1 swapped two wrong parts before finding slurry; telling them apart needs a full measure or a spread chart. After a repair, nothing marks the service on the SPC chart, and the old excursion points invite a second, unneeded pull. A qual wafer only says pass/fail and never names the bad metric.

### Onboarding and feel (P3, newcomer)

- **The biggest reason a newcomer leaves:** nothing says "tap the yellow card to review". The help says wafers "are measured", which reads as automatic. P3's first shift ended with 0 shipped, 9 lots waiting for review and −$7,350.
- The How to play box opens with 11 jargon steps (ILD, CMP, SC1). There's no "do this first" cue, and the glow on the next tool is a faint amber arc.
- **Click fatigue:** about 110 review decisions for 11 lots, mostly all-green Accepts. The real decisions get lost in the noise.
- **Drift shows only inside each tool's SPC tab**, never on the floor. P3 caught no drifts before alarm in either run. P1, who knew to look, caught 17 early with 0 alarms: the telegraph works for someone who knows where to look.
- **Feedback is almost all toasts, capped at 3.** "Loaded…" messages push out alarms and hot lots. There's no shipping moment and no reason given for low yield (shipments at 80%, 27%, 46%, 12.6%).
- **Phone:** tool labels are about 7–8 px, the speed buttons are 46×36 (under 44 px tall), the floor and panel are far apart, and SPC charts have no y-axis numbers. On empty charts the target line runs through the "No lots measured yet" text like a strikethrough.

### Recommendations, ranked (combined)

| # | Recommendation | Raised by | Fits |
|---|---|---|---|
| 1 | Make losing possible and judgment pay: lose conditions, a trust or money cost for shipping bad wafers and for late or ignored hot lots, maybe yield loss inside spec (from Cpk) so centering matters | P1, P2, P3 | M2 (David decides the trust formula, §1b) |
| 2 | Small fairness and determinism fixes: qual backfire (B1), etch RNG in render (B2), residue guard band, toasts above dialogs, empty clean toast | P1, P2, P3 | B2 = M1 item 2; the rest are small items |
| 3 | Keyboard and save safety: Space always pauses (except in sliders), focus stays in dialogs, no autofocus on Accept, Help doesn't replace a dialog, confirm before a seed link or Start over replaces a save | P2 | small item |
| 4 | Onboarding: a guided first lot ("tap the yellow card to review"), shorter, plainer help, advice that reacts to what actually went wrong | P3 | before public demos |
| 5 | Make the telegraph fair and visible: a drift cue on the floor, ramp counted in lots run (not minutes), a spread/range chart with control limits, a maintenance marker on SPC | P1, P3 | M4 |
| 6 | "Auto-accept all-green" as a deliberately dumb automation upgrade (it also accepts lots from a drifting tool), to end the click fatigue | P3 | M5 |
| 7 | Yield autopsy on shipping ("lost 54%: PECVD thickness, CMP dishing"), saved to the notebook | P3 | M6 notebook |
| 8 | Realism items above: litho focus→overlay, oxide rework at step 1, Deal-Grove constants, barrier/seed note | P1 | David's call; M1 item 4 for Deal-Grove |

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
