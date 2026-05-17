# Course Form Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move course creation and editing into a dedicated full-screen Course Form page, and slim the Course panel down to a picker.

**Architecture:** A new `#courseFormView` full-screen fixed overlay (hidden by default, shown for Add or Edit) operates on a *draft* copy of a course; nothing is written to `localStorage` until Save. The Course `<details>` panel loses its name field, 9/18 toggle and par grid — those live on the form page now. All changes are inline in the single-file `index.html`.

**Tech Stack:** Vanilla HTML/CSS/JS in one file (`index.html`). No build, no dependencies, no framework.

**Spec:** `docs/superpowers/specs/2026-05-17-course-form-page-design.md`

**Testing convention:** This project has **no automated tests** by deliberate choice (see `CLAUDE.md` — "No TDD ... Trust the browser to validate"). Every task is verified manually in a browser. To run the app, open `index.html` in a browser (double-click it, or run `start index.html`). After an edit, reload the tab. The service worker only registers over http(s) — for the offline check in Task 4, serve the folder instead (e.g. `python -m http.server 8000`, then load `http://localhost:8000`).

**Branch:** Work proceeds on the current branch (`worktree-new-feature`). Commit after each task.

---

## File Structure

| File | Responsibility | Change |
|------|----------------|--------|
| `index.html` | The entire app — HTML, CSS, JS inline | Modified throughout |

All work is in `index.html`. No new files (project constraint: single-file app).

New ids introduced: `courseFormView`, `courseFormCancel`, `courseFormTitle`, `courseFormSave`, `courseFormName`, `courseFormHint`, `courseFormHoles`, `courseFormPars`, `courseFormParTotal`, `editCourse`.

New JS symbols: `courseFormMode`, `courseFormEditId`, `courseFormDraft`, `courseFormInitial`, `openCourseForm()`, `closeCourseForm()`, `renderCourseForm()`, `updateCourseFormParTotal()`, `saveCourseForm()`.

---

## Task 1: Course Form page — markup and styles

Adds the full-screen form view to the DOM and its CSS. The view stays `hidden` and unwired, so the app is visually and behaviorally unchanged after this task.

**Files:**
- Modify: `index.html` — CSS in the `<style>` block; HTML after the `.container` element.

- [ ] **Step 1: Add the form's light-mode CSS**

In the `<style>` block, immediately **before** the line `@media (prefers-color-scheme: dark) {`, add:

```css
  /* Course form page */
  .course-form-view {
    position: fixed;
    inset: 0;
    z-index: 50;
    background: #f5f5f7;
    display: flex;
    flex-direction: column;
  }
  .course-form-view[hidden] { display: none; }
  .course-form-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    background: #0f6e56;
    color: white;
    padding: calc(env(safe-area-inset-top) + 12px) 16px 12px;
  }
  .course-form-header button {
    background: transparent;
    border: none;
    color: white;
    font-family: inherit;
    font-size: 15px;
    cursor: pointer;
    padding: 4px 2px;
  }
  .course-form-header .cf-save { font-weight: 600; }
  .course-form-title { font-size: 16px; font-weight: 600; }
  .course-form-body {
    flex: 1;
    overflow-y: auto;
    width: 100%;
    max-width: 600px;
    margin: 0 auto;
    padding: 16px 16px calc(env(safe-area-inset-bottom) + 24px);
  }
  .cf-label {
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    color: #8e8e93;
    margin: 16px 0 5px;
  }
  .cf-label:first-child { margin-top: 0; }
  .cf-name {
    width: 100%;
    height: 40px;
    border: 0.5px solid rgba(0,0,0,0.15);
    border-radius: 8px;
    background: white;
    color: inherit;
    font-family: inherit;
    font-size: 15px;
    padding: 0 12px;
  }
  .cf-hint {
    font-size: 12px;
    color: #d23f3f;
    min-height: 15px;
    margin-top: 6px;
  }
  .cf-par-total {
    text-align: center;
    font-size: 13px;
    font-weight: 600;
    color: #6b6b70;
    margin-top: 16px;
  }
```

