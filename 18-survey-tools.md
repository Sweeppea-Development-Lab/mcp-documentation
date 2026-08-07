# Survey Tools

> **Category:** Surveys | **Tools:** 7 | **Auth required:** Yes

Create and manage surveys attached to sweepstakes, collect responses, and read aggregated reports. **The Surveys module is disabled by default** — if the API returns `403 "module is not enabled"`, do NOT retry; contact Sweeppea support to request access.

Key behaviors of this module:

- **Structural limits:** max **5 questions per page**, **50 pages**, **250 questions total**, **30 options per question**.
- **Field types:** `text`, `textarea`, `radio`, `checkbox`, `select`, `slider`, `rating`, `nps`, `yesno`, `date`.
- **Locking:** once a survey has responses, `IsLocked` becomes `true` and its question structure can no longer be replaced (409).
- **Token display rule:** never show `SurveyToken`/`QuestionToken`/`ResponseToken`/`ParticipantToken` UUIDs to the user. The PublicLink IS meant to be shared with participants.
- **PascalCase nested objects:** `settings` and `questions` use PascalCase keys internally, exactly as the API expects (e.g., `QuestionText`, `FieldType`, `ThankYouMessage`).
- Visual configuration (`Visuals`, `Pages`, `LogoFile`) is managed in the web app and ignored by these tools.

---

## fetch_surveys

Fetch a paginated list of surveys, optionally filtered by sweepstakes, enabled state or archived state. An empty filter returns all surveys in the account.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | No | Filter by a specific sweepstakes (UUID v4) |
| `status` | boolean | No | `true` = enabled only, `false` = disabled only, omit = both |
| `archived` | boolean | No | Filter by archived state; omit for both |
| `page` | number | No | Page number (default 1) |
| `limit` | number | No | Results per page (default 50, max 200) |

**Notes:**

- `IsLocked` is derived at read time: `true` when the survey already has responses (its question structure can no longer be replaced).
- Questions are **NOT** included here — use `get_survey` for the full question set.

---

## get_survey

Fetch the full detail of a single survey, including its complete question set (sorted by Page then Order, no pagination — max 250 questions).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `survey_token` | string | Yes | The survey token (UUID v4). Get it via `fetch_surveys` |

**Returns:** Full survey with its complete question set.

**Notes:**

- `IsLocked: true` means the survey has responses and its questions can no longer be replaced.
- For aggregated results use `fetch_survey_report`; for individual responses use `fetch_survey_responses`.

---

## create_survey

Create a survey attached to a sweepstakes, optionally with its full question set. Use `fetch_sweepstakes` first to get the `sweepstakes_token`.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | The sweepstakes token (UUID v4) the survey belongs to |
| `survey_name` | string | Yes | Survey name (max 200 chars) |
| `status` | boolean | No | Initial enabled state (default `true`) |
| `settings` | object | No | Settings object with PascalCase keys: `Description`, `QuestionsPerPage` (1-5), `CollectContactInfo`, `ShowProgressBar`, `ShuffleQuestions`, `AllowMultipleResponses`, `ThankYouMessage`, `RedirectUrl`, `StartDate`, `EndDate`, `MaxResponses`, `ShowCountdown`, `EnableSharing`, `Language` (`"en"` \| `"es"`) |
| `questions` | array | No | Questions array with PascalCase keys per item: `{"QuestionText"` (required)`, "FieldType"` (required: `text` \| `textarea` \| `radio` \| `checkbox` \| `select` \| `slider` \| `rating` \| `nps` \| `yesno` \| `date`)`, "Page", "QuestionDescription", "Layout", "Required", "Options": [...` max 30`], "Settings": {`e.g. `RatingMax`, `RatingIcon}}` |

**Notes:**

- **Structural limits:** max 5 questions per page, 50 pages, 250 questions total, 30 options per question.
- **Validation is all-or-nothing:** any invalid question rejects the whole request and nothing is written.
- Pages are renumbered 1..N without gaps; question `Order` comes from array position.
- Visual configuration (`Visuals`, `Pages`, `LogoFile`) is managed in the web app and ignored here.
- To modify questions later use `update_survey` — only possible while the survey has no responses.

---

## update_survey

