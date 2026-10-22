# Start here

This kit is built around your single-file FabSim v2 (the isometric fab with 8 tools, hidden drift, SPC charts and cash). `index.html` in this folder is that game, copied from your artifact exactly as published.

## Set it up (10 minutes)

1. Unzip this folder somewhere you keep code and rename it `fabsim2`.
2. Create the repo and push it. With the GitHub CLI:
   ```
   cd fabsim2
   git init -b main
   git add .
   git commit -m "Import FabSim v2 and Claude Code kit"
   gh repo create fabsim2 --public --source=. --push
   ```
   Or create an empty repo `fabsim2` on github.com and follow its push instructions.
3. Turn on GitHub Pages: repo **Settings → Pages → Deploy from a branch → `main` / `/ (root)`**. After a minute the game is live at `https://chirality89.github.io/fabsim2/`.
4. Open Claude Code in the `fabsim2` folder.

## First prompt

> Read CLAUDE.md, PROJECT.md and ROADMAP.md. Use plan mode. We're on M0. Don't change any code yet. Load index.html in a headless browser, run one simulated shift with a simple bot, and tell me whether it runs cleanly. Then write a plan to tasks/todo.md for M1.

## Notes

- **FabForge stays as it is.** Leave `chirality89/fabforge` alone as an archive. Its Lab upgrade tree and tech-generation idea could feed the roguelite meta layer later, but nothing from it is needed now.
- The only external request in the game is Google Fonts. It falls back to system fonts if blocked.
- I checked that the file loads, that tests can reach `PHYSICS` and the game state directly, and that a simple bot can play a full shift. The shift took cash from $20,000 to $38,575 with 4 lots shipped and 3 tool alarms. That is a sanity check, not a balance result.
