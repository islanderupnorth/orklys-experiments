# Stripe Connect Pricing Strategy for Orklys

## Summary

This document outlines how Stripe Connect pricing applies to Orklys' platform where Energy Communities (ECs) collect investments from members and distribute dividends back to them, with wallet functionality for re-investment.

---

## Account Structure

```
Orklys (Platform Account)
├── Energy Community A (Express Connected Account)
│   ├── Member 1 (Express Connected Account) ←── has wallet
│   ├── Member 2 (Express Connected Account) ←── has wallet
│   └── Member N...
├── Energy Community B (Express Connected Account)
│   └── Members...
└── Energy Community N...
```

**Account Type: Express** (required for wallet functionality)

---

## Pricing Overview

### Fixed Fees

| Fee Type               | Amount        | When Charged                                  |
| ---------------------- | ------------- | --------------------------------------------- |
| Monthly active account | €2            | Per account that receives a payout that month |
| Per payout             | 0.25% + €0.25 | Each payout to a bank account                 |

### Transaction Fees (Denmark)

| Card Type          | Fee            |
| ------------------ | -------------- |
| European cards     | 1.4% + 1.80 kr |
| Non-European cards | 2.9% + 1.80 kr |

### Key Insight

**"Active account" = received a payout to their bank that month**

- Funds staying in Orklys wallet = NOT active = **€0 fee**
- Internal transfers (re-investment in Energy Community projects on Orklys) = **€0 fee**
- Only actual bank withdrawals trigger fees

---

## Use Cases & Costs

### Use Case 1: New Member Buys Shares (First Time)

**Flow:**

1. Member signs up on Orklys
2. Redirected to Stripe to create Express account (KYC/identity verification)
3. Returns to Orklys, account connected
4. Member pays for shares (card payment to EC)

**Costs:**

| Step                    | Who Pays              | Cost                           |
| ----------------------- | --------------------- | ------------------------------ |
| Account creation        | —                     | Free                           |
| Card payment (1,000 kr) | Deducted from payment | 1.4% + 1.80 kr = **15.80 kr**  |
| Orklys application fee  | EC pays to Orklys     | Your choice (e.g., 2% = 20 kr) |

**Member pays:** 1,000 kr
**EC receives:** 1,000 - 15.80 - 20 = **964.20 kr**
**Orklys receives:** **20 kr**
**Stripe receives:** **15.80 kr**

---

### Use Case 2: EC Pays Dividends to Members

**Flow:**

1. EC triggers dividend payout (e.g., 10,000 kr total to 100 members)
2. Funds distributed to member wallets (internal ledger)
3. No actual Stripe payout occurs

**Costs:**

| Action                         | Cost   |
| ------------------------------ | ------ |
| Dividend to wallets (internal) | **€0** |
| No bank payouts triggered      | **€0** |

**Total cost to Orklys: €0**

The dividend is an internal ledger operation — money is held by Orklys/Stripe, tracked in your database as member balances.

---

### Use Case 3: Member Withdraws from Wallet to Bank

**Flow:**

1. Member has 500 kr in Orklys wallet
2. Member requests withdrawal
3. Orklys triggers Stripe payout to member's bank

**Costs:**

| Fee                        | Amount                          |
| -------------------------- | ------------------------------- |
| Active account fee         | €2 (~15.50 kr)                  |
| Payout fee (0.25% + €0.25) | 1.25 kr + 1.94 kr = **3.19 kr** |

**Total cost: ~18.69 kr** for this withdrawal

---

### Use Case 4: Member Re-invests Dividends in Another EC

**Flow:**

1. Member has 500 kr dividend in wallet
2. Member invests in EC Project B
3. Internal transfer: Member wallet → EC Project B

**Costs:**

| Action                                       | Cost   |
| -------------------------------------------- | ------ |
| Internal wallet transfer                     | **€0** |
| No card processing (funds already in system) | **€0** |
| No payout triggered                          | **€0** |

**Total cost: €0**

This is the power of the wallet model — re-investment costs nothing.

---

### Use Case 5: EC Withdraws Funds to Bank (for project costs)

**Flow:**