- [ ] **Step 2: Add the form's dark-mode CSS**

Inside the `@media (prefers-color-scheme: dark) { ... }` block, immediately **before** its closing `}` (right after the existing `.toast { ... }` rule), add:

```css
    .course-form-view { background: #000; }
    .cf-name { background: #2c2c2e; color: #f5f5f7; border-color: rgba(255,255,255,0.12); }
    .cf-par-total { color: #8e8e93; }
```

- [ ] **Step 3: Add the Course Form view markup**

Find the `</div>` that closes `<div class="container">` (it is immediately followed by `<div id="toast" class="toast"></div>`). Insert the following **between** that `</div>` and the toast `<div>`:

```html
<div class="course-form-view" id="courseFormView" hidden>
  <div class="course-form-header">
    <button id="courseFormCancel">Cancel</button>
    <span class="course-form-title" id="courseFormTitle">New course</span>
    <button id="courseFormSave" class="cf-save">Save</button>
  </div>
  <div class="course-form-body">
    <div class="cf-label">Name</div>
    <input type="text" class="cf-name" id="courseFormName" placeholder="Course name" autocomplete="off" autocapitalize="words">
    <div class="cf-hint" id="courseFormHint"></div>
    <div class="cf-label">Holes</div>
    <div class="segmented" id="courseFormHoles">
      <button data-holes="9">9</button>
      <button data-holes="18">18</button>
    </div>
    <div class="cf-label">Par per hole</div>
    <div class="par-grid" id="courseFormPars"></div>
    <div class="cf-par-total" id="courseFormParTotal">Par 72</div>
  </div>
</div>
```

- [ ] **Step 4: Verify in the browser**

Open `index.html` in a browser. Expected: the app looks and behaves exactly as before; no Course Form is visible. Open DevTools → Console: no errors. In DevTools → Elements, find `<div id="courseFormView" ... hidden>` and untick the `hidden` attribute: a full-screen view appears with a teal header (Cancel · "New course" · Save) and an empty body with "Name" / "Holes" / "Par per hole" labels. Toggle your OS to dark mode and confirm the view background goes black and the name input is dark. Re-tick `hidden`.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Course Form page markup and styles"
```

---

## Task 2: Wire the Course Form page for adding courses

Adds the form's JavaScript (draft state, render, open/close, in-form editing, add-mode Save) and repoints the panel's existing "New" button at it. After this task you can add a course through the new page. The panel still shows its old name/holes/par-grid/Duplicate controls — that is intentional and removed in Task 4.

**Files:**
- Modify: `index.html` — new `// --- Course form page ---` JS section; replace the `#newCourse` click handler.

- [ ] **Step 1: Add the Course Form section — state, open, render, close**

Find the `// --- Reset round ---` comment in the `<script>`. Immediately **before** it, insert:

