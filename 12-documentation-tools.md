# Documentation Tools

> **Category:** Documentation | **Tools:** 1 | **Auth required:** Yes

Search and retrieve help articles from the Sweeppea knowledge base.

---

## fetch_documentation

Get help and support documentation articles. Supports pagination and full-text search.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `page` | number | No | Page number (default: `1`) |
| `limit` | number | No | Articles per page (max: `10`, default: `5`) |
| `search` | string | No | Search by article title or content keywords |

**Returns:** Paginated list of matching help articles with title, content preview, and metadata.

---

## When to Use

Always call `fetch_documentation` before creating a support ticket (`create_ticket`). Many common questions are answered in the knowledge base.

**Search keyword examples:**
- `"entry page"` — entry page setup and configuration
- `"official rules"` — rules requirements and wizard
- `"AMOE"` — alternate method of entry setup
- `"winner drawing"` — how to draw winners
- `"clone sweepstakes"` — duplicating a campaign
- `"age gate"` — age verification setup
- `"bonus entries"` — bonus entry configuration
- `"webhook"` — real-time CRM integration
- `"bonding registration"` — state registration requirements

---

## Workflow

```
User has a question or reports an issue
  → fetch_documentation (search for answer)
  → Present relevant article(s) to user
  → If not resolved: create_ticket
```
