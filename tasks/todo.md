# FabSim — Task Tracker

## Current: M0 — Move in

### Task: Get the game live and play it
- [x] Push `index.html` and the kit to a new GitHub repo (`chirality89/fabsim2`, on a working branch; merge to `main` before turning on Pages)
- [ ] Turn on GitHub Pages (Settings → Pages → Deploy from branch `main`, folder `/ (root)`)
- [ ] Confirm the live URL loads and plays on a phone
- [ ] Play 2–3 shifts by hand; log what feels wrong or unclear below
- [ ] Headless check: load the page, run one simulated shift with a simple bot, confirm no errors

**Acceptance criteria:**
- Live URL works on desktop and phone
- A simulated shift completes with no console errors (a failed Google Fonts request is fine)

**Playtest notes (David):**
-

**Progress notes:**
- 2026-10-22: Imported FabSim v2 and the kit into `fabsim2`; game code unchanged. Quick headless load: desktop (1280×800) renders the floor, lots and How to play dialog correctly. Phone (375 px) screenshot showed the dialog and top bar cut off on the right. Likely a limit of plain headless Chrome (it won't go narrower than ~500 px), not confirmed. Check on a real phone and with Playwright phone emulation (M1).

## Done
- (nothing yet)
