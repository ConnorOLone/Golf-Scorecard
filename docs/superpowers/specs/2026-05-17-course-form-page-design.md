# Course Form Page — Design Spec

**Date:** 2026-05-17
**App:** Golf Scorecard (single-file PWA, `index.html`)
**Status:** Approved design — ready for implementation planning

## 1. Summary

Today, creating and editing a golf course both happen inline inside the
collapsible **Course** `<details>` panel — name field, 9/18 toggle, and a
per-hole par grid, crammed alongside the course picker and management
buttons. This conflates *editing course content* with *picking and managing
courses*.

This change introduces a dedicated full-screen **Course Form page** that
handles both adding a new course and editing an existing one. The Course
panel slims down to a pure picker: course dropdown plus **Add course**,
**Edit**, and **Delete**.

## 2. Goals / Non-goals

**Goals**

- Move course creation and editing onto a dedicated, roomy, full-screen page.
- Use one shared form for both Add and Edit.
- Slim the Course panel to picking + management only.

**Non-goals**

- No new course fields (course rating, slope, tees, location) — fields stay
  name / holes / pars.
- No URL routing or hash navigation — the page is an in-app view toggle.
- The **Duplicate course** feature is removed (redundant).
- No data-model or storage changes; no multi-player.

## 3. The two screens

### 3.1 Course Form page

A full-screen in-app view, toggled show/hide with no URL routing — the same
in-app-view idea as the History view, but rendered as a full-screen fixed
overlay (`position: fixed; inset: 0`) rather than an in-flow section. Hidden
by default; shown when adding or editing.

**Layout:**

- **Sticky header** (brand teal `#0f6e56`, safe-area aware): **Cancel**
  (left) · title (center) · **Save** (right). Title is "New course" (add
  mode) or "Edit course" (edit mode).
- **Scrollable body:**
  - **Name** — text input.
  - **Holes** — 9/18 segmented control.
  - **Par per hole** — 3-column par grid (`<select>`, par 3–6 per hole),
    reusing the existing `.par-grid` / `.par-cell` styles.
  - **Par total** — live-updating computed total (e.g. "Par 72").

### 3.2 Slimmed Course panel

The Course `<details>` panel keeps only:

- Course **dropdown** — selects the active course. Unchanged behavior,
  including the existing "Save & Switch" prompt when an unsaved round is in
  progress.
- **+ Add course** — primary button → opens the form in Add mode.
- **Edit** — opens the form in Edit mode for the currently-selected course.
- **Delete** — unchanged (confirm, remove, fall back to the first course,
  start a fresh round).

Removed from the panel: the name input, the 9/18 toggle, the par grid, the
old **New** button, and the **Duplicate** button.

## 4. Behavior

### 4.1 Draft / working copy

When the form opens it operates on a **draft** object `{ name, holes, pars }`.
All edits mutate only the draft. `golf_courses_v1` and the live course
objects are untouched until **Save**. **Cancel** discards the draft.

This is a deliberate change from today's apply-instantly panel edits — a page
with a Cancel button needs a commit/discard model.

### 4.2 Add mode — via "+ Add course"

- Opens blank: name empty (placeholder "Course name"), holes = 18, default
  18-hole pars. Title "New course". The name field is focused on open.
- **Save:**
  1. Validate name (§4.4).
  2. If the current in-progress round has unsaved scores, confirm first
     (§5.3).
  3. Create `{ id: uid(), name, holes, pars }`, append to `courses`, set
     `currentId` to it, persist.
  4. Start a fresh round on the new course, persist.
  5. Close the form, re-render the scorecard.
- Replaces today's inline `newCourse` handler.

### 4.3 Edit mode — via "Edit"

- Opens pre-filled with the currently-selected course's `name` / `holes` /
  `pars`. Title "Edit course".
- **Save:**
  1. Validate name.
  2. Apply the hole-count-change edge-case rules (§5).
  3. Write the draft back onto the course object, persist.
  4. Close the form, re-render the scorecard.

### 4.4 Validation

