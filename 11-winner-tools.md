# Winner Tools

> **Category:** Winners | **Tools:** 5 | **Auth required:** Yes

Draw winners, schedule future drawings, and manage winner records.

---

## draw_winners

Execute an immediate random winner drawing from a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `how_many_winners` | number | Yes | Number of winners to draw (≥ 1) |
| `group` | string | Yes | Group UUID from `fetch_groups`, or `"allgroups"` to draw from all groups |
| `completed_entries` | boolean | No | Only draw from completed entries (recommended: `true`) |
| `exclude_spam` | boolean | No | Exclude flagged spam/bot entries (recommended: `true`) |
| `include_opted_out` | boolean | No | Include opt-out participants (default: `false`) |

**Selection method:** Weighted random — participants with bonus entries have proportionally higher probability of winning.

**Pre-draw checklist (verify before executing):**
- [ ] Official Rules exist (`fetch_rules` returns a document)
- [ ] Sweepstakes end date has passed (for one-time drawings)
- [ ] Sufficient participants (`count_participants`)
- [ ] Drawing date matches what is stated in the Official Rules

**After drawing:**
1. Call `fetch_winners` to retrieve the winner list.
2. Update the campaign brief note with drawing results.
3. Create a calendar event for the winner notification deadline.
4. Classify winners (Grand Prize, 1st Place, etc.) in the web interface.

---

## fetch_winners

Get winners from a sweepstakes with pagination.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `search` | string | No | Search by name or email |
| `page` | number | No | Page number |
| `items_per_page` | number | No | Results per page (default: `10`) |

**Returns:** Paginated list of winners sorted by draw date (most recent first), with winner token, name, email, draw date, and status.

---

## schedule_drawing

Schedule a future winner drawing.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `group` | string | Yes | Group UUID or `"allgroups"` |
| `selected_action` | number | Yes | `1`=Draw only, `2`=Draw & Notify winners |
| `schedule_mode` | string | Yes | `"schedule"` (use frequency) or `"settime"` (one-time at specific time) |
| `how_many_winners` | number | Yes | Number of winners to draw (≥ 1) |
| `end_date` | string | Yes | Drawing date (`YYYY-MM-DD`) |
| `end_time` | string | Yes | Drawing time (`HH:mm`) |
| `timezone` | number | No | Timezone ID (default: `7` = Eastern) |
| `frequency` | number | No | `0`=None, `1`=Hourly, `2`=Daily, `3`=Weekly, `4`=Monthly |
| `day_of_the_week` | number | No | For weekly: `1`–`7` |
| `week_of_the_month` | number | No | For monthly: `1`–`4` |
| `delivery_time` | string | No | Time to deliver notifications |
| `message` | string | No | Custom winner notification message |
| `publish_to_winners_page` | boolean | No | Auto-publish winner to winners page |
| `send_copy_to_me` | boolean | No | Send copy of notification to account owner |
| `add_to_calendar` | boolean | No | Add drawing to calendar |
| `exclude_spam` | boolean | No | Exclude spam entries (recommended: `true`) |
| `include_opted_out` | boolean | No | Include opt-outs (default: `false`) |

**Timezone IDs:** Use `fetch_timezones` to get the full list. Default is `7` (Eastern Standard Time — America/New_York).

---

## fetch_scheduled_drawings

Get all pending scheduled drawings for a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** List of scheduled drawings sorted by creation date (newest first), with schedule token, drawing date, time, winner count, and status.

**Use when:** Confirming a drawing was scheduled correctly, or finding the schedule token to cancel it.

---

## delete_scheduled_drawing

Cancel and delete a pending scheduled drawing. Only drawings with pending status can be deleted.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `schedule_token` | string | Yes | UUID of the scheduled drawing (from `fetch_scheduled_drawings`) |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool. Get the `schedule_token` from `fetch_scheduled_drawings` first.

---

## Post-Drawing Actions (Web Interface)

The following winner management steps must be completed through the Sweeppea web app:
- Classify winners (Grand Prize, 1st Place, 2nd Place, etc.)
- Send winner notification emails
- Verify winner eligibility
- Collect winner affidavits
- Publish the public Winners List page

---

## Workflow

```
# Immediate drawing
fetch_rules           → verify rules exist
count_participants    → verify sufficient entries
fetch_groups          → get group token
draw_winners          → execute drawing
fetch_winners         → retrieve winner list
update_note           → record results in campaign brief
create_calendar_event → winner notification deadline reminder

# Scheduled drawing
fetch_groups          → get group token
schedule_drawing      → schedule future drawing
fetch_scheduled_drawings → confirm it was created
```
