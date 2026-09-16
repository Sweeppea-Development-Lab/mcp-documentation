# Support Ticket Tools

> **Category:** Support Tickets | **Tools:** 7 | **Auth required:** Yes

Create and manage support requests with the Sweeppea support team. Always search `fetch_documentation` first before creating a ticket — many questions are answered in the help articles.

---

## fetch_open_tickets

Get open support tickets with pagination and filtering.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Search by title or description keyword |
| `platform` | string | No | Filter by platform: `renaissance` \| `api` \| `general` \| `overture` \| `winners` |
| `priority` | number | No | `1`=Low, `2`=Medium, `3`=High |
| `page` | number | No | Page number (20 tickets per page) |

**Returns:** Paginated list of open tickets with case number, title, priority, and creation date.

---

## fetch_closed_tickets

Get closed/resolved support tickets with pagination and filtering.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Search by title or description keyword |
| `platform` | string | No | Filter by platform |
| `priority` | number | No | `1`=Low, `2`=Medium, `3`=High |
| `page` | number | No | Page number (20 tickets per page) |

**Use when:** Reviewing resolved issues, referencing previous support interactions, or auditing ticket history.

---

## get_ticket

Get full details of a support ticket by case number.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `case_number` | string | Yes | Case number string (e.g., `"HYXTNJV"`) |

**Returns:** Full ticket with title, description, priority, status, notes, attached files, collaborators, and all activity history.

---

## create_ticket

Create a new support ticket for a platform, optionally assigned to one admin.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Brief summary (max 200 chars) |
| `description` | string | Yes | Detailed description of the issue (max 20,000 chars) |
| `priority` | number | Yes | `1`=Low, `2`=Medium, `3`=High |
| `platform` | string | No | Platform the ticket is about: `general`, `renaissance`, `overture`, `winners`, `soprano`, `symphony`, `sonata`, `papyrus`, `website`, `blog`, `enewsletter`, `socialmedia`, `api`, `aws`, `shopify`, `instakes`, `mcp-server`, `sweeppea-cli`, `n8n`, `other`. Defaults to `general`. |
| `assign_to` | string | No | Email, full name, username or first name of an admin user to assign the ticket to exclusively. If omitted, assigned to all admins. |

**Platform:** set it whenever the user names the product the ticket is about ("a change in a Renaissance feature" → `renaissance`). Without it the ticket is filed under `general` and does not appear under the product's filter.

**Assignment resolution** (first stage with a single match wins): exact email → exact full name or username → whole words (`"martin"` → `"Martin Gaido"`, not `"Marcos Martinez"`) → unique partial match. When a stage matches more than one admin the tool returns an error listing the candidates — ask the user which one they mean and retry with the full name. Use `assign_to` whenever the user says who the ticket is for; never invent a name they did not mention.

**Allowed HTML in description:** `<b>`, `<strong>`, `<i>`, `<hr>`, `<br>`

**Priority guidelines:**
- `3` (High): Production outage, sweepstakes cannot accept entries, legal deadline at risk
- `2` (Medium): Feature broken, workaround available, drawing scheduled soon
- `1` (Low): Question, enhancement request, cosmetic issue

**Best practice:** Always call `fetch_documentation` first with relevant keywords. Create a ticket only if documentation does not resolve the issue.

---

## update_ticket

Update an open support ticket. Only open tickets can be updated.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `case_id` | string | Yes | Case ID of the ticket |
| `title` | string | No | New title |
| `description` | string | No | New or additional description |

**At least one of `title` or `description` must be provided.**

---

## resolve_ticket

Close and resolve an open support ticket.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `case_id` | string | Yes | Case ID of the ticket to resolve |

**Use when:** The issue has been resolved and no further support action is needed.

---

## delete_ticket

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete an open support ticket. Only open tickets can be deleted.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `case_id` | string | Yes | Case ID of the ticket to delete |
| `confirm` | boolean | Yes | Must be `true`. Assert it only after the user explicitly confirmed THIS deletion, having been told what is lost. The server rejects the call without it. |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool. Consider resolving instead of deleting to preserve history.

---

## Workflow

```
User reports an issue
  → fetch_documentation (search for answer)
  → If not resolved: create_ticket (with detailed description)
  → fetch_open_tickets (monitor status)
  → get_ticket (check for updates)
  → resolve_ticket (when resolved)
```
