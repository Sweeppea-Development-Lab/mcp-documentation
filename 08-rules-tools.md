# Rules Tools

> **Category:** Rules | **Tools:** 5 | **Auth required:** Yes

Create and manage Official Rules documents. Every legally compliant sweepstakes in the United States **must have Official Rules**. Without rules, the promotion is legally unprotected.

---

## fetch_rules

Get all official rules documents for a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** List of rules documents (primary and secondary) with token, title, primary status, and creation date.

**CRITICAL:** Always call `fetch_rules` before creating new rules. If primary rules already exist, new rules will be **secondary** (not displayed on the Entry Page or assigned to the short URL). The user can change primary status in the app under Official Rules.

---

## create_rules_wizard

Generate complete Official Rules via a 14-step wizard. Complete HTML rules are generated server-side. **This is the primary and recommended method for creating rules.**

**Required Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `sweepstakes_token` | string | UUID of the sweepstakes |
| `arv` | number | `1` = ARV ≥ $5,000 (requires bonding/registration for FL/NY), `2` = ARV < $5,000 |
| `alcohol_sweeps` | number | `1` = Yes (alcohol industry), `2` = No |
| `sweepstakes_name` | string | Official name (6–60 characters) |
| `start_date` | string | `YYYY-MM-DD` — must match sweepstakes config |
| `start_time` | string | `HH:MM` (24-hour) — must match sweepstakes config |
| `start_timezone` | string | IANA timezone identifier (e.g., `America/New_York`) |
| `end_date` | string | `YYYY-MM-DD` — must match sweepstakes config |
| `end_time` | string | `HH:MM` (24-hour) — must match sweepstakes config |
| `end_timezone` | string | IANA timezone identifier |
| `prize_description` | string | Detailed prize description (max 5,000 chars) |
| `prize_include_travel` | boolean | Prize includes travel (triggers travel-specific disclosures) |
| `prize_is_vehicle` | boolean | Prize is a vehicle (triggers vehicle-specific disclosures) |
| `prize_value` | number | Total combined ARV in USD (no $ symbol, numeric only, must be > 0) |
| `entry_period_selector` | number | `1` = Single drawing at conclusion, `2` = Multiple/periodic drawings |
| `sponsor_name` | string | Full legal name of the sponsor (use `get_business` data) |
| `sponsor_address` | string | Sponsor street address (use `get_business` data) |
| `sponsor_city` | string | Sponsor city (use `get_business` data) |
| `sponsor_state` | string | Sponsor state abbreviation (use `get_business` data) |
| `sponsor_zip_code` | string | Sponsor ZIP code (use `get_business` data) |
| `method_of_entry` | number | `1`=Website, `2`=SMS, `3`=Social, `4`=Other, `5`=Purchase, `6`=Donation, `7`=Subscription, `8`=Purchase+Website |
| `min_age` | number | `1`=18+, `2`=21+, `3`=13+ |
| `states` | number | Geographic eligibility: `1`=Contiguous US (48), `2`=50 US States, `3`=50 States + DC, `4`=Specific states list, `5`=North America, `6`=US + Canada, `7`=US + Mexico, `8`=US + Puerto Rico, `9`=US + All Territories, `10`=Worldwide |
| `privacy_policy_url` | string | Valid URL (must start with `https://`) to sponsor's privacy policy |
| `sweeppea_entry_page` | number | `1`=Sweeppea-hosted entry page, `2`=Custom entry page, `3`=No digital entry page |

**Conditional Parameters:**

| Parameter | Required when | Description |
|---|---|---|
| `winner_drawing_date` | `entry_period_selector=1` | Date of the single drawing (`YYYY-MM-DD`) |
| `winners_to_draw` | `entry_period_selector=1` | Number of winners to draw |
| `entry_period_items` | `entry_period_selector=2` | Array of multiple entry periods |
| `social_media_entry_description` | `method_of_entry=3` | How social media entry works |
| `other_description` | `method_of_entry=4` | Description of the "other" entry method |
| `list_of_states` | `states=4` | Array of US state abbreviations |
| `custom_entry_page` | `sweeppea_entry_page=2` | URL of the custom entry page |

**Auto-populated from prior steps:**
- Sponsor name/address/city/state/ZIP → use `get_business` data
- Start/end dates and times → use data from `create_sweepstakes`
- Sweepstakes name → use the name from sweepstakes creation

**Automatic compliance built into the wizard:**
- "No Purchase Necessary" statement (when applicable)
- "Void Where Prohibited" statement
- Tax disclosure (IRS Form 1099-MISC for ARV > $600; additional language for ARV > $2,000)
- Alcohol industry clauses (when `alcohol_sweeps=1`)

---

## create_rule

Create a manual rules document with custom HTML content. Use only when the wizard does not meet your needs (e.g., highly customized legal language).

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `title` | string | Yes | Document title (max 100 chars) |
| `document_content` | string | Yes | Full HTML rules content (max 1,000,000 chars) |

**Warning:** Secondary rules warning applies — call `fetch_rules` first.

---

## update_rule

Update an existing official rules document. Supports partial updates.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `rules_token` | string | Yes | UUID of the rules document to update |
| `title` | string | No | New title |
| `document_content` | string | No | New full HTML content |
| `abbreviated_rules_shopify` | string | No | Abbreviated rules for Shopify integration |

**Use when:** Correcting dates, prize descriptions, or sponsor information after creation.

---

## delete_rule

**DESTRUCTIVE — IRREVERSIBLE.** Permanently delete an official rules document.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `rules_token` | string | Yes | UUID of the rules document to delete |

**CRITICAL:** Always ask for explicit user confirmation. Deleting the primary rules document means the sweepstakes no longer has legally required rules.

---

## Compliance Checklist for Rules

Before finalizing rules, verify:

- [ ] Sponsor name and address match `get_business` data
- [ ] Start/end dates match the sweepstakes configuration exactly
- [ ] Prize description is detailed and includes quantity per winner
- [ ] ARV is stated for each prize
- [ ] Eligibility states are correct and complete
- [ ] Entry method matches the configured entry page
- [ ] Privacy Policy URL is valid and uses `https://`
- [ ] "No Purchase Necessary" statement included (if applicable)
- [ ] "Void Where Prohibited" statement included
- [ ] Minimum age is correct (13, 18, or 21)
- [ ] Tax disclosure included for prizes > $600 ARV

---

## ARV Compliance Warnings

| Condition | Action Required |
|---|---|
| ARV > $5,000 + open to FL residents | Register and bond in Florida, OR exclude Florida |
| ARV > $5,000 + open to NY residents | Register and bond in New York, OR exclude New York |
| ARV > $500 + sponsor has store in RI | Register in Rhode Island, OR exclude Rhode Island |