Update an existing survey. Partial update: only the fields you send are touched — **EXCEPT `questions`, which is a FULL REPLACEMENT** of the entire question set (sending `[]` deletes all questions; omitting it leaves them untouched).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `survey_token` | string | Yes | The survey token (UUID v4) of the survey to update |
| `survey_name` | string | No | New survey name (max 200 chars, cannot be emptied) |
| `status` | boolean | No | `true` enables, `false` disables the survey |
| `archived` | boolean | No | `true` archives, `false` restores |
| `settings` | object | No | Settings object with PascalCase keys (written key-by-key, only sent keys change): `Description`, `QuestionsPerPage` (1-5), `CollectContactInfo`, `ShowProgressBar`, `ShuffleQuestions`, `AllowMultipleResponses`, `ThankYouMessage`, `RedirectUrl`, `StartDate`, `EndDate`, `MaxResponses`, `ShowCountdown`, `EnableSharing`, `Language` (`"en"` \| `"es"`) |
| `questions` | array | No | **FULL REPLACEMENT** of the question set — rejected with 409 if the survey has responses. Same PascalCase shape as `create_survey` |

**At least one of `survey_name`, `status`, `archived`, `settings` or `questions` must be provided.**

**Notes:**

- If the survey already has responses its structure is **locked**: replacing `questions` is rejected with 409 — do not retry; responses protect the structure.
- Use `status: false` to disable a survey without losing data (the safe alternative to `delete_survey`).
- Visual configuration (`Visuals`/`Pages`/`LogoFile`) is ignored here.

**Best practice:** Call `get_survey` first to check `IsLocked` before attempting to replace questions.

---

## fetch_survey_responses

Fetch individual survey responses (paginated, newest first) with each answer.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `survey_token` | string | Yes | The survey token (UUID v4). Get it via `fetch_surveys` |
| `completed` | boolean | No | `true` = completed only, `false` = abandoned only, omit = both |
| `from_date` | string | No | Inclusive start date `YYYY-MM-DD` |
| `to_date` | string | No | Inclusive end date `YYYY-MM-DD`, covers the whole day |
| `include_metadata` | boolean | No | Include device/browser/IP metadata — **PII**, default `false`. Only set `true` if the user explicitly needs it |
| `page` | number | No | Page number (default 1) |
| `limit` | number | No | Results per page (default 25, max 100) |

**PRIVACY warning:** `include_metadata` adds PII (IP address, user agent, device). Keep it `false` unless the user explicitly needs it, and **NEVER** copy that metadata into notes, tickets or any other field.

**Notes:**

- `ParticipantToken` is `null` when no contact info was collected.
- For aggregated results use `fetch_survey_report`; for unlimited export point the user to the Export CSV button in the app.

---

## fetch_survey_report

Fetch the aggregated report of a survey: visit/start/complete totals, completion rate and times, device and browser breakdown, daily timeline, and per-question answer distributions.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `survey_token` | string | Yes | The survey token (UUID v4). Get it via `fetch_surveys` |
| `timeline_days` | number | No | Days back for the daily timeline (default 30, max 365) |

**Notes:**

- Distributions are capped at **25 buckets** — the long tail counts toward `TotalAnswers` but does not travel (`Truncated` flag).
- A survey with no traffic returns the full structure with zeros — report that honestly, never invent activity.
- `CompletionRate` = Completes/Visits as an integer percentage; `CompletionTimes` are in seconds.
- For raw individual responses use `fetch_survey_responses`.

---

## delete_survey

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a survey: its questions, ALL collected responses, stats and uploaded files.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `survey_token` | string | Yes | The survey token (UUID v4) of the survey to delete |
| `confirm` | boolean | No | Required `true` ONLY when the survey already has responses (409 otherwise). Pass it ONLY after the user explicitly confirmed losing all responses |

**CRITICAL:** Always ask for explicit user confirmation for this specific survey before calling this tool, spelling out what will be lost.

**Notes:**

- If the survey has responses the API additionally requires `confirm: true` (409 otherwise).
- Participants created from the survey and the sweepstakes itself are **preserved**.
- To disable a survey WITHOUT losing data, use `update_survey` with `status: false` instead — always offer that alternative first.
- Call `get_survey` first to check the response count and tell the user exactly what will be lost.

---

## Workflow

```
User wants to survey participants of a sweepstakes
  → fetch_sweepstakes (get the sweepstakes token)
  → create_survey (name + settings + questions, respecting structural limits)
  → Share the PublicLink with participants
  → fetch_surveys (monitor; IsLocked shows if responses arrived)
  → fetch_survey_report (aggregated results, completion rate, distributions)
  → fetch_survey_responses (individual answers; keep include_metadata false)
  → To pause: update_survey with status false (never delete to "pause")
```
