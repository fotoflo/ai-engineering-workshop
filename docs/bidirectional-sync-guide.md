# 🔄 Bidirectional Firebase ↔ PostgreSQL Sync System

## Overview

The bidirectional sync system enables seamless data synchronization between Firebase Firestore and PostgreSQL, ensuring data consistency across both platforms while preventing data corruption and infinite sync loops. The system is designed for serverless environments and handles schema differences between Firebase's document model and PostgreSQL's relational model.

## 🎯 Key Features

- **Schema-Aware Transformations**: Handles field name differences (e.g., `bikeID` ↔ `productId`)
- **Comprehensive Validation**: Zod-based validation at every transformation step
- **Conflict Resolution**: Timestamp-based conflict resolution prevents data loss
- **Loop Prevention**: Sync metadata prevents infinite loops between systems
- **Error Recovery**: Detailed error reporting and graceful failure handling
- **Production Safety**: No direct production access during development/testing

## 🏗️ Architecture

### Core Components

#### 1. Data Transformers (`src/lib/dataTransformers.ts`)

**Purpose**: Converts data between Firebase and Prisma schemas with validation

**Key Functions:**

- `transformFirebaseToPrisma()` - Firebase → PostgreSQL conversion
- `transformPrismaToFirebase()` - PostgreSQL → Firebase conversion
- `validateFirebaseBooking()` / `validatePrismaBooking()` - Schema validation
- `reportTransformationError()` - Error logging

#### 2. Sync Services (`src/server/services/sync/firebaseSync.ts`)

**Purpose**: Handles Firebase Admin SDK operations with transformation integration

**Key Functions:**

- `upsertBooking()` - Sync booking to Firebase with validation
- `upsertUser()` - Sync user to Firebase with validation
- `checkFirebaseHealth()` - Connection health monitoring

#### 3. API Endpoints

**Firebase → PostgreSQL:**

- `/api/admin/bookings/upsert` - Receives Firebase booking data
- `/api/admin/users/upsert` - Receives Firebase user data

**PostgreSQL → Firebase:**

- Direct calls to `firebaseSync.ts` from API routes (bookings, users)

#### 4. Firebase Functions (`flexbike-functions/src/trigger-sync.ts`)

**Purpose**: Listens for Firestore changes and triggers PostgreSQL sync

**Triggers:**

- `onBookingCreated/Updated` - Sync booking changes
- `onUserCreated/Updated` - Sync user changes
- `onCompanyCreated/Updated` - Sync company changes
- `onProductCreated/Updated` - Sync product changes

**Note**: Deletes are not handled as the system doesn't perform deletions across platforms.

## 🔄 Sync Flow Diagrams

### Firebase → PostgreSQL Flow

```
Firebase Document Change
        ↓
Firebase Cloud Function Trigger
        ↓
makePostgreSQLRequest() → /api/admin/*/upsert
        ↓
transformFirebaseToPrisma() + Validation
        ↓
Prisma.upsert() with sync metadata
        ↓
Success/Error Response
```

### PostgreSQL → Firebase Flow

```
API Route Mutation (bookings/users)
        ↓
Prisma.create/update
        ↓
firebaseSync.upsert*() + Validation
        ↓
transformPrismaToFirebase()
        ↓
Firebase Admin SDK .set() with sync metadata
```

## 📊 Schema Mappings

### Bookings Collection

| Firebase Field       | PostgreSQL Field     | Type             | Notes                                |
| -------------------- | -------------------- | ---------------- | ------------------------------------ |
| `bikeID`             | `productId`          | String           | Required                             |
| `companyID`          | `companyId`          | String           | Optional                             |
| `riderID`            | `userId`             | String           | Required (or "vendor-booking-guest") |
| `riderInfo`          | -                    | Object           | Extracted to user fields             |
| `start`              | `startDate`          | Date             | ISO string conversion                |
| `end`                | `endDate`            | Date             | ISO string conversion                |
| `totalPrice`         | `totalPrice`         | Number → Decimal | Safe conversion                      |
| `price`              | `price`              | Number → Decimal | Safe conversion                      |
| `deposit`            | `deposit`            | Number → Decimal | Safe conversion                      |
| `whatsappNumber`     | `whatsappNumber`     | String           | On booking                           |
| `marketplaceBooking` | `marketplaceBooking` | Boolean          | Booking type                         |
| `status`             | `status`             | String           | Status normalization                 |
| `currency`           | `currency`           | String           | Default: "IDR"                       |

### Decimal Conversion Rationale

