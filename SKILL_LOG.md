# SKILL LOG

## get-current-user

**Created:** 2026-10-07

**Conversation:** User requested creating a new skill called "get-current-user" that fetches the authenticated user's profile from the 4Geeks Academy (BreatheCode) API.

**Source conversation details:**
- API endpoint: `GET https://breathecode.herokuapp.com/v1/admissions/user/me`
- Auth header: `Authorization: Token <token>`
- API documented in `breatheCode.md`
- Required: call API with custom `Authorization` header, parse JSON response (id, email, name, profile_academy with linked academies)

**Design decisions:**
- Uses `curl` via `exec` rather than `web_fetch` because the latter doesn't support custom HTTP headers.
- Token expected in `TOKEN_4GEEKS` env var or requested via `secrets` tool.
- Follows the same SKILL.md structure as existing skills (`note-in-drive`, `task-in-calendar`).

## get-my-projects

**Created:** 2026-10-09

**Conversation:** User requested creating a new skill called "get-my-projects" that fetches the current user's assigned tasks/projects from the BreatheCode API.

**Source conversation details:**
- API endpoint: `GET /v1/assignment/user/me/task`
- Auth header: `Authorization: Token <token>`
- Token key in `.env`: `TOKEN_4GEEKS`
- Known params: academy_id = 62926, cohort id = 1838
- Query options: `task_status` (PENDING, DONE, APPROVED, REJECTED), `task_type` (PROJECT, EXERCISE, LESSON), `cohort`, `limit`, `offset`
- Response: array of task objects with status, type, dates, review metadata
- API documented in `breatheCode.md`

**Design decisions:**
- Uses `curl` via `exec` for custom auth header (same pattern as `get-current-user`).
- Defaults to `task_type=PROJECT` and `cohort=1838` as sensible starting filters.
- Token sourced from `TOKEN_4GEEKS` env var (loaded from `.env`), with `secrets` fallback.
- Covers pagination in Special cases for large result sets.

## get-my-pending-tasks

**Created:** 2026-10-09

**Conversation:** User requested creating a new skill called "get-my-pending-tasks" that fetches only the current user's PENDING tasks from the BreatheCode API.

**Source conversation details:**
- API endpoint: `GET /v1/assignment/user/me/task`
- Auth header: `Authorization: Token <token>`
- Token key in `.env`: `TOKEN_4GEEKS`
- Known params: academy_id = 62926, cohort id = 1838
- Filters used: `task_status=PENDING`, `cohort=1838` (no task_type filter — all task types included)
- Response: array of task objects with status, type, dates, review metadata
- API documented in `breatheCode.md`

**Design decisions:**
- Same curl+auth pattern as the sibling skills.
- Always applies `task_status=PENDING` automatically — that's the core purpose.
- Does **not** filter by `task_type` so exercises, lessons, and quizzes show up alongside projects.
- Defaults to `cohort=1838`.
- Includes a Special case for refining by task_type if the user asks for a specific type.
- Special "all caught up" messaging when no pending tasks exist.

## get-my-progress

**Created:** 2026-10-09

**Conversation:** User requested creating a new skill called "get-my-progress" that returns a general progress overview of the course.

**Source conversation details:**
- API endpoints used:
  - `GET /v1/assignment/user/me/task?cohort=1838&limit=100` — all tasks to compute stats by status
  - `GET /v1/admissions/academy/cohort/me?academy=62926&educational_status=ACTIVE` — active cohort info
  - `GET /v1/activity/me?cohort=1838` — activity data
- Auth header: `Authorization: Token <token>`
- Token key in `.env`: `TOKEN_4GEEKS`
- Known params: academy_id = 62926, cohort id = 1838
- Response: composed summary from three endpoints

**Design decisions:**
- Combines three endpoints for a holistic progress view rather than a single API call.
- Computes task completion stats by status count (pending / done / approved / rejected).
- Includes cohort info (name, schedule, educational status) for context.
- Adds activity data when available.
- Parallel curl recommended for efficiency, but sequential fallback is okay.
- Designed to present a summary card, not raw data.
- Handles partial data gracefully if one endpoint fails.

## get-task-detail

**Created:** 2026-10-21

**Conversation:** User requested creating a new skill called "get-task-detail" that fetches full details of a specific task by its ID from the BreatheCode API.

**Source conversation details:**
- API endpoint: `GET /v1/assignment/task/{task_id}`
- Auth header: `Authorization: Token <token>`
- Token key in `.env`: `TOKEN_4GEEKS`
- Path param: `task_id` (numeric)
- Response: complete task object with description, score, feedback, URLs, timestamps
- API documented in `breatheCode.md`

**Design decisions:**
- Same curl+auth pattern as sibling skills.
- Requires the user to provide a numeric task ID; if missing, the skill instructs the agent to ask.
- Handles 404 specifically (task not found) in addition to the standard 401/network cases.
- Presents a structured detail view with description, score, feedback, URLs, and dates.

## Token Validation — get-current-user (live test)

**Date:** 2026-10-21

**Action:** Ran `get-current-user` skill against the production API to validate that the stored `TOKEN_4GEEKS` token is functional.

**API call:**
```
GET https://breathecode.herokuapp.com/v1/admissions/user/me
Authorization: Token ***
```

**Response (HTTP 401):**
```json
{"error":"Invalid or Inactive Token","is_authenticated":"False","status_code":401}
```