```js
  // --- Course form page ---
  const courseFormView     = document.getElementById('courseFormView');
  const courseFormTitle    = document.getElementById('courseFormTitle');
  const courseFormName     = document.getElementById('courseFormName');
  const courseFormHint     = document.getElementById('courseFormHint');
  const courseFormHoles    = document.getElementById('courseFormHoles');
  const courseFormPars     = document.getElementById('courseFormPars');
  const courseFormParTotal = document.getElementById('courseFormParTotal');

  let courseFormMode = null;
  let courseFormEditId = null;
  let courseFormDraft = null;
  let courseFormInitial = null;

  function updateCourseFormParTotal() {
    courseFormParTotal.textContent = 'Par ' + courseFormDraft.pars.reduce((a, b) => a + b, 0);
  }

  function renderCourseForm() {
    courseFormHoles.querySelectorAll('button').forEach(b => {
      b.classList.toggle('active', parseInt(b.dataset.holes, 10) === courseFormDraft.holes);
    });
    courseFormPars.innerHTML = '';
    for (let i = 0; i < courseFormDraft.holes; i++) {
      const wrap = document.createElement('div');
      wrap.className = 'par-cell';
      wrap.innerHTML = `
        <span>H${i+1}</span>
        <select data-cf-par="${i}">
          <option value="3"${courseFormDraft.pars[i]===3?' selected':''}>3</option>
          <option value="4"${courseFormDraft.pars[i]===4?' selected':''}>4</option>
          <option value="5"${courseFormDraft.pars[i]===5?' selected':''}>5</option>
          <option value="6"${courseFormDraft.pars[i]===6?' selected':''}>6</option>
        </select>
      `;
      courseFormPars.appendChild(wrap);
    }
    updateCourseFormParTotal();
  }

  function openCourseForm(mode, courseId) {
    courseFormMode = mode;
    if (mode === 'edit') {
      const c = courses.find(x => x.id === courseId) || currentCourse();
      courseFormEditId = c.id;
      courseFormDraft = { name: c.name, holes: c.holes, pars: c.pars.slice() };
    } else {
      courseFormEditId = null;
      courseFormDraft = { name: '', holes: 18, pars: defaultPars(18) };
    }
    courseFormInitial = JSON.stringify(courseFormDraft);
    courseFormTitle.textContent = mode === 'edit' ? 'Edit course' : 'New course';
    courseFormHint.textContent = '';
    courseFormName.value = courseFormDraft.name;
    renderCourseForm();
    courseFormView.hidden = false;
    if (mode === 'add') courseFormName.focus();
  }

  async function closeCourseForm(force) {
    if (!force && JSON.stringify(courseFormDraft) !== courseFormInitial) {
      const ok = await confirmDialog('Discard changes?', { okText: 'Discard', destructive: true });
      if (!ok) return;
    }
    courseFormView.hidden = true;
    courseFormMode = null;
    courseFormEditId = null;
    courseFormDraft = null;
    courseFormInitial = null;
  }
```

- [ ] **Step 2: Add the Save handler, in-form handlers, and button wiring**

Immediately **after** the code from Step 1 (still before `// --- Reset round ---`), insert:

```js
  async function saveCourseForm() {
    const name = courseFormDraft.name.trim();
    if (!name) {
      courseFormHint.textContent = 'Enter a course name';
      courseFormName.focus();
      return;
    }
    const hasScores = round.scores.some(s => s != null);
    if (hasScores && !roundSaved) {
      const ok = await confirmDialog('Adding a course will clear the current round. Continue?', { okText: 'Add course', destructive: true });
      if (!ok) return;
    }
    const c = { id: uid(), name, holes: courseFormDraft.holes, pars: courseFormDraft.pars.slice() };
    courses.push(c);
    currentId = c.id;
    saveCourses();
    round = newRound(currentId);
    saveRound();
    markRoundDirty();
    closeCourseForm(true);
    render();
  }

  courseFormName.addEventListener('input', () => {
    courseFormDraft.name = courseFormName.value;
    if (courseFormName.value.trim()) courseFormHint.textContent = '';
  });

  courseFormHoles.addEventListener('click', (e) => {
    const b = e.target.closest('button[data-holes]');
    if (!b) return;
    const newHoles = parseInt(b.dataset.holes, 10);
    if (newHoles === courseFormDraft.holes) return;
    if (newHoles > courseFormDraft.holes) {
      courseFormDraft.pars = courseFormDraft.pars.concat(defaultPars(18).slice(courseFormDraft.holes, newHoles));
    } else {
      courseFormDraft.pars = courseFormDraft.pars.slice(0, newHoles);
    }
    courseFormDraft.holes = newHoles;
    renderCourseForm();
  });

  courseFormPars.addEventListener('change', (e) => {
    const sel = e.target.closest('select[data-cf-par]');
    if (!sel) return;
    courseFormDraft.pars[parseInt(sel.dataset.cfPar, 10)] = parseInt(sel.value, 10);
    updateCourseFormParTotal();
  });

  document.getElementById('courseFormSave').addEventListener('click', saveCourseForm);
  document.getElementById('courseFormCancel').addEventListener('click', () => closeCourseForm(false));
```