1. EC has collected 100,000 kr from members
2. EC needs to pay contractor, requests withdrawal
3. Orklys triggers payout to EC's bank

**Costs:**

| Fee                                      | Amount            |
| ---------------------------------------- | ----------------- |
| Active account fee                       | €2                |
| Payout fee (0.25% of 100,000 kr + €0.25) | ~250 kr + 1.94 kr |

**Total cost: ~267 kr** for 100,000 kr withdrawal

---

### Use Case 6: Monthly Platform Costs at Scale

**Scenario:** 50 ECs, 2,000 total members

| Activity                          | Count | Cost               |
| --------------------------------- | ----- | ------------------ |
| Members keeping funds in wallet   | 1,600 | €0                 |
| Members withdrawing this month    | 400   | 400 × €2 = €800    |
| Payout fees (avg €100 withdrawal) | 400   | 400 × €0.50 = €200 |
| ECs withdrawing this month        | 20    | 20 × €2 = €40      |
| EC payout fees (avg €5,000)       | 20    | 20 × €12.75 = €255 |

**Total monthly platform cost: ~€1,295**
**Cost per member: ~€0.65/month**

---

## Revenue Model for Orklys

### Application Fees (Per Transaction)

Orklys can charge a percentage on every transaction:

| Transaction Type      | Suggested Fee | Example (1,000 kr) |
| --------------------- | ------------- | ------------------ |
| Member investment     | 1-3%          | 10-30 kr           |
| Dividend distribution | 0-1%          | 0-10 kr            |
| Withdrawal            | Fixed fee?    | 10-25 kr           |

### Example Revenue (Same 50 EC / 2,000 member scenario)

| Revenue Stream                     | Monthly Estimate |
| ---------------------------------- | ---------------- |
| Investment fees (2% on 500,000 kr) | 10,000 kr        |
| Withdrawal fees (10 kr × 420)      | 4,200 kr         |
| **Total revenue**                  | **14,200 kr**    |
| **Stripe costs**                   | **~10,000 kr**   |
| **Net margin**                     | **~4,200 kr**    |

---

## Architecture Requirements

### 1. Wallet Ledger (Database)

Track member balances internally:

- Credits (dividends received, refunds)
- Debits (investments, withdrawals)
- Running balance

### 2. Internal Transfer System

Move funds between wallets without Stripe:

- Member wallet → EC (investment)
- EC → Member wallet (dividend)
- No actual money movement until withdrawal

### 3. Withdrawal Flow

Trigger real Stripe payouts:

- Validate balance
- Create Stripe Transfer/Payout
- Update ledger
- Handle failures

### 4. Stripe Connect Onboarding

For both ECs and Members:

- Express account creation flow
- Return URL handling
- Account status tracking

---

## Comparison: With vs Without Wallet

| Metric                       | Without Wallet | With Wallet |
| ---------------------------- | -------------- | ----------- |
| Monthly fees (2,000 members) | €4,000         | ~€800       |
| Re-investment cost           | Full card fee  | €0          |
| User experience              | Fragmented     | Seamless    |
| Complexity                   | Lower          | Higher      |
| Lock-in / retention          | Low            | High        |

---

## Recommendations

1. **Use Express accounts** for both ECs and members (wallet requires this)
2. **Encourage wallet retention** — lower costs, better UX, higher retention
3. **Charge withdrawal fees** — offset Stripe costs, discourage unnecessary withdrawals
4. **Set competitive application fees** — 1.5-2.5% on investments
5. **Consider Treasury later** — for proper stored-value accounts if regulatory needed

---

## Open Questions

1. **Regulatory**: Does holding member funds require financial licensing in Denmark/EU?
2. **Treasury**: Should we apply for Stripe Treasury for proper embedded finance?
3. **Withdrawal limits**: Minimum withdrawal amount to batch small payouts?
4. **Fee structure**: Who absorbs Stripe fees — platform, EC, or member?

---

## Sources

- [Stripe Connect Pricing](https://stripe.com/connect/pricing)
- [Platform Pricing Tools Documentation](https://docs.stripe.com/connect/platform-pricing-tools)
- [Stripe Denmark Fee Calculator](https://affonso.io/resources/stripe-fee-calculator/denmark)
