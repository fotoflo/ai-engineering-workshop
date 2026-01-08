# Interactive Cron Job Runner

A utility script to run Flexbike cron jobs with different environment configurations interactively.

## Usage

```bash
# Run interactively (recommended)
node scripts/run-cron-interactive.js

# Or make it executable and run directly
chmod +x scripts/run-cron-interactive.js
./scripts/run-cron-interactive.js
```

## Available Scripts

1. **backfill-company-ratings** - Update company ratings and review counts from reviews
2. **add-test-ratings** - Add test ratings to companies for development
3. **populate-locations-and-counts** - Update location data and product counts

## Available Environments

1. **development** - `.env.local` (Local development environment)
2. **production** - `.env.production` (Production environment)
3. **main** - `.env` (Main environment file)

## Features

- ✅ Interactive menu system
- ✅ Environment variable loading from selected .env file
- ✅ Script validation and error handling
- ✅ Colored output for better UX
- ✅ Confirmation before execution
- ✅ Proper exit codes and error reporting

## Example Usage

```
🚀 Flexbike Cron Job Runner
===========================

📜 Available Scripts:
  1. backfill-company-ratings
     Update company ratings and review counts from reviews
  2. add-test-ratings
     Add test ratings to companies for development
  3. populate-locations-and-counts
     Update location data and product counts

Choose a script (1-3): 1

🌍 Available Environments:
  1. development
     Local development environment (.env.local)
  2. production
     Production environment (.env.production)
  3. main
     Main environment file (.env)

Choose an environment (1-3): 1

⚡ Ready to execute:
   Script: backfill-company-ratings
   Environment: development
   File: backfill-company-ratings.js
   Env File: .env.local

Proceed? (y/N): y

Running backfill-company-ratings with development environment
===============================================================

✅ Loaded 45 environment variables
ℹ️  Executing: node scripts/backfill-company-ratings.js
ℹ️  Using environment: .env.local
✅ backfill-company-ratings completed successfully!
🎉 Cron job completed successfully!
```

## Security Notes

- The script loads environment variables from the selected .env file
- Make sure your .env files contain the correct `DATABASE_URL` and other required variables
- For production runs, use the `.env.production` file with production database credentials

## Manual Execution

If you prefer to run scripts manually:

```bash
# Load environment and run script directly
DATABASE_URL="your-connection-string" node scripts/backfill-company-ratings.js

# Or use the interactive runner
node scripts/run-cron-interactive.js
```
