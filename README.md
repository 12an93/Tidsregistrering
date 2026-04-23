# Tidsregistrering

> A single-file time tracking web app for Swedish work weeks. Tracks hours, car commutes, absences, and per-project time — with no server, no build step, and optional direct-to-disk storage.

![Made with](https://img.shields.io/badge/made%20with-vanilla%20JS-f6f2ea?style=flat-square&labelColor=1a1d1a)
![Single file](https://img.shields.io/badge/single%20file-HTML-2e4a36?style=flat-square)
![No build](https://img.shields.io/badge/no%20build-step-b68636?style=flat-square)

---

## What it is

A self-contained `tidsregistrering.html` file — about 78 KB, zero dependencies — that gives you a clean weekly timesheet with Swedish holidays, absence tracking, per-client project logging, and auto-sync to a local JSON file via the File System Access API. Drop it anywhere, open it in a Chromium browser, and it works.

Built originally to replace a brittle Excel timesheet (`.xlsm`) but kept deliberately minimal: one HTML file, one JSON file, your data stays on your disk.

## Why it exists

Excel timesheets rot. Formulas break when rows move, macros stop working after updates, and sharing between devices means emailing files around. A web app running entirely in the browser solves this — but only if the data lives somewhere you control. This app writes directly to a JSON file you choose (ideally inside a OneDrive/Dropbox/iCloud folder), so sync works for free and nothing depends on a vendor staying in business.

## Features

### Weekly time entry
- **Start, lunch out, lunch in, end** for each weekday
- **Forgiving time input**: type `7`, `7:`, `730`, `0730`, `7.30`, `7,30`, or `7:30` — all parse to `07:30` on Tab/Enter
- **Auto-calculated hours** per day and week, with overtime computed weekly (hours > 40h/week), matching the Swedish workweek norm
- **Car-to-work checkbox** per day with yearly tally — useful for commute deduction on Swedish tax returns

### Absence types
Dropdown per day covers the common Swedish categories:
- Arbete (Work) · Semester · Sjuk · VAB · Föräldraledig · Tjänstledig · Kompledig · ATK · Ledig · Röd dag

ATK entries show a small inline hours field (default 8h, accepts `4:30`, `4,5`, `2h30m`, etc.) since ATK is typically drawn down in fractions of a day.

### Swedish holidays
Computed automatically for any year (2021–2031 in the picker, extensible). Uses Gauss's algorithm for Easter and derives moveable feasts from there:
- Fixed: Nyårsdagen, Trettondedag jul, Första maj, Nationaldagen, Julafton, Juldagen, Annandag jul, Nyårsafton
- Easter-based: Långfredagen, Annandag påsk, Kristi himmelsfärdsdag
- Calendar-computed: Midsommarafton, Midsommardagen, Allhelgonaafton, Alla helgons dag

Holidays auto-mark as "Röd dag" but can be overridden if you actually worked that day.

### Day view (project entries)
Click any day name to open a modal for granular per-project logging. Each entry has:
- Start time, end time (auto-calculated duration)
- Client
- Project
- Comment

Meant for consultants and project managers who bill by client. A live reconciliation row shows whether your project entries sum to the day's total, so you catch gaps before invoicing.

Two shortcuts speed up entry:
- **Fyll från dagens tider** auto-creates one or two entries (split by lunch) so you only edit the client/project fields
- **New entry** pre-fills the start time with the previous entry's end time

### Year summary
Three panels update in real time:
- **Sammanställning** — worked hours, workdays, overtime, car days, semester days, sick days
- **Frånvaro & bil** — all absence types with counts (days) or hours (for ATK)
- **Kunder & uppdrag** — top clients with a bar chart and top 5 client-project combinations

### Storage
Three layers, explicitly indicated by a status bar at the top:

1. **Primary (recommended): File System Access API** — picks a local JSON file you own. Every change writes through, debounced 400 ms. Put the file in OneDrive/Dropbox/iCloud and you get cross-device sync for free.
2. **Cache: browser localStorage** — always on, so the app works instantly even before a file is connected and survives a brief disk unavailability.
3. **Fallback: JSON export/import** — manual backup with file picker or copy-paste text.

File-handle references persist across sessions via IndexedDB. If browser permissions lapse, a yellow status bar offers a single "Reconnect" click.

### Week utilities
- **Idag** jumps to the current ISO week
- **Kopiera föregående** copies last week's times into this week (skips rows already marked as holidays or semester)
- **Rensa vecka** clears all entries for the shown week with a confirmation dialog

### Excel import
The repo includes logic for importing from the original Excel timesheet format (`Tid_2026.xlsx`). The conversion script:
- Reads day rows from the sheet (day name in col B, date in col C, times in D–G, note in J, car marker in K)
- Maps Excel notes: `LEDIG` → status Ledig, `SJUK` → status Sjuk, `ATK` → status ATK; other notes preserved as plain text on work days
- Converts decimal-day times (Excel's internal format) to `HH:MM`
- Outputs a JSON file you can restore via the Backup dialog

## Tech

- **Vanilla JavaScript** — no React, no build tools, no package.json
- **CSS variables** for theming (paper-like light theme with green/ochre accents)
- **Google Fonts** — Fraunces (display) and Inter (body)
- **File System Access API** for direct disk writes (Chromium-only)
- **IndexedDB** for persisting file handles across sessions

## Browser support

| Browser | Status |
|---|---|
| Chrome, Edge, Opera, Brave, Arc (Chromium ≥ 86) | Full support including direct file sync |
| Firefox, Safari | Works with localStorage; file sync falls back to manual JSON export/import |

The app degrades gracefully — on unsupported browsers the storage status bar guides you to download the HTML and open it locally in a Chromium browser for the full experience.

## Getting started

1. Download `tidsregistrering.html`
2. Open it in Chrome or Edge (double-click works if it's saved to disk)
3. Click **Skapa ny fil** in the storage bar at the top
4. Save the JSON file somewhere synced (e.g. `OneDrive/Tider/tider.json`)
5. Start entering times

To sync to a second machine: download the HTML there too, open it, click **Öppna befintlig** in the storage bar, point to the same JSON file.

To import from an existing Excel file: open the Backup dialog, choose **Välj fil…**, select the generated `tidsregistrering-import.json`, then **Återställ**.

## File structure

```
tidsregistrering.html        ← the whole app (open this)
tidsregistrering-import.json ← optional: converted Excel data for one-time import
```

That's it. No `node_modules`, no `dist`, no config files.

## Data format

Storage is a flat JSON object keyed by ISO date (`YYYY-MM-DD`):

```json
{
  "2026-01-12": {
    "in": "07:00",
    "lunchOut": "12:00",
    "lunchIn": "12:35",
    "out": "16:00",
    "car": true,
    "status": "work",
    "note": "Internt möte",
    "entries": [
      {
        "start": "07:00",
        "end": "09:30",
        "client": "Kommun X",
        "project": "Migration",
        "comment": "Setup"
      },
      {
        "start": "09:30",
        "end": "12:00",
        "client": "Saab",
        "project": "Review",
        "comment": ""
      }
    ]
  },
  "2026-01-15": {
    "status": "atk",
    "atkHours": 4,
    "car": false,
    "note": "",
    "in": "", "lunchOut": "", "lunchIn": "", "out": ""
  }
}
```

Days with no meaningful data are omitted to keep the file compact. The format is deliberately simple so it can be edited by hand, diffed in git, or read by other tools.

## Keyboard shortcuts

- **Tab** — move between fields, parses times on the way
- **Enter** inside a time field — commit and blur
- **Escape** — close the day view modal

## Known quirks

- The `atk` hours field defaults to 8h for historical reasons (one full workday); override on first entry
- Overtime is calculated per ISO week, not per day — matches the Swedish `Installationsavtalet`/most collective agreements, but check yours
- "Röd dag" auto-marking fires only on days with no stored data, so edits are never silently overwritten

## Privacy

No telemetry, no analytics, no external API calls (except loading Google Fonts from fonts.googleapis.com — swap for self-hosted if you care about that). Your time data never leaves the machine you're working on.

## License

Pick one that suits you — this is the kind of thing you might want MIT or 0BSD for. Add a `LICENSE` file before publishing.

## Credits

Built by Tolvers, 2026. Replaces an older Excel workflow that had outlived its usefulness.
