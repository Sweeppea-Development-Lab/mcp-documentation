# Calendar Tools

> **Category:** Calendar | **Tools:** 5 | **Auth required:** Yes

Manage campaign milestone calendar events. The calendar API operates in UTC, so dates must account for timezone offset when relevant.

---

## fetch_calendar_events

Get all calendar events for the account.

**Parameters:** None

**Returns:** List of all calendar events with token, title, dates, times, location, status, and completion state.

---

## get_calendar_event

Get a single calendar event by token.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `event_token` | string | Yes | UUID of the calendar event |

**Returns:** Full event details including all optional fields.

---

## create_calendar_event

Create a new calendar event.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Event title |
| `start_date` | string | Yes | Start date (ISO 8601: `YYYY-MM-DD`) |
| `end_date` | string | Yes | End date (ISO 8601: `YYYY-MM-DD`) |
| `description` | string | No | Event description |
| `location` | string | No | Event location |
| `start_time` | string | No | `HH:MM` (24-hour) |
| `end_time` | string | No | `HH:MM` (24-hour) |
| `all_day` | boolean | No | Mark as all-day event |
| `color` | string | No | Hex color code |
| `event_url` | string | No | Related URL |
| `latitude` | number | No | GPS latitude |
| `longitude` | number | No | GPS longitude |
| `notification` | boolean | No | Enable notification |
| `people_involved` | string | No | Participants or team members |
| `private_event` | boolean | No | Mark as private |
| `repeat_this_event` | boolean | No | Set as recurring event |
| `sms_notification` | boolean | No | Send SMS notification |
| `status` | string | No | Event status |
| `completed` | boolean | No | Mark as completed |

**IMPORTANT — UTC Date Limitation:**
The calendar API uses UTC. Today's date may be rejected as "past" depending on the user's timezone. **Always use tomorrow's date or later** when creating calendar events. If the user asks for today, explain this limitation.

**Recommended events to create automatically:**
1. **Launch Date** — When the sweepstakes goes live
2. **Close Date** — When entries close
3. **Drawing Date** — When winners will be drawn
4. **Winner Notification Deadline** — Deadline for winners to respond

---

## update_calendar_event

Update an existing calendar event. Supports partial updates.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `event_token` | string | Yes | UUID of the event to update |
| `title` | string | No | New title |
| `start_date` | string | No | New start date |
| `end_date` | string | No | New end date |
| `start_time` | string | No | New start time |
| `end_time` | string | No | New end time |
| `description` | string | No | New description |
| `location` | string | No | New location |
| `all_day` | boolean | No | Update all-day status |
| `color` | string | No | New color |
| `event_url` | string | No | New URL |
| `status` | string | No | New status |
| `completed` | boolean | No | Mark as completed |

**Restriction:** Cannot update events to past dates.

---

## delete_calendar_event

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a calendar event.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `event_token` | string | Yes | UUID of the event to delete |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool.

---

## Best Practices

- Create calendar events automatically at sweepstakes creation (`create_in_calendar: true` in `create_sweepstakes`).
- After drawing winners, create a "Winner Notification Deadline" reminder event.
- Use descriptive titles: `"[Sweepstakes Name] — Launch"`, `"[Sweepstakes Name] — Drawing"`.
- When cloning a sweepstakes, update the cloned events to match new dates.
