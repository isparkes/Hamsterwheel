# HamsterWheel — User Guide

HamsterWheel is a lightweight time tracker that lives entirely in a single browser tab. No account, no server — all data is stored locally in your browser.

---

## Quick start

1. Open `index.html` in a browser.
2. Add a task under **Tasks**.
3. Click the task name (or the ▶ button) to start tracking.
4. When a check-in fires, pick what you're working on — or just dismiss it.

---

## Tasks

### Adding a task
Use the form at the bottom of the Tasks card. Type a name, optionally toggle **Recurring**, then click **Add task** (or press Enter).

### Recurring vs one-off
| Type | Behaviour |
|---|---|
| **Recurring** | Never expires. Use for things you do every day (standups, email, etc.). Shown with a green badge. |
| **One-off** | Expires after the configured retention period (default 7 days) since it was last used. Shown with a countdown badge. |

Toggle between the two types at any time using the 🔁 / 📌 button on each task row.

### Starting and stopping
- Click a task name or its ▶ button to begin tracking it.
- Clicking a different task stops the current one and starts the new one.
- Click ■ (or **Stop** on the active card) to stop without starting anything new.

### Deleting a task
Click the trash icon on any task row. If that task is currently being tracked, tracking stops first.

---

## Check-ins

A check-in fires automatically at the configured interval (default every 30 minutes). It opens a modal asking what you're working on.

- **Pick an existing task** to switch to it (or keep the highlighted active one running).
- **Type in the bottom field** and press **Add & start** to create a new task on the fly.
- **Skip** dismisses the modal without changing anything.

If the browser tab is hidden when a check-in fires, a browser notification is sent (you'll be asked for permission on first use).

The countdown to the next check-in is shown in the header badge.

---

## Time log

Every tracked session creates a log entry. The 100 most recent entries are shown, newest first.

### Columns
`Task · category tag | Start → End | Duration | ×`

### Editing an entry

| Target | Action |
|---|---|
| Task name | Click the name to rename it inline. Press Enter to save, Escape to cancel. |
| Start or end time | Click the time to reveal a time picker. Change the value to save, Escape to cancel. |
| Category | Click the coloured badge (or **+ tag**) to open the category picker. Type a name or pick a recent chip. |
| Delete | Hover the row and click **×**. |

> Editing a time only changes the hour/minute — the date stays the same.

### Adding a manual entry
Click **+ Add entry** in the log header to open the entry form.

| Field | Notes |
|---|---|
| Task | Type any name. If it matches an existing task (case-insensitive) the entry is linked to it; otherwise a new task is created automatically. |
| Start | Required. Date and time the session started. |
| End | Optional. If left blank, and the new entry falls between two existing ones, the end time is automatically set to the start time of the next entry. |

---

## Categories

Categories tag what type of work an entry represents (e.g. *Deep work*, *Admin*, *Meetings*).

- Click **+ tag** on any log entry to open the picker.
- Type a new name or click one of the recent chips.
- Click **✕ clear** to remove the category from an entry.
- Each category gets a consistent colour across the whole app.
- The last 20 used categories are remembered as quick picks.

---

## Week report

The table at the bottom shows tracked time broken down by category for the current Mon–Sun week. Each cell shows the total duration for that category on that day. The rightmost column is the weekly total per category; the bottom row is the daily total.

---

## Settings

Click the ⚙ icon in the header.

| Setting | Default | Description |
|---|---|---|
| Check-in every … minutes | 30 | How often the check-in modal fires. |
| Keep non-recurring tasks for … days | 7 | One-off tasks that haven't been used within this window are automatically removed at the next check-in. |

### Backup and restore
- **Download backup** — exports all data as a JSON file.
- **Restore from file** — imports a backup, replacing all current data. You'll be asked to confirm.

---

## Keyboard shortcuts

| Key | Action |
|---|---|
| Escape | Close modal / cancel inline edit |
| Enter | Confirm inline edit / submit modal new-task field |

---

## Data and privacy

All data is stored in your browser's `localStorage` under the key `hamster_data`. Nothing is sent anywhere. Clearing site data in your browser will erase it — use **Download backup** regularly if you want to keep a history.
