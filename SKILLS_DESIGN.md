# SKILLS DESIGN

## note-in-drive

- What does this skill do?
Saves a note, idea, or summary as an editable Google Doc.

- What input does the agent need?
Provide the content as text; the agent can generate the dated title, and asks if content is missing.

- What does a good output look like?
Success is a Doc in Drive with the required prefix and a returned link.

## task-in-calendar

- What does this skill do?
Creates a 30-minute Calendar task.

- What input does the agent need?
Provide the task plus an unambiguous date and start time; it asks for missing details, uses the AI -  title prefix, and always uses 30 minutes even if you request another duration.

- What does a good output look like?
Success is a Calendar event at the requested time, with the task description, and a returned confirmation and link when available.

## get-current-user

- What does this skill do?
Fetches the current user's profile from the 4Geeks Academy (BreatheCode) API.

- What input does the agent need?
The BreatheCode API token (stored in `TOKEN_4GEEKS` env var, in `secrets`, or provided by the user).

- What does a good output look like?
The user's name, email, and linked academies (with roles) displayed in a clear format.

## get-my-projects

- What does this skill do?
Fetches the current user's assigned tasks/projects from the 4Geeks Academy API.

- What input does the agent need?
The `TOKEN_4GEEKS` token (from `.env` or `secrets`). Optionally filters like task status or type.

- What does a good output look like?
A list of projects/tasks with title, status, type, and relevant metadata in a readable format.

## get-my-pending-tasks

- What does this skill do?
Fetches only the current user's pending (unfinished) tasks from the 4Geeks Academy API.

- What input does the agent need?
The `TOKEN_4GEEKS` token (from `.env` or `secrets`). The user may optionally specify a task type to narrow down.

- What does a good output look like?
A list of pending tasks with title, type, and status, or a friendly "all caught up" if none.

## get-my-progress

- What does this skill do?
Returns a general progress summary: task completion stats, active cohort info, and recent activity.

- What does the agent need?
The `TOKEN_4GEEKS` token. The skill calls three endpoints and composes the results.

- What does a good output look like?
A summary card with cohort name/status, task breakdown (pending/done/approved/rejected), completion rate, and activity highlights.

## get-task-detail

- What does this skill do?
Fetches the full detail of a single task by its numeric ID: description, score, feedback, URLs, dates.

- What does the agent need?
The `TOKEN_4GEEKS` token and a numeric task ID from the user.

- What does a good output look like?
A structured detail card with title, type, status, description, score, feedback, GitHub URL, live URL, and key dates.

# SILL_LOG.md — get-my-pending-tasks

**Date:** 2026-10-21 19:37 UTC  
**Skill:** `get-my-pending-tasks` — Fetches the current user's pending tasks from the 4Geeks Academy (BreatheCode) API, filtered by PENDING status.

---

## Request

**Endpoint:** `GET https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=PENDING&cohort=1838`  
**Headers:**
```
Authorization: Token <TOKEN_4GEEKS>
```
**Query Params:**
- `task_status=PENDING`
- `cohort=1838` (primary cohort: spain-aie-devs-pt-1)

## Response (HTTP 200)

```json
[]
```

## Summary

- **Total pending tasks:** 0
- **HTTP Status:** 200 OK
- **Cohort queried:** 1838 ("spain-aie-devs-pt-1")
- **Result:** The user has no pending tasks — all caught up! 🎉

## Notes

- An empty array `[]` with 200 means no tasks currently have `PENDING` status.
- The `get-my-progress` skill would be needed to see DONE / APPROVED / REJECTED tasks for a fuller picture.