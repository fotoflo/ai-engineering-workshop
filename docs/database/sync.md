# Database Sync Scripts

This document explains how to sync your local and preview environments with production data using the database dump and restore scripts.

## Overview

Two scripts are provided to manage database synchronization:

1. **`dump-database.js`** - Dumps database to a file
2. **`restore-database.js`** - Restores database dump to local/preview environments

## Prerequisites

1. **PostgreSQL tools**: Ensure `pg_dump` and `pg_restore` are installed

   ```bash
   # macOS with Homebrew
   brew install postgresql

   # Ubuntu/Debian
   sudo apt-get install postgresql-client
   ```

2. **Environment variables**: Set up your `.env` file with database URLs

## Environment Setup

### For Dumping Production Data

Set your production database URL in `.env`:

```bash
# Production database (required for dumping)
DATABASE_URL=postgresql://username:password@host:port/database
```

### For Restoring to Local/Preview

Set your local and preview database URLs in `.env`:

```bash
# Local database (required for restoring)
DATABASE_URL=postgresql://username:password@localhost:5432/flexbike

# Preview database (optional)
PREVIEW_DATABASE_URL=postgresql://username:password@host:port/database
```

## Usage

### Step 1: Dump Production Database

```bash
# Using npm script
pnpm run db:dump

# Or directly
node scripts/dump-database.js
```

This will:

- Connect to your production database
- Create a backup file in `backups/` directory
- Show progress and file size information

### Step 2: Restore to Local/Preview

```bash
# Using npm script
pnpm run db:restore-database

# Or directly
node scripts/restore-database.js
```

This will:

- Find the most recent production dump
- Ask for confirmation before proceeding
- Reset local and preview databases
- Restore production data
- Handle permission issues gracefully

## Security Notes

- Database URLs are read from environment variables only
- Passwords are masked in console output
- Scripts validate environment setup before running
- Only the `public` schema is dumped/restored (avoids Supabase system schema issues)

## Troubleshooting

### Common Issues

1. **"DATABASE_URL environment variable is not set"**

   - Ensure your `.env` file exists and contains the correct DATABASE_URL
   - Check that the file is in the project root

2. **"No production dump files found"**

   - Run the dump script first: `pnpm run db:dump`
   - Check that the `backups/` directory exists and contains `.sql` files

3. **Permission denied errors**

   - This is normal when restoring between different Supabase projects
   - The scripts handle these gracefully and continue with available data

4. **Connection refused**
   - Verify your database URLs are correct
   - Ensure your network can reach the database hosts
   - Check firewall settings

### Manual Recovery

If automated scripts fail, you can manually:

1. **Dump production manually:**

   ```bash
   pg_dump "your-production-url" --schema=public --no-owner --no-privileges --clean --if-exists --format=custom --compress=9 --file=backups/manual-dump.sql
   ```

2. **Restore manually:**

   ```bash
   # Reset local database
   psql "your-local-url" -c "DROP SCHEMA public CASCADE;"
   psql "your-local-url" -c "CREATE SCHEMA public;"

   # Restore from dump
   pg_restore --clean --if-exists --no-owner --no-privileges --dbname="your-local-url" backups/manual-dump.sql
   ```

## File Locations

- **Scripts**: `scripts/dump-database.js`, `scripts/restore-database.js`
- **Backups**: `backups/production-dump-*.sql`
- **Environment**: `.env` (not committed to git)

## Best Practices

1. **Always backup first**: The restore script creates backups of your current data
2. **Test locally first**: Restore to local environment before touching preview
3. **Review changes**: Check the dump file size and creation time before restoring
4. **Use with caution**: Production data contains real user information
5. **Keep backups**: Don't delete old backup files immediately

## Support

If you encounter issues:

1. Check the troubleshooting section above
2. Verify your environment variables
3. Ensure database connectivity
4. Check PostgreSQL client tools are installed
5. Review error messages for specific guidance
