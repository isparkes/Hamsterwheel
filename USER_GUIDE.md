# HamsterWheel — User Guide

HamsterWheel is a lightweight time tracker that lives entirely in a single browser tab. No account, no server — all data is stored locally in your browser.

---

## Quick start

1. Open `index.html` in a browser.
2. Add a task under **Tasks**.
3. Click the task name (or the ▶ button) to start tracking.
4. When a check-in fires, pick what you're working on — or snooze it.

---

## Tasks

### Adding a task
Use the form at the bottom of the Tasks card. Type a name, optionally toggle **Recurring**, then click **Add task** (or press Enter). As you type, the task list above filters in real time to show only matching tasks.

### Recurring vs one-off

| Type | Behaviour |
|---|---|
| **Recurring** | Never expires. Use for things you do every day (standups, email, etc.). Shown with a green badge. |
| **One-off** | Expires after the configured retention period (default 7 days) since it was last used. Shown with a countdown badge that turns red at ≤ 2 days. |

Toggle between the two types at any time using the 🔁 / 📌 button on each task row.

### Sorting tasks
Use the segmented control in the Tasks card header to change the sort order.

| Sort | Behaviour |
|---|---|
| **Added** | Order tasks were created (default). |
| **A–Z** | Alphabetical by name. |
| **Recent** | Most recently used first. |

Click the active sort button again to reverse the direction (↑ / ↓).

### Starting and stopping
- Click a task name or its ▶ button to begin tracking it.
- Clicking a different task stops the current one and starts the new one immediately.
- Click ■ (or **Stop** on the "Now tracking" card) to stop without starting anything new.

### Deleting a task
Click the trash icon on any task row. If that task is currently being tracked, tracking stops first.

---

## Now tracking card

