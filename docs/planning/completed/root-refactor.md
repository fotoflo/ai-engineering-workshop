# Root Directory Refactoring Proposal

## Current State Analysis

The root directory contains **80+ files** including scripts, logs, data files, and configuration files. This makes it difficult to navigate and understand the project structure.

## Proposed Directory Structure

### 1. **Scripts Organization** (`scripts/`)

Move all utility scripts into organized subdirectories:

#### `scripts/migrations/` - Database migration scripts

- `add-cancel-reason.js`
- `add-missing-columns.js`
- `add-production-indexes.sql`
- `apply-production-schema.js`
- `create_revenue_history_table.sql`
- `fix-preview-schema.js`
- `fix-production-constraint.js`
- `fix-production-schema.js`
- `manual_migration.sql`
- `safe-migration-with-shadow.js`
- `check_columns.sql` (move to `scripts/sql/`)

#### `scripts/import/` - Data import scripts

- `import-all-data.js`
- `import-single-booking.js`
- `re-import-bookings.js`
- `full-api-import.ts`
- `export-new-records-for-restore.js`
- `extract-emulator-data.js`
- `get-real-test-data.js`
- `populate-emulator-with-real-data.js`
- `populate-via-cli.js`
- `simple-populate.js`
- `generate-emulator-population-commands.js`

#### `scripts/validation/` - Validation and check scripts

- `check-bike-id.js`
- `check-booking-data.js`
- `check-bookings-schema.js`
- `check-collection-fields.js`
- `check-companyids.js`
- `check-empty-images.js`
- `check-emulator-company.js`
- `check-firebase-data.js`
- `check-local-db.js`
- `check-location-data.js`
- `check-production-auth.js`
- `check-production-booking.js`
- `check-production-company.js`
- `check-production-db.js`
- `check-production-tables.js`
- `check-region-mapping.js`
- `check-thailand-companies.js`
- `check-user.js`
- `validate-preview.js`

#### `scripts/analysis/` - Analysis and investigation scripts

- `analyze-booking-fields.js`
- `analyze-bookings.js`
- `analyze-production-schema.js`
- `investigate-auth-schema.js`
- `compare-firebase-data.js`
- `check-bollu-blocked-details.js`
- `check-bollu-garage.js`

#### `scripts/debugging/` - Debug scripts (already exists, add these)

- `debug-auth-flow.js`
- `debug-booking.js`
- `debug-production-admin.js`
- `debug-rollback.js`
- `debug-validation.js`

#### `scripts/fixes/` - One-time fix scripts

- `fix-admin-user-id.js`
- `fix-booking-data.js`
- `update-user-id.js`
- `test-chiang-mai-fix.js`

#### `scripts/backfill/` - Data backfill scripts

- `run-all-companies-backfill.js`
- `run-all-companies-occupancy-backfill.js`
- `run-occupancy-backfill.js`
- `run-revenue-backfill.js`

#### `scripts/sync/` - Sync scripts

- `sync-all-to-firebase.js`
- `sync-to-firebase-rest.js`
- `resync-specific-record.js`

#### `scripts/find/` - Find/search scripts

- `find-recent-bikes.js`
- `find-recent-bookings.js`
- `find-recent-companies.js`
- `find-recent-users.js`

#### `scripts/create/` - Creation scripts

- `create-green-island-motors-simple.js`
- `create-green-island-products-final.js`
- `create-missing-products.js`
- `create-test-offline-booking.js`

#### `scripts/tests/` - Test utility scripts

- `test-blog-proxy.js`
- `test-book-now-redirects.js`
- `test-bookings-import.ts`
- `test-data-fidelity.js`
- `test-modal-logic.js`
- `test-roundtrip.js`
- `test-search-api.js`
- `test-users-ensure.js`
- `test-users-ensure-curl.sh`
- `simple-roundtrip-test.js`

#### `scripts/sql/` - SQL scripts (already exists, add these)

- `direct-sql-execute.js`
- `check_columns.sql` (from root)

#### `scripts/shell/` - Shell scripts

- `ngrok-emu.sh`
- `ngrok-status.sh`
- `run-population-commands.sh`

### 2. **Logs Directory** (`logs/`)

Create a `logs/` directory and move all log files:

- `import.log`
- `messages_import_final.log`
- `messages_import.log`
- `migration_10k.log`
- `migration.log`
- `restore.log`
- `server.log`
- `firebase-debug.log`
- `firestore-debug.log`

**Note**: `logs/` is already in `.gitignore` (line 69), so we just need to create the directory.

### 3. **Data Files Organization**

**Note**: `backups/` directory already exists and is gitignored (contains SQL dumps and Firebase exports).

#### Move to `backups/` (backup files - already gitignored):

- `new-records-backup-firebase-2025-10-30T05-02-01.json`
- `new-records-backup-simple-2025-10-30T05-02-01.json`