Name is required. On Save, if the trimmed name is empty: show an inline hint
("Enter a course name"), focus the name field, and stay on the page.

### 4.5 Cancel

- Draft unchanged from when the form opened → close immediately.
- Unsaved changes present → "Discard changes?" confirm (existing
  `confirmDialog`). Confirm closes the form; cancel stays.

### 4.6 Returning

Save and Cancel both hide the form view and re-render the scorecard
underneath.

## 5. Edge cases — editing the active course mid-round

Edit always targets the currently-selected course, which owns the in-progress
round. (To edit a different course, the user selects it in the dropdown
first, which switches to it.) Three cases:

### 5.1 Par values change, hole count unchanged

The common, safe case. The round's score array still fits. Save applies the
new pars and re-renders; to-par recomputes; **scores are kept**. No confirm.

### 5.2 Hole count change (9 ↔ 18)

The score array no longer fits — the round must be regenerated. On Save:

- Round has unsaved scores → destructive confirm: *"Changing hole count will
  clear the in-progress round. Continue?"* → apply + regenerate a fresh round.
- No scores → apply + regenerate the round silently.

Toggling 9/18 *inside the form* only resizes the draft's par array (extend
with default pars / truncate) and re-renders the grid — no round effect until
Save.

### 5.3 Add mode discards the current round

Save in Add mode replaces the current round with a fresh one on the new
course. If the current round has unsaved scores → destructive confirm first
(same pattern as today's New-course guard).

### 5.4 Definitions & rules

- **"Unsaved scores"** = the round has at least one entered score **and** is
  not already saved to history (`roundSaved` is false).
- Confirms render *over* the form page (modal `z-index` wins). Confirm →
  apply + close the form; Cancel → stay on the form.
- The draft commits only after any confirm passes — Cancel anywhere leaves
  real course data untouched.
- This change does **not** add a "save round to history first" option here —
  that stays exclusive to course-switching. Mid-round hole-count changes are
  rare; a plain confirm is sufficient.

## 6. Implementation notes

- Single-file: all changes in `index.html` (HTML + CSS + JS inline), per
  project constraints. No new files, no dependencies, vanilla JS.
- **New markup:** a `#courseFormView` full-screen view (`position: fixed;
  inset: 0`, hidden by default, `z-index` below the modals), with its header
  and body, plus CSS for both light and `prefers-color-scheme: dark`.
- **New JS:** `openCourseForm(mode, courseId?)`, `closeCourseForm()`, the
  draft state, the form's par-grid render, the Save/Cancel handlers, and name
  validation.
- **Reuse:** `.segmented` styles for the 9/18 toggle; `.par-grid` /
  `.par-cell` styles for the par grid; `confirmDialog` for confirms; the
  `uid()`, `defaultPars()`, `newRound()`, and `render()` helpers.
- **Remove:** the `duplicateCourse` handler and its button; the old
  `newCourse` inline-create handler; the panel's name-input, 9/18-toggle, and
  par-grid change handlers (their logic moves into the form, operating on the
  draft).
- **Keep:** the `courseSelect` change handler (including its Save-&-Switch
  flow) and the `deleteCourse` handler.
- **Data model:** `golf_courses_v1` shape is unchanged
  (`{ courses: [{ id, name, holes, pars }], currentId }`). No new storage
  keys, no migration. Backup / export is unaffected.

## 7. Testing (manual — no TDD, per project convention)

- Add a course (both 9 and 18 holes) → verify it is created, becomes active,
  and starts a fresh round.
- Edit a course's name → the header course name updates.
- Edit pars only, mid-round → scores kept, to-par recomputes.
- Edit hole count mid-round with scores entered → confirm is shown; round
  regenerates on confirm.
- Add a course mid-round with unsaved scores → confirm is shown.
- Cancel with no changes → closes silently; Cancel with changes → "Discard
  changes?" prompt.
- Empty name on Save → blocked with the inline hint.
- Offline (service worker) still works; data persists across reload.
- Light and dark mode both render the form correctly.
