# CLAUDE.md — FabSim

Read this file, then `PROJECT.md` (workflow), `tasks/lessons.md` (rules from past bugs) and `ROADMAP.md` (what's next), at the start of every session. If anything here conflicts with a request, stop and ask David.

## 0. The game already exists

`index.html` is a **working single-file game** (FabSim v2, about 1,300 lines of vanilla HTML, CSS and JavaScript, no build step). Do not rebuild it, rewrite it in React, or add a bundler. All work extends this file.

What exists today:

- **An isometric fab floor** with 8 tools: wet clean, oxidation furnace, stepper (litho), plasma etcher, ion implanter, PECVD, copper plating, copper CMP.
- **An 11-step route** per lot of 25 wafers: clean → oxide → litho 1 → trench etch → litho 2 → well implant → ILD deposition → litho 3 → ILD trench etch → copper fill → copper CMP, then ship. The stepper serves three layers and the etcher two, which is the dispatch puzzle.
- **Click-to-assign dispatch.** Tap a lot, then tap the glowing tool. **The recipe locks the moment the lot loads.** Each route layer keeps its own recipe.
- **Metrology judgment.** After each step, 3 of 25 wafers are measured (slots 1, 13, 25) with gauge noise. The player chooses Accept, Rework or Scrap, or pays to measure all 25. Rework rules differ by tool (implant, oxidation, PECVD and plating can never be reworked).
- **Hidden drift.** Every tool except clean eventually develops one of three hidden faults that ramps up over `DRIFT_RAMP_MIN` (200 minutes), shows on the SPC charts, and ends in an alarm. The player diagnoses by running a qual wafer or replacing a part. Right part is cheap; wrong part costs $750 and the fault stays.
- **Money and pressure.** Start with $20,000. Each lot costs $1,000 to start and is worth $7,000 at 100% yield. Hot lots (customer wafers) are worth double and due sooner. Late lots lose 30%. A 2-hour Q-time sits between clean and furnace.
- **Shifts.** A shift is 720 game minutes (about 6 real minutes at 1×). A shift report shows the ledger and advice. Progress saves to localStorage (`SAVE_KEY = 'fabsim-v3'`).
- **Real physics where it exists:** Deal-Grove oxidation with an Arrhenius rate constant (Ea = 1.23 eV), Preston's equation for CMP, Faraday's law for plating, Poisson-style defect-limited yield. Litho CD vs dose, etch rate, implant range and PECVD rate are empirical scalings.

Code layout inside `index.html`'s `<script>`:

```
1. CONFIG    every number that defines the game (costs, times, specs, TOOLS, ROUTE, PARAMS, METRICS)
2. PHYSICS   PHYSICS.<TOOL>(recipe, fault, ctx) → true lot-mean values
3. HELPERS   random numbers, formatting, spec judging
4. STATE     the one object `s` holding the whole game; newGame, save, load
5. RULES     dispatch, run completion, metrology decisions, drift, shipping, shift end
6. RENDER    floor, lot cards, side panel, dialogs
7. INPUT     taps, sliders, keyboard
8. LOOP      requestAnimationFrame clock
```

## 1. The target: a roguelite on top of this

The game identity doesn't change. We add a **run structure**, a **choice layer between shifts**, and **persistent knowledge**.

- **A run = one product ramp** of about 5 shifts (about 30 real minutes). A customer gives the player a spec and a deadline. Win by passing the qual lot boss and ramping to volume.
- **Death = bankruptcy OR losing the customer.** Lose when `cash < 0` or `customerTrust <= 0`. Cash exists today (it only turns red). `customerTrust` is new.
- **In the design notes a "quarter" is one shift in the code.** Keep "shift" in the UI unless David renames it.
- **Information is the currency.** Metrology costs money and the sample is only 3 wafers. Skipping the full measure is a gamble. This already exists; protect it.

### 1a. New mechanics (agreed design)

**Branching map between shifts.** Instead of the current "Start shift N+1" button, the player picks 1 of 2–3 nodes: *Standard* (baseline), *Pull-in* (customer pulls a deadline in: more hot lots, bonus cash, a curse attached), *Maintenance* (tools' faults reset but the shift starts with a throughput hit), *Event* (a text choice with a tradeoff). Node type is visible; which fault drifts next is hidden. Fault selection is weighted by node type (weights in CONFIG or CONTENT).

**Telegraphed excursions.** The drift ramp and SPC charts are already the telegraph. Formalize it: each fault gets an explicit lead time and the metrology that shows it. Player response windows: *early* (pull the tool at the first SPC trend: cheap), *late* (after bad lots ship), *missed* (alarm, tool down, expensive). Drift acts on **physics inputs** (e.g. dose sensor over-reads); models stay exact.

**Automation upgrades (in-run).** After each shift, pick 1 of 3: dispatch rules (FIFO, hot-lot first, tool matching), auto-PM scheduler, alert filter, auto-hold on SPC violation. **Automation is dumb on purpose** (auto-hold pins good lots on a sensor glitch; FIFO feeds a drifting tool). It frees attention and never replaces judgment. **Automation resets every run.**

**Between runs (persists).** An **engineering notebook** of diagnosed root causes and their signatures. `s.lessons` already records these within a game; persist it in its own localStorage key. Later: unlockable starting fabs (BEOL, MEMS, QubitFab), ascension levels, a daily seed.

### 1b. Open decisions (ask David in the planning step; don't assume)

1. **Customer trust formula.** Proposal: drops on late lots, low-yield shipments and scrapped hot lots; rises on on-time, high-yield shipments.
2. **Run length and deadline.** Default 5 shifts, with the deadline as a target shipped-lot count and yield by the final shift.
3. **Free play.** Default: keep today's endless mode as a "Free play" option.
4. **Naming:** "shift" or "quarter" in the UI.

## 2. Who you're working with

David is a semiconductor fab / packaging engineer (deep domain expert) and a **JavaScript beginner**.

- Explain every change in plain words. No unexplained code jargon.
- Prefer readable code over clever code. Short functions, descriptive names, comments on anything non-obvious.
- **Ask before** adding a framework, build tool, runtime library or npm package. The only pre-approved dev dependency is `@playwright/test`, for tests.
- **Defer to David on process physics and fab realism.** Push back (politely, with reasons) on code quality, structure and bugs.
- When unsure what David means, ask one clear question instead of guessing on a big change.

## 3. How we work (extends `PROJECT.md`)

- **Plan first.** Enter plan mode, write the plan to `tasks/todo.md` with acceptance criteria, and wait for David's OK before editing code.
- **One `ROADMAP.md` item per session.** Tick it when done.
- After any bug or correction, add a rule to `tasks/lessons.md` (what happened → root cause → rule).
- Commit after each working step with a plain-English message. Never commit with failing tests unless David says so.
- Ask before refactoring more than ~100 lines or touching more than one numbered section at once.
- Use the agents in `.claude/agents/` (section 10) only when their job is needed. No agents for small edits.

## 4. Code conventions

- **One file until it hurts.** If `index.html` passes about 2,500 lines, propose splitting the script into `game.js` loaded by a plain `<script src>`. No bundler. Don't split without approval.
- **Balance lives in CONFIG.** Costs, times, specs, drift rates and thresholds are constants at the top. Tune there, never inline.
- **New content is data.** New events, upgrades and curses are entries in tables, not new logic. A new fault today needs a data entry in `TOOLS[x].faults` **and** a branch in `PHYSICS[x]`. Milestone M4 moves faults into a CONTENT table so they are data-only.
- **Render code never changes game state.** RENDER reads `s`; RULES and INPUT change it. The floor SVG is built once and patched in place so taps never hit a replaced element; keep it that way.
- **Don't rebuild a panel under the player's finger.** Sliders update in place (see `refreshPanelLive`). Keep that.
- New sections slot in this order: CONFIG, RNG, PHYSICS, CONTENT, HELPERS, STATE, RUN, RULES, RENDER, INPUT, LOOP.
- Saved state: bump `SAVE_KEY` and `s.v` if the shape of `s` changes so old saves aren't loaded wrongly. Keep the **notebook in its own key** so a state change never wipes it.
- Wrap all `localStorage` access in try/catch (it is today). The game must run when storage is blocked.

## 5. The physics is sacred

- Use real equations where they exist (Deal-Grove, Arrhenius, Preston, Faraday, Poisson yield). The other models are labeled empirical scalings; don't change coefficients without David.
- Randomness and drift change **inputs and noise** (a faulty dose sensor, gauge noise, wafer spread), never the equations.
- **Never fudge a model to tune difficulty.** Tune with costs, specs, time, fault weights and CONFIG values.
- `PHYSICS.<TOOL>` returns true lot means. **Known exceptions:** `PHYSICS.ETCH` (RF jitter) and `PHYSICS.CMP` (per-wafer scatter) call `gauss()` internally. M1 moves these onto the seeded generator; don't add more.
- Every physics function documents units for inputs and outputs in a comment.

## 6. Locked game rules (verified in the code, don't change without asking)

- Click-to-assign dispatch. **Recipe locks on load** (`t.locked` is a copy).
- Metrology is a 3-wafer sample with gauge noise; decisions are Accept / Rework / Scrap; full measure costs extra.
- Rework rules per tool (`reworkRule`): implant and oxidation are permanent; re-etch only when under-etched; touch-up polish only with residue; litho can always be redone.
- A wrong diagnosis costs money and leaves the fault in place.
- Pause / play / 2× time control. The clock stops while a dialog is open.
- Lose conditions (new): `cash < 0` or `customerTrust <= 0`.

## 7. Seeded randomness

Today `Math.random()` appears in 5 places: `gauss`, `expRand`, `pick`, the first hot-lot time in `newGame`, and hot-lot spacing in `tick`. The target is **one seeded generator**.

- M1 adds an `RNG` section with a small seeded generator (mulberry32), stores `s.seed` in the saved state, shows the seed on screen, and replaces every `Math.random()`.
- **Until then, add no new `Math.random()` calls.**
- Same seed → same run. This enables reproducible bug reports, the daily seed and the playtest bot. Always include the seed when reporting a bug or test failure.

## 8. Content and public information

**Fault card** (target shape for M4; today these live inside `TOOLS[x].faults`):

```js
{ id, tool, name, sig,               // sig = what the player sees (existing text)
  draft: true,                       // Claude drafts; David removes the flag after review
  nodeWeights: { standard: 1, pullIn: 2, maintenance: 0, event: 1 },
  telegraph: { leadMin: 200, seenBy: ['cd'] },   // minutes of visible drift, and which metric shows it
  effect: { input: 'dose', scale: -0.14 },       // how it changes a physics input at full drift
  redHerrings: ['...'],
  costs: { early: 300, late: 600, missed: 600 },
  physicsNote: 'why this signature is physically plausible' }
```

**Fairness rules:** every fault must be diagnosable from in-game evidence (SPC charts, metrology, the recipe), and every telegraph must be visible in a metric the player can read. Every signature must be physically plausible. Different faults on one tool must produce different signatures, as the existing ones do.

**Public information only.** Textbook and published numbers only. No real company, tool-vendor or brand names, no fab-specific recipes, nothing resembling non-public process data. If unsure, flag it rather than use it. The only external request is Google Fonts (Barlow); the game must work with fallback fonts if it fails to load.

## 9. Testing and definition of done

Tests use Playwright against the real page. The script's top-level `const` and `let` bindings (`PHYSICS`, `s`, `tick`, `loadLot`, `decide`, `ROUTE`) are reachable from `page.evaluate`, so no refactor is needed to test them.

1. **Physics checks** (`tests/physics.spec.js`): textbook values David supplies, asserted against `PHYSICS`. For example, dry oxide thickness at a given temperature and time. Never edit an expected value to make a test pass; ask.
2. **Bot runs** (`tests/bot.spec.js`): about 100 seeded runs through `tick`, `loadLot` and `decide`. Pass if: no exception, no `NaN`/`Infinity` in state, win rate in the target band (set in CONFIG), and run length near 30 real minutes at 1×.
3. **Screenshots** (`tests/screens.spec.js`): desktop (1280×800) and phone (375×812). Actually look at them and report anything broken, overlapping or unreadable.

A change is **done** when all tests pass, screenshots are checked, `tasks/todo.md` is updated, `tasks/lessons.md` has any new rule, the `ROADMAP.md` box is ticked, and David gets a **3-bullet summary**: what changed, why, how to see it in the game.

If the test setup doesn't exist yet, say so and propose the smallest setup before writing features that depend on it.

## 10. Agents (`.claude/agents/`)

| Agent | Use it for | Edits code? |
|---|---|---|
| `physics-reviewer` | Any change to PHYSICS, METRICS or fault cards | No |
| `content-drafter` | Batches of faults, events, upgrades, curses | CONTENT only |
| `playtester` | Run all three test layers and look at the screenshots before "done" | No |
| `code-explainer` | Review a diff for beginner readability and rule violations; write David's summary | No |

Parallel agents only for independent content. Never two agents editing the same section.

## 11. Hosting

- Static site on **GitHub Pages** from the repo root: `index.html` is the whole game. Every push to `main` updates the live site.
- **No backend.** Saves and the notebook use localStorage. The daily seed comes from the browser date. The share card is drawn on a canvas.
- The repo is public and on David's personal account. Section 8 (public information only) applies to everything committed, including commit messages and screenshots.
- Don't add a server, database or leaderboard without asking.

## 12. Glossary (use the game's own words)

- **Lot** 25 wafers moving together. **Route** the ordered steps. **Layer** one litho or etch pass. **Recipe** the parameters locked on load.
- **Metrology** measuring a sample. **SPC** the control chart of lot means. **Drift** a fault ramping up unseen. **Alarm** the tool goes down. **Qual wafer** a test run after maintenance.
- **Q-time** the clean-to-furnace deadline. **Hot lot** a customer lot worth double and due sooner. **Yellow room** the litho bay. **Copper area** the contamination-segregated back end.
- **ILD** inter-layer dielectric. **CMP** chemical-mechanical polish. **CD** critical dimension. **Cpk** process capability index.
- **Excursion** a process going out of control. **Telegraph** the early warning. **Ramp** scaling a product to volume. **Qual lot** the customer's qualification lot.
