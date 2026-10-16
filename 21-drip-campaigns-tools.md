# Drip Campaigns Tools

> **Category:** Drip Campaigns | **Tools:** 7 | **Auth required:** Yes

Read the account automated email and SMS sequences, their delivery reports and who is enrolled, and pause, resume or take one contact out. **Drips are created, edited, activated and deleted only in the Sweeppea app.** Those actions run a launch checklist — template, sender, 10DLC registration, content screening and plan allowance — that the API deliberately does not reproduce, which is why there is no create endpoint. This is a documented design decision, not a missing feature: say that plainly instead of improvising a workaround.

Key behaviors of this module:

- **The gate is split — but not the way Codes and Messaging split it.** The four reads need the **Drip Marketing** module. The three writes (`pause_drip`, `resume_drip`, `remove_drip_contact`) need **Drip Marketing AND Send Message**, with a distinct 403 message for each, and either one missing fails the call. Neither gate could be observed on the test account — `/messaging/drips` answered 200 — so treat it as documented, not verified, and on a 403 report which module the API actually named.
- **`can_run` is a plan signal, not a module signal.** The API documents it as whether the plan lets drips be **activated or resumed**. A `false` there is not evidence the module is off. Read `plan_limit` alongside it — passed through in PascalCase: `MaxAllowed` (0 = unlimited), `CurrentCount`, `LimitReached` — which means no further drip can be created, an app action in any case — and an `Unlimited` boolean (true when the plan sets no drip limit, that is `MaxAllowed` 0).

`can_run` is **independent of the counters**. Measured on a live account (Sweeppea CLI, 2026-10-08): `CurrentCount: 0`, `MaxAllowed: 3`, `LimitReached: false` and `CanRun: false` in the same response. So a false `can_run` is never explained by the allowance. It tells you **that** the plan does not allow it, never **why**: the cause only surfaces when a write is refused, and what a refusal carries has now been measured. A pause refused on a draft answered 409 with `api_data.Status` naming the status — **not** a `Reason` (verified live through the MCP server, 2026-10-13). That is the first `api_data` ever observed on a drip refusal: the envelope is real, the documented `Reason` vocabulary still is not.
- **`remove_drip_contact` is permanent.** The API allows **one enrolment per address per drip, ever**: a removed contact is never enrolled again by that drip, not even if they enter the sweepstakes again. Messages already sent stay in the report. It does **not** unsubscribe anyone — that is `add_suppressions`.
- **The missing writes are missing on purpose.** Creating, editing, activating, archiving and deleting a drip have no endpoints, and the API maintainer confirmed in October 2026 that they are left out deliberately. Email templates and sender profiles have no endpoints either, for the same reason. Never propose or invent a tool for any of them.
- **Only `remove_drip_contact` requires confirmation.** `pause_drip` and `resume_drip` are each other's inverse, so gating them would teach users to confirm without reading.
- **Step numbers are inconsistent — across the category and inside one payload.** `get_drip` returns `Steps[].Index` **0-based**, and so does `steps[].Index` in `fetch_drip_report`: both measured against a live two-step drip on 2026-10-12, no longer documentation alone. `NextStep` in `fetch_drip_enrollments` and the step on each `top_links` entry of `fetch_drip_report` are **1-based** — still documented only, since no drip with clicks or enrolments has existed to check them. `Index` N is the same step as `NextStep` N+1. Note that `fetch_drip_report` carries both schemes at once, so the conversion is needed even within a single response.
- **The three reads pass nested objects through in PascalCase**, exactly as api-v3 sends them. Only the envelope is renamed: `can_run`, `plan_limit`, `top_links`, `total_results`. Inside, read `Steps[].Index`, `BlockedReason`, `StaffHold`, `NextStep`, `SourceType`, `ExitReason`, `MaxAllowed`, `CurrentCount`. The writes do map their payloads.
- **`source_token` is not a UUID.** It is a ParticipantToken **or** a lead / subscriber id, and `SourceType` (`participant | amoe | lead | subscriber`) tells you which. Never pass it to `get_participant` without checking, and never validate it as a UUID.
- **`fetch_drip_enrollments` never returns email addresses or phone numbers.** You cannot look a contact up by address; `source_token` is the only handle. `ExitReason` is a closed vocabulary: `unsubscribed`, `suppressed`, `no_consent`, `removed`, `clicked`, `won`, `invalid`, `undeliverable`, `over_quota`.
- **The list counters count PEOPLE, not messages.** `fetch_drips` reports enrolled / active / completed / exited contacts. Message statistics (sent, delivered, opened, clicked…) exist only in `fetch_drip_report`. Reporting an `Enrolled` of 1,240 as "1,240 sent" is wrong.
- **Retention floors.** Message rows and ended enrolments are both kept 180 days. An absent enrolment row means "not in the last 180 days", not "never enrolled".
- **`fetch_drip_report` excludes test sends**, and its daily rows are UTC days — a drip that was only tested reports zeros, correctly.
- **In `fetch_drip_report`, `null` and `[]` mean different things in `top_links`.** `[]` is nobody clicked; `null` is the links could not be read, which is unknown. The client passes both through unchanged — do not report them the same way.
- **Wildcards come back unresolved**, written as `[FIRST_NAME]`, and are filled per contact at send time.
- **The three writes are recorded in the account log.**

