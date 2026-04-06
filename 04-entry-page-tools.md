# Entry Page Tools

> **Category:** Entry Page | **Tools:** 3 | **Auth required:** Yes

Configure the sweepstakes entry page: form fields, display settings, compliance toggles, age gate, AMOE, bonus entries, social follows, and more.

---

## get_entry_fields

Get all form fields defined for a sweepstakes entry page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** List of form fields with name, type, label, and whether each field is required.

**CRITICAL:** Always call this before `add_participant` to discover field names and required fields. Field names use underscores for spaces (e.g., `First_Name`, `Zip_Code`).

**Notes:**
- Default fields are Name and Email.
- Additional fields (Phone, Date of Birth, Address, City, State, ZIP) must be added through the Sweeppea web interface under Entry Page > FIELDS.
- For alcohol/cannabis promotions, a Date of Birth field is required.

---

## get_entry_settings

Get all current entry page settings for a sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** Complete entry page configuration including display settings, colors, spacing, compliance toggles, confirmation page, winners page, age gate, AMOE, geolocation, analytics, social media follows, sharing rewards, bonus entries, and sponsor profiles.

**Use when:** Before calling `update_entry_settings` to review current values, or auditing entry page compliance settings.

---

## update_entry_settings

Partially update entry page settings. Supports 80+ configurable fields.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |
| `settings` | object | Yes | Object with 1–5 PascalCase field names to update |

**LIMIT:** Maximum 5 fields per request. If more than 5 fields need updating, break into multiple sequential requests.

**Color fields format:**
```json
{ "EntryPageBackgroundColor": { "hexa": "#1a1b2e" } }
{ "AgeGateBackgroundColor": { "hex": "#ffffff" } }
```

**Measure fields:** `EntryPageWidthMeasure` only accepts `"%"` or `"px"`.

---

### Configurable Fields Reference

#### Display & Layout
| Field | Type | Description |
|---|---|---|
| `EntryPageHeadline` | string | Main headline text |
| `EntryPageDescription` | string | Description text below headline |
| `EntryPageAbbreviatedRules` | string | Short rules summary |
| `EntryPageWidth` | number | Page width value |
| `EntryPageWidthMeasure` | string | `"px"` or `"%"` |
| `EntryPageBorder` | boolean | Show border |
| `EntryPageRadius` | number | Border radius (px) |
| `EntryPageMarginTop` | number | Top margin (px) |
| `EntryPageMarginBottom` | number | Bottom margin (px) |
| `EntryPageShowOverlay` | boolean | Show background overlay |

#### Colors
| Field | Format | Description |
|---|---|---|
| `EntryPageBackgroundColor` | `{ "hexa": "#RRGGBB" }` | Outer background color |
| `EntryPageBackgroundInnerColor` | `{ "hexa": "#RRGGBB" }` | Inner panel background |
| `EntryPageTextColor` | `{ "hexa": "#RRGGBB" }` | Text color |
| `EntryPageButtonColor` | `{ "hexa": "#RRGGBB" }` | Button color |

#### Compliance (Required)
| Field | Type | Description |
|---|---|---|
| `TermsConditionsSwitch` | boolean | **REQUIRED** — "I agree with Official Rules" checkbox |
| `TermsConditionsMessage` | string | Custom terms message |
| `SelectedOfficialRules` | string | Rules document token to link |
| `ReCaptcha` | boolean | Enable Google reCAPTCHA |

#### Age Gate (ONLY for alcohol/cannabis — 21+ promotions)
| Field | Type | Description |
|---|---|---|
| `ActivateAgeGateSwitch` | boolean | Enable age verification gate. ONLY activate when min_age is 21+ (alcohol/cannabis). NEVER activate for 18+ or 13+ promotions. |
| `AgeGateHeadline` | string | Age gate headline |
| `AgeGateDescription` | string | Age gate description |
| `AgeGateMinAge` | number | Minimum age (13, 18, or 21) |
| `AgeGateBackgroundColor` | `{ "hex": "#RRGGBB" }` | Age gate background |
| `AgeGateTextColor` | `{ "hex": "#RRGGBB" }` | Age gate text color |

