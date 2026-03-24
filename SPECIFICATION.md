# HamsterWheel — Product Specification

## Overview

HamsterWheel is a single-file (`index.html`) browser-based time tracker with no server, build step, or external dependencies. All state is persisted in `localStorage`. The core interaction model is a recurring popup that asks the user what they are currently working on, logging each response as a timed session.

---

## Data Model

All state is stored as a single JSON object in `localStorage` under the key `hamster_data`.

```
{
  settings: {
    intervalMinutes:   number,   // check-in popup frequency (default 5)
    retentionDays:     number,   // one-off task lifetime (default 7)
    logRetentionWeeks: number,   // log history window in weeks (default 2)
    alertSound:        boolean,
    alertTitleFlash:   boolean,
    alertFavicon:      boolean,
    taskSort:          string,   // 'added-asc'|'added-desc'|'alpha-asc'|'alpha-desc'|'recent'
    lastLocalBackup:   string,   // ISO 8601 timestamp of last local JSON download
    gistId:            string,
    gistAutoBackup:    boolean,
    gistLastBackup:    string,   // ISO 8601
    fsAutoBackup:      boolean,
    fsFileName:        string,
    fsLastBackup:      string    // ISO 8601
  },
  tasks:      Task[],
  log:        Entry[],
  active: {
    taskId:  string | null,      // ID of currently tracked task
    entryId: string | null       // ID of the open log entry
  },
  categories: string[]           // known category names, most-recently-used first (max 20)
}
```

The GitHub Personal Access Token is stored separately under the localStorage key `hamster_gist_token` and is never serialised into the state object.

### Task

```
{
  id:        string          // UUID
  name:      string
  recurring: boolean
  createdAt: string          // ISO 8601
  lastUsed:  string | null   // ISO 8601
}
```

### Entry

```
{
  id:        string          // UUID
  taskId:    string
  taskName:  string          // snapshot of task name at time of creation
  startTime: string          // ISO 8601
  endTime:   string | null   // null while the entry is active
  category:  string | null
}
```

> **Backward compatibility:** Entries created before the active-tracking feature may carry a `timestamp` field instead of `startTime`/`endTime`. All rendering and reporting code falls back to `entry.timestamp` when `entry.startTime` is absent.

---

## Features

### 1. Check-in Timer

- A countdown runs continuously in the page header showing time until the next check-in.
- When the interval elapses, a modal popup opens and the countdown resets.
- The interval is configurable in settings (1–480 minutes, default 5).
- A **Test check-in popup** button in settings triggers the popup immediately.
- If the browser tab is hidden when the interval fires, a Web Notifications API alert is sent (requires permission). Clicking it focuses the tab and opens the modal.
- `snoozeCheckin(minutes)` delays the next check-in by the given number of minutes (15, 30, or 60). It sets `nextCheckinAt = Date.now() + snoozeMs`, clears the existing `snoozeTimeoutId`, schedules a new one, and dismisses the modal.

### 2. Pause

A **⏥ Pause** toggle button sits in the header next to the settings icon. State is held in the module-level boolean `isPaused`.

- **Activating pause** (`isPaused = true`): calls `stopCurrentTask()`, clears `timerIntervalId` and `snoozeTimeoutId`, sets `nextCheckinAt = null`, displays "Paused" in the countdown badge, and adds `.is-paused` (orange) to the button.
- **Deactivating pause** (`isPaused = false`): calls `startTimer()` to resume the countdown, then immediately calls `openModal()`.
- `triggerCheckin()` returns early without opening the modal when `isPaused` is `true`.
- `updateCountdown()` skips its update when `isPaused` is `true`.

### 3. Tasks

#### Adding tasks
- Tasks are added via the form at the bottom of the Tasks card.
- Each task has a name and a recurring toggle.
- The card is collapsible: clicking the card header toggles the list and add-task form open or closed.
- Typing in the new-task input filters the task list in real time (`filterTaskList()`); items whose name does not contain the query are hidden. The filter is cleared after a task is added.

