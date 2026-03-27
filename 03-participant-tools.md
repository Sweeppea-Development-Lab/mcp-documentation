# Participant Tools

> **Category:** Participants | **Tools:** 6 | **Auth required:** Yes

Manage sweepstakes participants: add, search, list, count, update bonus entries, and delete entries.

---

## add_participant

Add a new participant to a sweepstakes. For testing and administrative purposes only — not intended for high-volume bulk imports.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the target sweepstakes |
| `email` | string | Yes | Participant email address |
| `fields` | object | Yes | Form field values (discovered via `get_entry_fields`) |
| `phone` | string | No | Phone number in `(XXX) XXX-XXXX` format for US, E.164 for international |
| `bonus_entries` | number | No | Additional entries to award (default: `0`) |

**Fields format:**
Use underscores to replace spaces in field names. Example:
```json
{
  "First_Name": "Jane",
  "Last_Name": "Doe",
  "Zip_Code": "10001"
}
```

**IMPORTANT:** Always call `get_entry_fields` before `add_participant` to discover the correct field names and which are required.

**Privacy requirements:**
- The email and phone must belong to the participant and were provided with their consent.
- Never add participant data without explicit consent.

---

## get_participant

Fetch full details of a single participant by token, email, or phone.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `participant_token` | string | No* | UUID of the specific participant |
| `email` | string | No* | Participant email |
| `phone` | string | No* | Participant phone number (10 digits) |

*At least one of `participant_token`, `email`, or `phone` is required.

**Returns:** Full participant record including entry fields, opt-in status, bonus entries, entry date, and group membership.

---

## fetch_participants

Get a paginated list of participants with optional filtering.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `search` | string | No | Search by name, email, or phone |
| `opt_in_date` | string | No | Filter by opt-in date (`YYYY-MM-DD`) |
| `start_date` | string | No | Filter entries from this date (`YYYY-MM-DD`) |
| `end_date` | string | No | Filter entries until this date (`YYYY-MM-DD`) |
| `page` | number | No | Page number (20 participants per page) |

**Returns:** Paginated list with participant details and pagination metadata.

**Pagination:** 20 participants per page. Use the `page` parameter to navigate.

---

## count_participants

Get participant counts with optional type and date filtering.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `filter_type` | string | No | `all` \| `participants` \| `amoe` \| `optouts` |
| `start_date` | string | No | Start of date range (`YYYY-MM-DD`) |
| `end_date` | string | No | End of date range (`YYYY-MM-DD`) |

**Use when:** Quick statistics before drawing winners, reporting totals, or checking campaign progress.

---

## update_bonus_entries

Overwrite the bonus entries value for a participant in a sweepstakes. Use `get_participant` first to check current bonus entries.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `participant_token` | string | Yes | UUID of the participant |
| `bonus_entries` | integer | Yes | New bonus entries value (0–1,000,000). Overwrites the current value. |

**Use when:** Adjusting bonus entries for a participant after a promotion, reward, or correction.

---

## delete_participant

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a single participant from a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `participant_token` | string | Yes | UUID of the participant to delete |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool. This removes the participant from all groups and winner pools.

---

## Workflow

```
get_entry_fields        → discover required form fields
add_participant         → add with correct field names
fetch_participants      → browse with search/date filters
get_participant         → look up specific person
update_bonus_entries    → adjust bonus entries for a participant
count_participants      → get statistics for reporting or pre-draw checks
```