#### AMOE (Required when entry involves purchase)
| Field | Type | Description |
|---|---|---|
| `ActivateAmoeSwitch` | boolean | Enable Alternate Method of Entry |
| `AmoeHeadline` | string | AMOE page headline |
| `AmoeDescription` | string | AMOE page description |
| `AmoeEntries` | number | Entries awarded per AMOE submission |
| `EnableInternationalAMOEForm` | boolean | Accept international AMOE submissions |

#### Confirmation Page
| Field | Type | Description |
|---|---|---|
| `ConfirmationPageHeadline` | string | Post-entry confirmation headline |
| `ConfirmationPageDescription` | string | Confirmation message |
| `ExternalConfirmationPageURI` | string | Redirect to external URL after entry |
| `ConfirmationYoutubeUrl` | string | YouTube video on confirmation page |
| `WebExpirationMessage` | string | Message shown after sweepstakes ends |

#### Winners Page
| Field | Type | Description |
|---|---|---|
| `ActivateWinnersSwitch` | boolean | Enable public winners page |
| `WinnersPageHeadline` | string | Winners page headline |
| `WinnersPageDescription` | string | Winners page description |
| `ExternalWinnersPageURI` | string | Link to external winners page |

#### Analytics & Widgets
| Field | Type | Description |
|---|---|---|
| `CollectStatistics` | boolean | Enable analytics collection |
| `EnableShareWidget` | boolean | Show sharing buttons |
| `EnableProgressBar` | boolean | Show entries progress bar |
| `EnableSweepstakesCountdown` | boolean | Show countdown timer |
| `EnableNumberOfParticipants` | boolean | Display participant count |
| `EnableSocialWidget` | boolean | Enable social media widget |

#### Opt-ins
| Field | Type | Description |
|---|---|---|
| `EmailOptInSwitch` | boolean | Show email marketing opt-in |
| `EmailOptInMessage` | string | Email opt-in label text |
| `SMSTextOptInSwitch` | boolean | Show SMS marketing opt-in |

#### Geolocation (for GPS/IP physical boundaries ONLY — NOT for state-level eligibility)
| Field | Type | Description |
|---|---|---|
| `GeoLocation` | boolean | Enable GPS/IP-based geolocation detection. For physical location boundaries (e.g., store radius), NOT for state-level restrictions. Use `states` parameter in `create_rules_wizard` for state eligibility. |
| `GeoLocationIsRequiredToRenderPage` | boolean | Require geolocation to view page |
| `AllowParticipantsWithinFences` | boolean | Restrict to geographic fences (physical boundaries) |

#### Bonus Entries
| Field | Type | Description |
|---|---|---|
| `BonusEntriesSwitch` | boolean | Enable bonus entries system |
| `BonusEntriesValue` | number | Default bonus entry value |
| `BonusEntriesOnRegister` | number | Entries for registering |
| `BonusEntriesOnShareFacebook` | number | Entries for sharing on Facebook |
| `BonusEntriesOnShareX` | number | Entries for sharing on X (Twitter) |
| `BonusEntriesOnShareInstagram` | number | Entries for sharing on Instagram |
| `BonusEntriesOnShareTikTok` | number | Entries for sharing on TikTok |
| `BonusEntriesOnShareLinkedIn` | number | Entries for sharing on LinkedIn |
| `BonusEntriesOnSharePinterest` | number | Entries for sharing on Pinterest |
| `BonusEntriesOnShareThreads` | number | Entries for sharing on Threads |
| `BonusEntriesOnShareReddit` | number | Entries for sharing on Reddit |
| `BonusEntriesOnShareSnapchat` | number | Entries for sharing on Snapchat |
| `BonusEntriesOnShareWhatsApp` | number | Entries for sharing on WhatsApp |
| `BonusEntriesOnShareTelegram` | number | Entries for sharing on Telegram |
| `BonusEntriesOnShareGenericLink` | number | Entries for generic link share |
| `BonusEntriesOnShareEmail` | number | Entries for email share |
| `BothSocialShareParticipantsAwardedSwitch` | boolean | Award both sharer and receiver |
| `BothEmailShareParticipantsAwardedSwitch` | boolean | Award both email sharer and receiver |

