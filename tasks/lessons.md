# Lessons Learned

Add an entry after any bug or correction. Format: what happened → root cause → rule to prevent it.

## Template

### What Happened
[2–3 sentences]

### Root Cause
[Why it happened]

### Rule to Prevent
✅ [Specific rule]
✅ [Test or check that would catch it]

---


## Bot reset its repair memory mid-repair

### What Happened
In the M0 headless check, the bot swapped the wrong part 50 times in one shift. A tool's three possible faults should take at most two wrong guesses.

### Root Cause
The bot cleared its "parts tried" count whenever a tool wasn't `down`. A wrong part puts the tool in `maint` for 30 minutes and then back to `down`, so the count was wiped every time and the bot always guessed the first fault.

### Rule to Prevent
✅ Bot memory about a tool resets only when the tool is healthy (`idle` or `run`), never during `maint`
✅ In bot reports, flag wrong parts above 2 per alarm. It means the bot is broken, not the game

## Notes edit silently did nothing

### What Happened
The M0 bot-check results were supposed to go into `tasks/todo.md` and were reported as recorded, but they never landed. The PR description said they were there.

### Root Cause
A scripted find-and-replace looked for "Does the How to play box…" but the file said "does". The replace found no match, made no change and raised no error.

### Rule to Prevent
✅ After any scripted text edit, check the diff shows the new text before committing or reporting it as done
✅ Prefer the Edit tool for notes, since it fails loudly when the old text isn't found
