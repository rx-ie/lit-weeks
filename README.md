# Liturgical Weeks

Import liturgical-week ranges from an ICS calendar into a Google Sheet.

This repository contains a Google Apps Script that reads an iCalendar (ICS) feed of liturgical-week events, detects weeks (Monday start), infers the human-readable week name (e.g. "Week 3 in Ordinary Time", "1st Week of Advent"), extends the series forward week-by-week to the end of the calendar year, and writes a sheet named "Liturgical Weeks" with columns Start Date, End Date, and Week Name.

This is useful if you maintain a liturgical calendar as an ICS feed and want a simple tabular export for planning, printing, or publishing.

## Features

- Parse an ICS feed and detect VEVENT entries.
- Extract a week name from event SUMMARY values using common liturgical patterns (Ordinary Time, Advent, Lent, Easter, Christmas).
- Normalize dates to Monday start and produce Monday–Saturday ranges per week.
- Extend detected weeks forward until 31 December of the current year.
- Output to a Google Sheet named `Liturgical Weeks` with date formatting.

## Files

- `create-table` — Google Apps Script source (single-file) containing the import logic.
- `LICENSE` — repository license.

## Requirements

- A Google account with access to Google Sheets and Google Apps Script.
- A publicly accessible ICS URL (or one accessible from Apps Script via UrlFetchApp).

## Installation / Setup

1. Open the Google Sheet you want the Liturgical Weeks written into (or create a new one).
2. From the sheet choose Extensions → Apps Script to open the script editor.
3. Create a new script file and paste the contents of the repository's `create-table` file into it.
4. Edit the top of the script and set the ICS_URL constant to point to your calendar's ICS feed:

```javascript
const ICS_URL = 'https://example.com/path/to/calendar.ics';
```

5. Save the script, authorize it when prompted (it uses UrlFetchApp and SpreadsheetApp), and run the `importLiturgicalWeekRanges` function.

After the script runs it will create (or replace) a sheet called `Liturgical Weeks` and write a header row and one row per week detected/created.

## Behavior / Notes

- Date range: only events from today through 31 December of the current year are considered. Past events (before today) are ignored.
- Week boundaries: the script converts an event date to the Monday of that week and writes a Start Date (Monday) and End Date (Saturday).
- Name patterns: the script attempts to parse the SUMMARY text for several common patterns. Supported patterns include:
  - "Week N in Ordinary Time" (case-insensitive)
  - "Nth week of Advent" (e.g. "1st week of Advent")
  - "Nth week of Lent"
  - "Nth week of Easter" (or "Eastertide")
  - "Nth week of Christmas"

  If a SUMMARY does not match any of the supported patterns the event is ignored.

- Sequence extension: the script deduplicates detected weeks, sorts them, and then continues the weekly sequence forward (incrementing the ordinal where applicable) until the year-end cutoff.

## Example output

| Start Date | End Date   | Week Name                     |
|------------|------------|-------------------------------|
| 2026-02-02 | 2026-02-07 | Week 5 in Ordinary Time       |
| 2026-02-09 | 2026-02-14 | Week 6 in Ordinary Time       |

Dates are written as Date values in the spreadsheet and formatted as `yyyy-mm-dd` by the script.

## Scheduling

If you want the sheet refreshed automatically, add a time-driven trigger in Apps Script (Edit → Current project's triggers) to run `importLiturgicalWeekRanges` daily or at whatever cadence you prefer.

## Troubleshooting

- If nothing appears in the spreadsheet, check:
  - That `ICS_URL` is correct and reachable from Apps Script.
  - That the ICS feed contains VEVENT entries with SUMMARY and DTSTART properties.
  - Script authorization: the first run requires granting permissions for UrlFetchApp and SpreadsheetApp.

- If your calendar uses different SUMMARY wording, update `extractWeekName` in `create-table` to add or adjust regex patterns to match your feed.

## Functions (overview)

- `importLiturgicalWeekRanges()` — main entry point: fetches ICS, parses events, detects weeks, writes sheet.
- `extractWeekName(summary)` — returns a normalized human-readable week name or `null` if none matched.
- `nextWeekName(name)` — given a week name, returns the next week's name (increments ordinal where applicable).
- `ordinal(n)` — format an integer as an ordinal string (1st, 2nd, 3rd, 4th).
- `getMonday(date)` — returns a Date set to the Monday of the given date's week.
- `addDays(date, days)` — helper to add days to a Date.
- `parseICS(icsText)` / `unfoldICSLines(icsText)` / `parseICSDate(value)` — lightweight ICS parsing helpers.
- `formatDateISO(date)` — format a Date as `YYYY-MM-DD` for deduplication and internal keys.

## Contributing

Contributions welcome. If you add new SUMMARY patterns, please include tests or example SUMMARY strings in a pull request so the pattern matching remains clear.

## License

See the repository `LICENSE` file for license details.

