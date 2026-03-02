# HamsterWheel — Product Specification

## Overview

HamsterWheel is a single-file (`index.html`) browser-based time tracker with no server, build step, or external dependencies. All state is persisted in `localStorage`. The core interaction model is a recurring popup that asks the user what they are currently working on, logging each response as a timed session.

---

## Data Model

All state is stored as a single JSON object in `localStorage` under the key `hamster_data`.

```
{
  settings: {
    intervalMinutes: number,   // check-in popup frequency
    retentionDays:   number    // one-off task lifetime
  },
  tasks: Task[],
  log:   Entry[],
  active: {
    taskId:  string | null,    // ID of currently tracked task
    entryId: string | null     // ID of the open log entry
  },
  categories: string[]         // known category names, most-recently-used first (max 20)
}
```

### Task

```
{
  id:        string   // UUID
  name:      string
  recurring: boolean
  createdAt: string   // ISO 8601
  lastUsed:  string | null  // ISO 8601
}
```

### Entry

```
{
  id:        string         // UUID
  taskId:    string
  taskName:  string         // snapshot of task name at time of creation
  startTime: string         // ISO 8601
  endTime:   string | null  // null while the entry is active
  category:  string | null
}
```

> **Backward compatibility:** Entries created before the active-tracking feature may carry a `timestamp` field instead of `startTime`/`endTime`. All rendering and reporting code falls back to `entry.timestamp` when `entry.startTime` is absent.

---

## Features

### 1. Check-in Timer

- A countdown runs continuously in the page header showing time until the next check-in.
- When the interval elapses, a modal popup opens and the countdown resets.
- The interval is configurable in settings (1–480 minutes, default 30).
- A "Test check-in popup" button in settings triggers the popup immediately.
- If the browser tab is hidden when the interval fires, a Web Notifications API alert is sent (requires permission). Clicking it focuses the tab and opens the modal.

### 2. Tasks

#### Adding tasks
- Tasks are added via the form at the bottom of the Tasks card.
- Each task has a name and a recurring toggle.
- The card is collapsible: clicking the card header toggles the list and add-task form open or closed. The card is open by default.

#### Recurring vs one-off
- **Recurring** tasks are never automatically removed.
- **One-off** tasks are pruned when they have not been used within `retentionDays` days, measured from `lastUsed` (or `createdAt` if never used).
- The currently active task is never pruned regardless of its retention status.
- Each one-off task displays a badge showing how many days remain; turns red at ≤ 2 days.

#### Task actions
- **▶ / ■** button — starts tracking the task (or stops it if it is the active task).
- **🔁 / 📌** button — toggles between recurring and one-off.
- **Trash** button — deletes the task; if it is the active task, tracking is stopped first.
- Clicking the task name also starts tracking.

### 3. Active Task Tracking

- Exactly one task can be tracked at a time.
- Starting a task:
  1. If a different task is already active, its log entry is closed (`endTime = now`).
  2. A new log entry is created with `startTime = now`, `endTime = null`.
  3. `state.active` is updated to point to the new task and entry.
- Starting the currently active task again dismisses the modal/interaction without creating a new entry.
- Stopping (`stopCurrentTask`): sets `endTime = now` on the active entry, clears `state.active`.
- On page reload, the active task and its open entry are restored from `localStorage`. The entry resumes from its saved `startTime`, so elapsed time continues to accumulate.

#### "Now tracking" card
- Shown only when a task is active; hidden otherwise.
- Displays: task name, pulsing green dot, live elapsed duration (updates every second).
- Contains a **Stop** button.

### 4. Check-in Modal

- Lists all current tasks as buttons.
- The currently active task is highlighted green with a ▶ prefix.
- Clicking the active task dismisses the modal; tracking continues uninterrupted.
- Clicking any other task switches tracking to that task.
- A text input at the bottom allows creating a new task and immediately starting it.
- The modal subtitle shows the current time (no active task) or the active task name and elapsed duration (task running).
- The modal is dismissed by clicking outside, pressing Escape, or selecting/creating a task.

### 5. Time Log

Displayed as a scrollable list (newest first, capped at 100 entries). The card is collapsible: clicking the card header toggles the list open or closed; a chevron (▼/◀) indicates the current state.

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
- Pressing Enter or tabbing away (with a non-empty value) saves the new name to `entry.taskName`.
- Pressing Escape or clearing the input and blurring cancels without saving.
- Opening the name editor closes any open category picker, and vice versa.
- The active entry's name in the "Now tracking" card is not updated retroactively — it still shows the name from when tracking started.

