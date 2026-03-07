# Account Tools

> **Category:** Account | **Tools:** 4 | **Auth required:** Yes

These tools retrieve account-level information: user profile, business details, subscription plan, and API connectivity.

---

## health_check

Verify the connection to the Sweeppea API and validate your API key.

**Parameters:** None

**Use when:** Starting any session to confirm connectivity before making other calls.

**Example response:**
```json
{
  "status": "ok",
  "message": "API connection successful"
}
```

**Notes:**
- Always run this first in any workflow.
- If this fails, stop and ask the user to verify their API key.

---

## get_profile

Get user profile information for the authenticated Sweeppea account.

**Parameters:** None

**Returns:** User name, email, phone number, and account settings.

**Use when:** You need to identify the current user or retrieve their contact information.

**Notes:**
- Use the returned email/phone as fallback contact info for winner notifications.

---

## get_business

Get business information for the authenticated Sweeppea account.

**Parameters:** None

**Returns:** Legal company name, street address, city, state, ZIP code, and business settings.

**Use when:** Auto-populating Sponsor fields in Official Rules (sponsor name, address).

**Notes:**
- **Always call this before `create_rules_wizard`** to auto-populate sponsor details.
- The business address determines state-level compliance requirements.

---

## get_plan

Get subscription plan details for the Sweeppea account.

**Parameters:** None

**Returns:** Plan name, pricing, limits (max active sweepstakes, max total sweepstakes), available features, and current usage.

**Use when:** Checking if the account can create additional sweepstakes before attempting `create_sweepstakes`.

**Notes:**
- Default limits: 3 active sweepstakes, 10 total (active + inactive).
- Warn the user if they are at their limit before attempting to create a new sweepstakes.

---

## Recommended Call Order

```
health_check → get_profile → get_business → get_plan
```

Run all four at session start before any other operations.
