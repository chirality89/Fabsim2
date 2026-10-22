# CLAUDE.md — FabSim

Read this whole file at the start of every session, then read `ROADMAP.md` to see the current milestone. If anything here conflicts with a request, stop and ask David.

## 1. What this is

FabSim is an isometric, physics-grounded wafer fab simulator, built as a **roguelite**.

- **A run = one product ramp.** A customer hands the player a device spec, a deadline and a cash budget. Win by hitting target yield, passing the qual lot boss, and ramping to volume.
- **Death = bankruptcy OR losing the customer.** You lose when `cash < 0` or `customerTrust <= 0`.
- **Target run length:** 20–40 minutes. Moment-to-moment play is streamlined to fit.
- **Information is the currency.** Metrology is fog of war: every measurement costs time and money; skipping it is a gamble.

Current process tools: Lithography, Ion Implantation, Oxidation, Plasma Etch. Planned unlocks: BEOL (CMP, thin film deposition), MEMS, QubitFab.

## 2. The run loop (locked)

```
Run start (spec, deadline, cash, seed)
  └─ ×5 quarters:
       1. MAP      pick 1 of 2–3 nodes: Standard | Pull-in | Maintenance | Event
       2. QUARTER  dispatch lots, run metrology, watch for telegraphs
       3. CHECK    cash < 0 or trust <= 0 → run over
       4. UPGRADE  pick 1 of 3 (mostly automation)
  └─ Qual lot boss (hit Cpk target) → Ramp to volume → win
On death: notebook saves diagnosed signatures → new run, new seed
```

### 2a. Branching map
- Node **type** is visible to the player. Which excursion gets drawn is hidden.
- **Standard:** baseline risk. **Pull-in:** pays cash, tighter deadline, attaches a curse. **Maintenance:** no throughput, resets tool drift. **Event:** text choice with a tradeoff (vendor offer, audit prep, staffing).
- **The excursion deck is weighted by node type.** Pull-ins draw nastier cards. Weights live in CONFIG.

### 2b. Telegraphed excursions
- Every excursion has a `telegraph`: the signal, the lead time, and the metrology needed to see it.
- The drift is real physics acting on **inputs** (e.g. chamber wall buildup shifting etch rate). Models stay exact.
- Player response windows: **early** = cheap PM or re-qual; **late** = hold lots, scrap some; **missed** = excursion hits, diagnosis required, wrong guess costs money.
- Skipping metrology removes the warning. That's intended.

### 2c. Automation upgrades
- Quarter 1 is fully manual. Upgrades add rules: dispatch rules (FIFO, hot-lot priority, tool matching), auto-PM scheduler, alert filter, auto-hold on SPC violation.
- **Automation is dumb on purpose.** Auto-hold can pin good lots on a sensor glitch; FIFO will feed a drifting tool. It frees attention; it never replaces judgment.
- **Automation resets every run.** Only the notebook, fab unlocks and ascension level persist.

## 3. Who you're working with

David is a semiconductor fab / packaging engineer (deep domain expert) and a **JavaScript beginner**.

- Explain every change in plain words. No unexplained jargon about code.
- Prefer readable code over clever code. Short functions, descriptive names, comments on anything non-obvious.
- **Ask before** adding a framework, build tool, npm package, or new source file.
- **Defer to David on process physics and fab realism.** Push back (politely, with reasons) on code quality, structure and bugs.
- When unsure what David means, ask one clear question instead of guessing on a big change.

## 4. How we work in Claude Code

- **Plan first.** For any feature, start in plan mode: restate the goal, list the sections you'll touch, and wait for David's OK before editing.
- **One feature per session**, matching the current item in `ROADMAP.md`.
- Commit after each working step with a plain-English message. Never commit with failing tests unless David says so.
- Use the agents in `.claude/agents/` (section 12) only when their job is needed. Don't spawn agents for small edits.
- Ask before refactoring more than ~100 lines or touching more than one section.
- Don't re-read the whole file repeatedly — search for the section you need.

## 5. Stack and file layout

```
fabsim2/
├── CLAUDE.md
├── ROADMAP.md          current milestone + checklist
├── CHANGELOG.md        one line per change
├── index.html          the game (single file)
├── tests/
│   ├── physics.test.js textbook values from David
│   ├── bot.js          seeded bot runs
│   └── screens.spec.js Playwright desktop + phone
└── .claude/agents/     physics-reviewer, content-drafter, playtester, code-explainer
```

- Single-file vanilla HTML/JS. No React, no bundler — yet. React + Vite later, only when David decides.
- Inside `index.html`, keep these labeled sections in this order:

```
// ===== CONFIG =====     all balance knobs: costs, specs, timings, deck weights
// ===== RNG =====        the one seeded random generator
// ===== PHYSICS =====    pure process models
// ===== CONTENT =====    excursions, map events, upgrades, curses, bosses, glossary (data only)
// ===== RUN STATE =====  run, quarter, map, cash, trust, WIP, automation rules, notebook
// ===== UI =====         rendering, input, modals
```

- **Content is data.** Adding an excursion, event or upgrade means adding an entry to a CONTENT table, not new logic. If a card needs new logic, say so and ask.
- Physics and content must be testable outside the browser (tests load them via Node). Keep them free of DOM calls.
- If `index.html` passes ~3,000 lines, propose splitting into `index.html` / `physics.js` / `content.js` — don't do it without approval.

