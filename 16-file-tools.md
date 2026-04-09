# File Management Tools

> **Category:** Files / Drive | **Tools:** 4 | **Auth required:** Yes

Upload, list, delete, and share files from the user's Drive.

---

## upload_file

Upload a file to the user's Drive. The file must be base64-encoded.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `filename` | string | Yes | Original filename with extension (e.g., `"report.pdf"`) |
| `mime_type` | string | Yes | MIME type (e.g., `"application/pdf"`, `"image/png"`) |
| `file_data` | string | Yes | Base64-encoded file content |
| `private` | boolean | No | Privacy flag (default: `true`) |

**Constraints:**
- Max file size: **10 MB**
- Allowed types: PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX, TXT, CSV, JPG, JPEG, PNG, GIF, WEBP, SVG, BMP
- Filenames are sanitized: spaces → underscores, special characters removed
- Upload rejected if it exceeds the user's storage quota

**Returns:** File token, filename, MIME type, size, category, and creation date.

---

## fetch_files

List all files in the user's Drive with storage usage, categories, and pagination.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `page` | number | No | Page number (default: `1`) |
| `limit` | number | No | Files per page (default: `50`, max: `200`) |

**Returns:** Array of files (token, filename, path, size, MIME type, privacy, opened status), categories, storage usage (used/max/remaining MB), and pagination metadata.

---

## delete_file

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete a file from the user's Drive. Removes from S3 storage and the database. Storage quota is freed immediately.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `file_token` | string | Yes | UUID of the file to delete. Get via `fetch_files`. |

**CRITICAL:** Always ask for explicit user confirmation before calling this tool.

---

## send_file

Send a file from the user's Drive as an email attachment using the Sweeppea email template.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `file_token` | string | Yes | UUID of the file to send. Get via `fetch_files`. |
| `recipient_email` | string | Yes | Destination email address |
| `email_subject` | string | No | Custom subject line (default: `"File shared from Sweeppea"`) |
| `email_message` | string | No | Additional message text for the email body |

**Constraints:**
- Max attachment size: **5 MB** (most email providers reject larger attachments)
- Only file owners can send files
- Each transmission is recorded in the file's sharing history

---

## Workflow

```
upload_file        → upload a document or image to Drive
fetch_files        → list files, check storage usage
send_file          → share a file via email
delete_file        → remove a file permanently (confirm first)
```
