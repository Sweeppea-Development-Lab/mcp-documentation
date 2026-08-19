# Legal Compliance Guide

> Sweepstakes in the United States are subject to strict federal and state laws. This guide covers the mandatory requirements that every Sweeppea sweepstakes must satisfy before launch.

---

## Mandatory Pre-Launch Checklist

Every sweepstakes must satisfy ALL of the following before going live:

### 1. Official Rules Document
Every sweepstakes **must have Official Rules**. Without rules, the promotion is legally unprotected.

- Use `create_rules_wizard` (recommended) or `create_rule`
- Verify with `fetch_rules` after creation

### 2. "No Purchase Necessary" Statement
If any entry method involves a purchase, donation, or subscription, the rules **must** state "No Purchase Necessary" AND an AMOE must be configured.

### 3. AMOE (Alternate Method of Entry)
Required whenever entry involves any form of payment. The AMOE must provide **"equal dignity"** — the exact same number of entries as the paid method.

Configure via:
- `update_entry_settings` with `ActivateAmoeSwitch: true` and `AmoeEntries: [number]`
- OR: Entry Page > AMOE module in the web interface

### 4. "Void Where Prohibited" Statement
The Official Rules must include this statement. The `create_rules_wizard` handles this automatically.

### 5. Eligibility Requirements
Rules must specify:
- Minimum age (13, 18, or 21)
- Geographic restrictions (which US states/territories are eligible)
- Disqualifications (employees of sponsor, their immediate family members, etc.)

### 6. Start and End Dates with Timezone
Must be precise to the minute. **Both** the sweepstakes configuration and the Official Rules must show matching dates.

### 7. Prize Description and ARV
Every prize must be described in detail with its **Approximate Retail Value (ARV)** stated. This is legally required for transparency and tax purposes.

### 8. Sponsor Identification
Full legal name and physical mailing address of the sponsor. Use `get_business` to auto-populate.

### 9. Winner Selection Method
Rules must specify:
- Random drawing
- Date of drawing
- Number of winners
- Notification timeline

### 10. Privacy Policy Link
Must include a valid URL (with `https://`) to the sponsor's privacy policy.

### 11. Winner Notification Timeline
Must specify how and when winners will be contacted and the deadline to respond.

### 12. Tax Disclosure
- Prizes of **US$2,000 or more per winner** create IRS reporting obligations for the winner (Form 1099-MISC)
- Recommend confirming current requirements with a tax professional
- The rules wizard includes the appropriate tax language automatically when the ARV requires it

### 13. State Registration and Bonding
| Condition | Requirement |
|---|---|
| ARV > $5,000 + open to FL residents | Register and bond in Florida (file at least 7 days before the start date), OR exclude Florida |
| ARV > $5,000 + open to NY residents | Register and bond in New York (file at least 30 days before the start date), OR exclude New York |
| ARV > $500 + sponsor is a retailer with a physical location in RI + entry requires visiting the store (all three conditions) | Register in Rhode Island, OR exclude Rhode Island |

### 14. Age Gate (ONLY for alcohol/cannabis — 21+ promotions)
Required ONLY for alcohol (21+) and cannabis (21+) promotions. NEVER activate Age Gate when minimum age is 18 or 13.

Configure via:
- `update_entry_settings` with `ActivateAgeGateSwitch: true` and `AgeGateMinAge: 3` (the enum code for 21+ — **not** the number 21, which the API rejects)
- OR: Entry Page > AGE GATE module

### 15. Official Rules Consent Checkbox
The Entry Page **must** display "I agree with the Official Rules" as a mandatory checkbox.

Configure via:
- `update_entry_settings` with `TermsConditionsSwitch: true`
- OR: Entry Page > toggle in the web interface

### 16. Canada — Mathematical Skill-Testing Question
**Always** applies when the promotion is open to Canadian residents (`states` values 7, 8, 9 or 10 in `create_rules_wizard`): the Official Rules must state that the winner will be required to correctly answer a mathematical skill-testing question to receive the prize.