## 5a. Hosting

- The game ships as a static site on **GitHub Pages**: `index.html` at the repo root, nothing else required. Every push to `main` updates the live site.
- **No backend.** Notebook, unlocks and ascension level persist in `localStorage`, wrapped in try/catch so the game still runs if storage is blocked. The daily seed comes from the browser date. The share card is drawn on a canvas.
- No external requests at runtime: no CDN scripts, fonts or analytics. The game must work offline once loaded.
- The repo is public and on David's personal account. Section 10 (public information only) applies to everything committed, including commit messages and screenshots.
- Test on a phone-sized viewport before every deploy. Don't add a server, database or leaderboard without asking.

## 6. The physics is sacred

- Models are **exact, not approximations**: Deal-Grove oxidation, Arrhenius temperature dependence, plasma etch-rate scaling, CD vs. exposure dose.
- Randomness and drift only change **inputs**: tool offsets, drift after PM, chamber buildup, incoming wafer quality, latent defects.
- **Never fudge a model to tune difficulty.** Tune with costs, specs, time, deck weights and CONFIG values.
- Physics functions are **pure** (no side effects, no RNG inside) with **units in comments** for every input and output.

## 7. Locked game rules — do not change without asking

- Click-to-assign lot dispatch (automation may assign for the player once unlocked).
- Recipe locks on LOAD.
- **Implant and oxidation are permanent — no rework.**
- Metrology is a judgment call: rework / accept / scrap. Metrology costs time and money.
- Diagnosis = pick a root cause; a **wrong guess costs money**.
- Time control: pause / play / 2x.
- Lose: `cash < 0` or `customerTrust <= 0`. Five quarters, then qual lot boss.
- Everything in section 2 is locked too.

## 8. Seeded randomness only

- **No `Math.random()` anywhere.** All randomness comes from the single seeded generator in RNG.
- Same seed → same run. Enables reproducible bugs, the daily seed, and the playtest bot.
- Always include the seed when reporting a bug or test failure.

## 9. Content templates

**Excursion card**
```js
{
  id: "chamber-wall-buildup",
  draft: true,
  rootCause: "...",
  tools: ["Plasma Etch"],
  nodeWeights: { standard: 1, pullIn: 2, maintenance: 0, event: 1 },
  telegraph: { signal: "etch rate creeping up on SPC", leadHrs: 24, seenBy: "etch-rate monitor" },
  signatures: { waferMap: "...", spc: "...", toolLog: "..." },
  redHerrings: ["..."],
  fix: "...",
  cost: { early: {...}, late: {...}, missed: {...} },
  physicsNote: "why this is physically plausible"
}
```

**Map event, upgrade, curse, boss:** small fixed templates with `id`, `name`, `description`, `effect` (CONFIG changes or flags), `cashPayout` (curses), `unlock`.

**Fairness rules**
- Every excursion must be diagnosable from in-game evidence, and every telegraph must be detectable by some metrology the player can buy.
- Every signature must be physically plausible. Write the `physicsNote`.
- Claude drafts content with `draft: true`. **David reviews all process content** and removes the flag.

## 10. Public information only

- Textbook and publicly published numbers only.
- No real company, tool vendor or brand names. No fab-specific recipes, nothing resembling non-public process data.
- If unsure whether a value is public/textbook, flag it rather than use it.

## 11. Testing and definition of done

1. **Physics checks** — textbook values David supplies. Never edit an expected value to make a test pass; ask.
2. **Bot runs** — ~100 seeded runs by a simple bot. Pass if: no crash, no `NaN`/`Infinity`, win rate in the target band (CONFIG), run length 20–40 simulated minutes.
3. **Playwright screenshots** — desktop and phone. Actually look at them.

A change is **done** when tests pass (or you've said which don't exist yet), screenshots are checked, one line is added to `CHANGELOG.md`, the `ROADMAP.md` checkbox is ticked, and David gets a **3-bullet summary**: what changed, why, how to see it in the game.

## 12. Agents

| Agent | Use it for | Edits code? |
|---|---|---|
| `physics-reviewer` | Any change to PHYSICS or a new excursion's `physicsNote` | No |
| `content-drafter` | Batches of excursion/event/upgrade cards (can run in parallel) | CONTENT only |
| `playtester` | Run all three test layers and review screenshots before "done" | No |
| `code-explainer` | Review a diff for beginner readability and rule violations | No |

Parallel agents only for independent content. Never two agents editing the same section.

## 13. Glossary

- **Lot** — a batch of wafers moving together. **WIP** — work in progress.
- **Recipe** — parameter set run on a tool. **PM** — preventive maintenance.
- **Metrology** — measuring wafers. **CD** — critical dimension.
- **SPC** — statistical process control charts. **Cpk** — process capability index.
- **Excursion** — a process going out of control. **Telegraph** — the early warning signal before it hits.
- **Root cause** — the actual source. **Rework** — strip and redo (not possible for implant/oxidation). **Scrap** — discard wafers.
- **Qual lot** — qualification lot the customer judges. **Ramp** — scaling a new product to volume.
- **Pull-in** — customer moving a deadline earlier. **Yield** — fraction of good die. **Wafer map** — die-by-die pass/fail picture.
