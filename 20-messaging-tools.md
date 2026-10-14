# Messaging Tools

> **Category:** Messaging | **Tools:** 12 | **Auth required:** Yes

Read email and SMS campaigns with their delivery reports and recipients, manage the account do-not-contact list, and send a direct message to one of your own participants. **Only the writes of this category are gated by the Send Message module.** The six reads (`fetch_messaging_usage`, `fetch_campaigns`, `get_campaign`, `fetch_campaign_report`, `fetch_campaign_recipients`, `fetch_suppressions`) do not document a module-not-enabled 403. On a genuine module 403, do NOT retry; contact Sweeppea support to request access.

Key behaviors of this module:

- **Campaigns cannot be created through the API, on purpose.** This server can only read, pause, resume and cancel them, and the API maintainer confirmed in October 2026 that creation is left out deliberately. Email templates and sender profiles have no endpoints at all, for the same reason — never propose or invent a tool for them. If the user wants a new campaign, say so plainly and point them at the Sweeppea app — do not improvise a workaround with direct sends.
- **Direct sending is a separate switch from the module.** An account can report `ModuleEnabled: true` and still be refused: observed in production on 2026-09-30, `DirectMessageEnabled: false` produced `403 "Direct messages through the API are not enabled for your account yet. Contact Sweeppea support to enable it."` on both `send_message` and `send_code`. That is account configuration, not a failure — do not retry it as if it were transient.
- **`fetch_messaging_usage` is the diagnostic.** A send can be refused for module access, a channel outside the plan, an exhausted monthly allowance, an active sending pause or a missing 10DLC number. Only this call tells those causes apart — call it before explaining a refusal to the user.
- **Sends are real and unrecallable.** `send_message` and `send_code` require `confirm: true`. There is no address field: the recipient is a participant token inside the account, so neither tool can reach anyone else. Never send one to "test" that it works.
- **SMS is shorter than email.** `message` accepts up to 5,000 characters, but only **1,600** when the channel includes SMS.
- **Rate limits are server-side:** 200 sends per API key per 24h, and 3 per recipient per 24h. Account suppressions are enforced on every send.
- **Suppressions are asymmetric by design.** `add_suppressions` is the safe direction and needs no confirmation. `remove_suppression` requires `confirm: true` and only works on entries added by hand (reason `manual`) — a recipient's own opt-out, a STOP reply or a complaint is **never** removable. Check the reason with `fetch_suppressions` first.
- **`cancel_campaign` cannot be undone** and releases the monthly allowance. `pause_campaign` keeps the allowance held so the campaign can be resumed — offer it first.
- **Report rates over the right denominator.** `fetch_campaign_report` computes rates over messages **sent**, not over the audience. `rows_expired: true` means the per-recipient detail has aged out.
- **Recipient addresses and suppression entries are PII.** Report them to the account owner when asked; never copy them into notes, tickets or another context.

## fetch_messaging_usage

Takes no parameters. CALL THIS FIRST when a send is refused: it is the only tool that distinguishes module access, plan channels, an exhausted allowance, a sending pause and a missing 10DLC number.

No parameters.

## fetch_campaigns

Campaigns CANNOT be created through this API — this server can only read, pause, resume and cancel them. If the user wants a new campaign, say so and point them at the Sweeppea app.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | No | Only campaigns of this sweepstakes (UUID) |
| channel | string | No | email or sms |
| category | string | No | marketing, transactional or referral |
| status | string | No | draft, scheduled, queued, sending, paused, completed, cancelled, failed, blocked |
| search | string | No | Part of the campaign name (max 100 chars) |
| archived | boolean | No | true lists archived campaigns instead (default: false) |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 100) |

## get_campaign

Content, audience, sender profile, counters and the allowed actions. Read the action list before offering a pause, resume or cancel — it reflects what the current status allows.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) |

## fetch_campaign_report

Rates are computed over messages SENT, not over the audience — name the denominator when quoting a percentage. `rows_expired` true means the per-recipient detail has aged out.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) |

## fetch_campaign_recipients

`search` takes ONE EXACT address, not a fragment — narrow with the status filter instead. Recipient addresses are personal data.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) |
| status | string | No | queued, sending, sent, delivered, opened, clicked, bounced, complained, failed, suppressed, unsubscribed, cancelled |
| search | string | No | ONE exact email address or phone number (max 254 chars) |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 100) |

## pause_campaign

Nothing more goes out once the call answers, but messages already handed to the carrier cannot be pulled back. The allowance stays held so the campaign can be resumed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) of a campaign in status queued or sending |

## resume_campaign

Every problem that would stop it again is re-checked first and returned together — read the whole list back to the user instead of retrying.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) of a paused campaign |

## cancel_campaign

CANNOT BE UNDONE — a cancelled campaign cannot be resumed and messages already sent stay sent. If the user only wants to stop it for now, use `pause_campaign`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_token | string | Yes | Campaign token (UUID) in status scheduled, queued, sending, paused or blocked |
| confirm | boolean | Yes | Must be true — the campaign ends permanently |

## fetch_suppressions

Every address that unsubscribed, replied STOP, complained, bounced or was added by hand. Only reason `manual` can ever be removed — check it before promising the user an undo.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| channel | string | No | email or sms |
| search | string | No | ONE exact email address or phone number (max 254 chars) |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 100) |

## add_suppressions

The safe direction — suppressing costs nothing and protects the account, so no confirmation is required. Never downgrades an existing opt-out. The response separates Added from AlreadySuppressed and Invalid.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| channel | string | Yes | email or sms — one channel per call |
| addresses | array | Yes | 1 to 500 email addresses, or phone numbers (E.164 or 10-digit US/CA) |

## remove_suppression

Only entries added by hand (reason `manual`) can be removed — a recipient own opt-out, a STOP reply or a complaint is NEVER removable. Check the reason with `fetch_suppressions` first.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| channel | string | Yes | email or sms |
| address | string | No | The address. Required unless address_hash is sent |
| address_hash | string | No | The 64-character AddressHash from fetch_suppressions. Wins over address |
| confirm | boolean | Yes | Must be true — the address becomes contactable again |

## send_message

SENDS A REAL MESSAGE — it cannot be recalled. No address field: the recipient is a participant token inside the account, so this cannot message anyone else. Max 5,000 characters, but 1,600 when the channel includes SMS. Rate limits (200 per API key / 24h, 3 per recipient / 24h) and suppressions are enforced server-side.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| participant_token | string | Yes | Participant token (UUID) of a participant of your account |
| channel | string | Yes | email, sms or both |
| message | string | Yes | Plain text. Max 5,000 chars — max 1,600 when the channel includes SMS |
| subject | string | No | Email subject (max 200 chars). Defaults to a generic subject |
| idempotency_key | string | No | 8 to 64 chars [A-Za-z0-9_-]. Retrying with the same key never sends twice |
| confirm | boolean | Yes | Must be true — a real message goes out and cannot be recalled |
