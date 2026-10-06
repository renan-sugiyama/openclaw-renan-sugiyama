---
name: task-in-calendar
description: Creates a 30-minute task in Google Calendar.
---

# Skill: task in Calendar

## When to use
When I ask you to create, schedule, or add a task to Google Calendar.

## Prerequisites

- The Zapier MCP server must be connected.

## Procedure

1. If the task was not provided, ask what task should be created.
2. If the date or start time is missing or ambiguous, ask for the missing information.
3. Create a short, descriptive title for the task.
4. Always prefix the title with:
   `AI - `
5. Set the start date and time according to the user's request.
6. Set the end time to exactly 30 minutes after the start time.
7. Create the calendar entry using the appropriate Google Calendar action from the Zapier MCP server.
8. Return the created task's date, start time, end time, and link when available.

## Expected output

A Google Calendar entry exists with:

- A title starting with `AI - `
- The requested task description
- The requested start date and time
- A duration of exactly 30 minutes

The agent returned confirmation and the calendar entry link when available.

## Special cases

- If the task text is empty, do not create anything.
- If the date is missing, ask for it.
- If the start time is missing, ask for it.
- If the requested duration is different from 30 minutes, ignore the requested duration and create a 30-minute calendar entry.
- Never create a calendar entry without the `AI - ` prefix.