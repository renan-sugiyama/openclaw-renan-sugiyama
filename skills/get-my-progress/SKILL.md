---
name: get-my-progress
description: Fetches a general progress overview of the user's course performance from the 4Geeks Academy (BreatheCode) API, including task completion stats, activity, and cohort info.
---

# Skill: Get My Progress

## When to use

When the user asks about their overall progress, performance, completion stats, how they're doing in the course, or a summary of their course status.

## Prerequisites

- The BreatheCode API token must be available in the environment variable `TOKEN_4GEEKS` (loaded from `.env`). If missing, use the `secrets` tool to request it.
- The API base URL is `https://breathecode.herokuapp.com`.

## Procedure

1. **Ensure you have the API token.** If not already available:
   - Check `TOKEN_4GEEKS` from the environment (sourced from `.env`).
   - If absent, use the `secrets` tool with action `request`, name `TOKEN_4GEEKS`, and reason "Need the BreatheCode API token to fetch course progress."

2. **Make parallel GET requests** to gather a complete progress picture. Use `curl` via `exec` (since `web_fetch` does not support custom headers). Make all calls for a broad, unfiltered view, then compute summaries:

   a) **Tasks (all statuses)** — to compute completion stats:
      ```
      GET /v1/assignment/user/me/task?cohort=1838&limit=100
      ```

   b) **Cohort info** — to know the current cohort name, slug, schedule:
      ```
      GET /v1/admissions/academy/cohort/me?academy=62926&educational_status=ACTIVE
      ```
      
   c) **Activity** — to know learning time and exercise stats:
      ```
      GET /v1/activity/me?cohort=1838
      ```

   Example batch command:
   ```sh
   TOKEN=$(cat .env | grep TOKEN_4GEEKS | cut -d= -f2-)
   curl -s -H "Authorization: Token $TOKEN" \
     "https://breathecode.herokuapp.com/v1/assignment/user/me/task?cohort=1838&limit=100" \
     -o /tmp/tasks.json &
   curl -s -H "Authorization: Token $TOKEN" \
     "https://breathecode.herokuapp.com/v1/admissions/academy/cohort/me?academy=62926&educational_status=ACTIVE" \
     -o /tmp/cohort.json &
   curl -s -H "Authorization: Token $TOKEN" \
     "https://breathecode.herokuapp.com/v1/activity/me?cohort=1838" \
     -o /tmp/activity.json &
   wait
   ```

   Or, if the agent prefers simpler sequential calls, make each request and accumulate the results.

3. **Compute and compose the progress summary** from the responses:

   **From tasks (`/v1/assignment/user/me/task`):**
   - Count tasks by `task_status`:
     - `PENDING` — still to do
     - `DONE` — completed but not yet reviewed
     - `APPROVED` — completed and approved
     - `REJECTED` — needs corrections
   - Count by `task_type` if helpful (projects vs exercises vs lessons vs quizzes).
   - Calculate completion rate: `(DONE + APPROVED) / total * 100`.

   **From cohort (`/v1/admissions/academy/cohort/me`):**
   - Cohort name, slug, schedule
   - `educational_status` — ACTIVE, GRADUATED, etc.
   - `role` — student, etc.
   - `created_at` — when enrolled.

   **From activity (`/v1/activity/me`):**
   - Learning time (if available)
   - Exercises or sessions completed
   - Date range of activity data.

4. **Present a clean, readable progress report** covering:
   - **Course**: cohort name, schedule, status
   - **Tasks summary**: total tasks, how many pending, done, approved, rejected
   - **Completion rate**: percentage of tasks that are done/approved
   - **Recent activity**: any available activity stats

## Expected output

A concise progress report with cohort info, task completion breakdown (pending / done / approved / rejected), completion percentage, and activity highlights.

## Special cases

- **401 Unauthorized**: The token is invalid or expired. Inform the user.
- **Empty task list**: The user has no assigned tasks yet. Report that they may be newly enrolled.
- **No active cohort**: If the cohort endpoint returns no `ACTIVE` enrollment, check for other statuses or show available cohort history.
- **No activity data**: The activity endpoint may be empty. Omit that section rather than showing a blank.
- **Network errors**: The API is unreachable. Report the error.
- **Token not found**: If `TOKEN_4GEEKS` is unavailable, ask the user to provide the token.
- **Partial data**: If one endpoint fails but others succeed, present what's available and note which data couldn't be fetched.