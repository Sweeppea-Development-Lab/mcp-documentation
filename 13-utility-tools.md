# Utility Tools

> **Category:** Utilities | **Tools:** 5 | **Auth required:** Yes

Reference data tools for US geographic and phone data, international country lookup, and timezone management.

---

## fetch_timezones

Get all available timezones with IANA identifiers, abbreviations, and UTC offsets.

**Parameters:** None

**Returns:** Full list of timezones with ID, IANA identifier (e.g., `America/New_York`), abbreviation (e.g., `EST`), UTC offset, and display name.

**Default timezone:** ID `7` = Eastern Standard Time (America/New_York)

**Use when:**
- Setting up sweepstakes start/end times
- Scheduling drawings in the user's local timezone
- Populating the `timezone` parameter in `schedule_drawing`
- Creating Official Rules with correct timezone references

**Infer the timezone** from the sponsor's business address state whenever possible (from `get_business`).

---

## fetch_states

Get all US states and territories.

**Parameters:** None

**Returns:** Complete list including all 50 states, Washington DC, Puerto Rico, US Virgin Islands, Guam, American Samoa, and Northern Mariana Islands, with full names and two-letter abbreviations.

**Use when:**
- Populating the `list_of_states` parameter for `create_rules_wizard` when `states=4`
- Validating sponsor address state abbreviations
- Building geographic eligibility lists

---

## fetch_zipcodes

Search US ZIP codes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | Yes | ZIP code number, city name, or state name |

**Returns:** Up to 10 matching ZIP codes with city, state, and county information.

**Use when:** Verifying the sponsor's ZIP code from `get_business`, or looking up ZIP codes for geolocation features.

---

## fetch_areacodes

Search US telephone area codes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Area code number or state name |

**Returns:** Up to 10 matching area codes with state and region information.

**Use when:** Validating phone number area codes, or finding area codes for a specific state.

---

## fetch_countries

Search countries by name, dial code, or ISO abbreviation.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | string | No | Country name (English or Spanish), international dial code, or ISO 2-letter abbreviation |

**Returns:** Up to 10 matching countries with English name, Spanish name, dial code, and ISO code.

**Use when:**
- Sweepstakes has international eligibility considerations
- Validating international phone numbers for AMOE entries
- Populating country selectors on custom entry forms

---

## Quick Reference

| Tool | Use for |
|---|---|
| `fetch_timezones` | Setting dates/times, scheduling drawings |
| `fetch_states` | Geographic eligibility lists, state validation |
| `fetch_zipcodes` | Address verification, geolocation |
| `fetch_areacodes` | Phone number validation |
| `fetch_countries` | International eligibility, phone formatting |
