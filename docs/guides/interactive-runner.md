# Interactive Runner Documentation

The Interactive Runner is a flexible script execution tool for running maintenance and migration tasks across different environments.

## Table of Contents

- [Quick Start](#quick-start)
- [Usage Modes](#usage-modes)
- [Available Scripts](#available-scripts)
- [Environments](#environments)
- [Date Range Options](#date-range-options)
- [Examples](#examples)
- [Environment Setup](#environment-setup)
- [Troubleshooting](#troubleshooting)

## Quick Start

### Interactive Mode (Default)

```bash
npm run interactive-runner
```

This launches a menu-driven interface where you can select:

1. The script to run
2. The environment to use
3. Date range (for applicable scripts)

### Non-Interactive Mode

```bash
npm run interactive-runner -- --script <script-name> --env <environment> [--date-range <range>]
```

Perfect for automation, CI/CD, or quick executions.

## Usage Modes

### Interactive Mode

The default mode with a user-friendly menu interface:

```bash
npm run interactive-runner
```

**Features:**

- ✅ Visual menu for script selection
- ✅ Environment file validation
- ✅ Guided date range selection
- ✅ Clear descriptions for each option

### Non-Interactive Mode

Command-line driven execution:

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range all
```

**Features:**

- ✅ No prompts or user input required
- ✅ Scriptable and automatable
- ✅ Validates all inputs before execution
- ✅ Clear error messages

### Help Mode

```bash
npm run interactive-runner -- --help
```

Displays this help information.

## Available Scripts

### `firebase-incremental-sync`

**Description:** Sync recent Firebase data (bookings, bikes, reviews) to database

**Use Cases:**

- Regular data synchronization from Firebase to Supabase
- Incremental updates to avoid full migrations
- Recovery from failed syncs

**Date Range Support:** ✅ Yes

**Example:**

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range today
```

---

### `locations-pipeline`

**Description:** Seed → backfill (coords-aware) → populate counts

**Use Cases:**

- Initial location data setup
- Updating location hierarchies
- Refreshing product counts by location

**Date Range Support:** ❌ No

**Example:**

```bash
npm run interactive-runner -- --script locations-pipeline --env production
```

---

### `backfill-company-locations`

**Description:** Backfill company city/region using address + coordinates

**Use Cases:**

- Fixing missing location data for companies
- Geocoding company addresses
- Location normalization

**Date Range Support:** ❌ No

---

### `populate-locations-and-counts`

**Description:** Normalize slugs and update product counts for locations

**Use Cases:**

- Refreshing location-based product counts
- Slug normalization
- Search index updates

**Date Range Support:** ❌ No

---

### `seed-locations`

**Description:** Seed countries/regions/cities from CSV

**Use Cases:**

- Initial location database setup
- Adding new regions/cities
- Location data updates

**Date Range Support:** ❌ No

---

### `backfill-company-ratings`

**Description:** Update company ratings and review counts from reviews

**Use Cases:**

- Recalculating aggregate ratings
- Fixing rating inconsistencies
- After bulk review imports

**Date Range Support:** ❌ No

---

### `add-test-ratings`

**Description:** Add test ratings to companies for development

**Use Cases:**

- Development environment setup
- Testing rating features
- Demo data generation

**Date Range Support:** ❌ No

## Environments

### Development (`development`)

**Environment File:** `.env.local`

**When to Use:**

- Local development
- Testing changes before deployment
- Debugging

**Database:** Local or development database

---

### Preview (`preview`)

**Environment File:** `.vercel/.env.preview.local`

**When to Use:**

- Testing in staging environment
- Pre-production validation
- Feature branch deployments

**Database:** Preview/staging database

**Setup:**

```bash
vercel pull --environment=preview
```

---

### Production (`production`)

**Environment File:** `.vercel/.env.production.local`

**When to Use:**

- Live data operations
- Production migrations
- Real user-facing changes

**Database:** Production database

**Setup:**

```bash
vercel pull --environment=production
```

⚠️ **Important:** Always test in preview before running on production!

## Date Range Options

Available for scripts that support incremental/date-based operations (like `firebase-incremental-sync`).

### `today`

Processes records from today (00:00:00 onwards)

**Example:**

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range today
```

---

### `7days`

Processes records from the last 7 days

**Example:**

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range 7days
```

---

### `30days`

Processes records from the last 30 days

**Example:**

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range 30days
```

---

### `all`

Processes all records since 2020-01-01

**Example:**

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range all
```

⚠️ **Note:** This can take a long time and process thousands of records.

## Examples

### Common Use Cases

#### Daily Production Sync

Sync today's Firebase changes to production:

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range today
```

#### Weekly Preview Sync

Sync last week's data to preview environment:

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env preview --date-range 7days
```

#### Full Production Migration

Import all historical data:

```bash
npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range all
```

#### Update Location Data

Refresh location counts and slugs:

```bash
npm run interactive-runner -- --script populate-locations-and-counts --env production
```

#### Backfill Company Ratings

Recalculate all company ratings:

```bash
npm run interactive-runner -- --script backfill-company-ratings --env production
```

### Automation Examples

#### Cron Job (Daily Sync)

Add to your crontab:

```bash
0 2 * * * cd /path/to/flexbike-next && npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range today
```

#### GitHub Actions

```yaml
- name: Sync Firebase Data
  run: |
    vercel pull --environment=production --token=${{ secrets.VERCEL_TOKEN }}
    npm run interactive-runner -- --script firebase-incremental-sync --env production --date-range today
```

## Environment Setup

### Required Environment Variables

#### For All Scripts

- `DATABASE_URL` - Primary database connection (can use pgbouncer pooler)

#### For Migration Scripts

- `MIGRATIONS_DATABASE_URL` - Direct database connection for migrations (port 5432, no pgbouncer)

#### For Firebase Scripts

- `GOOGLE_APPLICATION_CREDENTIALS` - Path to Firebase service account JSON
- `FIREBASE_PROJECT_ID` - Firebase project ID
- `FIREBASE_PRIVATE_KEY` - Firebase private key
- `FIREBASE_CLIENT_EMAIL` - Firebase client email

### Database Connection Notes

**Regular Scripts:** Use `DATABASE_URL` with connection pooling (pgbouncer)

```
DATABASE_URL=postgresql://user:pass@host:6543/db?pgbouncer=true
```

**Migration Scripts:** Use `MIGRATIONS_DATABASE_URL` with direct connection

```
MIGRATIONS_DATABASE_URL=postgresql://user:pass@host:5432/db?sslmode=require
```

⚠️ **Important:** Migration scripts require direct connections (port 5432) because pgbouncer doesn't support session-level settings like `session_replication_role`.

### Setting Up Environment Files

#### Preview Environment

```bash
# Download from Vercel
vercel pull --environment=preview

# Verify file exists
ls -la .vercel/.env.preview.local
```

#### Production Environment

```bash
# Download from Vercel
vercel pull --environment=production

# Verify file exists
ls -la .vercel/.env.production.local
```

## Troubleshooting

### Error: Environment file not found

**Problem:** The script can't find `.vercel/.env.preview.local` or `.vercel/.env.production.local`

**Solution:**

```bash
vercel pull --environment=preview    # For preview
vercel pull --environment=production # For production
```

### Error: Script not found

**Problem:** Invalid script name provided

**Solution:** Check available scripts:

```bash
npm run interactive-runner -- --help
```

Or run in interactive mode to see the list:

```bash
npm run interactive-runner
```

### Error: Database connection failed

**Problem:** `MIGRATIONS_DATABASE_URL` or `DATABASE_URL` not set correctly

**Solution:**

1. Verify environment file exists and is loaded
2. Check that `MIGRATIONS_DATABASE_URL` is set for migration scripts
3. Ensure connection string uses correct port (5432 for direct, 6543 for pooler)
4. Add `sslmode=require` for direct connections

### Migration shows "Using DATABASE_URL" instead of "MIGRATIONS_DATABASE_URL"

**Problem:** `MIGRATIONS_DATABASE_URL` is not set in environment file

**Solution:** Add to `.vercel/.env.production.local`:

```bash
MIGRATIONS_DATABASE_URL=postgresql://user:pass@host:5432/db?sslmode=require
```

### Bookings not appearing after migration

**Problem:** Silent transaction rollback due to pgbouncer or missing foreign keys

**Solutions:**

1. Ensure using `MIGRATIONS_DATABASE_URL` (direct connection)
2. Check for missing user records (import users before bookings)
3. Review error logs for foreign key violations
4. Verify product/company references exist

### Performance Issues

**Problem:** Script runs very slowly

**Solutions:**

1. Use smaller date ranges (e.g., `today` instead of `all`)
2. Run during off-peak hours
3. Check database connection latency
4. Consider batch size adjustments in migration scripts

## Advanced Usage

### Custom Scripts

To add a new script to the runner:

1. Add to `AVAILABLE_SCRIPTS` in `scripts/interactive-runner.js`:

```javascript
{
  name: "my-custom-script",
  file: "path/to/script.js",
  description: "What this script does"
}
```

2. Place script in `scripts/` directory

3. Run with:

```bash
npm run interactive-runner -- --script my-custom-script --env development
```

### Environment-Specific Configuration

You can customize behavior per environment by checking in your script:

```javascript
const env = process.env.NODE_ENV || "development";
const isProd = env === "production";

if (isProd) {
  // Production-specific logic
  console.log("Running in production mode");
}
```

### Logging and Monitoring

Migration scripts include detailed logging:

- 📋 Database connection details
- 🔢 Record counts and progress
- ⚠️ Error messages with context
- ✅ Success confirmations

Monitor output for:

- Connection details (host, port, database)
- Which URL is being used (MIGRATIONS_DATABASE_URL vs DATABASE_URL)
- Sample records being processed
- Any errors or warnings

## Best Practices

1. **Always test in preview first**

   ```bash
   npm run interactive-runner -- --script firebase-incremental-sync --env preview --date-range today
   ```

2. **Use appropriate date ranges**

   - `today` for daily syncs
   - `7days` for weekly catch-ups
   - `all` only when necessary

3. **Monitor logs carefully**

   - Check database connection details
   - Verify correct environment is being used
   - Watch for errors or warnings

4. **Keep environment files updated**

   ```bash
   vercel pull --environment=production
   ```

5. **Set up MIGRATIONS_DATABASE_URL correctly**
   - Use port 5432 (direct connection)
   - Add `sslmode=require`
   - No pgbouncer parameter

## Support

For issues or questions:

1. Check this documentation
2. Review script-specific documentation in `scripts/firebase-to-supabase/`
3. Check migration logs for error details
4. Consult the team's technical documentation
