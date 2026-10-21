---
name: get-task-detail
description: Fetches the full detail of a specific task by its ID from the 4Geeks Academy (BreatheCode) API.
---

# Skill: Get Task Detail

## When to use

When the user asks for details, description, score, feedback, or full information about a specific task or project by its ID.

## Prerequisites

- The BreatheCode API token must be available in the environment variable `TOKEN_4GEEKS` (loaded from `.env`). If missing, use the `secrets` tool to request it.
- The API base URL is `https://breathecode.herokuapp.com`.
- The user must provide a numeric task ID.

## Procedure

1. **Ensure you have the API token.** If not already available:
   - Check `TOKEN_4GEEKS` from the environment (sourced from `.env`).
   - If absent, use the `secrets` tool with action `request`, name `TOKEN_4GEEKS`, and reason "Need the BreatheCode API token to fetch task details."

2. **Get the task ID from the user.** If they didn't provide one, ask for it. The task ID is a numeric identifier (e.g. `1234`).

3. **Make the GET request** to:
   ```
   https://breathecode.herokuapp.com/v1/assignment/task/{task_id}
   ```
   - Use `exec` with `curl` (since `web_fetch` does not support custom headers).
   - Include the `Authorization: Token <token>` header.
   - Replace `{task_id}` with the actual numeric ID.
   - Example command:
     ```sh
     curl -s -H "Authorization: Token ***}" \
       "https://breathecode.herokuapp.com/v1/assignment/task/1234"
     ```

4. **Parse the JSON response.** The response is a single task object containing:
   - `id` — numeric task ID
   - `title` — task title
   - `task_type` — `PROJECT`, `EXERCISE`, `LESSON`, or `QUIZ`
   - `task_status` — `PENDING`, `DONE`, `APPROVED`, or `REJECTED`
   - `revision_status` — current review status
   - `description` — task description or instructions
   - `github_url` — associated GitHub repo URL (if any)
   - `live_url` — deployment URL (if any)
   - `score` — task score or grade (if graded)
   - `feedback` — reviewer feedback or comments (if reviewed)
   - `created_at` / `updated_at` — timestamps
   - `delivery_url` — submission URL (if delivered)

5. **Present the details** in a clear, structured format:
   - Title, type, and status at the top
   - Description (if available)
   - Score and feedback (if graded/reviewed)
   - GitHub URL and live URL (if any)
   - Key dates

## Expected output

The agent returns the full task detail — title, type, status, description, score, feedback, URLs, and dates — in a readable format.

## Special cases

- **401 Unauthorized**: The token is invalid or expired. Inform the user and ask for a fresh token.
- **404 Not Found**: No task exists with that ID. Inform the user and ask them to verify the ID.
- **Network errors**: The API is unreachable. Report the error and suggest checking connectivity.
- **Token not found**: If `TOKEN_4GEEKS` is unavailable in both env and secrets, ask the user to provide the token directly.
- **Missing task ID**: If the user didn't provide a task ID, ask for it before making the API call.