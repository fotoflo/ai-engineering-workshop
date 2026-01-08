# Stripe Booking Integration

This document outlines the implementation of Stripe payment integration for bike bookings according to the PRD.

## Overview

The Stripe integration enables users to pay for bike bookings with one-time or subscription payments, applying a 3% transaction fee. The system uses a single booking record as the source of truth and handles edits cleanly during the checkout process.

## Features Implemented

### ✅ Database Schema Updates

- Added new booking statuses: `draft`, `pending_payment`, `confirmed`, `review_required`
- Added Stripe-related fields to `Booking` model:
  - `version` - for tracking booking changes
  - `priceBaseCents`, `feeCents`, `totalCents` - pricing in cents
  - `feeRateBps` - transaction fee rate (default 300 = 3%)
  - `stripeCheckoutId` - Stripe session ID
  - `checksum` - for data integrity validation
  - `lockedUntil` - booking lock during payment
- Added company payment settings:
  - `acceptsStripe` - whether company accepts Stripe payments
  - `acceptsSubscription` - whether company accepts subscriptions
  - `paymentModes` - array of supported payment modes

### ✅ API Routes

- **POST `/api/stripe/create-checkout-session`** - Creates Stripe checkout sessions
- **POST `/api/stripe/webhook`** - Handles Stripe webhooks
- **GET/PATCH/DELETE `/api/bookings/[id]`** - Booking management with version control

### ✅ Payment Flow

1. **Booking Creation** - Bookings start in `draft` status
2. **Payment Initiation** - Booking locked with `pending_payment` status
3. **Stripe Checkout** - User redirected to Stripe with booking metadata
4. **Webhook Processing** - Payment confirmation updates booking to `confirmed`
5. **Version Management** - Price changes during checkout create new versions

### ✅ UI Components

- **BookingPaymentClient** - Payment selection and Stripe integration
- **Payment pages** - Success, cancel, and payment selection pages
- **Pricing breakdown** - Shows base amount, fees, and total

### ✅ Transaction Fee Calculation

```typescript
const feeCents = Math.round(baseCents * (feeRateBps / 10000));
const totalCents = baseCents + feeCents;
```

### ✅ Payment Modes

- **Booking Fee Only** - 15% of total as booking fee
- **Full Sum** - Complete rental cost

### ✅ Payment Types

- **One-time Payment** - Single payment for booking
- **Subscription** - Recurring monthly payments (if company supports)

### ✅ Data Integrity

- **Checksum validation** - Ensures booking wasn't modified during payment
- **Version control** - Tracks booking changes and handles conflicts
- **Booking locks** - Prevents modifications during payment (30-minute timeout)

### ✅ Edit Handling

- **Non-price edits** (notes, etc.) - Allowed during checkout, updates checksum
- **Price edits** (dates, address) - Cancels current session, creates new version
- **Automatic cleanup** - Expired sessions revert bookings to draft status

### ✅ Slack Notifications

- Sends notifications to `#stripe-transactions` channel
- Includes booking details, amounts, customer info, and status
- Triggered on payment success, failure, and status changes

## Environment Variables Required

Add these to your `.env.local` file:

```bash
# Stripe Configuration
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Slack Notifications
SLACK_STRIPE_TRANSACTIONS_WEBHOOK_URL=https://hooks.slack.com/services/...

# Application
NEXT_PUBLIC_HOST=http://localhost:3002
```

## File Structure

```
src/
├── lib/
│   ├── stripe.ts                 # Stripe configuration and utilities
│   ├── bookingService.ts         # Booking business logic
│   └── bookingDraft.ts          # Draft persistence (existing)
├── app/
│   ├── api/
│   │   ├── stripe/
│   │   │   ├── create-checkout-session/route.ts
│   │   │   └── webhook/route.ts
│   │   └── bookings/[id]/route.ts
│   ├── components/
│   │   └── BookingPaymentClient.tsx
│   └── [country]/[city]/[category]/[slug]/book/
│       ├── payment/page.tsx
│       ├── success/page.tsx
│       └── cancel/page.tsx
└── prisma/
    └── schema.prisma            # Updated with Stripe fields
```

## Usage

### 1. Company Setup

Companies need to be configured to accept Stripe payments:

```typescript
await prisma.companies.update({
  where: { id: companyId },
  data: {
    acceptsStripe: true,
    acceptsSubscription: true, // optional
    paymentModes: ["full_sum", "booking_fee_only"],
  },
});
```

Or via CLI (recommended for quick testing):