PostgreSQL uses the `Decimal` type for monetary values to maintain precision and support large numbers (beyond JavaScript's safe integer range of ±9,007,199,254,740,991). While JavaScript uses `number` (float64), PostgreSQL's Decimal provides exact decimal arithmetic and prevents rounding errors in financial calculations. The conversion ensures data integrity for pricing, deposits, and totals.

### Users Collection

| Firebase Field   | PostgreSQL Field | Type           | Notes                                 |
| ---------------- | ---------------- | -------------- | ------------------------------------- |
| `companyIDs`     | `companyId`      | Array → String | ⚠️ **RISK**: Takes first company only |
| `whatsappNumber` | `whatsappNumber` | String         | Direct mapping                        |
| `email`          | `email`          | String         | Direct mapping                        |
| `name`           | `name`           | String         | Direct mapping                        |

### CompanyIDs Multiple Assignment Risk

**⚠️ Critical Issue**: The current transformation takes only the first company from `companyIDs` array, but users may belong to multiple companies. This needs investigation:

```sql
-- Check for users with multiple companies
SELECT id, "companyId" FROM users WHERE "companyId" LIKE '%,%' LIMIT 10;
```

**Potential Solutions:**

1. **Multi-company support**: Change `companyId` to array in PostgreSQL schema
2. **Primary company**: Add `primaryCompanyId` field and logic to determine primary
3. **Company relationship table**: Create separate `user_companies` table for many-to-many relationship

**Recommendation**: Run the query above to assess the impact before implementing sync.

## 🛡️ Safety & Validation

### Validation Layers

1. **Input Validation**: Zod schemas validate incoming data structure
2. **Transformation Validation**: Pre/post validation of transformations
3. **Output Validation**: Final schema validation before database writes
4. **Error Reporting**: Detailed logging with context for debugging

### Post-Transform Validation Testing

**Unit Test Requirement**: Add comprehensive tests that verify entities match exactly after transformation and sync field removal:

```typescript
describe("Post-Transform Entity Matching", () => {
  test("Firebase → PostgreSQL roundtrip preserves data integrity", async () => {
    // 1. Create test Firebase booking data
    const firebaseBooking = createTestFirebaseBooking();

    // 2. Transform to PostgreSQL format
    const postgresBooking = transformFirebaseToPrisma(firebaseBooking);

    // 3. Transform back to Firebase format
    const firebaseBookingRoundtrip = transformPrismaToFirebase(postgresBooking);

    // 4. Remove sync metadata fields from Firebase
    const { _sync, ...firebaseWithoutSync } = firebaseBookingRoundtrip;

    // 5. Verify exact match (excluding sync fields)
    expect(firebaseWithoutSync).toEqual(firebaseBooking);
  });

  test("PostgreSQL → Firebase roundtrip preserves data integrity", async () => {
    // Similar test in reverse direction
    const postgresBooking = createTestPostgresBooking();
    const firebaseBooking = transformPrismaToFirebase(postgresBooking);
    const postgresBookingRoundtrip = transformFirebaseToPrisma(firebaseBooking);
    // Verify exact match
  });
});
```

This ensures no data corruption occurs during bidirectional transformations.

### Conflict Resolution

- **Timestamp-based**: Last-write-wins using `updatedAt` comparison
- **Sync Metadata**: `_sync.origin` and `updatedBySource` prevent loops
- **Version Tracking**: `syncVersion` for debugging conflicts

### Error Handling

```typescript
// Example error reporting
reportTransformationError(
  "prisma-to-firebase",
  "booking",
  bookingId,
  error,
  originalData
);
```

## 🧪 Testing & Development

### Development Setup

1. **Start Firebase Emulator:**

   ```bash
   cd flexbike-functions
   firebase emulators:start --only firestore --project project-s-v2
   ```

2. **Create Test Data:**

   ```bash
   cd flexbike-next
   node scripts/create-emulator-test-data.js
   ```

3. **Start Next.js with Emulator:**
   ```bash
   FIRESTORE_EMULATOR_HOST=localhost:8080 npm run dev
   ```

### Running Tests

```bash
# Run transformation tests
npm test -- --testPathPattern=dataTransformers.test.ts

# Run bidirectional sync integration test
node scripts/test-bidirectional-sync.js
```

### Test Data Structure

The emulator is populated with:

- **2 Users**: Test customers with company affiliations
- **2 Companies**: One with instant booking, one without
- **2 Products**: Bikes linked to companies
- **3 Bookings**: Mix of marketplace and vendor bookings

## 🚀 Production Deployment

### Environment Overview

- **Preview**: `preview.flexbike.app` (Vercel preview deployments)
  - Uses preview Supabase database
  - **No preview Firebase instance** - shares production Firebase
  - Deploy to preview first for testing
- **Production**: `flexbike.app` (Vercel production)
  - Uses production Supabase database
  - Uses production Firebase instance

### Prerequisites

1. **Firebase Admin Credentials**: Set environment variables:

   ```
   FIREBASE_PROJECT_ID=project-s-v2
   FIREBASE_CLIENT_EMAIL=...
   FIREBASE_PRIVATE_KEY=...
   ```

2. **Sync API Key**: Generate secure key for API authentication:

   ```
   SYNC_API_KEY=production-key-here
   ```

3. **PostgreSQL Webhooks** (Serverless-compatible):
   - Use Supabase CLI for webhook testing during development
   - Configure Supabase webhook for `bookings` and `users` tables in production
   - Point to `/api/admin/sync/webhook`

### Supabase CLI Webhook Testing

For development and testing, use Supabase CLI instead of slow deployment cycles:

```bash
# Start local Supabase
supabase start

# Forward webhook to local Next.js
supabase functions serve webhook-handler --no-verify-jwt

# Test webhook with sample data
curl -X POST http://localhost:54321/functions/v1/webhook-handler \
  -H "Content-Type: application/json" \
  -d '{"table":"bookings","eventType":"INSERT","new":{"id":"test"}}'
```

### Deployment Steps

1. **Deploy Firebase Functions:**

   ```bash
   cd flexbike-functions
   firebase deploy --only functions
   ```

2. **Enable Sync Flags:**

   ```bash
   # In production environment
   ENABLE_BIDIRECTIONAL_SYNC=true
   FIRESTORE_EMULATOR_HOST= # Leave empty for production
   ```

3. **Monitor Sync Health:**
   ```bash
   curl https://your-app.vercel.app/api/admin/sync/health
   ```

### Rollback Plan

If sync issues occur:

1. **Disable Firebase Functions** temporarily
2. **Check sync logs** for error patterns
3. **Validate data integrity** between systems
4. **Gradual re-enable** with monitoring

## 🔍 Monitoring & Debugging

### Health Check Endpoint

```bash
curl https://your-app.vercel.app/api/admin/sync/health
```

Returns:

```json
{
  "firebase": { "status": "healthy", "emulator": false },
  "realtime": { "status": "configured" },
  "sync_stats": {
    "bookings_from_firebase": 150,
    "users_from_firebase": 45,
    "bookings_to_firebase": 148,
    "users_to_firebase": 44
  }
}
```

### Common Issues & Solutions

#### Data Inconsistencies

- **Symptom**: Same record has different data in Firebase vs PostgreSQL
- **Check**: Run data integrity validation in test script
- **Fix**: Identify transformation bugs, redeploy with fixes

#### Infinite Loops

- **Symptom**: High sync volume, performance issues
- **Check**: Verify `_sync.origin` metadata is present
- **Fix**: Ensure sync metadata is added to all operations

#### Transformation Errors

- **Symptom**: Sync failures with transformation errors
- **Check**: Review application logs for detailed error messages
- **Fix**: Update transformation logic, redeploy

## 📈 Performance Considerations

### Batch Processing

- Firebase Functions can batch multiple document changes
- API endpoints support single-record sync for real-time updates

### Rate Limiting

- Firebase has quotas for reads/writes per minute
- Implement exponential backoff for failed sync attempts

### Monitoring Metrics

- Sync success/failure rates
- Average transformation time
- Data consistency checks
- Conflict resolution frequency

## 🔐 Security

### Authentication

- **Bearer Token**: API endpoints require `SYNC_API_KEY`
- **Origin Filtering**: `_sync.origin` prevents unauthorized syncs
- **Timestamp Validation**: Prevents replay attacks

### Data Privacy

- **Minimal Data**: Only sync necessary fields
- **PII Handling**: Ensure sensitive data is properly mapped
- **Audit Trail**: All sync operations are logged

## 🛠️ Development Workflow

### Adding New Fields

1. **Update Zod Schemas** in `dataTransformers.ts`
2. **Add Transformation Logic** for both directions
3. **Update Tests** to cover new fields
4. **Test End-to-End** with emulator
5. **Deploy Gradually** with feature flags

### Modifying Existing Mappings

1. **Identify Impact** on existing data
2. **Update Transformation Logic**
3. **Run Migration** for existing records if needed
4. **Test Thoroughly** with production-like data
5. **Monitor After Deployment**

## 📚 API Reference

### Sync Health Check

```typescript
GET / api / admin / sync / health;
// Returns sync system health status
```

### Manual Sync Trigger

```typescript
POST / api / admin / sync / test;
// Test sync functionality with sample data
```

### Webhook Receiver

```typescript
POST / api / admin / sync / webhook;
// Receives PostgreSQL change events via Supabase webhooks
```

## 🎯 Success Metrics

- **Data Consistency**: <1% data mismatches between systems
- **Sync Latency**: <5 seconds for real-time updates
- **Error Rate**: <0.1% sync operation failures
- **Uptime**: 99.9% sync system availability

## 📞 Support & Troubleshooting

### Common Issues

1. **"Transformation failed" errors**: Check Zod schema validation
2. **Missing sync metadata**: Ensure all sync operations add metadata
3. **Firebase permission errors**: Verify service account credentials
4. **PostgreSQL connection issues**: Check database connectivity

### Debug Commands

```bash
# Check Firebase emulator data
firebase emulators:export export-data --project project-s-v2

# Query PostgreSQL sync metadata
SELECT id, "updatedBySource", "syncVersion" FROM bookings LIMIT 10;

# Check Firebase sync metadata
db.collection('bookings').where('_sync.origin', '==', 'postgres').limit(10);
```

This documentation provides a comprehensive guide for understanding, maintaining, and extending the bidirectional sync system. The system is designed to be robust, testable, and safe for production use.
