---
name: write-outlook-calendar
description: Add personal events to Outlook/Microsoft 365 calendar from chat text, screenshots, appointment slips, reminders, or loose notes. Use when the user asks to write, add, create, or import calendar items, especially when Teams, phone, or watch calendar sync depends on Outlook; includes browser-based Outlook entry, privacy checks, reminder handling, verification, and ICS fallback.
---

# Write Outlook Calendar

## Purpose

Turn user-provided text or images into private Outlook calendar events, then write them into the user's Microsoft 365 calendar through the best available route. Prefer a direct calendar connector/API if one exists. Otherwise use Outlook Calendar in the user's signed-in browser. Produce an `.ics` fallback when direct writing is blocked.

This file is intentionally ASCII-only so it can be copied into other AI tools without encoding issues.

## Extract Event Details

Extract these fields before writing:

- `title`: concise event name.
- `date`: absolute year/month/day. If the user gives only month/day, infer the current year unless context clearly points elsewhere.
- `start` and `end`: preserve explicit times.
- `location`: clinic, office, store, room, venue, or address.
- `body`: practical details such as doctor, department, ticket number, room, order info, prep notes, or original constraints.
- `reminders`: preserve explicit reminder requests.

For vague time periods, use practical defaults and disclose them:

- morning: `09:00-10:00`
- noon: `12:00-13:00`
- afternoon / tea time: `15:00-17:00`
- evening: `19:00-20:30`
- task/reminder with no time: `09:00-09:30`

For screenshots, inspect the image and transcribe only necessary details. Do not copy excessive private text into the calendar body.

## Privacy Rules

Create personal calendar events only:

- Do not add attendees unless the user explicitly asks.
- Do not enable Teams meeting unless explicitly requested.
- Do not share or publish calendars.
- For sensitive items such as medical appointments, keep the title useful but not overly detailed if the user asks for discretion.
- If the browser form shows attendees, meeting invite, sharing, or send controls, verify no recipients are present before saving.

## Preferred Tool Order

1. Search available tools for an Outlook/Microsoft calendar create-event connector or Microsoft Graph wrapper.
2. If no connector exists, use the user's signed-in browser with Outlook Calendar.
3. If browser authentication is blocked, leave the prefilled Outlook tab open for user login and create an `.ics` fallback.

## Outlook Browser Workflow

Use a browser session that has the user's Microsoft login state. Chrome often works better than an in-app browser if Outlook was previously used there.

Use Outlook's calendar compose deeplink:

```text
https://outlook.office.com/calendar/deeplink/compose?path=/calendar/action/compose&subject=...&startdt=...&enddt=...&location=...&body=...
```

Use ISO datetimes with timezone offsets:

```text
2026-07-28T09:30:00+08:00
```

After navigation:

1. Wait for the form to load.
2. Detect the `Save` button. In Traditional Chinese Outlook this is the save button in the top-left command bar.
3. Confirm the title, time, location, and body are populated.
4. Confirm no attendees and no Teams meeting unless requested.
5. Click `Save`.
6. Wait briefly for Outlook to process the save.

If the page lands on Microsoft login, keep the tab as a handoff and tell the user to sign in. The deeplink usually preserves the event data and opens the compose form after login.

## Multiple Reminders

Outlook event compose may not reliably expose multiple custom reminders. When the user requests multiple reminders, create reminder events in addition to the main event.

Example:

- Main event: `2026-09-20 15:00-17:00 Eat at Taoyuan Island`
- Five-day reminder: `2026-09-15 09:00-09:15 Reminder: Taoyuan Island in 5 days`
- One-day reminder: `2026-09-19 09:00-09:15 Reminder: Taoyuan Island tomorrow`

Mention this approach in the final answer so the user understands why extra reminder entries exist.

## Verification

After saving:

- Prefer searching Outlook for a distinctive title if month navigation is unreliable.
- For current-month events, month view visual confirmation is acceptable.
- Report only confirmed results.
- If save clicks succeeded but visual verification is inconclusive, say that saves were clicked and explain the verification limitation.

## ICS Fallback

When direct save is blocked, generate a standards-friendly `.ics`:

- UTF-8 with BOM.
- CRLF line endings.
- `VERSION:2.0`, `CALSCALE:GREGORIAN`, `METHOD:PUBLISH`.
- Use local floating times or `TZID:Asia/Taipei` consistently.
- Include `VALARM` for reminders.
- Keep descriptions concise; long or badly escaped non-ASCII text can break some Outlook imports.

Minimal event template:

```ics
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Codex//Outlook Calendar Event//EN
CALSCALE:GREGORIAN
METHOD:PUBLISH
BEGIN:VEVENT
UID:unique-id@example.local
DTSTAMP:20260724T000000Z
DTSTART:20260728T093000
DTEND:20260728T100000
SUMMARY:Hospital appointment
LOCATION:Hospital campus, building, floor, clinic room
DESCRIPTION:Department, doctor, ticket number, room, and check-in time.
BEGIN:VALARM
ACTION:DISPLAY
DESCRIPTION:Hospital appointment.
TRIGGER:-PT1H
END:VALARM
END:VEVENT
END:VCALENDAR
```

If using PowerShell to write the file:

```powershell
$text = ($lines -join "`r`n") + "`r`n"
[System.IO.File]::WriteAllText($path, $text, [System.Text.UTF8Encoding]::new($true))
```

## Final Response

Keep the final short:

- State whether the event was written directly, left ready after login, or saved as an ICS fallback.
- List the event date/time/title/location.
- Mention reminder assumptions.
- If a browser tab is waiting for login, tell the user exactly what to do next.
