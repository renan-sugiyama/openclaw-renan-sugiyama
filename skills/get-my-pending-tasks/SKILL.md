---
name: get-my-pending-tasks
description: Fetches the current user's pending tasks from the 4Geeks Academy (BreatheCode) API, filtered by PENDING status.
---

# Skill: Get My Pending Tasks

## When to use

When the user asks to see their pending, unfinished, or outstanding tasks, projects, assignments, or homework from 4Geeks Academy.

## Prerequisites

- The BreatheCode API token must be available in the environment variable `TOKEN_4GEEKS` (loaded from `.env`). If missing, use the `secrets` tool to request it.
- The API base URL is `https://breathecode.herokuapp.com`.

## Procedure

1. **Ensure you have the API token.** If not already available:
   - Check `TOKEN_4GEEKS` from the environment (sourced from `.env`).
   - If absent, use the `secrets` tool with action `request`, name `TOKEN_4GEEKS`, and reason "Need the BreatheCode API token to fetch pending tasks."

2. **Make the GET request** to the tasks endpoint with `task_status=PENDING`:
   ```
   https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=PENDING&cohort=1838
   ```
   - Use `exec` with `curl` (since `web_fetch` does not support custom headers).
   - Include the `Authorization: Token <token>` header.
   - Always filter by `task_status=PENDING`.
   - Default to `cohort=1838` (the user's cohort).
   - Do **not** filter by `task_type` so all task types (projects, exercises, lessons, quizzes) appear.
   - Example command:
     ```sh
     curl -s -H "Authorization: Token ***}" \
       "https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_status=PENDING&cohort=1838"
     ```

3. **Parse the JSON response.** The response is an array of task objects, each containing:
   - `id` — numeric task ID
   - `title` — task title
   - `task_type` — `PROJECT`, `EXERCISE`, `LESSON`, or `QUIZ`
   - `task_status` — always `PENDING` due to the filter
   - `revision_status` — review status if applicable
   - `created_at` / `updated_at` — timestamps
   - `github_url` — associated GitHub repo URL (if any)
   - `live_url` — deployment URL (if any)
   - `description` — task description or instructions

4. **Present the results** in a clear format:
   - Show each pending task with its title, type, and any relevant dates.
   - If there are no pending tasks, let the user know they're all caught up.
   - If the user wants details on a specific task, include description and URLs.

## Expected output

The agent returns the user's list of pending tasks with title, type, and relevant metadata in a readable format.

## Special cases

- **401 Unauthorized**: The token is invalid or expired. Inform the user and ask for a fresh token.
- **Empty results (no pending tasks)**: Let the user know they're all caught up — no pending work!
- **Network errors**: The API is unreachable. Report the error and suggest checking connectivity.
- **Token not found**: If `TOKEN_4GEEKS` is unavailable in both env and secrets, ask the user to provide the token directly.
- **Pagination**: If the user has many pending tasks, use `limit` and `offset` query params to fetch all pages.
- **Filter clarification**: If the user asks for pending tasks of a specific type (e.g. "pending projects"), add `task_type=PROJECT` to the query. If they mention a different cohort, use that instead of `1838`.