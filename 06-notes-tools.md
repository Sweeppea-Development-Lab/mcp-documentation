# Notes Tools

> **Category:** Notes | **Tools:** 5 | **Auth required:** Yes

Create and manage encrypted campaign notes. All note content is encrypted at rest using AES-256-CBC encryption. Notes are account-level (not tied to a specific sweepstakes) and returned in reverse chronological order.

---

## fetch_notes

Get all notes for the account.

**Parameters:** None

**Returns:** List of notes (automatically decrypted) in reverse chronological order, with token, title, pinned status, and creation date.

---

## create_note

Create a new encrypted note.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Note title (max 100 chars, must be unique, **not encrypted**) |
| `content` | string | Yes | Note content (max 100,000 chars, encrypted with AES-256-CBC) |
| `pinned` | boolean | No | Pin the note to the top (default: `false`) |

**Security warning:** The `title` field is **NOT encrypted**. Do not put sensitive information in the title.

**Allowed HTML tags in content:** `h1-h6`, `p`, `a`, `br`, `strong`, `i`, `ul`, `ol`, `li`

**Do NOT use** `<br>&nbsp;` between sections — it creates ugly blank blocks. Rely on natural HTML block element spacing (headings, paragraphs).

**Use when:** Documenting campaign briefs, recording meeting notes, storing research, or tracking campaign milestones.

**Recommended naming convention:**
```
Campaign Brief — [Sweepstakes Name]
Meeting Notes — [Date] — [Topic]
Winner Draw Results — [Sweepstakes Name] — [Date]
```

---

## get_note

Fetch a single note by token. Content is automatically decrypted.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `note_token` | string | Yes | UUID of the note |

**Returns:** Full note with decrypted content, title, pinned status, creation date, and last update date.

**Use when:** Reading a specific note after finding it with `fetch_notes`.

---

## update_note

Update an existing note. Supports partial updates (only provide fields you want to change).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `note_token` | string | Yes | UUID of the note to update |
| `title` | string | No | New title |
| `content` | string | No | New content (replaces existing content) |
| `pinned` | boolean | No | Update pin status |

**Use when:** Adding winner draw results to a campaign brief note, or updating campaign status.

---

## delete_note

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a note.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `note_token` | string | Yes | UUID of the note to delete |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool.

---

## Best Practices

- Create a pinned campaign brief note at sweepstakes creation time.
- Update the note with drawing results after winners are drawn.
- Use the note to document all campaign decisions and legal considerations.
- Never store raw PII (emails, phone numbers) in note titles (not encrypted).
- Use structured HTML (headings + paragraphs) rather than `<br>` spam.

**Example campaign brief structure:**
```html
<h2>Campaign Overview</h2>
<p>Sweepstakes Name: Win a Trip to Miami<br>Handler: WINMIAMI2026<br>Period: March 1 – March 31, 2026</p>

<h2>Prize</h2>
<p>Grand Prize: Round-trip airfare + 3 nights hotel (ARV: $2,500)</p>

<h2>Entry Method</h2>
<p>Website only — no purchase necessary. AMOE configured.</p>

<h2>Drawing</h2>
<p>Scheduled: April 5, 2026 at 12:00 PM ET<br>Winners: 1 Grand Prize</p>
```
