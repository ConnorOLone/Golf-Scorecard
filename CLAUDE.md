# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Structure & Edit Rules

- **Single-file application.** All code is in `index.html` (~1865 lines). No build process, no separate files to manage.
- **Don't split the file.** Keep CSS, HTML, and JavaScript inline. Connor prefers monolithic structure.
- **Edit directly.** When modifying features, edit the relevant sections in index.html without creating new files.
- **Service Worker is separate.** `sw.js` handles offline caching; edit it independently if cache strategy changes.

## Data Model

**localStorage keys** (don't change names—they determine data shape):
- `golf_courses_v1`: `{ courses: [...], currentId }` — list of courses with id, name, holes, pars array
- `golf_round_v2`: `{ courseId, scores: [...], date }` — current round (array length must match course.holes)
- `golf_history_v1`: array of saved rounds with date, courseId, courseName, holes, scores, pars, total, toPar, thru

**Migration path:** Legacy `golf_round_v1` is read once at init and pars are migrated to courses. After migration, it's never touched.

## Rendering & State

- **render()** is the entry point — calls renderHeader, renderCoursePanel, renderHoles, updateSummary in sequence.
- **updateHoleRow(i)** updates a single hole without full re-render (use after score changes).
- **State is reactive to localStorage.** Render functions read from `courses`, `currentId`, `round` (in-memory variables). Always call saveRound/saveCourses after mutations.
- **No framework.** Direct DOM manipulation via querySelector, innerHTML, addEventListener. Be careful with reflows.

## Score Entry

Three paths converge at `onScoreChanged(i)`:
1. Stepper buttons (±) — increment/decrement, clamped 1–25, null when decremented below 1
2. Modal keypad (tap score number) — direct assignment 1–12
3. Long-press hole row (550ms) — clears to null (has vibration feedback)

After any change:
- Call `saveRound()` first (persists)
- Call `updateHoleRow(i, { bump: true })` (animation + label refresh)
- Call `updateSummary()` (recalculates to-par, splits)
- Call `scrollToNextHole()` (auto-focuses next unfilled)
- Call `markRoundDirty()` (button state)

## Testing Requirements

**Always test after feature changes:**
1. **Offline mode:** DevTools → Application → Service Workers → simulate offline, reload, verify app works
2. **Data persistence:** Change a score, reload page, check it's still there
3. **Modal interactions:** Escape key, backdrop click, Enter submit, number key shortcuts in keypad
4. **Edge cases:** Switch courses mid-round (clears scores), toggle 9/18 holes (resizes array), empty round save (error dialog), full localStorage (backup may fail)

## No TDD

Connor works directly without tests. Trust the browser to validate. If you add features, test them manually in the browser. No test files.

## Colors & Styling

- Primary brand: `#0f6e56` (teal, buttons, headers, highlights)
- Dark mode is in `@media (prefers-color-scheme: dark)` block — keep both light & dark vars in sync
- Use system font stack: `-apple-system, BlinkMacSystemFont, "SF Pro Text", system-ui, sans-serif`
- Safe-area insets are applied to body padding (notch support)

## When Modifying Storage

- **Never delete old keys.** Add migration logic if renaming.
- **Validate on load.** Check type, array length, structure before use.
- **Test round mismatch.** If courseId doesn't match current course or holes don't align, generate fresh round.
- **Backup includes all keys.** If adding new storage, update buildBackup() and isValidBackup().

## When Adding Features

- Update HTML structure only if necessary (keep it minimal)
- Add CSS for new elements (follow existing selectors: `.class-name { }`)
- Add event listeners via `document.addEventListener` or element.addEventListener; attach to relevant parent elements
- If UI is complex, use helper functions (e.g., `renderX()`, `computeY()`)
- Always persist changes to localStorage immediately, not on blur/submit
- Test offline before committing

## Constraints

- Don't add npm dependencies or build tooling. Stay vanilla JavaScript.
- Don't create separate CSS or JS files.
- Don't use frameworks (React, Vue, etc.).
- Avoid commenting code. Write clear variable names instead.
- Keep the file under 2000 lines if possible (already at 1865; be mindful of bloat).