- [ ] **Step 3: Repoint the "New" button at the form**

Find the `// --- New / duplicate / delete ---` comment and the `#newCourse` click handler directly below it. Replace this entire block:

```js
  // --- New / duplicate / delete ---
  document.getElementById('newCourse').addEventListener('click', async () => {
    const hasScores = round.scores.some(s => s != null);
    if (hasScores) {
      const ok = await confirmDialog('Starting a new course will clear the current scores. Continue?', { okText: 'New course', destructive: true });
      if (!ok) return;
    }
    const c = { id: uid(), name: 'New Course', holes: 18, pars: defaultPars(18) };
    courses.push(c);
    currentId = c.id;
    saveCourses();
    round = newRound(currentId);
    saveRound();
    markRoundDirty();
    render();
    document.getElementById('coursePanel').open = true;
    courseNameIn.focus();
    courseNameIn.select();
  });
```

with:

```js
  // --- New / duplicate / delete ---
  document.getElementById('newCourse').addEventListener('click', () => openCourseForm('add'));
```

- [ ] **Step 4: Verify in the browser**

Reload `index.html`. Open the **Course** panel, tap **New**. Expected: the full-screen form opens, titled "New course", name field focused/empty. Type "Test 9", tap the **9** holes button — the par grid shrinks to 9 rows and "Par total" updates. Change a par `<select>` — the total updates. Tap **Save** — you return to the scorecard; the header shows "Test 9 · Par …"; the scorecard has 9 holes, all empty.

Then test the guard paths:
- Tap **New**, tap **Cancel** immediately → closes with no prompt.
- Tap **New**, type a letter, tap **Cancel** → "Discard changes?" prompt appears.
- Tap **New**, leave name blank, tap **Save** → red "Enter a course name" hint shows, form stays open.
- On the scorecard, enter a score on hole 1, then tap **New**, type a name, tap **Save** → "Adding a course will clear the current round. Continue?" confirm appears.

Console: no errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: wire Course Form page for adding courses"
```

---

## Task 3: Support editing courses via the Course Form page

Adds an **Edit** button to the panel and extends `saveCourseForm` to handle edit mode, including the mid-round hole-count rules from spec §5.

**Files:**
- Modify: `index.html` — add `#editCourse` button to the panel; rewrite `saveCourseForm`; wire the Edit button.

- [ ] **Step 1: Add the Edit button to the panel**

Find this block in the Course panel markup:

```html
      <div class="panel-buttons">
        <button id="newCourse">New</button>
        <button id="duplicateCourse">Duplicate</button>
        <button id="deleteCourse" class="danger">Delete</button>
      </div>
```

Replace it with:

```html
      <div class="panel-buttons">
        <button id="newCourse">New</button>
        <button id="editCourse">Edit</button>
        <button id="duplicateCourse">Duplicate</button>
        <button id="deleteCourse" class="danger">Delete</button>
      </div>
```

- [ ] **Step 2: Rewrite `saveCourseForm` to handle Add and Edit**

Replace the **entire** `saveCourseForm` function (added in Task 2, Step 2) with:

