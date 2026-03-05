# HamsterWheel

A lightweight, single-file time tracker that runs entirely in your browser.

## Usage

Open `index.html` in any modern browser. No server, no install, no dependencies.

## How it works

HamsterWheel prompts you at a configurable interval to confirm or switch what you're working on. Every prompt is a chance to log your time without interrupting your flow.

1. **Add tasks** — type a name and mark it recurring or one-off
2. **Start tracking** — click a task name to start the timer, or wait for a check-in popup
3. **Confirm at check-ins** — pick the current task, switch to another, or snooze
4. **Tag your entries** — click `+ tag` on any log entry to categorise it
5. **Review the week** — navigate the weekly report; export a PDF

## Features

- **Check-in popups** — configurable interval (default 5 min); snooze by 15m / 30m / 1h
- **Start-time picker** — when a check-in fires, choose when the current task actually started
- **Live timer** — "Now tracking" card shows elapsed time and a quick local-backup button
- **Task list** — recurring (kept forever) or one-off (auto-removed after N days); sortable by added date, A–Z, or most recent use; collapsible
- **Time log** — every session recorded with start/end/duration; newest first; collapsible
- **Inline editing** — rename any log entry; edit start/end times; tag categories
- **Gap indicators** — flagged gaps between entries with extend/pull/fill options
- **Categories** — colour-coded tags; consistent colour per name; recent chips for quick pick
- **Weekly report** — time by category × day; navigate back through previous weeks
- **PDF export** — printable weekly summary with per-task table and daily breakdown
- **Auto-prune** — log entries older than the configured window are removed automatically
- **Browser notifications** — fires when the tab is hidden
- **Cloud backup** — GitHub Gist integration (create/update/restore); optional auto-backup
- **File system backup** — write directly to a local file on every save (Chromium only)
- **Local backup button** — one-click JSON download from the tracking card; flashes amber if overdue (>24 h)
- **Backup & restore** — export/import full state as JSON at any time

## Settings

Open settings with the gear icon (top right).

| Setting | Default | Description |
|---|---|---|
| Check-in interval | 5 min | How often the popup appears |
| Task retention | 7 days | How long one-off tasks stay after last use |
| Log retention | 2 weeks | How many full weeks of log history to keep |

## Data

All data is stored in `localStorage` under the key `hamster_data`. Nothing leaves your machine unless you explicitly use the Gist or file system backup features.

Use **Download backup** in settings (or the button on the tracking card) to export a dated `.json` file. Use **Restore from file** to load it back.
