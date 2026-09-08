# Sweepstakes Tools

> **Category:** Sweepstakes | **Tools:** 7 | **Auth required:** Yes

Manage the full lifecycle of sweepstakes campaigns: create, update, clone, pause, reactivate, and delete.

---

## fetch_sweepstakes

Get the sweepstakes associated with your account, paginated and summarized.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `page` | number | No | 1-based page number. Default `1` |
| `page_size` | number | No | Items per page. Default `25`, maximum `100` |
| `include_settings` | boolean | No | Include the full `Settings` object of each sweepstakes (~1.5 KB each: SMS/email templates). Default `false` |
| `include_archived` | boolean | No | Include archived sweepstakes. Default `true` |

**Returns:** A summary per sweepstakes — token, name, handler, type, start/end date and time, timezone id, language, status, archived — plus a computed lifecycle `state` (`scheduled`, `running`, `ended`, `unknown`), `has_ended` and `days_until_end`. The response also carries `pagination` (`page`, `page_size`, `returned`, `total`, `total_pages`, `has_more`) and `filters`.

**Use when:** Finding an existing sweepstakes token for subsequent operations, or checking for handler collisions before creating a new one.

**Notes:**
- Read `state` before giving operational advice. A sweepstakes with `state: "ended"` cannot receive entries — never suggest sharing its entry link or waiting for participants.
- Read `pagination.has_more`. If it is `true`, request the next page or tell the user how many were not listed; never present a partial list as complete.
- `include_settings: true` was the old default shape and is heavy. For a single sweepstakes prefer `get_entry_settings`.
- The lifecycle comparison is at date level in UTC; near midnight in the sweepstakes timezone treat `state` as approximate.

---

## create_sweepstakes

Create a new sweepstakes programmatically.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_name` | string | Yes | Official name (used in rules and entry page) |
| `sweepstakes_type` | number | No | Defaults to `2` (Email — recommended). `1`=SMS and `3`=Social exist but are rarely used; only specify if the user explicitly requests them. Cannot be changed after creation. |
| `handler` | string | Yes | Unique identifier — alphanumeric only, max 20 chars, no spaces |
| `entrants_are_18_or_older` | boolean | No | Required `true` when the name refers to a minor audience. Confirms the person submitting the entry is an adult |
| `start_date` | string | Yes | `YYYY-MM-DD` |
| `end_date` | string | Yes | `YYYY-MM-DD` |
| `start_time` | string | Yes | `HH:MM` (24-hour) |
| `end_time` | string | Yes | `HH:MM` (24-hour) |
| `create_in_calendar` | boolean | No | Automatically create calendar events (recommended: `true`) |
| `sync_with_winners` | boolean | No | Sync with winners page (recommended: `true`) |
| `delete_if_deleted` | boolean | No | Auto-delete linked data when sweepstakes is deleted |
| `delete_if_acct_deleted` | boolean | No | Auto-delete when account is deleted |

**Returns:** `sweepstakes_token` (UUID) — save this, required for all subsequent operations.

**Limits:**
- Single per-plan cap on total sweepstakes (no separate active/total distinction).
- Default plans have lower caps than custom plans. Call `get_plan` to read this account's actual `MaxSweepstakesAllowed`.
- Warn the user if they are at or near their cap before calling `create_sweepstakes`.

**Handler rules:**
- Must be unique across the account
- Only alphanumeric characters allowed
- Max 20 characters
- Cannot be changed after creation

**Notes:**
- Always call `fetch_sweepstakes` first to check for handler collisions.
- **Minor audiences:** the server rejects a `sweepstakes_name` that refers to children or teenagers (kids, niños, teens, high school…) unless `entrants_are_18_or_older: true` is passed. Entry pages collect personal data from whoever enters, so entrants must be adults; a parent or guardian entering on a child's behalf is the compliant setup. Ask who actually enters before setting the flag.
- Set `create_in_calendar: true` and `sync_with_winners: true` by default.
- Save the returned `sweepstakes_token` — it is required for ALL subsequent operations.

---

## update_sweepstakes

Update an existing sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID from `fetch_sweepstakes` or `create_sweepstakes` |
| `sweepstakes_name` | string | No | New name |
| `start_date` | string | No | `YYYY-MM-DD` |
| `end_date` | string | No | `YYYY-MM-DD` |
| `start_time` | string | No | `HH:MM` (24-hour) |
| `end_time` | string | No | `HH:MM` (24-hour) |

**Cannot change:** Handler, sweepstakes type, and prize value/description — these are permanent after creation. Prize changes are not supported by any MCP tool; they must be handled through Sweeppea support.

**Notes:**
- If you update dates, also update the Official Rules document to match.

---

## clone_sweepstakes

Duplicate an existing sweepstakes with new parameters.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `handler` | string | Yes | Handler of the sweepstakes to clone |
| `handler_new` | string | Yes | Unique handler for the new sweepstakes |
| `sweepstakes_name` | string | Yes | Name for the new sweepstakes |
| `start_date` | string | Yes | `YYYY-MM-DD` |
| `end_date` | string | Yes | `YYYY-MM-DD` |
| `start_time` | string | Yes | `HH:MM` (24-hour) |
| `end_time` | string | Yes | `HH:MM` (24-hour) |

**What gets cloned:** Entry page configuration, short links, groups, calendar events, and all settings. Participants are NOT cloned.

**Use when:** Running recurring promotions (monthly, seasonal) based on a successful template.

---

## pause_sweepstakes

Temporarily disable a sweepstakes. Participants cannot enter while paused. All data is preserved.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes to pause |

**Use when:** Temporarily taking the sweepstakes offline for maintenance or legal review without losing data.

---

## unpause_sweepstakes

Reactivate a paused sweepstakes, allowing participants to enter again.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes to reactivate |

---

## delete_sweepstakes

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a sweepstakes and ALL associated data including participants, statistics, rules, winners, groups, and automations.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes to delete |
| `confirm` | boolean | Yes | Must be `true`. Assert it only after the user explicitly confirmed THIS deletion, having been told what is lost. The server rejects the call without it. |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool. State clearly what will be lost. This action cannot be undone.

---

## Token Usage

The `sweepstakes_token` (UUID) is the primary key for all operations. It is returned by `create_sweepstakes` and listed in `fetch_sweepstakes`. **Never display tokens to users** — use them internally for tool chaining only.