When a task is active, a green card appears at the top of the page showing:
- The task name and a pulsing green dot
- Live elapsed time, updated every second
- A **Stop** button
- A **download icon button** for a quick local backup (see [Local backup button](#local-backup-button))

---

## Check-ins

A check-in fires automatically at the configured interval (default every 5 minutes). It opens a modal asking what you're working on.

- **Pick an existing task** to switch to it (the currently active task is highlighted green). As you type in the bottom field, the task list filters in real time to show only matching tasks.
- **Type in the bottom field** and press **Add & start** to create a new task on the fly.
- **Continue** dismisses the modal without changing anything.

### Snooze
If you're in the middle of something and don't want to be interrupted again soon, use the snooze buttons at the bottom of the modal: **15m**, **30m**, or **1h**. The next check-in is delayed by that amount.

### Start-time picker
When a task is already running and a check-in fires, a **"Started at"** dropdown appears above the task list. Use it to back-date the start of a new task — useful when you switched activities a while ago but only just got the check-in. The dropdown lists 5-minute intervals from when the current task started up to now.

If the browser tab is hidden when a check-in fires, a browser notification is sent (you'll be asked for permission on first use). The countdown to the next check-in is shown in the header badge.

---

## Time log

Every tracked session creates a log entry. The 100 most recent entries are shown, newest first. Click the log card header to collapse or expand it.

### Columns
`Task · category | Start → End | Duration | ×`

### Editing an entry

| Target | Action |
|---|---|
| Task name | Click the name to rename it inline. Press Enter to save, Escape to cancel. |
| Start or end time | Click the time (dotted underline appears on hover) to reveal a time picker. Change the value or press Enter to save, Escape to cancel. Start must be before end. |
| Category | Click the coloured badge or **+ tag** to open the category picker. |
| Delete | Hover the row and click **×**. |

> Editing a time only changes the hour and minute — the date stays the same.

### Backdating a running task

For the currently active entry, quick-backdate buttons appear next to the start time: **−5m**, **−15m**, **−30m**, **−1h**. Each snaps the start time to the previous wall-clock boundary for that interval — for example, clicking **−15m** at 10:23 sets the start to 10:15.

- Clicking a button also moves the **end time of the preceding entry** to the same point, keeping the timeline seamless. If the resulting gap between the preceding entry's end time and the new start time is more than 15 minutes, you will be asked to confirm before the preceding entry is adjusted. If you decline, only the active entry's start time is changed.
- Buttons that would move the start to before (or equal to) the preceding entry's own start time are hidden automatically.
- Buttons that would resolve to the same boundary as a shorter interval are also hidden to avoid duplicates.
- For precise control, click the start time itself to open the full time picker.

### Category picker

- Type a new category name, or click one of the recent-use chips.
- Press Enter, click a chip, or click away (with something typed) to save.
- Click **✕ clear** to remove the category from an entry.
- Press Escape, or click away with an empty input, to cancel.

Each category gets a consistent colour across the whole app based on its name.

### Gap indicators

When two consecutive log entries have a gap of more than 1 minute between them, a dashed row appears showing the gap duration. Hover it to reveal three actions:

| Button | Effect |
|---|---|
| **Extend end ↑** | Stretches the earlier entry's end time forward to meet the later entry's start. |
| **↓ Pull start** | Pulls the later entry's start time back to meet the earlier entry's end. |
| **+ Fill gap** | Opens the Add Entry panel pre-filled with the gap's start and end times. |

### Adding a manual entry
Click **+ Add entry** in the log header to open the entry form.

| Field | Notes |
|---|---|
| Task | Type any name. Matches an existing task case-insensitively and links to it; otherwise creates a new task. |
| Start | Required. Date and time the session started. |
| End | Optional. If blank and the entry falls between two existing entries, the end is auto-set to the next entry's start time. |

---

## Categories

Categories tag what type of work an entry represents (e.g. *Deep work*, *Admin*, *Meetings*).

- The last 20 used categories are remembered as quick-pick chips.
- Each category name always gets the same colour everywhere in the app.

---

## Weekly report

The report below the log shows tracked time broken down by **category × day** for a Mon–Sun week.

- Each cell shows the total duration for that category on that day.
- The rightmost column is the category's weekly total; the bottom row is the daily total.
- Today's column is highlighted.

### Navigating weeks
Use the **‹** and **›** buttons in the report header to step back and forward through previous weeks. The title updates to show "This week", "Last week", or "N weeks ago". The **›** button is hidden when you're on the current week.

### Exporting a PDF
Click the **PDF** button in the report header. A new tab opens with a printable report for whichever week is currently displayed, containing:

- **Weekly Summary by Task** — each task's time per day and weekly total.
- **Daily Breakdown** — for each day that has entries: individual sessions with task name, category, start/end times, and duration.

From the print dialog choose **Save as PDF** to download the file.

> If nothing opens, your browser may be blocking pop-ups for this page. Allow them and try again.

---

## Header controls

The header bar contains:
- **Countdown badge** — time until the next check-in.
- **⏥ Pause button** — toggles pause mode. When paused: the current task is ended, the countdown stops (badge shows "Paused"), and no check-in popups appear. Clicking again resumes the timer and immediately opens the "What are you working on?" modal. The button turns orange when active.
- **⚙ Settings button** — opens the settings panel.

---

## Settings

Click the **⚙** icon in the header.

| Setting | Default | Description |
|---|---|---|
| Check-in every … minutes | 5 | How often the check-in modal fires (1–480). |
| Keep non-recurring tasks for … days | 7 | One-off tasks unused beyond this window are removed at the next check-in (1–365). |
| Keep log history for … weeks | 2 | Log entries older than this are pruned automatically (1–52). |

Saving settings restarts the countdown timer from zero. A **Test check-in popup** button triggers a check-in immediately.

---

## Backup and restore

### Local backup button

The "Now tracking" card has a small **download icon** button in its header. Click it to download a JSON backup immediately. The file is named `hamsterwheel-backup-YYYY-MM-DD_HH-MM.json`. If no local backup has been taken in the last 24 hours, the button pulses amber as a reminder.

### Settings panel

| Action | Description |
|---|---|
| **Download backup** | Exports all data as a dated `.json` file (`hamsterwheel-backup-YYYY-MM-DD.json`). |
| **Restore from file** | Imports a backup file, replacing all current data. You'll be asked to confirm first. |

### Cloud backup (GitHub Gist)

Store your data in a secret GitHub Gist so it survives browser data clears and syncs across machines.

1. Create a fine-grained Personal Access Token at **github.com/settings/personal-access-tokens** with the **Gists: read & write** scope.
2. Paste the token into the token field in Settings. It is stored only in your browser and is never included in file backups.
3. Click **Backup now** to create or update the Gist immediately.
4. Enable **Auto-backup** to back up automatically 10 seconds after any data change.
5. Click **Restore** to fetch and restore data from the Gist (you'll be asked to confirm).

### File system backup (Chromium only)

Write your data directly to a local file on disk — works in Chrome, Edge, and Brave.

1. Click **Choose file…** to pick a destination file (creates it if new).
2. Enable **Auto-save when data changes** to write automatically 10 seconds after any change.
3. If you reload the page, click **Reconnect** to reattach the file handle (browser security requires this step).
4. Click **Clear** to disconnect the file without deleting it.

---

## Keyboard shortcuts

| Key | Action |
|---|---|
| Escape | Close modal / cancel inline edit |
| Enter | Confirm inline edit / submit modal new-task field |

---

## Data and privacy

All data lives in your browser's `localStorage` under the key `hamster_data`. Nothing is sent anywhere unless you explicitly configure Gist backup. Clearing site data in your browser will erase it — use one of the backup options if you want to preserve your history.

The GitHub token (if set) is stored separately under `hamster_gist_token` and is never exported in JSON backups.