#### Recurring vs one-off
- **Recurring** tasks are never automatically removed.
- **One-off** tasks are pruned when they have not been used within `retentionDays` days, measured from `lastUsed` (or `createdAt` if never used).
- The currently active task is never pruned regardless of its retention status.
- Each one-off task displays a badge showing how many days remain; turns red at ≤ 2 days.

#### Task sort
The segmented control in the Tasks card header controls sort order. `setTaskSort(base)` toggles direction (`asc`/`desc`) if the base is already active, otherwise sets the base with `asc`. The value is stored in `state.settings.taskSort` as one of:

| Value | Meaning |
|---|---|
| `added-asc` / `added-desc` | Creation order |
| `alpha-asc` / `alpha-desc` | Alphabetical by name |
| `recent` | Most recently used first (no direction toggle) |

Legacy values `'manual'` and `'alpha'` are migrated to `'added-asc'` and `'alpha-asc'` on read. `getSortedTasks()` is a shared helper used by both `renderTasks()` and `openModal()`.

#### Task actions
- **▶ / ■** button — starts tracking the task (or stops it if it is the active task).
- **🔁 / 📌** button — toggles between recurring and one-off.
- **Trash** button — deletes the task; if it is the active task, tracking is stopped first.
- Clicking the task name also starts tracking.

### 4. Active Task Tracking

- Exactly one task can be tracked at a time.
- Starting a task:
  1. If a different task is already active, its log entry is closed (`endTime = now`).
  2. A new log entry is created with `startTime = now` (or a caller-supplied ISO string), `endTime = null`.
  3. `state.active` is updated to point to the new task and entry.
- Starting the currently active task again dismisses the modal without creating a new entry.
- Stopping (`stopCurrentTask`): sets `endTime = now` on the active entry, clears `state.active`.
- On page reload, the active task and its open entry are restored from `localStorage`. The entry resumes from its saved `startTime`, so elapsed time continues to accumulate.

#### "Now tracking" card
- Shown only when a task is active; hidden otherwise.
- Displays: pulsing green dot, task name, live elapsed duration (updates every second).
- Contains a **Stop** button and a **local backup button** (see §10).

#### Start-time picker in modal
- When a task is already active and the check-in modal opens, a `"Started at"` dropdown is populated with 5-minute interval timestamps from just after the current entry's `startTime` up to now (options at least 30 s before now), plus a final "Now" option (selected by default).
- Selecting an earlier time and then switching tasks sets the new entry's `startTime` to the chosen value and also sets the closing `endTime` of the previous entry to the same value, back-dating the switch.
- The picker is hidden when no task is active.

### 5. Check-in Modal

- Lists all current tasks as buttons (sorted by `getSortedTasks()`).
- The currently active task is highlighted green with a ▶ prefix.
- Clicking the active task dismisses the modal; tracking continues uninterrupted.
- Clicking any other task switches tracking to that task (using the selected start time from the picker).
- A text input at the bottom allows creating a new task and immediately starting it.
- The modal subtitle shows the current time (no active task) or the active task name and elapsed duration (task running).
- **Snooze buttons** — 15m / 30m / 1h — dismiss the modal and delay the next check-in.
- **Continue** — dismisses the modal without changing anything.
- The modal is dismissed by clicking outside, pressing Escape, or selecting/creating a task.
- Typing in the new-task input filters the task button list in real time (`filterModalTasks()`); buttons whose name does not contain the query string are hidden.

### 6. Time Log

Displayed as a scrollable list (newest first, capped at 100 rendered entries). The card is collapsible.

| Column | Content |
|---|---|
| Task name | Entry task name + category badge or `+ tag` button |
| Start | Start time (time only if today; date + time otherwise) |
| → | Arrow separator |
| End | End time, or a pulsing green dot for the active entry |
| Duration | Elapsed duration; live-updating for the active entry |