**Verified live on 2026-10-12/13** against `ZZ Test Drip` (two email steps, one real enrolment):

- `pause_drip` on an active drip answers 200 with `Status: "paused"`, `PausedAt` set and `released_units: 0` when nothing was queued. Pausing again answers 409: `This drip cannot be paused: its status is "paused". Only an active drip can be paused.`
- `resume_drip` answers 200 with `Status: "active"`, `PausedAt: null` and a `note` carrying the API own wording that the sequencer runs every 5 minutes. Resuming an active drip answers 409: `This drip cannot be resumed: its status is "active". Only a paused drip can be resumed.`
- Editing a paused drip in the app and resuming by API answers 409: `This drip was changed after it was paused. Resume it from Drip Marketing in the app, where the full checklist runs.` **The refusal message carries the cause in prose.** Whether `Data.Reason` accompanies it has still never been observed, so the documented vocabulary stays documented, not verified.
- An active enrolment reads `{EnrollmentToken, SourceToken, SourceType, Status, ExitReason: null, EnrolledAt, LastSentAt, NextStep, NextStepAt, EndedAt: null}`. With the step at `Index 0` already sent, `NextStep` read **2** — the 1-based claim, confirmed.
- `remove_drip_contact` remains the only endpoint of the category never exercised.

- **Two different Status vocabularies.** `fetch_drips` filters on `draft | active | paused | completed | blocked`; `fetch_drip_enrollments` filters on `active | completed | exited`. They are not interchangeable.
- **The OpenAPI document declares no enums** for `Channel`, `Status` or `Days` — the valid values live only in the prose of the per-endpoint docs. They are written into the tool schemas by hand and match the documented sets exactly.
- **The 180-day report window is a retention limit.** Message rows are kept 180 days, which is why `days` stops there. A longer window does not exist.
- **`pause_drip` is immediate but not retroactive.** Steps already handed to delivery still finish, so a message can land just after the pause. Contacts keep their place, and the allowance reserved for queued messages returns to the monthly balance. Only an **active** drip can be paused: every other status answers 409 with a message naming the status it is in. Verified live on 2026-10-12 against a draft: `This drip cannot be paused: its status is "draft". Only an active drip can be paused.` An already-paused drip answers 409 the same way and stays paused.
- **`resume_drip` re-runs the app's checks** — plan allows drips and that channel, SMS has a registered 10DLC number, the channel is not paused by Sweeppea, no step held by support — and a refusal names its cause in the message, in prose. The docs also describe a machine-readable code in `api_data.Reason`, never yet observed on a real refusal: `Blocked`, `EditedSincePause`, `StepsNotActivated` (409); `PlanDoesNotAllowSharing`, `ChannelNotInPlan`, `NoRegisteredSmsSender`, `StaffHold` (403); `ChannelPaused` (423). `PlanDoesNotAllowSharing` means the plan does not include running drips at all. A **423** means Sweeppea paused sending on that channel: not an account action, not fixable by retrying. A drip edited after pausing, or blocked by the platform, must be resumed in the app. On success no step is sent twice, and the drip picks up within 5 minutes.

## fetch_drips

List the account drip campaigns — automated email or SMS sequences. Defaults to 25 per page, maximum 100. The response carries `can_run` — whether the **plan** lets drips be activated or resumed — plus `plan_limit`, passed through in PascalCase (`MaxAllowed`, where 0 means unlimited; `CurrentCount`; `LimitReached`; `Unlimited`). A `can_run: false` does not mean the module is disabled; report the field rather than interpreting it as one.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sweepstakes_token | string | No | Only drips of this sweepstakes (UUID) |
| channel | string | No | email or sms |
| status | string | No | draft, active, paused, completed or blocked |
| search | string | No | Part of the drip name (max 100 chars) |
| archived | boolean | No | true lists archived drips instead (default: false) |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 100) |