### 6. Categories

- Any log entry can be tagged with a free-text category.
- Clicking `+ tag` (or an existing category badge) opens an **inline picker** that expands within the log row.
- The picker contains:
  - A text input (pre-filled with the current category if set).
  - Up to 10 chips for recently used categories.
  - A **✕ clear** chip when a category is currently set.
- Saving: press Enter, click a chip, or click/tab away (if the input has a value).
- Cancelling: press Escape, or click away with an empty input.
- `mousedown` is used on chips (instead of `click`) so the chip action completes before the input's `blur` event fires.
- All category text is set via `textContent` / DOM methods — no HTML injection path.
- Categories are stored in `state.categories` ordered by most recent use (max 20 entries). Duplicate names are deduplicated on save.
- Colors are assigned deterministically per category name by hashing the name to an index into an 8-color palette. The same name always gets the same color.

### 7. Weekly Report

- Located below the time log.
- Shows the current Monday–Sunday week (ISO week, local timezone).
- Dimensions: categories (rows) × days (columns) + Total row + Total column.
- Row order: named categories alphabetically, "Uncategorized" last.
- Today's column is highlighted.
- Duration format in cells: `< 1m` / `45m` / `1h 30m` (rounded to nearest minute, no seconds).
- The active entry's contribution is included using `Date.now()` as the effective end time.
- The table is horizontally scrollable on narrow viewports (`min-width: 460px`).
- Re-renders on every `renderAll()` call (task start/stop, categorisation, log clear, check-in).

### 8. Settings

| Setting | Range | Default |
|---|---|---|
| Check-in interval | 1–480 min | 30 min |
| Task retention | 1–365 days | 7 days |

Saving settings restarts the countdown timer from zero.

### 9. Backup & Restore

- **Download backup** — serialises `state` to indented JSON and triggers a browser file download named `hamsterwheel-backup-YYYY-MM-DD.json`. No data is modified.
- **Restore from file** — opens a file picker (`.json` only). On selection:
  1. Parses the file; alerts on JSON parse error.
  2. Validates that `tasks` and `log` are arrays; alerts if not.
  3. Shows a confirmation dialog summarising entry and task counts.
  4. On confirmation, replaces `state` entirely, fills in any missing fields for backward compatibility, saves to `localStorage`, and re-renders everything.
  - The file input value is cleared after each use so the same file can be re-selected.

---

## Architecture

- **Single file** — all HTML, CSS, and JavaScript in `index.html`.
- **No build step** — open directly via `file://` in any modern browser.
- **No external dependencies** — no frameworks, no CDN resources.
- **Storage** — `localStorage` only; nothing is sent over the network.
- **Timers** — one `setInterval` for the check-in countdown (fires every 1 second); one `setInterval` for the check-in popup (fires every `intervalMinutes` minutes).
- **Rendering** — imperative DOM manipulation; no virtual DOM. `renderAll()` is the top-level render call. `updateActiveDuration()` performs targeted live updates without a full re-render.

---

## Key Functions

| Function | Purpose |
|---|---|
| `startTask(taskId)` | Begin tracking a task; close any open entry first |
| `stopCurrentTask()` | Close the active entry without starting another |
| `triggerCheckin()` | Fire notification + prune tasks + open modal |
| `openCategoryPicker(entryId)` | Show inline category editor for a log entry |
| `setCategory(entryId, cat)` | Persist a category and update the known-categories list |
| `openNameEditor(entryId)` | Show inline name input for a log entry |
| `setEntryName(entryId, name)` | Persist a renamed log entry task name |
| `buildWeekData(days)` | Aggregate log entries into a category × day ms matrix |
| `exportData()` | Download current state as a JSON file |
| `importData(event)` | Validate and restore state from an uploaded JSON file |
| `renderAll()` | Re-render active card, task list, log, and week report |
| `updateActiveDuration()` | Targeted live update of the active entry's duration display |
| `pruneExpiredTasks()` | Remove one-off tasks past their retention window |
| `toggleLog(event)` | Collapse or expand the time log card |
| `toggleTasks(event)` | Collapse or expand the tasks card |
| `saveData()` | Write `state` to `localStorage` |
