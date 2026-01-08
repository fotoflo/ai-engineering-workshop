# Import and Backfill Strategy - New API-Based Approach

This document outlines the new import and backfill strategy that replaces the old script-based approach with API-based operations that are simple to operate, run, and highly testable.

## Overview

The new strategy consists of:

1. **API-Based Bulk Imports** - Use existing `/api/admin/*/bulk-import` endpoints
2. **Unified Post-Import Processing** - Single API call to handle all post-processing
3. **Enhanced Location Backfilling** - Prioritizes `companies.area` field for location resolution
4. **Comprehensive Cron Jobs** - Daily maintenance using the new APIs
5. **Simple CLI Interface** - Easy-to-use command line tool
6. **Extensive Test Suite** - Highly testable with integration tests

## Key Improvements Over Old System

### ✅ **API-Based Architecture**

- **Before**: Standalone scripts that directly accessed database
- **After**: HTTP API endpoints that can be called programmatically
- **Benefits**: Testable, monitorable, reusable, works with API-based imports

### ✅ **Unified Post-Import Processing**

- **Before**: Multiple separate scripts run manually in sequence
- **After**: Single `/api/admin/post-import/process` endpoint
- **Benefits**: Atomic operations, easier error handling, better monitoring

### ✅ **Enhanced Location Backfilling**

- **Before**: Complex script with LLM fallback, didn't use `area` field effectively
- **After**: Direct mapping from `companies.area` field with smart fallbacks
- **Benefits**: Faster, more accurate, prioritizes explicit area data

### ✅ **Comprehensive Testing**

- **Before**: Scripts were hard to test reliably
- **After**: Full integration test suite for all operations
- **Benefits**: Confidence in changes, regression prevention

## Architecture

```
┌─────────────────┐    ┌──────────────────────┐    ┌─────────────────┐
│   CLI Tool      │    │   API Endpoints      │    │   Cron Jobs     │
│                 │    │                      │    │                 │
│ • run-full-     │───▶│ • /admin/*/bulk-     │◀───│ • Daily full    │
│   import-       │    │   import             │    │   processing    │
│   pipeline.ts   │    │ • /admin/post-import/│    │ • Rating updates│
│                 │    │   process            │    │ • Count updates │
│                 │    │ • /admin/locations/  │    │                 │
│                 │    │   backfill           │    │                 │
└─────────────────┘    └──────────────────────┘    └─────────────────┘
                              ▲
                              │
                              ▼
                    ┌──────────────────────┐
                    │   Database           │
                    │                      │
                    │ • companies          │
                    │ • products           │
                    │ • cities/regions/    │
                    │   countries          │
                    │ • company_locations  │
                    │                      │
                    └──────────────────────┘
```

## API Endpoints

### `/api/admin/post-import/process` - Unified Post-Import Processing

Runs all post-import steps in the correct order.

**Request:**

```json
{
  "steps": [
    "normalize-slugs",
    "backfill-locations",
    "update-ratings",
    "update-counts"
  ],
  "options": {
    "locationLimit": 5000,
    "dryRun": false
  }
}
```

**Response:**

```json
{
  "success": true,
  "totalDuration": 45230,
  "steps": {
    "normalize-slugs": {
      "companiesProcessed": 1250,
      "slugsFixed": 45,
      "chiangMaiFix": 12,
      "duration": 1250
    },
    "backfill-locations": {
      "candidates": 380,
      "updated": 245,
      "skipped": 135,
      "duration": 8900
    },
    "update-ratings": {
      "companiesProcessed": 1250,
      "ratingsUpdated": 892,
      "duration": 3400
    },
    "update-counts": {
      "countriesUpdated": 8,
      "regionsUpdated": 24,
      "citiesUpdated": 156,
      "duration": 2100
    }
  }
}
```

### `/api/admin/locations/backfill` - Location Backfilling

Backfills missing company location data using the `companies.area` field as primary source.

**Request:**

```json
{
  "limit": 5000,
  "dryRun": false
}
```

**Key Features:**

- **Prioritizes `companies.area` field** with direct mappings for known areas
- **Smart fallbacks** to address parsing and coordinate-based resolution
- **Updates both companies and company_locations tables**

### Existing Bulk Import Endpoints

- `/api/admin/companies/bulk-import`
- `/api/admin/products/bulk-import`
- `/api/admin/users/bulk-import`
- `/api/admin/bookings/bulk-import`

## Cron Jobs

### `/api/cron/full-post-import-process` - Daily Full Processing

Runs complete post-import processing daily at 4 AM UTC.

- **Schedule**: `0 4 * * *` (daily at 4 AM UTC)
- **Function**: Runs all post-processing steps
- **Monitoring**: Slack notifications with detailed results

### Existing Cron Jobs (Updated)