## get_drip

Fetch ONE drip in full: every step with its delay and content, a summary of the audience, and the sending settings. Check `status` and `blocked_reason` before offering a pause or resume, and `staff_hold` before promising anything will send. The drip object arrives in PascalCase, exactly as api-v3 sends it: `Steps[]` entries carry `StepToken`, `Name`, `Delay` (`Unit`, `Value`) and `Content` (`Subject`, `Preheader`, `Blocks`), and the drip also carries `Audience` and `Settings`. `Settings.SendWindow` limits the hours and days anything goes out; what happens to someone who enters **outside** that window — enrolled and waiting, or not enrolled at all — is neither documented nor observed, so do not tell a user which it will be. `Steps[].Index` is **0-based** here, verified against a live drip — see the step-numbering note above. Step content comes back with merge tags unresolved, exactly as written; they are filled per contact at send time, so never show a raw tag as the final copy.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) |

## fetch_drip_report

Statistics of a drip campaign: per step, in total, day by day, and the links people clicked. The activity window is 7, 30, 90 or 180 days and defaults to 30 — anything else is rejected, and 180 is the ceiling because message rows are only kept that long. Rates are computed server-side; quote them as given and name the window. `top_links` holds up to 10 most-clicked links and excludes the unsubscribe link. `[]` means nobody clicked; `null` means the links could not be read, which is unknown — both reach you unchanged. This response carries both numbering schemes: `steps[].Index` is **0-based** (measured) and the step on each `top_links` entry is **1-based** (documented, unobserved).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) |
| days | number | No | Activity window: 7, 30, 90 or 180 (default: 30) |

## fetch_drip_enrollments

List the contacts enrolled in a drip: where each is in the sequence, what is scheduled next, and why anyone left. Defaults to 25 per page, maximum 100. Rows arrive in PascalCase, exactly as api-v3 sends them. Each row carries `SourceType` (`participant | amoe | lead | subscriber`), so read it before treating the id as a participant. **Email addresses and phone numbers are never returned** — `SourceToken` is the only handle. `NextStep` is **1-based**. `ExitReason` is a closed vocabulary: quote the value rather than paraphrasing it into a cause you were not given. Enrollment rows identify people — report them to the account owner and do not copy them elsewhere.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) |
| status | string | No | active, completed or exited — a different vocabulary from the drip statuses |
| source_token | string | No | Only this contact. A ParticipantToken OR a lead / subscriber id — not always a UUID |
| page | number | No | Page number (default: 1) |
| items_per_page | number | No | Results per page (default: 25, max: 100) |

## pause_drip

Pause an ACTIVE drip: no new contact is enrolled and no step goes out until it is resumed. Reversible with `resume_drip`, which is why it needs no confirmation. Pausing is immediate and atomic, but steps already handed to delivery still finish — a message can land just after the pause, so do not promise everything in flight stopped. Contacts keep their place. The response reports `released_units`, the allowance reserved for queued messages that returns to the monthly balance while paused. Only an active drip can be paused: any other status answers 409 naming the status it is in (verified on a draft). An already-paused drip answers 409 the same way and stays paused.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) of a drip in status active |

## resume_drip

Resume a drip you paused. It only works on a drip still in status `paused` that was NOT edited since, and the account can still send — otherwise 409, with the cause spelled out in the message. Resuming a drip that is not paused answers 409 naming the status it is in. A **423** means Sweeppea itself paused sending on that channel: not something the account did, and not something retrying fixes. Sending restarts within a few minutes (the sequencer runs on an interval), so do not report a step as already sent.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) of a paused drip, not edited since |

## remove_drip_contact

Take ONE contact out of a drip so it receives no further steps. `confirm: true` is REQUIRED. **This is permanent**: the API allows one enrolment per address per drip, ever — a removed contact is never enrolled again by this drip, not even if they enter the sweepstakes again. Messages already sent stay in the report. It removes someone from this one drip and does **not** unsubscribe them (that is `add_suppressions`). To stop the sequence for everyone, use `pause_drip`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| drip_token | string | Yes | Drip campaign token (UUID) |
| source_token | string | Yes | The contact: a ParticipantToken OR a lead / subscriber id — not always a UUID |
| confirm | boolean | Yes | Must be true — the contact stops receiving every remaining step |
