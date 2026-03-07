# Group Tools

> **Category:** Groups | **Tools:** 4 | **Auth required:** Yes

Groups allow you to organize and segment participants within a sweepstakes. Each sweepstakes has a default primary group ("Participants (Primary)"). You can create additional groups for segmentation (e.g., VIP customers, AMOE entries, social media entries).

---

## fetch_groups

Get all groups for a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** List of groups with token, name, participant count, primary status, and locked status.

**Use when:** Before drawing winners to get the group token, or to check existing segmentation.

---

## create_group

Create a new participant group within a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `group_name` | string | Yes | Unique name within the sweepstakes |

**Returns:** New group token and details.

**Use when:** Setting up campaign segmentation before participants start entering.

**Examples of useful groups:**
- "VIP Customers"
- "Social Media Entries"
- "AMOE Entries"
- "Employee Contest"
- "Grand Prize Eligible"

---

## update_group

Rename an existing group.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `group_token` | string | Yes | UUID of the group to rename |
| `group_name` | string | Yes | New name for the group |

---

## delete_group

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a group.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `group_token` | string | Yes | UUID of the group to delete |

**Restrictions:**
- Cannot delete primary groups
- Cannot delete locked groups
- Cannot delete groups that contain participants

**CRITICAL:** Always ask for explicit user confirmation before calling this tool.

---

## Usage with Winner Drawings

When calling `draw_winners` or `schedule_drawing`, you must specify either a specific `group_token` or `"allgroups"` to draw from all groups simultaneously.

Call `fetch_groups` first to get the group token before drawing winners.