```js
  async function saveCourseForm() {
    const name = courseFormDraft.name.trim();
    if (!name) {
      courseFormHint.textContent = 'Enter a course name';
      courseFormName.focus();
      return;
    }
    if (courseFormMode === 'add') {
      const hasScores = round.scores.some(s => s != null);
      if (hasScores && !roundSaved) {
        const ok = await confirmDialog('Adding a course will clear the current round. Continue?', { okText: 'Add course', destructive: true });
        if (!ok) return;
      }
      const c = { id: uid(), name, holes: courseFormDraft.holes, pars: courseFormDraft.pars.slice() };
      courses.push(c);
      currentId = c.id;
      saveCourses();
      round = newRound(currentId);
      saveRound();
      markRoundDirty();
      closeCourseForm(true);
      render();
      return;
    }
    const c = courses.find(x => x.id === courseFormEditId);
    if (!c) { closeCourseForm(true); return; }
    const holesChanged = c.holes !== courseFormDraft.holes;
    const affectsRound = holesChanged && c.id === currentId;
    if (affectsRound) {
      const hasScores = round.scores.some(s => s != null);
      if (hasScores && !roundSaved) {
        const ok = await confirmDialog('Changing hole count will clear the in-progress round. Continue?', { okText: 'Change', destructive: true });
        if (!ok) return;
      }
    }
    c.name = name;
    c.holes = courseFormDraft.holes;
    c.pars = courseFormDraft.pars.slice();
    saveCourses();
    if (affectsRound) {
      round = newRound(currentId);
      saveRound();
      markRoundDirty();
    }
    closeCourseForm(true);
    render();
  }
```

- [ ] **Step 3: Wire the Edit button**

In the `// --- Course form page ---` section, immediately **after** the line
`document.getElementById('courseFormCancel').addEventListener('click', () => closeCourseForm(false));`, add:

```js
  document.getElementById('editCourse').addEventListener('click', () => openCourseForm('edit', currentId));
```

- [ ] **Step 4: Verify in the browser**

Reload `index.html`. Open the **Course** panel, tap **Edit**. Expected: the form opens titled "Edit course", pre-filled with the current course's name, holes and pars.

- Change the name, tap **Save** → the scorecard header shows the new name.
- Enter a score on hole 1. Tap **Edit**, change hole 3's par, tap **Save** → the score on hole 1 is **kept**; hole 3's "par" label and the to-par total reflect the new par.
- With a score still entered, tap **Edit**, toggle the holes count (e.g. 18→9), tap **Save** → "Changing hole count will clear the in-progress round. Continue?" confirm appears; on confirm the scorecard now has the new hole count and no scores.
- Tap **Edit**, change something, tap **Cancel** → "Discard changes?" prompt; on confirm the course is unchanged.

Console: no errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: support editing courses via the Course Form page"
```

---

## Task 4: Slim the Course panel and remove dead code

Removes the name field, 9/18 toggle, par grid and Duplicate button from the panel, plus their now-orphaned handlers and DOM refs. The panel becomes: dropdown + "+ Add course" + Edit + Delete. Steps remove JavaScript first, then markup, so the file always loads without error.

**Files:**
- Modify: `index.html` — remove handlers; update `renderCoursePanel`; remove DOM refs; replace panel markup; add primary-button CSS.

- [ ] **Step 1: Remove the `duplicateCourse` handler**

Delete this entire block:

```js
  document.getElementById('duplicateCourse').addEventListener('click', () => {
    const src = currentCourse();
    const c = { id: uid(), name: src.name + ' copy', holes: src.holes, pars: src.pars.slice() };
    courses.push(c);
    currentId = c.id;
    saveCourses();
    round = newRound(currentId);
    saveRound();
    markRoundDirty();
    render();
  });
```

Then update the now-stale section comment just above the `newCourse` handler — change `// --- New / duplicate / delete ---` to `// --- Add / delete course ---`.

- [ ] **Step 2: Remove the panel's par, rename, and holes-toggle handlers**

Delete this block (the `// --- Par select ---` handler):

```js
  // --- Par select ---
  parConfigEl.addEventListener('change', (e) => {
    const sel = e.target.closest('select[data-par-hole]');
    if (!sel) return;
    const i = parseInt(sel.dataset.parHole, 10);
    currentCourse().pars[i] = parseInt(sel.value, 10);
    saveCourses();
    renderHeader();
    renderHoles();
    updateSummary();
  });
```