Duration format: `30s` / `4m 22s` / `1h 05m`.

#### Live updates
`updateActiveDuration()` runs every second (piggybacking on the countdown interval). It updates the active entry's duration cell and the "Now tracking" card via targeted DOM writes — the log list is not re-rendered.

#### Renaming log entries
- Clicking a task name in the log opens an inline text input pre-filled with the current name.
- Pressing Enter or tabbing away (with a non-empty value) saves to `entry.taskName`.
- Pressing Escape cancels without saving.

#### Editing times
- Start and end time spans carry class `log-time-edit` (dotted underline on hover).
- Clicking opens a `<input type="time">` in place of the span.
- Saving validates that start < end; alerts and reverts on violation.
- Escape cancels.
- State variables: `activeTimeEditEntryId`, `activeTimeEditField` (`'start'` or `'end'`).

#### Quick-backdate buttons

For the currently active entry, `renderLog()` renders quick-backdate buttons (`backdate-btn`) next to the start time: −5m, −15m, −30m, −1h. Each snaps the start time to the previous wall-clock boundary for that interval. Buttons are omitted when:
- The target time is at or after the current start time.
- The target time is at or before the preceding entry's own start time.
- Two intervals resolve to the same boundary (de-duplicated via a `Set` of ISO strings).

Clicking calls `backdateEntry(entry.id, iso)`. `backdateEntry` sets the entry's `startTime` to the new ISO string, then adjusts the preceding (older) entry's `endTime` to match — **unless the gap between `prevEntry.endTime` and `newStartIso` exceeds 15 minutes**, in which case a `confirm()` dialog asks the user first. If the user declines, only the active entry's start time is changed and the function returns early after saving and re-rendering.

#### Gap indicators
When two consecutive log entries have a gap of more than 1 minute, a dashed separator row appears between them showing the gap duration. Three buttons appear on hover:

| Button | Action |
|---|---|
| **Extend end ↑** | `closeGapExtendEnd(entryId, newEndIso)` — sets older entry's `endTime` to newer entry's `startTime` |
| **↓ Pull start** | `closeGapPullStart(entryId, newStartIso)` — sets newer entry's `startTime` to older entry's `endTime` |
| **+ Fill gap** | Calls `toggleAddEntryPanel(startIso, endIso)` — opens the add-entry panel pre-filled with the gap bounds |

### 7. Adding Manual Entries

`toggleAddEntryPanel(startIso?, endIso?)` opens a form accepting optional ISO strings to pre-fill start and end times. On submit:
- If the task name matches an existing task (case-insensitive) it is linked; otherwise a new task is created.
- If no end time is provided and the entry falls between two existing entries, `endTime` is auto-set to the next entry's `startTime`.

### 8. Categories

- Any log entry can be tagged with a free-text category.
- Clicking `+ tag` (or an existing category badge) opens an **inline picker** that expands within the log row.
- The picker contains a text input, up to 10 recent-category chips, and a **✕ clear** chip when a category is set.
- Saving: Enter, click a chip, or click/tab away with a value.
- Cancelling: Escape, or click away with an empty input.
- `mousedown` is used on chips to fire before the input's `blur` event.
- All category text is set via `textContent` / DOM methods — no HTML injection path.
- `state.categories` — most-recently-used first, max 20, deduplicated on save.
- Colors: `CAT_PALETTE` (8 colors) assigned by consistent hash of category name. The same name always gets the same color everywhere.

### 9. Weekly Report

- Located below the time log.
- Shows a Monday–Sunday week (ISO week, local timezone).
- Dimensions: categories (rows) × days (columns) + Total column + Total footer row.
- Row order: named categories alphabetically, "Uncategorized" last.
- Today's column is highlighted (hidden for past weeks).
- Duration format in cells: `< 1m` / `45m` / `1h 30m` (nearest minute, no seconds).
- The active entry's contribution uses `Date.now()` as the effective end time.
- Horizontally scrollable on narrow viewports.
- Re-renders on every `renderAll()` call.

