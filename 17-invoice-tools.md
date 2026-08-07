# Invoice Tools

> **Category:** Invoices | **Tools:** 5 | **Auth required:** Yes

Create and manage invoices for the account. **The Invoices module is disabled by default** — if the API returns `403 "module is not enabled"`, do NOT retry; contact Sweeppea support to request access.

Key behaviors of this module:

- **Totals are computed server-side.** Subtotal, TaxAmount and Total are always calculated by the API from `items`, `discount_amount` and `tax_rate` — never calculate or promise totals yourself; report what the API returns. The computed total must be between **$1 and $1,000,000**, with a maximum of **60 line items**.
- **State machine:** `draft → pending → paid`, with `cancelled` reachable from `draft` or `pending`. Reverting to `draft` is rejected (400). Paid and cancelled invoices are **immutable history** — the API answers 409.
- **No automatic notifications.** Creating or publishing an invoice NEVER emails the recipient — sharing the PublicLink is up to the user.
- **Token display rule:** never show `InvoiceToken`/`PublicToken` UUIDs to the user. The PublicLink URL and InvoiceNumber ARE meant to be shared.
- **PascalCase nested objects:** `bill_to` and `items` use PascalCase keys internally, exactly as the API expects (e.g., `Name`, `Email`, `Description`, `Quantity`, `Rate`).

---

## fetch_invoices

Fetch a paginated list of invoices for the account, with optional filters by status and date range.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `status` | string | No | Filter by invoice status: `draft` \| `pending` \| `paid` \| `cancelled` |
| `from_date` | string | No | Inclusive start date in ISO format `YYYY-MM-DD` |
| `to_date` | string | No | Inclusive end date in ISO format `YYYY-MM-DD` |
| `page` | number | No | Page number (default 1) |
| `limit` | number | No | Results per page (default 50, max 200) |

**Returns:** Paginated list of invoices plus a Summary block and pagination info.

**Notes:**

- The Summary block (totals and counts) covers the **entire filtered set**, not just the current page.
- `IsOverdue` is derived at read time (the stored status stays `pending`).
- Archived invoices are included with `Archived: true`.
- For full details of one invoice (line items, notes, QR) use `get_invoice`.

---

## get_invoice

Fetch the full detail of a single invoice: line items, payment info, public link, QR code, stats and event timeline.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `invoice_token` | string | Yes | The invoice token (UUID v4). Get it via `fetch_invoices` |
| `include_events` | boolean | No | Include the event timeline (default `true`). Set `false` for a lighter payload |
| `include_qr_code` | boolean | No | Include the QR code as a data URL (default `true`). Set `false` for a lighter payload |

**Returns:** Full invoice with line items, payment info, public link, QR code, stats, derived fields, and event timeline.

**Notes:**

- The event timeline is capped at the **200 most recent events**.
- Set `include_events` and `include_qr_code` to `false` when you only need the invoice fields.

---

## create_invoice

Create a new invoice.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `bill_to` | object | Yes | Recipient object with PascalCase keys exactly as the API expects: `{"Name": "...", "Email": "..."}` required; optional keys: `Company`, `Phone`, `Address`, `City`, `State`, `ZipCode`, `Country`, `TaxId` |
| `items` | array | Yes | Line items array (min 1, max 60) with PascalCase keys: `[{"Description": "..."` (required)`, "Quantity": number` (max 100)`, "Rate": number` (max 1,000,000)`}]`. Amount per line is computed server-side |
| `title` | string | No | Invoice title/concept (max 200 chars) |
| `issue_date` | string | No | Issue date in ISO format `YYYY-MM-DD` (defaults to today) |
| `due_date` | string | No | Due date in ISO format `YYYY-MM-DD` (defaults to account setting) |
| `discount_amount` | number | No | Flat discount amount, capped at the subtotal |
| `tax_enabled` | boolean | No | Apply tax to the invoice (default `false`) |
| `tax_label` | string | No | Tax label shown on the invoice (default `"Tax"`) |
| `tax_rate` | number | No | Tax percentage 0–100 |
| `payment_method` | string | No | `card` \| `check` \| `transfer` (default `card`) |
| `payment_instructions` | string | No | Offline payment instructions — only for `check` or `transfer` methods |
| `notes_to_recipient` | string | No | Notes shown to the recipient (max 3000 chars) |
| `terms` | string | No | Terms and conditions text (max 3000 chars) |
| `status` | string | No | Initial status: `draft` \| `pending` (default `draft`). `pending` publishes immediately |