Delete this block (the `// --- Course rename ---` handler):

```js
  // --- Course rename ---
  courseNameIn.addEventListener('input', () => {
    const v = courseNameIn.value.trim() || 'Untitled';
    currentCourse().name = v;
    saveCourses();
    courseNameEl.textContent = v;
    // Update the dropdown option label without rebuilding everything
    const opt = courseSelect.querySelector(`option[value="${currentId}"]`);
    if (opt) opt.textContent = `${v} · ${currentCourse().holes}`;
  });
```

Delete this block (the `// --- 9 / 18 toggle ---` handler):

```js
  // --- 9 / 18 toggle ---
  holesToggle.addEventListener('click', async (e) => {
    const b = e.target.closest('button[data-holes]');
    if (!b) return;
    const newHoles = parseInt(b.dataset.holes, 10);
    const c = currentCourse();
    if (c.holes === newHoles) return;
    const hasScores = round.scores.some(s => s != null);
    if (hasScores) {
      const ok = await confirmDialog('Changing hole count will clear the current scores. Continue?', { okText: 'Change', destructive: true });
      if (!ok) return;
    }

    if (newHoles > c.holes) {
      // Extend pars with defaults
      const filler = defaultPars(18).slice(c.holes, newHoles);
      c.pars = c.pars.concat(filler);
    } else {
      c.pars = c.pars.slice(0, newHoles);
    }
    c.holes = newHoles;
    saveCourses();
    round = newRound(currentId);
    saveRound();
    markRoundDirty();
    render();
  });
```

- [ ] **Step 3: Simplify `renderCoursePanel`**

Replace the **entire** `renderCoursePanel` function:

```js
  function renderCoursePanel() {
    const c = currentCourse();
    // Course dropdown
    courseSelect.innerHTML = courses.map(co =>
      `<option value="${co.id}" ${co.id === currentId ? 'selected' : ''}>${escapeHtml(co.name)} · ${co.holes}</option>`
    ).join('');

    // Name input
    courseNameIn.value = c.name;

    // 9/18 segmented
    holesToggle.querySelectorAll('button').forEach(b => {
      b.classList.toggle('active', parseInt(b.dataset.holes, 10) === c.holes);
    });

    // Par grid
    parConfigEl.innerHTML = '';
    for (let i = 0; i < c.holes; i++) {
      const wrap = document.createElement('div');
      wrap.className = 'par-cell';
      wrap.innerHTML = `
        <span>H${i+1}</span>
        <select data-par-hole="${i}">
          <option value="3" ${c.pars[i]===3?'selected':''}>3</option>
          <option value="4" ${c.pars[i]===4?'selected':''}>4</option>
          <option value="5" ${c.pars[i]===5?'selected':''}>5</option>
          <option value="6" ${c.pars[i]===6?'selected':''}>6</option>
        </select>
      `;
      parConfigEl.appendChild(wrap);
    }
  }
```

with:

```js
  function renderCoursePanel() {
    courseSelect.innerHTML = courses.map(co =>
      `<option value="${co.id}" ${co.id === currentId ? 'selected' : ''}>${escapeHtml(co.name)} · ${co.holes}</option>`
    ).join('');
  }
```

- [ ] **Step 4: Remove the orphaned DOM refs**

In the `// --- DOM refs ---` block, delete these three lines:

```js
  const parConfigEl   = document.getElementById('parConfig');
  const courseNameIn  = document.getElementById('courseNameInput');
  const holesToggle   = document.getElementById('holesToggle');
```

Leave the other refs in that block (`holesEl`, `courseSelect`, `courseNameEl`, `dateBtn`, `dateInput`, `splitsEl`) untouched.

- [ ] **Step 5: Replace the Course panel markup**

Replace the entire `panel-body` of the Course panel:

