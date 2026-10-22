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
