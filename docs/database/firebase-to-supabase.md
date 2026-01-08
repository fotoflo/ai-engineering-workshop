# Firebase to Supabase Synchronization Setup

This document outlines the new Firebase to Supabase synchronization system that uses APIs for importing entities and supports both bulk imports and real-time trigger-based updates.

## Overview

The synchronization system consists of:

1. **Import APIs** - REST endpoints for importing individual entities
2. **Bulk Import Script** - Script for initial data migration using verification entities
3. **Firebase Triggers** - Cloud Functions for real-time synchronization
4. **Integration Tests** - Tests that verify imports work correctly

## Architecture

```
Firebase Firestore → Firebase Cloud Functions → Supabase APIs → PostgreSQL
     ↓                        ↓                        ↓
  Raw Data              Transform & Sync          Structured Data
```

## Import APIs

### Users Import API

- **Endpoint**: `POST /api/admin/users/import`
- **Purpose**: Import user data from Firebase
- **Required Fields**: `id`
- **Supported Fields**: All user schema fields

### Companies Import API

- **Endpoint**: `POST /api/admin/companies/import`
- **Purpose**: Import company data from Firebase
- **Required Fields**: `id`
- **Supported Fields**: All company schema fields

### Products Import API

- **Endpoint**: `POST /api/admin/products/import`
- **Purpose**: Import product data from Firebase
- **Required Fields**: `id`, `companyId`
- **Supported Fields**: All product and vehicle schema fields

### Bookings Import API (Enhanced)

- **Endpoint**: `POST /api/admin/bookings/import`
- **Purpose**: Import booking data from Firebase
- **Required Fields**: `id`, `productId`, `userId`, `startDate`, `endDate`
- **New Fields Added**:
  - `cancelReason` (String)
  - `handOverConfirmationCode` (String)
  - `handOverConfirmed` (Boolean)
  - `paidOut` (Boolean)
  - `reviewed` (Boolean)

## Bulk Import

### Running the Bulk Import

```bash
# Set the API base URL (defaults to localhost:3000)
export API_BASE=http://localhost:3000

# Run the bulk import
node scripts/firebase-to-supabase/bulk-verification-import.js
```

The script will:

1. Load all verification JSON files
2. Import users first
3. Import companies
4. Import products
5. Import bookings

### Verification Files

The following verification files are used for testing and bulk imports:

- `scripts/firebase-to-supabase/verification-users.json`
- `scripts/firebase-to-supabase/verification-companies.json`
- `scripts/firebase-to-supabase/verification-products.json`
- `scripts/firebase-to-supabase/verification-bookings.json`

## Real-time Synchronization

### Firebase Cloud Functions Setup

1. **Install Dependencies**:

```bash
cd flexbike-functions
npm install
```

2. **Deploy Functions**:

```bash
firebase deploy --only functions
```

3. **Environment Variables**:
   Set the following environment variables in Firebase Functions:

```
SUPABASE_API_BASE=https://your-app.vercel.app
SUPABASE_API_KEY=your-api-key
```

### Available Triggers

The following Firebase triggers are automatically deployed:

#### User Triggers

- `onUserCreated` - Syncs user creation
- `onUserUpdated` - Syncs user updates

#### Company Triggers

- `onCompanyCreated` - Syncs company creation
- `onCompanyUpdated` - Syncs company updates

#### Product Triggers

- `onProductCreated` - Syncs product creation
- `onProductUpdated` - Syncs product updates

#### Booking Triggers

- `onBookingCreated` - Syncs booking creation
- `onBookingUpdated` - Syncs booking updates

#### Utility Functions

- `manualSync` - Manually sync a specific document
- `healthCheck` - Health check endpoint

### Manual Sync

You can manually sync a document using the `manualSync` function:

```javascript
// Call the cloud function
const result = await firebase.functions().httpsCallable("manualSync")({
  collection: "users",
  documentId: "user123",
});
```

## Integration Tests

### Running Tests

```bash
# Run all import integration tests
npm test -- src/__tests__/integration/*-import.integration.test.ts

# Run specific test
npm test -- src/__tests__/integration/users-import.integration.test.ts
```

### Test Coverage

The integration tests cover:

- **Users Import**: Tests user creation and duplicate handling
- **Companies Import**: Tests company creation with expected verification data
- **Products Import**: Tests product creation with vehicle details
- **Bookings Import**: Tests booking creation with all required relationships

## Error Handling

### API Error Responses

All import APIs return standardized error responses:

```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional error details"
}
```

### Common Error Codes

- `P2002`: Unique constraint violation (duplicate entity)
- `P2003`: Foreign key constraint violation
- `P2000/P2001`: Validation errors

### Firebase Function Error Handling

Firebase functions include:

- Automatic retries for transient failures
- Structured logging for debugging
- Dead letter queue consideration for persistent failures

## Migration Strategy

### Phase 1: Bulk Import

1. Run bulk import script with verification data
2. Verify data integrity in Supabase
3. Run integration tests

### Phase 2: Real-time Sync

1. Deploy Firebase Cloud Functions
2. Enable triggers for new data
3. Monitor sync performance

### Phase 3: Cutover

1. Disable old Firebase reads
2. Enable Supabase as primary data source
3. Monitor for any missed data

## Monitoring

### Health Checks

Use the `healthCheck` function to verify system status:

```javascript
const result = await firebase.functions().httpsCallable("healthCheck")();
console.log(result.data); // { status: "healthy", timestamp: "...", version: "..." }
```

### Logging

All operations are logged with structured data:

- Import successes and failures
- Trigger execution times
- Error details and stack traces

## Security Considerations

1. **API Authentication**: Import APIs should be protected with admin authentication
2. **Rate Limiting**: Implement rate limiting to prevent abuse
3. **Data Validation**: All inputs are validated using Prisma schemas
4. **Audit Trail**: All imports are logged for audit purposes

## Performance Optimization

1. **Batch Operations**: Use batch operations for bulk imports
2. **Connection Pooling**: Database connections are pooled automatically
3. **Indexing**: Ensure proper database indexes for foreign keys
4. **Caching**: Consider caching for frequently accessed reference data

## Troubleshooting

### Common Issues

1. **Migration Blocks**: Run database migrations before importing
2. **Foreign Key Errors**: Import entities in dependency order
3. **Type Errors**: Regenerate Prisma types after schema changes
4. **Timeout Errors**: Increase function timeout for large imports

### Debug Commands

```bash
# Check database status
npx prisma migrate status

# Regenerate types
npx prisma generate

# Run specific tests
npm test -- --testNamePattern="should import user successfully"
```

## Future Enhancements

1. **Queue System**: Implement queue for failed operations
2. **Data Validation**: Add more sophisticated validation rules
3. **Monitoring Dashboard**: Create dashboard for sync monitoring
4. **Rollback Support**: Add rollback capabilities for failed imports
