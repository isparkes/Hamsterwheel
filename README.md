# 🐹 HamsterWheel

A lightweight, single-file time tracker that runs entirely in your browser.

## Usage

Open `index.html` in any modern browser. No server, no install, no dependencies.

## How it works

HamsterWheel prompts you at a configurable interval to confirm or switch what you're working on. Every prompt is a chance to log your time without interrupting your flow.

1. **Add tasks** — type a name and mark it recurring or one-off
2. **Start tracking** — click a task name or ▶ to start the timer, or wait for the check-in popup
3. **Tag your log entries** — click `+ tag` on any log entry to categorise it; click the task name to rename it
4. **Review the week** — the report at the bottom shows time by category across the current week
5. **Export a PDF** — click the PDF button in the week report header to get a printable weekly summary

## Features

- **Check-in popups** — appear on a configurable interval (default 5 min) and ask what you're working on
- **Live timer** — shows elapsed time for the currently active task
- **Task list** — tasks can be recurring (kept forever) or one-off (auto-removed after a configurable number of days); collapsible
- **Time log** — every tracked session is recorded with start time, end time, and duration; collapsible
- **Rename log entries** — click any task name in the log to edit it inline
- **Categories** — tag log entries; previously used tags appear as quick-select chips
- **Weekly report** — time by category × day for the current Mon–Sun week
- **PDF export** — generates a printable weekly report with a per-task summary table and a day-by-day entry breakdown
- **Browser notifications** — fires when the tab is in the background
- **Backup & restore** — export all data as JSON; restore from a backup file

## Settings

Open settings with the gear icon (top right).

| Setting | Default | Description |
|---|---|---|
| Check-in interval | 5 min | How often the popup appears |
| Task retention | 7 days | How long one-off tasks stay after last use |

## Data

All data is stored in `localStorage` under the key `hamster_data`. It is local to the browser and device — nothing leaves your machine.

Use **Download backup** in settings to export a dated `.json` file, and **Restore from file** to load it back.
