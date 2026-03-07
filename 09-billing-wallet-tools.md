# Billing & Wallet Tools

> **Category:** Billing & Wallet | **Tools:** 4 | **Auth required:** Yes

View billing transactions, invoices, wallet credits/debits, and data transfer usage per sweepstakes.

---

## fetch_billing_transactions

Get all billing transactions including invoices, amounts, and payment status.

**Parameters:** None

**Returns:** List of billing transactions sorted by creation date (newest first), with invoice ID, amount, status, and date.

**Use when:** Reviewing invoices, confirming payment history, or auditing billing records.

---

## fetch_billing_consumptions

Get monthly and yearly billing consumption totals.

**Parameters:** None

**Returns:** Aggregated usage data broken down by month and year, showing total consumption amounts.

**Use when:** Reviewing monthly spend, forecasting costs, or reporting usage to stakeholders.

---

## fetch_wallet_transactions

Get all wallet transactions including credits, debits, and payment details.

**Parameters:** None

**Returns:** Full wallet transaction history with credits applied, amounts spent, balance changes, and transaction dates.

**Use when:** Checking available wallet balance or reviewing how wallet credits have been applied.

---

## fetch_data_transfer

Get data transfer records for a specific sweepstakes.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `sweepstakes_token` | string | Yes | UUID of the sweepstakes |

**Returns:** Data transfer records for the sweepstakes including bytes transferred, payment status, and rates.

**Use when:** Understanding the data costs attributed to a specific campaign, or troubleshooting data transfer charges.

---

## When to Use These Tools

- **Monthly billing review:** `fetch_billing_transactions` + `fetch_billing_consumptions`
- **Wallet balance check:** `fetch_wallet_transactions`
- **Per-campaign cost analysis:** `fetch_data_transfer` for each sweepstakes
- **Audit or discrepancy investigation:** Combine all four tools