```bash
# Enable both modes for a company
pnpm stripe:enable-company --id <companyId>

# Enable only full_sum
pnpm stripe:enable-company --id <companyId> --modes full_sum

# Enable booking_fee_only and subscriptions
pnpm stripe:enable-company --id <companyId> --modes booking_fee_only --subscription
```

### 2. Creating a Booking

```typescript
import BookingService from "@/lib/bookingService";

const booking = await BookingService.createBooking({
  productId: "bike-id",
  userId: "user-id",
  startDate: new Date("2025-01-20"),
  endDate: new Date("2025-01-25"),
  currency: "IDR",
  // ... other fields
});
```

### 3. Processing Payment

```typescript
const session = await BookingService.createStripeCheckoutSession(
  booking.id,
  "full_sum", // or 'booking_fee_only'
  "one_time", // or 'subscription'
  successUrl,
  cancelUrl
);

// Redirect user to session.url
```

### 3.1 Proceed-to-Payment Flow (UI-driven)

In the app, once the user completes all steps on the booking overview, clicking "Proceed to Payment" will:

1. Create a draft booking via `POST /api/bookings`
2. Redirect to `/[country]/[city]/[category]/[slug]/book/payment?bookingId=...`
3. The payment page shows available modes and types and, on confirmation, creates a Stripe Checkout session and redirects to Stripe.

### 4. Webhook Handling

The webhook automatically:

- Validates booking version and checksum
- Updates booking status to `confirmed`
- Sends Slack notifications
- Handles payment failures and cancellations

## Security Features

- **Idempotency** - Stripe operations use idempotency keys
- **Checksum validation** - Prevents payment for modified bookings
- **Version control** - Handles concurrent modifications
- **Webhook signature verification** - Validates Stripe webhooks
- **Booking locks** - Prevents race conditions during payment

## Error Handling

- **Version mismatch** - Booking marked for review
- **Checksum failure** - Booking marked for review
- **Payment failure** - Booking reverted to draft
- **Session expiry** - Automatic cleanup and unlock

## Monitoring

The system provides observability through:

- Slack notifications for all transactions
- Console logging for debugging
- Booking status tracking
- Version history

## Stripe Webhook Configuration

When setting up your webhook endpoint in the Stripe Dashboard, you need to enable the following events:

### Required Webhook Events

Enable these events in your Stripe Dashboard (`Developers > Webhooks > Add endpoint`):

#### ✅ **Core Payment Events**

- `checkout.session.completed` - When payment is successful
- `checkout.session.expired` - When checkout session expires (30 min timeout)

#### ✅ **Payment Intent Events**

- `payment_intent.succeeded` - When one-time payment succeeds
- `payment_intent.payment_failed` - When payment fails

#### ✅ **Subscription Events** (if using subscriptions)

- `invoice.payment_succeeded` - When subscription payment succeeds
- `customer.subscription.deleted` - When subscription is cancelled

### Webhook Endpoint Setup

1. **Go to Stripe Dashboard** → Developers → Webhooks
2. **Click "Add endpoint"**
3. **Enter your endpoint URL**: `https://yourdomain.com/api/stripe/webhook`
4. **Select events** from the list above
5. **Copy the webhook signing secret** and add to your environment variables

### Webhook URL Format

```
https://yourdomain.com/api/stripe/webhook
```

For local development:

```
https://your-ngrok-url.ngrok.io/api/stripe/webhook
```

### Testing Webhooks Locally

For local development, use the Stripe CLI to forward webhooks:

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Login to your Stripe account
stripe login

# Forward webhooks to your local server
stripe listen --forward-to localhost:3002/api/stripe/webhook
```

The Stripe CLI will provide a webhook signing secret that you can use for local testing.

## Next Steps

To complete the integration:

1. **Set up Stripe webhook endpoint** in Stripe Dashboard (see above)
2. **Configure environment variables**
3. **Test payment flows** in Stripe test mode
4. **Set up Slack webhook** for notifications
5. **Add user authentication** for booking creation
6. **Implement email notifications**
7. **Add booking management UI** for admins

## Testing

Use Stripe test cards for testing:

- **4242 4242 4242 4242** - Successful payment
- **4000 0000 0000 0002** - Card declined
- **4000 0000 0000 9995** - Insufficient funds

## Production Checklist

- [ ] Switch to Stripe live keys
- [ ] Configure production webhook endpoint
- [ ] Set up monitoring and alerting
- [ ] Test all payment scenarios
- [ ] Verify Slack notifications
- [ ] Review security settings
- [ ] Set up backup and recovery procedures