#### Social Follows
| Field | Type | Description |
|---|---|---|
| `FollowFacebookSwitch` | boolean | Show Facebook follow button |
| `FollowXSwitch` | boolean | Show X (Twitter) follow button |
| `FollowInstagramSwitch` | boolean | Show Instagram follow button |
| `FollowTikTokSwitch` | boolean | Show TikTok follow button |
| `FollowLinkedInSwitch` | boolean | Show LinkedIn follow button |
| `FollowPinterestSwitch` | boolean | Show Pinterest follow button |
| `FollowThreadsSwitch` | boolean | Show Threads follow button |
| `FollowRedditSwitch` | boolean | Show Reddit follow button |
| `FollowSnapchatSwitch` | boolean | Show Snapchat follow button |
| `FollowYoutubeSwitch` | boolean | Show YouTube follow button |
| `FollowTwitchSwitch` | boolean | Show Twitch follow button |

#### Follow Bonus Entries
| Field | Type | Description |
|---|---|---|
| `BonusEntriesFollowFacebook` | number | Entries for following on Facebook |
| `BonusEntriesFollowX` | number | Entries for following on X |
| `BonusEntriesFollowInstagram` | number | Entries for following on Instagram |
| `BonusEntriesFollowTikTok` | number | Entries for following on TikTok |
| `BonusEntriesFollowLinkedIn` | number | Entries for following on LinkedIn |
| `BonusEntriesFollowPinterest` | number | Entries for following on Pinterest |
| `BonusEntriesFollowThreads` | number | Entries for following on Threads |
| `BonusEntriesFollowReddit` | number | Entries for following on Reddit |
| `BonusEntriesFollowSnapchat` | number | Entries for following on Snapchat |
| `BonusEntriesFollowYoutube` | number | Entries for following on YouTube |
| `BonusEntriesFollowTwitch` | number | Entries for following on Twitch |

#### Sponsor Social Profiles (for follow buttons)
| Field | Type | Description |
|---|---|---|
| `SponsorFacebookProfile` | string | Facebook profile URL or username |
| `SponsorXProfile` | string | X (Twitter) handle |
| `SponsorInstagramProfile` | string | Instagram handle |
| `SponsorTikTokProfile` | string | TikTok handle |
| `SponsorLinkedInProfile` | string | LinkedIn profile URL |
| `SponsorPinterestProfile` | string | Pinterest handle |
| `SponsorThreadsProfile` | string | Threads handle |
| `SponsorRedditProfile` | string | Reddit username |
| `SponsorSnapchatProfile` | string | Snapchat username |
| `SponsorYoutubeProfile` | string | YouTube channel URL |
| `SponsorTwitchProfile` | string | Twitch username |

---

## What Must Be Done in the Web Interface

The following CANNOT be configured via MCP and must be done at app.sweeppea.com:

| Feature | Location |
|---|---|
| Upload header/footer images | Entry Page > DESIGN |
| Add/remove/reorder form fields | Entry Page > FIELDS |
| IP validation, honeypot, session tokens | Entry Page > SECURITY |
| Keyphrases for exclusive access | Entry Page > FORM ACCESS |
| Webhooks (Zapier, direct API) | Entry Page > WEBHOOKS |

---

## Compliance Requirements

The following must be enabled for every sweepstakes:

- `TermsConditionsSwitch: true` — Required. Displays "I agree with the Official Rules" checkbox.
- `ActivateAgeGateSwitch: true` — Required ONLY for alcohol and cannabis promotions (21+). NEVER activate for 18+ or 13+ promotions.
- `ActivateAmoeSwitch: true` — Required when any entry method involves a purchase, donation, or subscription. Note: The AMOE page URL is managed automatically by Sweeppea — it is NOT the same as the entry page URL.
