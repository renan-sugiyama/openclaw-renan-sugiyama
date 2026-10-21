---
name: get-my-projects
description: Fetches the current user's assigned tasks/projects from the 4Geeks Academy (BreatheCode) API, optionally filtered by status, type, or cohort.
---

# Skill: Get My Projects

## When to use

When the user asks to see their projects, tasks, assignments, or homework from 4Geeks Academy — including pending, done, approved, or rejected work.

## Prerequisites

- The BreatheCode API token must be available in the environment variable `TOKEN_4GEEKS` (loaded from `.env`). If missing, use the `secrets` tool to request it.
- The API base URL is `https://breathecode.herokuapp.com`.

## Procedure

1. **Ensure you have the API token.** If not already available:
   - Check `TOKEN_4GEEKS` from the environment (sourced from `.env`).
   - If absent, use the `secrets` tool with action `request`, name `TOKEN_4GEEKS`, and reason "Need the BreatheCode API token to fetch the user's tasks."

2. **Determine any filters the user wants.** Ask if unclear, but sensible defaults are:
   - `task_type=PROJECT` (the user said "projects")
   - `cohort=1838` (the user's cohort)
   - For broader results, omit `task_type` or set it to `PROJECT,EXERCISE,LESSON`.

3. **Make the GET request** to:
   ```
   https://breathecode.herokuapp.com/v1/assignment/user/me/task
   ```
   - Use `exec` with `curl` (since `web_fetch` does not support custom headers).
   - Include the `Authorization: Token <token>` header.
   - Append query parameters as needed.
   - Example command:
     ```sh
     curl -s -H "Authorization: Token ${TOKEN_4GEEKS}" \
       "https://breathecode.herokuapp.com/v1/assignment/user/me/task?task_type=PROJECT&cohort=1838"
     ```

4. **Parse the JSON response.** The response is an array of task objects, each containing:
   - `id` — numeric task ID
   - `title` — task title
   - `task_type` — `PROJECT`, `EXERCISE`, `LESSON`, or `QUIZ`
   - `task_status` — `PENDING`, `DONE`, `APPROVED`, or `REJECTED`
   - `revision_status` — review status if applicable
   - `created_at` / `updated_at` — timestamps
   - `github_url` — associated GitHub repo URL (if any)
   - `live_url` — deployment URL (if any)
   - `description` — task description or instructions

5. **Present the results** in a clear format. Group or filter by what the user asked for:
   - If listing all: show title, status, type, and key dates.
   - If filtering by status: show only matching tasks.
   - If the user asked for a specific task detail, include the description and URLs.

## Expected output

The agent returns the user's project/task list (or filtered subset) with title, status, type, and relevant metadata in a readable format.

## Special cases

- **401 Unauthorized**: The token is invalid or expired. Inform the user and ask for a fresh token.
- **Empty results**: No tasks match the filters. Report that clearly rather than showing an empty table.
- **Network errors**: The API is unreachable. Report the error and suggest checking connectivity.
- **Token not found**: If `TOKEN_4GEEKS` is unavailable in both env and secrets, ask the user to provide the token directly.
- **Pagination**: If the user wants many results, use `limit` and `offset` query params and iterate through pages until all results are collected.