- `/api/cron/update-company-ratings` - Daily at 3 AM UTC
- `/api/cron/update-product-counts` - Daily at 2 AM UTC

## CLI Interface

### `scripts/run-full-import-pipeline.ts`

Simple command-line tool to run the complete import pipeline.

```bash
# Run full pipeline (import + post-process)
pnpm run db:import-pipeline

# Only run post-processing on existing data
pnpm run db:import-pipeline --skip-import

# Dry run to see what would be done
pnpm run db:import-pipeline --dry-run

# Only run import, skip post-processing
pnpm run db:import-pipeline --skip-post-process
```

## Location Backfilling Strategy

The new location backfill prioritizes the `companies.area` field:

### 1. Area Field Mapping

Direct mappings for known areas:

```javascript
const areaMappings = {
  "canggu": { city: "Canggu", region: "Bali", country: "Indonesia", ... },
  "seminyak": { city: "Seminyak", region: "Bali", country: "Indonesia", ... },
  "chiang mai": { city: "Chiang Mai", region: "Chiang Mai", country: "Thailand", ... },
  // ... more mappings
};
```

### 2. Fallback Strategies

- Address parsing using deterministic parser
- Coordinate-based nearest city lookup
- Database resolution against cities/regions tables

### 3. Data Updated

- `companies.city`, `companies.region`, `companies.country`
- `companies.citySlug`, `companies.regionSlug`, `companies.countrySlug`
- `company_locations` table with normalized data

## Testing Strategy

### Integration Tests

**`src/__tests__/integration/post-import-processing.integration.test.ts`**

- Tests the complete post-import workflow
- Verifies all steps execute correctly
- Checks data integrity after processing

**`src/__tests__/integration/location-backfill.integration.test.ts`**

- Tests location backfill with various scenarios
- Verifies area field prioritization
- Tests fallback strategies

### Test Coverage

- ✅ Full pipeline execution
- ✅ Individual step testing
- ✅ Dry-run functionality
- ✅ Error handling
- ✅ Data validation
- ✅ API endpoint testing

## Migration from Old System

### What Changes

1. **Replace script calls** with API calls in any automation
2. **Update cron jobs** to use new endpoints
3. **Use CLI tool** instead of manual script execution
4. **Remove old scripts** that are no longer needed

### Backward Compatibility

- Old bulk import scripts still work but are deprecated
- New system is additive, doesn't break existing functionality
- Gradual migration path available

## Operating the New System

### Daily Operations

1. **After bulk imports**: Call `/api/admin/post-import/process`
2. **Monitoring**: Check cron job Slack notifications
3. **Issues**: Review detailed error logs in API responses

### Manual Operations

```bash
# Quick post-processing after import
curl -X POST http://localhost:3000/api/admin/post-import/process \
  -H "Content-Type: application/json" \
  -d '{"steps": ["normalize-slugs", "backfill-locations", "update-ratings", "update-counts"]}'

# Location backfill only
curl -X POST http://localhost:3000/api/admin/locations/backfill \
  -H "Content-Type: application/json" \
  -d '{"limit": 5000}'

# Dry run to preview changes
curl -X POST http://localhost:3000/api/admin/post-import/process \
  -H "Content-Type: application/json" \
  -d '{"options": {"dryRun": true}}'
```

### Emergency Operations

- **Stop cron jobs** if issues detected
- **Manual API calls** for targeted fixes
- **Dry-run mode** to test changes safely
- **Detailed logging** for troubleshooting

## Performance Characteristics

### Expected Run Times

- **Slug normalization**: ~1-2 seconds per 1000 companies
- **Location backfill**: ~5-10 seconds per 1000 companies
- **Rating updates**: ~2-3 seconds per 1000 companies
- **Count updates**: ~1-2 seconds total

### Scalability

- **Batch processing**: Handles large datasets efficiently
- **Memory efficient**: Streams data instead of loading all at once
- **Database optimized**: Uses appropriate indexes and batch operations

## Monitoring and Alerting

### Slack Notifications

- **Success**: Detailed breakdown of each processing step
- **Warnings**: Partial failures with error details
- **Errors**: Critical failures with full error context

### Key Metrics to Monitor

- Processing duration trends
- Success/failure rates
- Data quality improvements (location fill rates, etc.)
- Performance regressions

## Future Enhancements

### Potential Improvements

1. **Real-time processing** - Process imports as they happen
2. **Incremental updates** - Only process changed data
3. **Advanced location resolution** - More sophisticated area mappings
4. **Parallel processing** - Distribute work across multiple workers
5. **Configurable pipelines** - Allow custom processing steps

### Maintenance Tasks

- Regular review of area mappings
- Performance optimization based on usage patterns
- Test suite expansion as new scenarios arise
- Documentation updates as processes evolve

---

This new strategy provides a robust, testable, and maintainable approach to import and backfill operations that will scale with the application's needs.
