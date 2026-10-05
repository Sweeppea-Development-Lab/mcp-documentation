# Codes & Coupons Tools

> **Category:** Codes & Coupons | **Tools:** 14 | **Auth required:** Yes

Import or generate promotional codes for a sweepstakes, assign them to participants, redeem them at the point of sale and control how the entry and AMOE pages hand them out. **Only the writes of this category are gated by the Codes & Coupons module.** The four reads (`fetch_code_stats`, `fetch_codes`, `get_code`, `get_code_settings`) do not document a module-not-enabled 403 — if one of them answers 403, the cause is something else. On a genuine module 403, do NOT retry; contact Sweeppea support to request access.

Key behaviors of this module:

- **Gating is asymmetric, and the split is exactly read vs write.** Verified against the OpenAPI spec (2026-09-30): the ten write endpoints document a `"Codes & Coupons module not enabled"` 403; the four read endpoints do not. Note also that the `PaidModuleManageCoupons` flag on the plan does **not** predict the gate — writes were observed succeeding in production on an account where that flag was `false`.
- **Two ways to name a code.** `get_code` and `redeem_code` accept either `coupon_token`, or `coupon_code` plus `sweepstakes_token` — the second form is what a point-of-sale system has in hand. `coupon_token` takes precedence when both are sent. Codes are matched **exactly and case-sensitively**.
- **`null` clears, omission preserves.** In `update_code` and `update_code_settings`, sending `null` EMPTIES the field while leaving it out keeps the current value. Never send `null` unless the user asked to clear that field.
- **Two encodings of "code type".** `generate_codes` takes `type` as a **string** (`alphanumeric` / `numeric`); `update_code_settings` takes `coupon_code_type` as an **integer** (1 numeric, 2 alphanumeric). They are not interchangeable — do not copy a value from one to the other.
- **Batch calls report what they skipped.** `create_codes` separates `Created` from `Duplicates` and `Invalid`; `delete_codes` separates `Deleted` from `Skipped`. Report those counts instead of assuming the whole batch went through.
- **Destructive calls need confirmation.** `delete_codes` and `unassign_code` require `confirm: true`. `unassign_code` also wipes any redemption recorded on the code — offer `unredeem_code` first when the user only wants to undo a redemption.
- **`send_code` puts a real message on the wire** through the Messaging Engine and requires `confirm: true`. It has no address field: the recipient is whoever holds the code.
- **Token display rule:** never show `CouponToken` or `ParticipantToken` UUIDs to the user. The code itself (`CouponCode`) is what they recognize.

## fetch_code_stats

Count the codes of a sweepstakes by status. Cheap — use it instead of paging through fetch_codes when the user only wants a number.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |

## fetch_codes

One page of a sweepstakes codes, with the participant each code is assigned to. Filtering, search and sorting happen server-side. `available` means not redeemed AND not assigned.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |
| filter | string | No | all, assigned, unassigned, redeemed, not-redeemed, voided, available (default: all) |
| search | string | No | Part of a code, or of the assigned participant email/phone (max 100 chars) |
| sort_by | string | No | CouponCode, ExpirationDate, Status, CreationDate, RedemptionDate (default: CreationDate desc) |
| sort_desc | boolean | No | true for descending order |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 500) |

## get_code

Fetch one code either by `coupon_token`, or by `coupon_code` plus `sweepstakes_token` — the second form is what a point-of-sale system has in hand. `coupon_token` wins when both are sent. Codes match exactly and case-sensitively.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | No | Code token (UUID). Required unless coupon_code is sent; takes precedence |
| sweepstakes_token | string | No | Sweepstakes token (UUID). Required together with coupon_code |
| coupon_code | string | No | The code itself (max 20 chars: letters, digits, - or _) |

## create_codes

Add codes you ALREADY have (from a POS, a CRM or a printed batch). The response reports Duplicates and Invalid separately from Created — report those counts rather than assuming the whole batch landed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |
| codes | array | Yes | 1 to 1,000 codes, each max 20 chars (letters, digits, - or _) |
| description | string | No | Applied to every code (max 5,000 chars) |
| value | string | No | Applied to every code (max 10 chars, e.g. $10, 15%) |
| external_url | string | No | https:// link applied to every code (max 500 chars) |
| expiration_date | string | No | YYYY-MM-DD, not in the past |
| expiration_time | string | No | HH:mm, 24-hour clock |

## generate_codes

Have Sweeppea invent the codes. `prefix + length + suffix` must not exceed 20 characters. NOTE: `type` is a STRING here, while `update_code_settings` uses the INTEGER `coupon_code_type` for the same idea — the two encodings are not interchangeable.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |
| quantity | number | Yes | How many codes, 1 to 5000 |
| length | number | Yes | Random characters per code, 4 to 20 |
| type | string | No | alphanumeric or numeric (default: alphanumeric) |
| case | string | No | upper, lower or mixed, alphanumeric only (default: upper) |
| prefix | string | No | Placed before every code |
| suffix | string | No | Placed after every code |
| description | string | No | Applied to every code (max 5,000 chars) |
| value | string | No | Applied to every code (max 10 chars) |
| external_url | string | No | https:// link (max 500 chars) |
| expiration_date | string | No | YYYY-MM-DD, not in the past |
| expiration_time | string | No | HH:mm, 24-hour clock |