### 17. Canada — Quebec (RACJ)
| Scenario | Requirement |
|---|---|
| Offered Canada-wide | No RACJ registration needed |
| Offered ONLY in Quebec, or prizes designated for Quebec winners | RACJ registration required **and** the Official Rules must be translated into French |

---

## Automatic Compliance Warnings

The AI agent must issue these warnings proactively when conditions are detected:

| Condition | Warning |
|---|---|
| ARV > $5,000 + FL/NY not excluded | Registration and bonding required for those states |
| Entry requires purchase + no AMOE | Promotion is an illegal lottery — AMOE required |
| Alcohol sweepstakes + age < 21 | Must be corrected for legal compliance |
| Rules dates ≠ sweepstakes dates | Dates must match exactly |
| No Official Rules document | Cannot launch without rules |
| No Privacy Policy URL | Required for Official Rules |
| Minimum age < 13 | Likely violates COPPA |
| About to delete sweepstakes | Permanent destruction of all data |

---

## Industry-Specific Requirements

### Alcohol Promotions
- The alcohol trigger is **who the sponsor is, not what the prize is**: `alcohol_sweeps=1` in `create_rules_wizard` applies ONLY when the sponsor **produces or manufactures** alcoholic products. A retailer, store, supermarket, bar or restaurant that merely sells alcohol is NOT an alcohol sponsor (`alcohol_sweeps=2`). A brewery giving away concert tickets IS an alcohol sponsor (`alcohol_sweeps=1`)
- Minimum age: **21+**
- Age Gate: **required**
- Seven states require State Alcohol Board approval before an alcohol promotion can run: **Alabama (AL), Indiana (IN), Maine (ME), Maryland (MD), North Carolina (NC), Vermont (VT), West Virginia (WV)**
  - Confirm current requirements and fees with the Sweeppea team before launch

### Cannabis Promotions
- Permitted **only in states where recreational cannabis is legal**
- Minimum age: **21+**
- Age Gate: **required**
- Must **exclude all states** where cannabis is not legal for recreational use
- Federal law still classifies cannabis as a controlled substance — recommend independent legal counsel
- Never assist with cannabis promotions in states where it is illegal

---

## Prohibited Promotions

The following are **never permitted**:

| Type | Reason |
|---|---|
| Illegal lotteries | Purchase + chance + no AMOE = illegal lottery |
| Gambling/casino-style promotions | Disguised gambling |
| Illegal substances (cocaine, heroin, meth, fentanyl, ecstasy, LSD, unauthorized pharmaceuticals, anabolic steroids) | Federally illegal |
| Weapons, explosives, or ammunition | Federally regulated |
| Tobacco or vaping/e-cigarette products | Not permitted on platform |
| Pornography or adult content | Not permitted on platform |
| Promotions targeting minors under 13 | Violates COPPA |
| Pyramid schemes or MLM recruitment | Fraudulent structure |
| Discriminatory promotions | Violates civil rights laws |
| Unregulated financial products (crypto giveaways with purchase, unregistered securities) | Financial regulation violations |

---

## Legal Definitions

**Sweepstakes:** A promotion where prizes are awarded by chance, with no purchase required to enter. Legal in all US states when properly structured.

**Contest:** A promotion where prizes are awarded based on skill. Different rules apply.

**Lottery:** A promotion where prizes are awarded by chance AND entry requires a purchase. Illegal without government licensing.

**AMOE:** Alternate Method of Entry — a free way to enter that provides equal dignity (same number of entries) as any paid entry method.

**ARV:** Approximate Retail Value — the estimated fair market value of a prize. Must be disclosed in Official Rules.

**Bonding:** A surety bond required for high-value sweepstakes in certain states (FL, NY) to ensure prize fulfillment.

---

## Resources

- Federal Trade Commission (FTC) Sweepstakes Guidelines
- Florida Statutes § 849.094 (Registration/Bonding)
- New York Arts and Cultural Affairs Law § 369-e (Registration/Bonding)
- COPPA (Children's Online Privacy Protection Act)
