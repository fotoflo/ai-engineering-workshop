# Offline Booking Test Script

This script tests the end-to-end creation of offline bookings that get synced from Firestore to PostgreSQL.

## What it does

1. **Creates bookings directly in Firestore** - Bypassing the regular Next.js booking API to simulate offline bookings
2. **Triggers Firebase functions** - The `syncBookingToPostgres` function automatically runs when bookings are created in Firestore
3. **Syncs to PostgreSQL** - The Firebase function sends data to the Next.js webhook which transforms and stores it in PostgreSQL

## Usage

### Prerequisites

1. **Firebase Emulator running**: Start with `pnpm emu`
2. **Next.js server running**: Start with `npm run dev`
3. **Database configured**: Ensure `DATABASE_URL` is set in your `.env.local` file

### Running the test

```bash
# Create a single test booking
node create-test-offline-booking.js

# Create multiple test bookings (e.g., 5)
node create-test-offline-booking.js 5
```

## Test Data Structure

The script creates bookings with the exact same fields as your real test data:

```javascript
{
  bikeID: "fPLjsLsfqpCe476PXUXG",
  companyID: "Uq7KTn71fpVBjfdhIXs4",
  riderName: "Test 332",
  whatsappNumber: null,
  start: Timestamp (Dec 8, 2025 07:00 UTC+7),
  end: Timestamp (Dec 11, 2025 07:00 UTC+7),
  deliveryAddress: null,
  deliveryTime: null,
  requestedDelivery: false,
  marketplaceBooking: false,
  deposit: 0,
  images: [],
  notes: null,
  price: null
}
```

## Verification Steps

### 1. Check Firestore (Firebase Emulator)
Visit: http://127.0.0.1:4000/firestore

Look for documents in the `bookings` collection with the generated UUIDs.

### 2. Check Firebase Functions Logs
The script will show function execution in the terminal output. You can also check `firebase-debug.log`:

```bash
grep "syncBookingToPostgres" firebase-debug.log
```

### 3. Check PostgreSQL Database
Use Prisma Studio or query directly:

```bash
npx prisma studio
```

Or check programmatically:

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

async function check() {
  const bookings = await prisma.bookings.findMany({
    where: { status: 'requested' },
    orderBy: { createdAt: 'desc' },
    take: 5
  });
  console.log('Recent bookings:', bookings);
}
```

## Architecture Flow

```
1. Script → Firestore (bookings collection)
2. Firestore Trigger → syncBookingToPostgres function
3. Function → HTTP POST to /api/sync/firebase-webhook
4. Webhook → Transform data + upsert to PostgreSQL
5. Success → Booking available in both systems
```

## Troubleshooting

### Firebase Functions not running
- Ensure `pnpm emu` is running
- Check that `FIREBASE_SYNC_ENV=EMULATOR` in your environment

### Database connection failed
- Ensure Next.js server is running (`npm run dev`)
- Check `DATABASE_URL` in `.env.local`
- Verify Supabase credentials if using remote database

### Webhook not accessible
- Ensure Next.js is running on port 3000
- Check `/api/health` endpoint returns healthy status

### Sync metadata prevents loops
The system uses `_sync` metadata to prevent infinite sync loops between Firebase and PostgreSQL.

## Files

- `create-test-offline-booking.js` - Main test script
- `flexbike-functions/functions/syncToPostgres.js` - Firebase sync function
- `src/app/api/sync/firebase-webhook/route.ts` - Next.js webhook endpoint
- `src/lib/transforms/bookings.ts` - Data transformation logic