#### Week navigation
`weekOffset` (module-level variable, default `0`) controls which week is displayed. `shiftWeek(delta)` clamps the offset to ≤ 0 (future weeks are not accessible). `getWeekDays(offset?)` uses `weekOffset` when no argument is supplied. The header title reads "This week", "Last week", or "N weeks ago". The **›** button is hidden when `weekOffset === 0`.

### 10. PDF Report

`generateWeekPDF()` opens a new browser tab with a self-contained print-ready HTML page for whichever week is currently displayed (`weekOffset`). It auto-triggers `window.print()` after 400 ms. If the popup is blocked, an alert is shown.

Two sections:

#### Weekly Summary by Task
One row per unique task name; one column per day; Total column; Total footer row. Empty cells show `—`.

#### Daily Breakdown
One subsection per day with entries, sorted ascending by start time: task name, category, start (HH:MM), end (HH:MM or `…`), duration. A "Day total" footer row. Each subsection has `page-break-inside: avoid`.

All user text is set via `textContent` / `esc()` — no HTML injection path.

### 11. Settings

| Setting | Range | Default |
|---|---|---|
| Check-in interval | 1–480 min | 5 min |
| Task retention | 1–365 days | 7 days |
| Log retention | 1–52 weeks | 2 weeks |

Saving settings restarts the countdown timer from zero.

### 12. Pruning

#### Task pruning — `pruneExpiredTasks()`
Filters `state.tasks`, removing one-off tasks where `Date.now() - ref >= retentionMs`. The active task is always kept. Called at startup and on every check-in.

#### Log pruning — `pruneLog()`
Filters `state.log`, removing entries whose `startTime` (or `timestamp`) is older than `logRetentionWeeks * 7` days. The active entry is always kept. Calls `saveData()` only if entries were removed. Called at startup and on every check-in.

### 13. Local Backup Button

A small download icon button in the "Now tracking" card header. Normal state: dim/transparent. Overdue state (last local backup > 24 h ago): slow amber pulse animation.

- Clicking calls `localBackupFromCard()`: runs `exportData()`, sets `state.settings.lastLocalBackup = now`, calls `saveData()`, and removes the overdue class.
- `renderActiveCard()` evaluates overdue status on every call (driven by the 1-second tick).

### 14. Backup & Restore

#### Local JSON export/import
- `exportData()` — serialises `state` to indented JSON and triggers a browser download named `hamsterwheel-backup-YYYY-MM-DD_HH-MM.json` (timestamp at time of export). Does not modify state or record `lastLocalBackup` (that is done by `localBackupFromCard()`).
- `importData(event)` — parses the file, validates `tasks` and `log` arrays, confirms with the user, replaces state, fills missing fields for backward compatibility, saves, and re-renders.

#### Cloud backup — GitHub Gist
- Token stored under `hamster_gist_token` (separate from state; never exported).
- `backupToGist(silent)` — creates (POST) or updates (PATCH) a secret Gist. `_gistSaving` flag prevents re-entrant calls.
- `restoreFromGist()` — fetches and restores with confirmation.
- `scheduleGistBackup()` — debounces 10 s after each `saveData()` call when auto-backup is on.

#### File system backup — File System Access API (Chromium only)
- `FileSystemFileHandle` stored in IndexedDB (`hamsterwheel` DB, `fsh` store, key `'backup'`).
- `initFsBackup()` — restores handle on load; if permission is `'granted'` sets `_fsHandle`; otherwise sets `_fsPendingHandle` (shows Reconnect button).
- `writeToFsHandle()` — writes JSON to the file and toasts the result.
- `_fsSaving` flag prevents re-entrant calls.
- `scheduleFsBackup()` — debounces 10 s after each `saveData()` call when auto-save is on.