```html
    <div class="panel-body">
      <div class="panel-row">
        <label>Course</label>
        <select id="courseSelect"></select>
      </div>
      <div class="panel-row">
        <label>Name</label>
        <input type="text" id="courseNameInput" placeholder="Course name" autocomplete="off" autocapitalize="words">
      </div>
      <div class="panel-row">
        <label>Holes</label>
        <div class="segmented" id="holesToggle">
          <button data-holes="9">9</button>
          <button data-holes="18">18</button>
        </div>
      </div>
      <div class="par-grid" id="parConfig"></div>
      <div class="panel-buttons">
        <button id="newCourse">New</button>
        <button id="editCourse">Edit</button>
        <button id="duplicateCourse">Duplicate</button>
        <button id="deleteCourse" class="danger">Delete</button>
      </div>
    </div>
```

with:

```html
    <div class="panel-body">
      <div class="panel-row">
        <label>Course</label>
        <select id="courseSelect"></select>
      </div>
      <div class="panel-buttons">
        <button id="newCourse" class="primary">+ Add course</button>
      </div>
      <div class="panel-buttons">
        <button id="editCourse">Edit</button>
        <button id="deleteCourse" class="danger">Delete</button>
      </div>
    </div>
```

- [ ] **Step 6: Add the primary panel-button style**

In the `<style>` block, find the rule `.panel-buttons button.danger { color: #d23f3f; }`. Immediately **after** it, add:

```css
  .panel-buttons button.primary {
    background: #0f6e56;
    color: white;
    border-color: transparent;
  }
  .panel-buttons button.primary:active { background: #0c5a47; }
```

- [ ] **Step 7: Verify — full regression pass**

Reload `index.html`. Console: **no errors on load**. Open the **Course** panel — it now shows only: the course dropdown, a teal **+ Add course** button, and an **Edit** / **Delete** row. No name field, no 9/18 toggle, no par grid, no Duplicate button.

Run the spec §7 checklist:
- **+ Add course** → form opens; create a 9-hole course and an 18-hole course → each is created, becomes active, starts an empty round.
- **Edit** → change the name → header updates.
- Enter scores, **Edit**, change a par only, **Save** → scores kept, to-par recomputes.
- With scores entered, **Edit**, change hole count, **Save** → confirm shown; round regenerates on confirm.
- With scores entered, **+ Add course**, **Save** → confirm shown.
- **+ Add course**, **Cancel** with no changes → silent close; with changes → "Discard changes?" prompt.
- Empty name on **Save** → blocked with the hint.
- **Delete** still works; the course **dropdown** still switches courses (and still shows the "Save & Switch" prompt when a round is in progress).
- Change a score, reload the page → the score persists.
- Toggle the OS between light and dark mode → the form page and panel render correctly in both.
- Offline check: serve the folder (`python -m http.server 8000`, load `http://localhost:8000`), then in DevTools → Application → Service Workers tick "Offline" and reload → the app still loads and works.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "refactor: slim the Course panel to a picker"
```

---

## Notes for the implementer

- **No `data-*` collisions:** the form's par selects use `data-cf-par`; the panel's old par selects used `data-par-hole`. They coexist between Tasks 1–4 without conflict, and `data-par-hole` is removed in Task 4.
- **Draft model:** the form never mutates a real course or `localStorage` until Save. `closeCourseForm(true)` skips the dirty check (used after a successful Save); `closeCourseForm(false)` runs it (used by Cancel).
- **Edit always targets the current course** — the panel's Edit button passes `currentId`. The `affectsRound` / non-current branch in `saveCourseForm` is defensive only.
- **Storage shape is unchanged** — `golf_courses_v1` stays `{ courses: [{id,name,holes,pars}], currentId }`. No migration, no backup changes.
- **Cancel does not call `render()`** — the form is a fixed overlay and never modifies the scorecard beneath it, so simply hiding it reveals the unchanged scorecard. Only Save re-renders (course/round changed). This is intentional, not a missed step against spec §4.6.
