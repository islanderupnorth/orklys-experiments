# Stripe Integration Guide

This document explains how Orklys integrates with Stripe to enable Energy Community share purchases. Use this as an onboarding guide to understand the payment architecture.

## Table of Contents

1. [Overview](#overview)
2. [Business Context & Membership Models](#business-context--membership-models)
3. [Data Model & Relationships](#data-model--relationships)
4. [Account & Product Creation Flow](#account--product-creation-flow)
5. [Checkout Flow](#checkout-flow)
6. [Platform Fees & Revenue](#platform-fees--revenue)
7. [Tax Handling](#tax-handling)
8. [Webhook Integration](#webhook-integration)
9. [Key Files & Code Locations](#key-files--code-locations)
10. [Testing](#testing)
11. [Troubleshooting](#troubleshooting)

---

## Overview

Orklys uses **Stripe Connect** with a **Direct Charges** model. This means:

- **Orklys** (Platform) has a main Stripe account
- **Each Energy Community** gets a connected Stripe account (managed by the platform)
- **Payments go directly** to the Energy Community's connected account
- **Platform fees** are automatically deducted as application fees

### Why This Model?

| Benefit                   | Description                                                                |
| ------------------------- | -------------------------------------------------------------------------- |
| **Regulatory Compliance** | Energy Communities receive funds directly, simplifying financial reporting |
| **Scalability**           | Each community is financially isolated                                     |
| **Tax Automation**        | Stripe handles VAT calculation per transaction                             |
| **Platform Control**      | Orklys manages all accounts without giving dashboard access                |

---

## Business Context & Membership Models

### Key Questions Explored

Before implementing the payment system, the following questions were clarified with leadership:

- Do users automatically become REC members when purchasing a share?
- Is there a parallel membership fee (monthly/annual)?
- At what point do users have voting rights (EC level or project level)?
- Can users own shares without being REC members?
- Can users be REC members without owning shares?

### Membership Options (CEO Decisions)

Energy Communities can choose from three membership models:

| Option       | Description                                               |
| ------------ | --------------------------------------------------------- |
| **Option 1** | Buy a share = automatic REC membership                    |
| **Option 2** | Buy a share without REC membership (just asset ownership) |
| **Option 3** | Buy a share AND pay a separate membership fee             |

**Key decisions:**

- REC can choose which model to use (configurable per community)
- A person can be a REC member without being a shareholder
- For Option 3: membership fee is annual (toggleable on/off)
- Default membership fee: ~200 DKK (configurable amount)

### Current Implementation Phase

For 2026, users are only **reserving** shares, not purchasing them:

- First payment is a one-off share reservation fee
- Membership functionality not needed until actual share sales begin
- This simplifies the initial Stripe integration

### Stripe Product Implications

The membership model affects Stripe setup:

- Share purchase = one-time payment (Stripe Checkout)
- Membership fee = recurring subscription (Stripe Subscriptions)
- May require separate checkout flows for shares vs. membership
- Future consideration: bundled checkout combining both

### Checkout Breakdown (Target UX)

When a user clicks "Become a Co-Owner", they should see:

- **Share cost**: e.g., 2890 DKK per share for 1000 kWh (toggleable)
- **Membership fee**: e.g., 190 DKK annual (toggleable)
- **Processing fee**: Includes Orklys + Stripe fees
- **MOMS (VAT)**: Tax amount

---

## Data Model & Relationships

Understanding the relationship between our database models and Stripe entities is crucial:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ORKLYS PLATFORM                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐       ┌──────────────────┐       ┌─────────────┐ │
│  │     User     │       │ EnergyCommunity  │       │  ECProject  │ │
│  │              │       │                  │       │             │ │
│  │ stripeAcct ──┼───┐   │  id              │   ┌───┼─ stripeProduct│
│  │ Id           │   │   │  name            │   │   │ Id          │ │
│  │              │   │   │  slug            │   │   │ stripePriceId│ │
│  └──────────────┘   │   │  ownerId ────────┼───┘   │ pricePerShare│ │
│                     │   │                  │       │ totalShares │ │
│                     │   └────────┬─────────┘       └──────┬──────┘ │
│                     │            │                        │        │
│                     │            │ 1:N                    │        │
│                     │            ▼                        │        │
│                     │   ┌──────────────────┐              │        │
│                     │   │   ECFinancials   │              │        │
│                     │   │ platformFeePercent              │        │
│                     │   └──────────────────┘              │        │
│                     │                                     │        │
└─────────────────────┼─────────────────────────────────────┼────────┘
                      │                                     │
                      ▼                                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         STRIPE CONNECT                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────┐       ┌──────────────┐       ┌─────────────┐ │
│  │ Connected Account│       │   Product    │       │    Price    │ │
│  │                  │       │              │       │             │ │
│  │  id: acct_xxx    │◄──────│  id: prod_xxx│◄──────│ id: price_xx│ │
│  │  country: DK     │       │  name        │       │ unit_amount │ │
│  │  capabilities    │       │  metadata    │       │ currency    │ │
│  └──────────────────┘       └──────────────┘       └─────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Relationships

| Orklys Entity               | Stripe Entity     | Relationship                                |
| --------------------------- | ----------------- | ------------------------------------------- |
| `User.stripeAccountId`      | Connected Account | 1:1 - Each organiser has one Stripe account |
| `ECProject.stripeProductId` | Product           | 1:1 - Each project offering has one product |
| `ECProject.stripePriceId`   | Price             | 1:1 - Current active price for the project  |

### Database Models

**User** (Organiser)

```prisma
model User {
  stripeAccountId String?  // Stripe Connect account ID (acct_xxx)
  // ... other fields
}
```

**ECProject** (Share Offering)

```prisma
model ECProject {
  stripeProductId String?  // Stripe product ID (prod_xxx)
  stripePriceId   String?  // Stripe price ID (price_xxx)
  pricePerShare   Decimal  // e.g., 1000.00 (€1,000)
  totalShares     Int      // e.g., 100
  reservedShares  Int      // Shares reserved but not paid
  soldShares      Int      // Shares fully purchased
  currentPhase    ProjectPhase  // RESERVATIONS or SHARES_SALE
  status          ProjectStatus // OPEN, PAUSED, CLOSED
  // ... other fields
}
```

**ECFinancials** (Community Settings)

```prisma
model ECFinancials {
  platformFeePercent Decimal?  // e.g., 5.00 (5%)
  // ... other fields
}
```

---

## Account & Product Creation Flow

When a new Energy Community is created during onboarding, the following happens automatically:

### Sequence Diagram

```
User                    Orklys                     Stripe
  │                        │                          │
  │ Complete Onboarding    │                          │
  │───────────────────────►│                          │
  │                        │                          │
  │                        │ 1. Create Connect Account│
  │                        │─────────────────────────►│
  │                        │      acct_xxx            │
  │                        │◄─────────────────────────│
  │                        │                          │
  │                        │ 2. Create Product        │
  │                        │─────────────────────────►│
  │                        │      prod_xxx            │
  │                        │◄─────────────────────────│
  │                        │                          │
  │                        │ 3. Create Price          │
  │                        │─────────────────────────►│
  │                        │      price_xxx           │
  │                        │◄─────────────────────────│
  │                        │                          │
  │                        │ 4. Save to Database      │
  │                        │  - User.stripeAccountId  │
  │                        │  - ECProject.stripeProductId
  │                        │  - ECProject.stripePriceId
  │                        │                          │
  │  Success               │                          │
  │◄───────────────────────│                          │
```

### Code Location

**File:** `server/routers/community/core.ts` - `createOnboardingCommunity` mutation

```typescript
// 1. Create Stripe Connect Account
const account = await stripe.accounts.create({
  controller: {
    fees: { payer: "application" }, // Platform pays Stripe fees
    losses: { payments: "application" }, // Platform covers disputes
    stripe_dashboard: { type: "none" }, // No dashboard access
    requirement_collection: "application",
  },
  capabilities: {
    card_payments: { requested: true },
    transfers: { requested: true },
  },
  business_type: "company",
  email: user.email,
  country: "DK",
});

// 2. Create Product on Connected Account
const product = await stripe.products.create(
  {
    name: `${communityName} Share`,
    description: `Share for ${communityName} energy community`,
    default_price_data: {
      unit_amount: 100000, // €1,000 in cents
      currency: "eur",
    },
    metadata: {
      communityId: community.id,
      communityName: community.name,
    },
  },
  { stripeAccount: account.id }
);

// 3. Create ECProject with Stripe references
const ecProject = await prisma.eCProject.create({
  data: {
    energyCommunityId: community.id,
    name: `${communityName} Shares`,
    stripeProductId: product.id,
    stripePriceId: product.default_price,
    pricePerShare: 1000,
    totalShares: 100,
    currentPhase: "RESERVATIONS",
    status: "OPEN",
  },
});
```

---

## Checkout Flow

When a supporter wants to purchase shares, they go through a project-based checkout:

### User Journey

```
1. Visit /projects/{community-slug}
2. Click "Become a Co-Owner"
3. Redirect to /checkout/{community-slug}
4. Select number of shares
5. Click "Proceed to Payment"
6. Redirect to Stripe Checkout (hosted page)
7. Complete payment
8. Redirect to /payment-completed
```

### Technical Flow

```
Frontend                    Backend                      Stripe
    │                          │                            │
    │ GET /checkout/slug       │                            │
    │─────────────────────────►│                            │
    │                          │                            │
    │                          │ getProjectForCheckout      │
    │                          │ (fetch project + community)│
    │                          │                            │
    │ Display checkout form    │                            │
    │◄─────────────────────────│                            │
    │                          │                            │
    │ Click "Proceed"          │                            │
    │─────────────────────────►│                            │
    │                          │                            │
    │                          │ createProjectCheckoutSession
    │                          │───────────────────────────►│
    │                          │                            │
    │                          │    Checkout Session URL    │
    │                          │◄───────────────────────────│
    │                          │                            │
    │ Redirect to Stripe       │                            │
    │◄─────────────────────────│                            │
    │                          │                            │
    │ ════════════════════════ STRIPE HOSTED CHECKOUT ═════│
    │                          │                            │
    │ Payment Complete         │                            │
    │──────────────────────────────────────────────────────►│
    │                          │                            │
    │ Redirect to success_url  │                            │
    │◄──────────────────────────────────────────────────────│
```

### Key API Endpoints

| Endpoint                                   | Purpose                              | File                                   |
| ------------------------------------------ | ------------------------------------ | -------------------------------------- |
| `community.projects.getProjectForCheckout` | Fetch project data for checkout page | `server/routers/community/projects.ts` |
| `stripe.createProjectCheckoutSession`      | Create Stripe Checkout session       | `server/routers/stripe-router.ts`      |
| `stripe.retrieveCheckoutSession`           | Get payment details after completion | `server/routers/stripe-router.ts`      |

### Checkout Session Creation

**File:** `server/routers/stripe-router.ts` - `createProjectCheckoutSession`

```typescript
const session = await stripe.checkout.sessions.create(
  {
    line_items: [
      {
        price: project.stripePriceId,
        adjustable_quantity: {
          enabled: true,
          minimum: project.minShares,
          maximum: Math.min(availableShares, project.maxSharesPerBuyer),
        },
        quantity: input.quantity,
      },
    ],
    payment_intent_data: {
      application_fee_amount: applicationFeeAmount, // Platform fee
    },
    automatic_tax: {
      enabled: true,
      liability: { type: "self" }, // Connected account liable
    },
    billing_address_collection: "required",
    phone_number_collection: { enabled: true },
    mode: "payment",
    success_url: `${baseUrl}/payment-completed?session_id={CHECKOUT_SESSION_ID}&project=${project.energyCommunityId}`,
    cancel_url: `${baseUrl}/projects/${community.slug}`,
    metadata: {
      projectId: project.id,
      communityId: community.id,
      communitySlug: community.slug,
    },
  },
  { stripeAccount: stripeAccountId }
);
```

---

## Platform Fees & Revenue

Orklys collects platform fees as Stripe application fees.

### How Fees Work

1. Fee percentage stored in `ECFinancials.platformFeePercent`
2. Calculated at checkout time based on share price × quantity
3. Automatically deducted by Stripe before funds reach connected account
4. Visible in Stripe Dashboard → Connect → Application Fees

### Fee Calculation

```typescript
const pricePerShareCents = Number(project.pricePerShare) * 100;
const feePercent = financials.platformFeePercent ?? 0;
const applicationFeeAmount = Math.round(
  pricePerShareCents * quantity * (feePercent / 100)
);
```

### Example Transaction

**Scenario:** 2 shares @ €1,000 each, 5% platform fee

```
Customer pays:
  2 × €1,000 shares          = €2,000.00
  VAT (25%)                  = €  500.00
  ─────────────────────────────────────
  Total charged              = €2,500.00

Money distribution:
  Stripe processing fee      = €   35.25  (1.4% + €0.25)
  Platform fee (5%)          = €  100.00  → Orklys
  ─────────────────────────────────────
  Energy Community receives  = €2,364.75
```

### Viewing Fees in Stripe Dashboard

1. Go to https://dashboard.stripe.com
2. Navigate to **Connect** → **Application fees**
3. Filter by date range or connected account

---

## Tax Handling

Orklys uses **Stripe Tax** for automatic VAT calculation.

### Configuration (One-Time Setup)

1. Go to https://dashboard.stripe.com/settings/tax
2. Enable Stripe Tax
3. Add Denmark (DK) as active jurisdiction
4. VAT rate: 25% (automatically applied)

### How It Works

- Stripe calculates VAT based on customer billing address
- VAT displayed as separate line item in checkout
- Tax collected along with payment
- Connected account is liable for tax remittance

### Tax in Checkout Session

```typescript
automatic_tax: {
  enabled: true,
  liability: {
    type: "self", // Connected account handles tax liability
  },
},
billing_address_collection: "required", // Needed for tax calculation
```

---

## Key Files & Code Locations

### Backend (tRPC Routers)

| File                                     | Purpose                                         |
| ---------------------------------------- | ----------------------------------------------- |
| `server/routers/stripe-router.ts`        | Stripe API endpoints (checkout, account status) |
| `server/routers/community/core.ts`       | Account & product creation during onboarding    |
| `server/routers/community/projects.ts`   | Project CRUD, checkout data fetching            |
| `server/routers/community/financials.ts` | Fee percentage management                       |
| `app/api/webhooks/stripe/route.ts`       | Webhook handler for payment events              |

### Frontend

| File                                                | Purpose                      |
| --------------------------------------------------- | ---------------------------- |
| `app/[locale]/(landing)/checkout/[slug]/page.tsx`   | Checkout page                |
| `components/features/checkout/CheckoutForm.tsx`     | Share selection UI           |
| `app/[locale]/(landing)/payment-completed/page.tsx` | Success page                 |
| `app/[locale]/dashboard/projects/page.tsx`          | Dashboard project management |

### Stripe Client

| File            | Purpose                   |
| --------------- | ------------------------- |
| `lib/stripe.ts` | Stripe SDK initialization |

---

## Testing

### Test Mode Credentials

- Use Stripe test mode keys in `.env`
- Test card: `4242 4242 4242 4242`
- Any future expiry, any 3-digit CVC

### Test Scenarios

| Scenario                | How to Test                                       |
| ----------------------- | ------------------------------------------------- |
| **Basic checkout**      | Complete purchase with test card                  |
| **Quantity adjustment** | Increase/decrease shares in Stripe checkout       |
| **Platform fee**        | Set fee %, complete purchase, verify in dashboard |
| **Tax calculation**     | Complete purchase, verify 25% VAT                 |
| **Insufficient shares** | Try to buy more than available                    |

### Viewing Test Data

1. Ensure you're in **Test mode** (toggle in Stripe Dashboard)
2. Payments appear under **Payments** tab
3. Connected accounts under **Connect** → **Accounts**
4. Application fees under **Connect** → **Application fees**

---

## Troubleshooting

### Common Issues

| Error                            | Cause                             | Solution                                     |
| -------------------------------- | --------------------------------- | -------------------------------------------- |
| "Stripe account not found"       | User missing `stripeAccountId`    | Re-run onboarding or manually create account |
| "Project pricing not configured" | ECProject missing `stripePriceId` | Create Stripe product for the project        |
| "No active project found"        | ECProject status not OPEN         | Update project status in database            |
| VAT not showing                  | Stripe Tax not enabled            | Enable in Stripe Dashboard settings          |
| Fee not collected                | `platformFeePercent` is 0 or null | Set fee in ECFinancials                      |

### Debugging Checklist

1. **Check database records:**

   - `User.stripeAccountId` exists?
   - `ECProject.stripeProductId` and `stripePriceId` exist?
   - `ECProject.status === "OPEN"`?

2. **Check Stripe Dashboard:**

   - Is the connected account active?
   - Does the product exist on the connected account?
   - Is the price active?

3. **Check logs:**
   - Server console for tRPC errors
   - Stripe Dashboard → Developers → Logs

### Manual Recovery

If automatic creation failed, you can manually create:

```typescript
// Create Stripe product manually
const product = await stripe.products.create(
  {
    name: "Community Name Share",
    default_price_data: {
      unit_amount: 100000,
      currency: "eur",
    },
  },
  { stripeAccount: "acct_xxx" }
);

// Update ECProject
await prisma.eCProject.update({
  where: { id: projectId },
  data: {
    stripeProductId: product.id,
    stripePriceId: product.default_price,
  },
});
```

---

## Webhook Integration

Stripe webhooks automatically sync payment data to the database after checkout completion.

### Implemented Events

| Event                        | Action                                                     | File                               |
| ---------------------------- | ---------------------------------------------------------- | ---------------------------------- |
| `checkout.session.completed` | Creates `ECShareReservation`, increments `reservedShares`  | `app/api/webhooks/stripe/route.ts` |
| `checkout.session.expired`   | Marks reservation as CANCELLED                             | `app/api/webhooks/stripe/route.ts` |
| `charge.refunded`            | Marks reservation as REFUNDED, decrements `reservedShares` | `app/api/webhooks/stripe/route.ts` |

### Setup Instructions

**1. Register Webhook in Stripe Dashboard:**

1. Go to https://dashboard.stripe.com/webhooks
2. Click "Add endpoint"
3. Enter URL: `https://yourdomain.com/api/webhooks/stripe`
4. Select events: `checkout.session.completed`, `checkout.session.expired`, `charge.refunded`
5. Click "Add endpoint"
6. Copy the signing secret (starts with `whsec_`)

**2. Add Environment Variable:**

```bash
STRIPE_WEBHOOK_SECRET=whsec_...
```

Add this to your deployment environment (e.g., Render.com environment variables).

**3. Local Development Testing:**

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# The CLI will display a webhook signing secret - use this for local testing
```

### Verification

After setup, test by completing a purchase:

1. Complete a test checkout with card `4242 4242 4242 4242`
2. Check server logs for: `✅ Created reservation...`
3. Verify in database:
   ```sql
   SELECT * FROM ec_share_reservations ORDER BY created_at DESC LIMIT 1;
   ```
4. Confirm project's `reservedShares` incremented

### Future Enhancements

- [ ] **Email Notifications** - Send confirmation emails on successful payment
- [ ] **Membership Products** - Recurring subscription for community membership
- [ ] **Phase Transitions** - Change from RESERVATIONS to SHARES_SALE with new pricing
- [ ] **Payout Tracking** - Listen for `payout.paid` for reporting

---

## Resources

- [Stripe Connect Documentation](https://stripe.com/docs/connect)
- [Direct Charges Guide](https://stripe.com/docs/connect/direct-charges)
- [Stripe Tax Documentation](https://stripe.com/docs/tax)
- [Checkout Sessions API](https://stripe.com/docs/api/checkout/sessions)
- [Application Fees](https://stripe.com/docs/connect/direct-charges#collecting-fees)

---

## Glossary

| Term                  | Definition                                                                    |
| --------------------- | ----------------------------------------------------------------------------- |
| **Connected Account** | Stripe account owned by an Energy Community organiser                         |
| **Direct Charges**    | Charges created directly on connected account (vs. destination charges)       |
| **Application Fee**   | Platform fee deducted from payment before reaching connected account          |
| **ECProject**         | Database model representing a share offering with Stripe product/price        |
| **Project Phase**     | Current state: RESERVATIONS (early interest) or SHARES_SALE (actual purchase) |
