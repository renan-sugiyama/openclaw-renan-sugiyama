---
name: get-current-user
description: Fetches the current authenticated user's profile from the 4Geeks Academy (BreatheCode) API.
---

# Skill: Get Current User

## When to use

When the user asks who they are, what their profile data is, or to look up their 4Geeks Academy account info (id, email, name, linked academies).

## Prerequisites

- The user's BreatheCode API token must be stored in an environment variable named `TOKEN_4GEEKS` or provided via a secret (use `secrets` tool to request it if missing).
- The API base URL is `https://breathecode.herokuapp.com`.

## Procedure

1. Ensure you have the API token. If not already available:
   - Check the environment variable `TOKEN_4GEEKS`.
   - If absent, use the `secrets` tool with action `request`, name `TOKEN_4GEEKS`, and reason "Need the BreatheCode API token to fetch the current user's profile."

2. Make a GET request to `https://breathecode.herokuapp.com/v1/admissions/user/me` with the header:
   ```
   Authorization: Token <token>
   ```
   - Use `exec` with `curl` to call the API, since `web_fetch` does not support custom headers.
   - Example command:
     ```sh
     curl -s -H "Authorization: Token $TOKEN_4GEEKS" \
       https://breathecode.herokuapp.com/v1/admissions/user/me
     ```

3. Parse the JSON response. The response is a user object containing:
   - `id` — numeric user ID
   - `email` — user's email
   - `name` — user's full name
   - `profile_academy` — array of academy profile objects, each with:
     - `academy` — object with `id`, `name`, `slug`, etc.
     - `role` — user's role in that academy
     - `educational_status` — if applicable

4. Present the data to the user in a clear format. Include at minimum:
   - Name
   - Email
   - Linked academies (name, slug, role)

## Expected output

The agent returns the current user's profile information (name, email, linked academies) in a concise, readable format.

## Special cases

- **401 Unauthorized**: The token is invalid or expired. Inform the user and ask them to provide a new token.
- **Network errors**: The API is unreachable. Report the error and suggest checking connectivity.
- **Empty profile_academy**: The user exists but has no academy affiliations. Display basic profile info without academies.
- **Token not found**: If the token is unavailable in both env vars and secrets, ask the user to provide it directly.