#### Toast notifications
`showToast(msg, isError)` — fixed bottom-right `#toast` element; 3.5 s auto-dismiss; `.toast-error` class for failures. Used for all backup results.

---

## Architecture

- **Single file** — all HTML, CSS, and JavaScript in `index.html`.
- **No build step** — open directly via `file://` in any modern browser.
- **No external dependencies** — no frameworks, no CDN resources.
- **Storage** — `localStorage` (state) + `IndexedDB` (filesystem handle) + `localStorage` (Gist token).
- **Timers** — one `setInterval` firing every second for the countdown and live-duration updates; a separate `setInterval` firing every `intervalMinutes` minutes for check-ins.
- **Rendering** — imperative DOM manipulation; no virtual DOM. `renderAll()` is the top-level render call. `updateActiveDuration()` performs targeted live updates without a full re-render.

---

## Key Functions

| Function | Purpose |
|---|---|
| `startTask(taskId, startISO?)` | Begin tracking a task; close any open entry first |
| `stopCurrentTask()` | Close the active entry without starting another |
| `togglePause()` | Toggle pause mode; stops task + timer when activating, restarts + opens modal when deactivating |
| `filterTaskList()` | Show/hide task card items based on new-task input text |
| `filterModalTasks()` | Show/hide modal task buttons based on new-task input text |
| `triggerCheckin()` | Fire notification + prune + open modal |
| `snoozeCheckin(minutes)` | Delay the next check-in and dismiss the modal |
| `populateModalStartPicker()` | Fill the "Started at" dropdown in the check-in modal |
| `getModalStartISO()` | Return the selected start time from the modal picker |
| `openCategoryPicker(entryId)` | Show inline category editor for a log entry |
| `setCategory(entryId, cat)` | Persist a category and update the known-categories list |
| `openNameEditor(entryId)` | Show inline name input for a log entry |
| `setEntryName(entryId, name)` | Persist a renamed log entry task name |
| `openTimeEditor(entryId, field)` | Show inline time input for start or end |
| `saveTimeEdit(entryId, field, value)` | Validate and persist an edited time |
| `backdateEntry(entryId, newStartIso)` | Set active entry start time; optionally adjust preceding entry's end time (prompts if gap > 15 min) |
| `getSortedTasks()` | Return tasks sorted per `state.settings.taskSort` |
| `setTaskSort(base)` | Update task sort, toggling direction if already active |
| `getWeekDays(offset?)` | Return 7 Date objects for the target Mon–Sun week |
| `shiftWeek(delta)` | Adjust `weekOffset` and re-render the weekly report |
| `buildWeekData(days)` | Aggregate log entries into a category × day ms matrix |
| `renderWeekReport()` | Render the weekly report table and update nav controls |
| `generateWeekPDF()` | Open a print-ready report tab and trigger `window.print()` |
| `exportData()` | Download current state as a JSON file |
| `localBackupFromCard()` | Download backup + record `lastLocalBackup` + update button |
| `importData(event)` | Validate and restore state from an uploaded JSON file |
| `backupToGist(silent)` | Create or update the GitHub Gist backup |
| `restoreFromGist()` | Fetch and restore state from the Gist |
| `scheduleGistBackup()` | Debounced auto-backup trigger |
| `writeToFsHandle()` | Write state JSON to the filesystem file handle |
| `scheduleFsBackup()` | Debounced auto-save trigger |
| `pruneExpiredTasks()` | Remove one-off tasks past their retention window |
| `pruneLog()` | Remove log entries older than `logRetentionWeeks` weeks |
| `renderAll()` | Re-render active card, task list, log, and week report |
| `updateActiveDuration()` | Targeted live update of active entry duration display |
| `showToast(msg, isError)` | Display a temporary notification in the bottom-right corner |
| `saveData()` | Write `state` to `localStorage`; trigger debounced auto-backups |
| `toggleLog(event)` | Collapse or expand the time log card |
| `toggleTasks(event)` | Collapse or expand the tasks card |
