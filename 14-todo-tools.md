# To-Do Tools

> **Category:** To-Do | **Tools:** 4 | **Auth required:** Yes (Admin only)

Internal task management for administrative users. Full CRUD for to-do items with priorities, deadlines, and resource tags.

---

## create_todo

Create an internal To-Do item.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Task title |
| `priority` | number | Yes | `1`=Low, `2`=Medium, `3`=High |
| `description` | string | No | Detailed task description |
| `deadline` | string | No | Due date (`YYYY-MM-DD`) |
| `resource_affected` | string | No | Resource or system affected by this task |
| `pin` | boolean | No | Pin to top of list |

**Allowed HTML in description:** `<b>`, `<strong>`, `<i>`, `<hr>`, `<br>`

**Do NOT use** `<br>&nbsp;` — use `<br>` for line breaks within text only.

---

## fetch_todos

Get To-Do items with pagination and filtering.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Search by title or description |
| `status` | string | No | Filter by status |
| `priority` | number | No | `1`=Low, `2`=Medium, `3`=High |
| `resource` | string | No | Filter by resource tag |
| `pinned` | boolean | No | Show only pinned items |
| `start_date` | string | No | Filter by start date (`YYYY-MM-DD`) |
| `end_date` | string | No | Filter by end date (`YYYY-MM-DD`) |
| `page` | number | No | Page number (20 per page) |

---

## update_todo

Update an existing To-Do item. Supports partial updates — only provide the fields you want to change.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `todo_token` | string | Yes | UUID of the To-Do item |
| `title` | string | No | New title (max 200 characters) |
| `description` | string | No | New description (max 20,000 characters) |
| `priority` | number | No | `1`=Low, `2`=Medium, `3`=High |
| `resource_affected` | string | No | Resource affected |
| `pin` | boolean | No | Pin to top of list |
| `deadline` | string | No | Deadline date (ISO 8601). Send empty string to clear. |
| `status` | boolean | No | `false`=Pending, `true`=Completed |
| `completion` | number | No | Completion percentage (0–100) |

**Allowed HTML in description:** `<b>`, `<strong>`, `<i>`, `<hr>`, `<br>`

---

## delete_todo

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a To-Do item.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `todo_token` | string | Yes | UUID of the To-Do item to delete |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool.

---

## Notes

- These tools are available to admin users only.
- Use To-Do items for internal operational tasks: infrastructure updates, compliance reviews, feature deployments.
- Not intended for participant-facing campaign tasks — use Calendar Events and Notes for campaign documentation.