**Notes:**

- Subtotal, TaxAmount and Total are **always computed server-side** from items, discount and tax rate — report what the API returns.
- The computed total must be between **$1 and $1,000,000**.
- Creating an invoice **never notifies the recipient** automatically — sharing the PublicLink is up to the user.

---

## update_invoice

Update an existing invoice. Partial update: anything omitted keeps its stored value, **BUT `items` and `bill_to` are replaced as a whole when sent** (never merged) — to edit one line item, send the complete new items array.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `invoice_token` | string | Yes | The invoice token (UUID v4) of the invoice to edit |
| `bill_to` | object | No | **FULL replacement** of the recipient object, PascalCase keys: `{"Name", "Email"}` required; optional: `Company`, `Phone`, `Address`, `City`, `State`, `ZipCode`, `Country`, `TaxId` |
| `items` | array | No | **FULL replacement** of the line items (max 60), PascalCase keys: `[{"Description"` (required)`, "Quantity", "Rate"}]` |
| `title` | string | No | Invoice title/concept (max 200 chars) |
| `issue_date` | string | No | Issue date `YYYY-MM-DD` |
| `due_date` | string | No | Due date `YYYY-MM-DD` |
| `discount_amount` | number | No | Flat discount amount, capped at the subtotal |
| `tax_enabled` | boolean | No | Apply tax |
| `tax_label` | string | No | Tax label |
| `tax_rate` | number | No | Tax percentage 0–100 |
| `payment_method` | string | No | `card` \| `check` \| `transfer`. `payment_instructions` is cleared when method is `card` |
| `payment_instructions` | string | No | Offline payment instructions for check/transfer |
| `notes_to_recipient` | string | No | Notes to the recipient (max 3000 chars) |
| `terms` | string | No | Terms and conditions (max 3000 chars) |
| `status` | string | No | Status transition: `pending` publishes the draft, `cancelled` cancels. Reverting to draft is not allowed |

**State machine:**

```
draft → pending → paid
draft → cancelled
pending → cancelled
```

- Reverting to `draft` is rejected (400).
- **Paid and cancelled invoices are immutable history** — the API answers 409; do not retry, explain to the user.
- Publishing does **NOT** email the recipient.
- Totals are recomputed server-side.

**Best practice:** Call `get_invoice` first to check the current status before attempting a transition.

---

## delete_invoice

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete an invoice and its events.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `invoice_token` | string | Yes | The invoice token (UUID v4) of the invoice to delete |

**CRITICAL:** Always ask for explicit user confirmation for this specific invoice before calling this tool.

**Notes:**

- Deleting is **NOT the same as cancelling**: to keep the invoice as history, use `update_invoice` with status `cancelled` instead — always offer that alternative first.
- **Money survives:** wallet transactions and Sweeppea commissions from a paid invoice are NEVER deleted.
- Call `get_invoice` first to confirm which invoice is being deleted and show the user its InvoiceNumber and status.

---

## Workflow

```
User wants to bill a client
  → create_invoice (bill_to + items; totals computed server-side)
  → update_invoice with status "pending" (publish — recipient is NOT emailed)
  → Share the PublicLink with the recipient (user's responsibility)
  → fetch_invoices (monitor status; Summary covers the whole filtered set)
  → get_invoice (full detail, payment info, event timeline)
  → To void: update_invoice with status "cancelled" (preferred over delete_invoice)
```