## update_code

Partial update of ONE code. The nullable fields accept `null`, which CLEARS them — omitting a field leaves it untouched. The code itself can only be renamed while unassigned and not redeemed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | Yes | Code token (UUID) |
| coupon_code | string | No | New code. Only while unassigned and not redeemed; unique per sweepstakes |
| description | string|null | No | Max 5,000 chars. null CLEARS it |
| value | string|null | No | Max 10 chars. null CLEARS it |
| external_url | string|null | No | https:// link, max 500 chars. null CLEARS it |
| expiration_date | string|null | No | YYYY-MM-DD. null CLEARS it |
| expiration_time | string|null | No | HH:mm. null CLEARS it |

## delete_codes

IRREVERSIBLE. Only codes nobody holds are deleted — assigned, redeemed and voided codes come back in Skipped. To take a code back from a participant instead, use `unassign_code`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_tokens | array | Yes | 1 to 1,000 code tokens (UUID). Duplicates are collapsed |
| confirm | boolean | Yes | Must be true — the server rejects the call without explicit user confirmation |

## assign_code

Assign an unassigned, not-redeemed code to a participant (regular or AMOE) of the SAME sweepstakes who holds no code yet. Does not notify anyone — use `send_code` for that.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | Yes | Code token (UUID) of an unassigned, not-redeemed code |
| participant_token | string | Yes | Participant token (UUID) of a participant holding no code |

## unassign_code

DESTRUCTIVE: the redemption recorded on the code is cleared along with the assignment. Voided codes are never released. To reverse only a redemption while keeping the participant, use `unredeem_code`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | Yes | Code token (UUID) of an assigned code |
| confirm | boolean | Yes | Must be true — the server rejects the call without explicit user confirmation |

## redeem_code

Redeem by `coupon_token`, or by `coupon_code` plus `sweepstakes_token` at the point of sale. Voided codes are never redeemable. Reversible with `unredeem_code`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | No | Code token (UUID). Takes precedence when both forms are sent |
| sweepstakes_token | string | No | Sweepstakes token (UUID). Required together with coupon_code |
| coupon_code | string | No | The code the customer presented |

## unredeem_code

Only a redeemed code changes; the assignment is kept. This is the safe way to undo `redeem_code`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | Yes | Code token (UUID) of a redeemed code |

## get_code_settings

Read-only. Call before `update_code_settings` so you change only what the user asked and can report the current values.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |

## update_code_settings

Registration modes: 0 none, 1 assign an existing code, 2 generate a new code, 3 ask the participant for their code. The reject/case-sensitive/label fields only matter in mode 3. NOTE: `coupon_code_type` is an INTEGER here (1 numeric, 2 alphanumeric) while `generate_codes` uses a string.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | Yes | Sweepstakes token (UUID) |
| coupon_entry_page_registration | number | No | Entry page mode: 0, 1, 2 or 3 |
| coupon_amoe_page_registration | number | No | AMOE page mode: 0, 1, 2 or 3 |
| what_to_do_when_no_coupon_code_is_available | number | No | 0 nothing, 1 generate a new code |
| coupon_code_type | number | No | Generated codes: 1 numeric, 2 alphanumeric |
| coupon_code_length | number | No | Generated code length, 4 to 20 |
| display_coupon_code_on_confirmation_page | boolean | No | Show the code on the confirmation page |
| send_coupon_code_to_participant_by_email | boolean | No | Email the code to the participant |
| send_coupon_code_to_participant_by_sms | boolean | No | Text the code to the participant |
| reject_invalid_coupon_code | boolean | No | Mode 3: refuse the registration when the code is invalid |
| coupon_code_case_sensitive | boolean | No | Mode 3: compare codes case-sensitively |
| invalid_coupon_code_error_message | string|null | No | Mode 3 message (max 500). null CLEARS it |
| coupon_request_field_label | string|null | No | Mode 3 field label (max 200). null CLEARS it |
| coupon_thank_you_page_message | string|null | No | Thank-you page message (max 1,000). null CLEARS it |

## send_code

SENDS A REAL MESSAGE through the Messaging Engine — it cannot be recalled. No address field: the recipient is whoever holds the code. Suppressions and rate limits are enforced server-side.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coupon_token | string | Yes | Code token (UUID) of a code already assigned to a participant |
| channel | string | Yes | email, sms or both |
| idempotency_key | string | No | 8 to 64 chars [A-Za-z0-9_-]. Retrying with the same key never sends twice |
| confirm | boolean | Yes | Must be true — a real message goes out and cannot be recalled |
