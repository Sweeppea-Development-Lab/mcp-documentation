# Sweepstakes Tools

> **Category:** Sweepstakes | **Tools:** 7 | **Auth required:** Yes

Manage the full lifecycle of sweepstakes campaigns: create, update, clone, pause, reactivate, and delete.

---

## fetch_sweepstakes

Get all sweepstakes associated with your account.

**Parameters:** None

**Returns:** List of all sweepstakes with token, name, handler, type, status, dates, and participant counts.

**Use when:** Finding an existing sweepstakes token for subsequent operations, or checking for handler collisions before creating a new one.

---

## create_sweepstakes

Create a new sweepstakes programmatically.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_name` | string | Yes | Official name (used in rules and entry page) |
| `sweepstakes_type` | number | Yes | `1`=SMS, `2`=Email, `3`=Social |
| `handler` | string | Yes | Unique identifier — alphanumeric only, max 20 chars, no spaces |
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
- Maximum 3 active sweepstakes simultaneously
- Maximum 10 total sweepstakes (active + inactive)

**Handler rules:**
- Must be unique across the account
- Only alphanumeric characters allowed
- Max 20 characters
- Cannot be changed after creation

**Notes:**
- Always call `fetch_sweepstakes` first to check for handler collisions.
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

**Cannot change:** Handler and sweepstakes type — these are permanent after creation.

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

**CRITICAL:** Always ask for explicit user confirmation before calling this tool. State clearly what will be lost. This action cannot be undone.

---

## Token Usage

The `sweepstakes_token` (UUID) is the primary key for all operations. It is returned by `create_sweepstakes` and listed in `fetch_sweepstakes`. **Never display tokens to users** — use them internally for tool chaining only.