#### Test/Fixture Data (Option B - Selected):

Move test fixtures to `tests/fixtures/` or `src/__tests__/fixtures/` (tracked in git):

- `emulator-bookings-clean.json` → `tests/fixtures/` or `src/__tests__/fixtures/`
- `firebase-test-data.json` → `tests/fixtures/` or `src/__tests__/fixtures/`
- `test-bike-import.json` → `tests/fixtures/` or `src/__tests__/fixtures/`
- `problematic-bookings-2025-12-01T12-50-02-143Z.json` → `tests/fixtures/` or `src/__tests__/fixtures/` (or could go to `backups/` if it's just analysis output)

**Note**: These will be tracked in git, so they can be used by automated tests in CI/CD.

### 4. **Documentation** (`docs/`)

**Important**: Understand the distinction:

- **`docs/agents/`** = Documentation FOR agents (codebase guide, conventions, how things work)
- **`agents/`** = Agent SPECIFICATIONS (how agents should behave/work)
- **Other `docs/`** = General project documentation

The Agent Writing Agent uses `docs/` as SOURCES to fill in agent specs, but agent specs themselves live in `agents/`.

Move markdown documentation files:

- `booking-relationships-2025-12-01T12-57-11-471Z.md` → `docs/migrations/`
- `firebase-booking-links-2025-12-01T12-51-51-732Z.md` → `docs/migrations/`
- `occupancy_calculation_documentation.md` → `docs/features/`
- `OFFLINE_BOOKING_TEST_README.md` → `docs/testing/`

### 5. **Keep in Root** (Configuration files)

These should stay in root as they're standard project configuration:

- `package.json`
- `package-lock.json`
- `pnpm-lock.yaml`
- `tsconfig.json`
- `tsconfig.tsbuildinfo`
- `next.config.mjs`
- `next-env.d.ts`
- `jsconfig.json`
- `jest.config.js`
- `jest.setup.js`
- `playwright.config.ts`
- `eslint.config.mjs`
- `postcss.config.mjs`
- `tailwind.config.mjs`
- `components.json`
- `vercel.json`
- `firebase.json`
- `firestore.rules`
- `firestore.indexes.json`
- `example.env`
- `README.md`
- `.gitignore` (if exists)

### 6. **Firebase Service Account** (Security consideration - URGENT)

**⚠️ SECURITY ISSUE**: The file `project-s-v2-firebase-adminsdk-b0xkg-d5cd382451.json` is currently **tracked in git** and contains sensitive credentials.

**Action Required**:

1. Add pattern to `.gitignore`: `*-firebase-adminsdk-*.json` or `*firebase-adminsdk*.json`
2. Remove from git tracking: `git rm --cached project-s-v2-firebase-adminsdk-b0xkg-d5cd382451.json`
3. Move to secure location: `config/firebase-service-account.json` (or use environment variables)
4. **Important**: If this file was already committed, consider rotating the service account credentials

**Recommendation**: Use environment variables instead of committing service account files.

### 7. **Assets** (`public/` or `assets/`)

- `flexbike - blue.svg` → `public/` (if used in app) or `assets/`

### 8. **Miscellaneous**

- `sitemap-urls.txt` → `scripts/generate-human-sitemap.js` output, could go to `data/` or `public/`

## Migration Plan

### Phase 1: Create Directory Structure

1. Create all new directories (`logs/`, `tests/fixtures/` or `src/__tests__/fixtures/`, script subdirectories)
2. Note: `logs/` and `backups/` are already gitignored; test fixtures will be tracked in git

### Phase 2: Move Files

1. Move scripts to appropriate subdirectories
2. Move logs to `logs/`
3. Move backup JSON files to `backups/` (already exists)
4. Move test fixture files to `tests/fixtures/` or `src/__tests__/fixtures/` (tracked in git)
5. Move documentation to `docs/`

### Phase 3: Update References

1. Search codebase for imports/references to moved scripts
2. Update any hardcoded paths
3. Update documentation that references these scripts
4. Update `package.json` scripts if they reference these files

### Phase 4: Cleanup

1. Remove empty directories
2. Update README with new structure
3. Add directory README files explaining purpose

## Benefits

1. **Clarity**: Root directory only contains essential configuration files
2. **Organization**: Scripts grouped by purpose (migrations, validation, debugging, etc.)
3. **Maintainability**: Easier to find and understand scripts
4. **Scalability**: Easy to add new scripts in appropriate locations
5. **Professional**: Cleaner project structure

## File Count Reduction

- **Before**: ~80 files in root
- **After**: ~20 configuration files in root
- **Reduction**: ~75% fewer files in root directory

## Notes

- Some scripts may be one-time use and could be archived in `scripts/archive/`
- Consider creating a `scripts/README.md` documenting the purpose of each subdirectory
- Review scripts to identify which are still actively used vs. historical