**Token source:** Environment variable `TOKEN_4GEEKS` (auto-injected by OpenClaw). Length: 40 characters.

**Result:** ❌ The token is **invalid or expired**. The API rejects it with a 401 status. A fresh token needs to be obtained (via `POST /v1/auth/login`) and stored before any BreatheCode skill can function.

**Action taken:** Token value not stored in plaintext; result logged here for reference.

# SKILL_LOG.md — get-current-user

**Date:** 2026-10-21 19:37 UTC  
**Skill:** `get-current-user` — Fetches the current authenticated user's profile from the 4Geeks Academy (BreatheCode) API.

---

## Request

**Endpoint:** `GET https://breathecode.herokuapp.com/v1/admissions/user/me`  
**Headers:**
```
Authorization: Token <TOKEN_4GEEKS>
```

## Response (HTTP 200)

```json
{
  "id": 21959,
  "email": "sugiyama.renan@gmail.com",
  "username": "sugiyama.renan@gmail.com",
  "first_name": "Renan Henrique",
  "last_name": "Cavicchioli Sugiyama",
  "date_joined": "2026-07-15T14:18:30.509678Z",
  "github": {
    "avatar_url": "https://avatars.githubusercontent.com/u/36682907?v=4",
    "name": "Renan Sugiyama",
    "username": "renan-sugiyama"
  },
  "google": null,
  "discord": null,
  "profile": {
    "id": 16243,
    "avatar_url": "https://storage.googleapis.com/breathecode-profile-pictures/a90b9458a6a7a4d1488bc21a4511fd82-120x120",
    "show_tutorial": true,
    "github_username": null
  },
  "cohorts": [
    {
      "cohort": {
        "id": 1838,
        "slug": "spain-aie-devs-pt-1",
        "name": "spain-aie-devs-pt-1",
        "kickoff_date": "2026-09-21T00:00:00Z",
        "ending_date": "2027-02-10T00:00:00Z"
      },
      "id": 62926,
      "role": "STUDENT",
      "finantial_status": "UP_TO_DATE",
      "educational_status": "ACTIVE"
    },
    "... (15 more cohorts)"
  ],
  "roles": [
    {
      "academy": {
        "id": 6,
        "name": "4Geeks Madrid",
        "slug": "madrid-spain"
      },
      "role": "student"
    }
  ],
  "permissions": [
    "Upload media",
    "Upload assignment telemetry",
    "Get my containers",
    "Join event",
    "Join live class",
    "Join mentorship",
    "Get my mentoring sessions",
    "Get my certificate",
    "Update my profile",
    "Create my profile",
    "Get my profile"
  ],
  "settings": {
    "lang": "es",
    "main_currency": null
  }
}
```

## Key Data Extracted

| Field | Value |
|---|---|
| **Full Name** | Renan Henrique Cavicchioli Sugiyama |
| **Email** | sugiyama.renan@gmail.com |
| **GitHub** | renan-sugiyama (https://avatars.githubusercontent.com/u/36682907) |
| **Primary Cohort** | spain-aie-devs-pt-1 (Cohort ID: 1838) |
| **Academy** | 4Geeks Madrid (slug: madrid-spain) |
| **Role** | STUDENT |
| **Financial Status** | UP_TO_DATE |
| **Educational Status** | ACTIVE |
| **Language** | es (Spanish) |
| **Joined** | 2026-07-15 |

## All Cohorts (16 total)

| # | Cohort Name | Slug | Status | Educational Status |
|---|---|---|---|---|
| 1 | spain-aie-devs-pt-1 (main) | spain-aie-devs-pt-1 | PREWORK | ACTIVE |
| 2 | Real-Time Communication | real-time-communication | INACTIVE | ACTIVE |
| 3 | Agentic Engineering | agentic-engineering | INACTIVE | ACTIVE |
| 4 | Ai Engineering Project Delivery | ai-engineering-project-delivery | INACTIVE | ACTIVE |
| 5 | Working with AI coding agents | working-with-ai-coding-agents-v3 | INACTIVE | **GRADUATED** |
| 6 | Personal assistants with Openclaw | personal-assistants-with-openclaw | INACTIVE | ACTIVE |
| 7 | Advanced personal assistants with Openclaw | advanced-personal-assitants-with-openclaw | INACTIVE | ACTIVE |
| 8 | Agentic Workflows | agentic-workflows | INACTIVE | ACTIVE |
| 9 | Implementing Data Pipelines | implementing-data-pipelines | INACTIVE | ACTIVE |
| 10 | Asynchronous processing and offloading | asynchronous-processing-and-offloading | INACTIVE | ACTIVE |
| 11 | Models training & RAG | models-training-rag | INACTIVE | ACTIVE |
| 12 | Application telemetry | application-telemetry | INACTIVE | ACTIVE |
| 13 | Coding Fundamentals with Typescript | coding-fundamentals-with-typescript-v3 | INACTIVE | ACTIVE |
| 14 | Intro to 4Geeks Method | intro-to-4geeks-method | INACTIVE | ACTIVE |
| 15 | Secure AI Applications | secure-ai-applications | INACTIVE | ACTIVE |
| 16 | Container applications with Docker | container-applications-with-docker | INACTIVE | **GRADUATED** |
| 17 | Architecture optimization | architecture-optimization | INACTIVE | ACTIVE |