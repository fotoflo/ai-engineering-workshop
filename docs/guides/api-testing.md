# API Testing Scripts

This directory contains scripts to test various API endpoints in the Flexbike application.

## Scripts Overview

### `test-api-endpoints.sh`

Comprehensive test suite for multiple API endpoints.

**Usage:**

```bash
./scripts/test-api-endpoints.sh
```

**Tests:**

- ✅ Users/ensure (user creation/update)
- ✅ Users/update-verification (OTP verification)
- ✅ Geo (location services)
- ✅ Health (server status)
- ✅ WATI/send-otp (WhatsApp integration)
- ✅ Search/view (bike search)
- ✅ Companies (company listings)
- ✅ Countries (country data)

### `test-users-ensure-curl.sh`

Focused testing for the `/api/users/ensure` endpoint with detailed scenarios.

**Usage:**

```bash
./scripts/test-users-ensure-curl.sh
```

**Test Scenarios:**

- ✅ Valid Indonesian phone numbers
- ✅ Valid Thai/Singapore phone numbers
- ✅ Invalid phone numbers (too short, missing country code)
- ✅ Missing/empty phone numbers
- ✅ Phone numbers with booking IDs
- ✅ Create vs Update operations
- ✅ Slack notification triggers

## Prerequisites

1. **Next.js server running:**

   ```bash
   npm run dev
   ```

2. **Database connection:** Ensure Prisma database is accessible

3. **Environment variables:** Required for external services (WATI, Slack, etc.)

## Test Results

### Expected Outputs

**Successful requests:**

```
Status: 200
Response: {"ok":true,"userId":"wa:6281717770552"}
✅ PASS
```

**Error cases:**

```
Status: 400
Response: {"ok":false,"error":"Valid phoneNumber is required (must be at least 8 digits)"}
✅ PASS
```

### Server Logs

Check your Next.js server console for detailed logging:

```
🔍 /api/users/ensure called with: { phoneNumber: '+6281717770552', ... }
🔄 Finding or creating user: { userId: 'wa:6281717770552', ... }
🆕 Creating new user
📢 Sending Slack notification for new user
✅ User upsert successful: { userId: 'wa:6281717770552' }
```

## Troubleshooting

### Common Issues

1. **Server not running:**

   ```bash
   npm run dev
   ```

2. **Database connection failed:**

   - Check database credentials
   - Run `npx prisma db push`

3. **Environment variables missing:**

   - Copy `.env.example` to `.env.local`
   - Fill in required API keys

4. **Port conflicts:**
   - Default port is 3000
   - Change with `PORT=3001 npm run dev`

### Debug Mode

For more detailed error information, the API includes development-mode error details:

```json
{
  "ok": false,
  "error": "Database connection failed",
  "details": {
    "code": "P1001",
    "meta": { "database": "postgresql" }
  }
}
```

## Manual Testing

You can also test endpoints manually with curl:

```bash
# Test users/ensure
curl -X POST "http://localhost:3000/api/users/ensure" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber":"+6281717770552","firstName":"Test","lastName":"User"}'

# Test with booking ID
curl -X POST "http://localhost:3000/api/users/ensure" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber":"+6281717770553","bookingId":"booking-123"}'
```

## Test Coverage

These scripts test the critical user journey:

1. **Phone verification** → `users/ensure`
2. **OTP verification** → `users/update-verification`
3. **Booking association** → Booking linking
4. **External integrations** → Slack notifications

Run these tests after any changes to user authentication or booking flows!
