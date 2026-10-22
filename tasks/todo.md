# FabSim — Task Tracker

## Current: M0 — Move in

### Task: Get the game live and play it
- [x] Push `index.html` and the kit to a new GitHub repo (`chirality89/fabsim2`, on a working branch; merge to `main` before turning on Pages)
- [x] Turn on GitHub Pages (Settings → Pages → Deploy from branch `main`, folder `/ (root)`)
- [x] Confirm the live URL loads and plays on a phone (https://chirality89.github.io/Fabsim2/)
- [ ] Play 2–3 shifts by hand; log what feels wrong or unclear below
- [ ] Headless check: load the page, run one simulated shift with a simple bot, confirm no errors

**Acceptance criteria:**
- Live URL works on desktop and phone
- A simulated shift completes with no console errors (a failed Google Fonts request is fine)

**Playtest notes (David):**
-

**Progress notes:**
- 2026-10-22: Imported FabSim v2 and the kit into `fabsim2`; game code unchanged. Quick headless load: desktop (1280×800) renders the floor, lots and How to play dialog correctly. Phone (375 px) screenshot showed the dialog and top bar cut off on the right. Likely a limit of plain headless Chrome (it won't go narrower than ~500 px), not confirmed. Check on a real phone and with Playwright phone emulation (M1).
- 2026-10-22: Merged PR #1 to `main`; Pages builds from `main` / root. First load gave a 404 because Pages had not been saved yet; after saving, the build ran and David confirmed the live link works. The address is case-sensitive: `Fabsim2`, capital F. Still to note during the hand playtest: does the How to play box fit the phone screen?

## Done
- 2026-10-22: M0 item 1 — game and kit in `chirality89/Fabsim2`, live on GitHub Pages
