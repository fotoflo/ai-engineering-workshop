# Flexbike Database & Schema Management

## Table of Contents

### Overview & Core Concepts

- [Overview](#overview) (line 60)
- [Prisma Schema Management](#prisma-schema-management) (line 105)
  - [Source of Truth](#source-of-truth) (line 107)
  - [Schema Update Process](#schema-update-process) (line 118)
  - [Type Generation & Validation](#type-generation--validation) (line 154)
  - [Schema Best Practices](#schema-best-practices) (line 207)

### Migration Strategy & Risk Assessment

- [Migration Strategy](#migration-strategy) (line 301)
  - [Critical: Code Impact Assessment](#critical-code-impact-assessment) (line 303)
  - [Code Consistency Rules](#code-consistency-rules) (line 336)
- [Core Principles](#core-principles) (line 394)
- [Migration Strategies](#migration-strategies) (line 429)
  - [Strategy 1: IF NOT EXISTS](#strategy-1-if-not-exists) (line 431)
  - [Strategy 2: Shadow Table Backup](#strategy-2-shadow-table-backup) (line 470)
  - [Strategy 3: Full Migration Scripts](#strategy-3-full-migration-scripts) (line 518)
- [Risk Assessment Framework](#risk-assessment-framework) (line 538)

### Environment & Execution Guidelines

- [Environment-Specific Guidelines](#environment-specific-guidelines) (line 567)
- [Execution Workflow](#execution-workflow) (line 590)
- [Tool Selection Guidelines](#tool-selection-guidelines) (line 617)
- [Recovery Procedures](#recovery-procedures) (line 639)
- [Success Metrics](#success-metrics) (line 682)

### Practical Examples & Implementation

- [Migration Examples](#migration-examples) (line 700)
- [Database Dump & Testing](#database-dump--testing) (line 793)
- [Future Improvements](#future-improvements) (line 919)

### API Development & Type Safety

- [API Development & Database Interactions](#api-development--database-interactions) (line 935)
  - [Database Access Patterns](#database-access-patterns) (line 952)
  - [API Route Structure](#api-route-structure) (line 981)
  - [Error Handling with Types](#error-handling-with-types) (line 1023)
  - [Transaction Patterns](#transaction-patterns) (line 1049)
  - [Performance Considerations](#performance-considerations) (line 1090)
- [Developer Environment Setup](#developer-environment-setup) (line 1121)
- [Type Safety & Validation](#type-safety--validation) (line 1182)
  - [Always Use Generated Types](#always-use-generated-types) (line 1184)
  - [Function Definitions with Zod Prisma Types](#function-definitions-with-zod-prisma-types) (line 1221)
  - [API Input/Output Validation](#api-inputoutput-validation) (line 1303)
  - [Custom Validation Rules](#custom-validation-rules) (line 1323)

### Summary & Conclusion

- [Conclusion](#conclusion) (line 1343)

---

## Overview

This document outlines Flexbike's comprehensive database management approach, including schema design, migrations, API development, and validation. Our strategy prioritizes **zero data loss**, **type safety**, and **maximum control** over automated tools that can be destructive.

**Referenced by:** `agents.sitemap.md` (Core Agents File)

---

## ⚠️ CRITICAL REQUIREMENT

**This entire document MUST be read in its entirety before beginning ANY database migration work.** Database migrations carry significant risk to business operations. Understanding the complete context, historical lessons, and safety procedures outlined here is essential for safe execution.

**For Migration Work:**

- Read all sections completely (1225+ lines)
- Understand the three migration strategies and when to use each
- Review all examples and testing requirements
- Plan your approach using the risk assessment framework
- Ensure you have backup and rollback procedures ready

**Failure to read this document completely may result in:**

- Data loss incidents
- Extended downtime
- Business disruption
- Violation of production safety protocols

---

**Key Components:**

- ✅ **Prisma Schema** - Source of truth for database structure, types, and validation
- ✅ **Migration Strategy** - Production-safe approaches for schema changes
- ✅ **API Development** - Database interaction patterns and best practices
- ✅ **Type Safety** - Zod validation and TypeScript integration

---

## Prisma Schema Management

### Source of Truth

The `prisma/schema.prisma` file is the **single source of truth** for:

- **Database Structure** - All tables, columns, relationships, constraints
- **Type Definitions** - Generated TypeScript types for application code
- **Validation Schemas** - Zod parsers for runtime data validation
- **API Contracts** - Expected shapes for database interactions

**Critical Rule:** Always update `schema.prisma` BEFORE making database changes.

### Schema Update Process

**Complete workflow for schema changes:**

```bash
# 1. Update schema.prisma with new/changed fields

# 2. Validate schema syntax and structure
npx prisma validate

# 3. Generate TypeScript types and Zod schemas
npx prisma generate

# 4. Create migration for review (optional, for complex changes)
npx prisma migrate dev --create-only

# 5. Review generated migration SQL (if created)
# 6. Apply manually with our safe migration strategy
```

**Always run after schema changes:**

```bash
npx prisma validate  # Check schema syntax and consistency
npx prisma generate  # Generate TypeScript types and Zod validation schemas
```

**Include in CI/CD and development workflows:**

```yaml
# In package.json scripts or CI pipeline
"validate:schema": "prisma validate",
"generate:types": "prisma generate",
"postinstall": "prisma generate"  # Run after npm install
```

### Type Generation & Validation

**Auto-generated Types:**

```typescript
// Generated from schema.prisma
type User = {
  id: string;
  email: string;
  role: UserRole;
  // ... all fields
};
```

**Zod Validation Schemas (Auto-generated by `prisma generate`):**

```typescript
// Generated by npx prisma generate
// Located in: @/generated/zod or wherever configured in prisma/schema.prisma

export const UserSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  role: z.enum(["CUSTOMER", "ADMIN", "SUPER_ADMIN"]),
});

export const UserCreateSchema = z.object({
  email: z.string().email(),
  firstName: z.string().optional(),
  lastName: z.string().optional(),
  role: z.enum(["CUSTOMER", "ADMIN", "SUPER_ADMIN"]).default("CUSTOMER"),
});

export const UserUpdateSchema = z.object({
  email: z.string().email().optional(),
  firstName: z.string().optional(),
  lastName: z.string().optional(),
  role: z.enum(["CUSTOMER", "ADMIN", "SUPER_ADMIN"]).optional(),
});

// Available for all models: Create, Update, Where, OrderBy, etc.
```

**Usage in APIs:**

```typescript
// Always validate input/output with generated schemas
export async function createUser(data: unknown) {
  const validatedData = UserCreateSchema.parse(data);
  return prisma.user.create({ data: validatedData });
}
```

### Schema Best Practices

**✅ ALWAYS RUN after schema changes:**

```bash
npx prisma validate   # Validate schema syntax and structure
npx prisma generate   # Generate TypeScript types and Zod validation schemas
```

#### Prisma Schema Design Patterns

**✅ REQUIRED for all models:**

```prisma
model Booking {
  id          String   @id @default(uuid())  // Always use UUID for IDs
  createdAt   DateTime @default(now())      // Always include timestamps
  updatedAt   DateTime @updatedAt           // Auto-updated timestamp

  // Relations with CASCADE delete for data integrity
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Indexes for performance (add based on query patterns)
  @@index([userId])              // Foreign key indexes
  @@index([createdAt])           // Common filter/sorting
  @@index([status, createdAt])   // Composite indexes for complex queries

  // Enums for constrained values (not free-form strings)
  status BookingStatus @default(PENDING)
}

model User {
  id            String    @id @default(uuid())
  email         String    @unique
  role          UserRole  @default(CUSTOMER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  // Relations
  bookings      Booking[]

  // Indexes
  @@index([email])        // For login lookups
  @@index([role])         // For admin queries
}
```

**✅ DO:**

- **Cascade Deletes:** Always specify `onDelete: Cascade` for required relations to maintain data integrity
- **Indexes:** Add `@@index([])` for frequently queried/filtered/sorted fields
- **Enums:** Use enums instead of strings for constrained values (`status`, `role`, `type` fields)
- **Timestamps:** Include `createdAt` and `updatedAt` on all models
- **UUID IDs:** Use `String @id @default(uuid())` for all primary keys
- **Unique Constraints:** Add `@unique` for fields that must be unique
- **Meaningful Names:** Name tables and fields clearly - avoid `map` attribute
- **Field Ordering:** Group related fields, put relations at bottom
- **Comments:** Add comments for complex business logic

**❌ DON'T:**

- **No Map Attribute:** Don't use `@@map("table_name")` - name models correctly from the start
- **No Cascade Without Thought:** Don't add cascade delete without considering business impact
- **No Missing Indexes:** Don't forget indexes on foreign keys and commonly queried fields
- **No Free-Form Strings:** Don't use `String` for status/role fields - use enums
- **No Optional Relations:** Don't make foreign keys optional unless truly nullable
- **No Missing Timestamps:** Don't create models without `createdAt`/`updatedAt`
- **No Auto-Increment IDs:** Don't use `Int @id @default(autoincrement())` - use UUIDs
- **No Over-Indexing:** Don't add unnecessary indexes - each slows down inserts/updates

**Schema Conventions:**

- **Model Names:** PascalCase, singular (e.g., `User`, `Booking`, not `Users`, `Bookings`)
- **Field Names:** camelCase (e.g., `userId`, `createdAt`, `isActive`)
- **Enum Names:** PascalCase (e.g., `UserRole`, `BookingStatus`)
- **Relation Fields:** Use singular name of related model (e.g., `user`, `company`)

#### General Best Practices

**✅ DO:**

- Keep schema.prisma updated with all changes
- Always run `prisma validate` and `prisma generate` after schema changes
- Use generated types throughout application
- Validate all inputs/outputs with auto-generated Zod schemas
- Review migration SQL before execution
- Document complex business rules in schema comments

**❌ DON'T:**

- Make direct database changes without updating schema
- Use raw SQL types when Prisma types exist
- Skip `prisma validate` or `prisma generate` after schema changes
- Skip Zod validation for performance (it's fast and critical for safety)
- Apply migrations without manual review
- Use shortened prisma variables (`const p = prisma`)
- Access prisma directly without proper imports (`prisma.bookings.findMany()`)
- Cast prisma as `any` (breaks type safety)

---

## Migration Strategy

### Critical: Code Impact Assessment

**⚠️ REMINDER: This entire document must be read before proceeding with any migration work.**

**Before any schema change that creates `_old` columns:**

1. **Search the entire codebase** for references to the column being changed:

   ```bash
   # Search for exact column name in quotes (handles camelCase vs snake_case)
   grep -r '"handoverConfirmed"' src/ --include="*.ts" --include="*.tsx"

   # Also search for camelCase variants
   grep -r "handoverConfirmed" src/ --include="*.ts" --include="*.tsx"
   ```

2. **Catalog ALL references:**

   **Find database queries that need updating:**

   ```bash
   # Find all references to the specific table/model
   grep -r "prisma\.bookings\." src/ --include="*.ts" --include="*.tsx"

   # Also search for camelCase variations
   grep -r "prisma\.bookings" src/ --include="*.ts" --include="*.tsx"
   ```

   **⚠️ IMPORTANT:** The standard pattern is `prisma.bookings.findMany()`, NOT:

   - `p.bookings.findMany()` (shortened variable)
   - `(prisma as any).bookings.findMany()` (type casting)
   - `bookings.findMany()` (missing prisma prefix)

   **Catalog by type:**

   - API route handlers (e.g., `src/app/api/bookings/route.ts`)
   - React components/hooks (e.g., `src/app/components/BookingsList.tsx`)
   - Utility functions (e.g., `src/lib/booking-utils.ts`)
   - Tests (e.g., `src/__tests__/bookings.test.ts`)
   - Type definitions (e.g., `src/lib/types.ts`)

3. **Choose backward compatibility approach:**

   - **Option A (RECOMMENDED)**: Add new column, keep old column, update references - **Option B (RISKY)**: Rename old column to `_old`, update references

4. **Update ALL Prisma references:**
   - Change `handoverConfirmed` to `handoverConfirmedAt` in all queries
   - Update TypeScript types (regenerate with `prisma generate`)
   - Update Zod schemas (auto-generated)
   - Test all affected functionality

### Code Consistency Rules

**NEVER use these patterns (they cause catastrophic errors):**

```typescript
// ❌ WRONG - Shortened variable names
const p = prisma;
p.bookings.findMany({...});

// ❌ WRONG - Creating your own Prisma client
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();
prisma.bookings.findMany({...});

// ❌ WRONG - Type casting
(prisma as any).bookings.findMany({...});
```

**ALWAYS use this pattern:**

```typescript
// ✅ CORRECT - Consistent prisma usage
import { prisma } from "@/server/db/prisma";

prisma.bookings.findMany({
  where: { handoverConfirmedAt: { not: null } },
});
```

**Migration Philosophy:**

**Gradual migration with backward compatibility is ALWAYS preferred over big-bang changes.** This approach:

- Minimizes risk by allowing incremental updates
- Provides easy rollback options
- Allows thorough testing at each step
- Prevents catastrophic failures from affecting the entire application

**Historical Context:**

- Multiple disruptive migrations recovered via reimporting the Firebase database
- Without reset capability, business operations would be severely impacted
- Need for **bulletproof** migration strategy in production

**Future Transition:**

- **Phase 1 (Soon):** Begin syncing Firebase database on each operation
- **Phase 2 (Future):** Eliminate Firebase database entirely
- **Critical:** Migration safety becomes even more vital as Firebase safety net is removed

**Philosophy:**

- **Manual control > Automation** when data integrity is at stake
- **Conservative > Convenient** for production databases
- **Explicit > Implicit** operations with clear audit trails

---

## Core Principles

### 1. Zero Data Loss Guarantee

- Every migration must preserve existing data
- Backup strategies for any potentially destructive operation
- Recovery procedures documented and tested

### 2. Manual Review Required

- No automated migrations in production
- All SQL reviewed by human before execution
- Clear audit trail of who approved what

### 3. Environment-Specific Strategies

- Development: May use automated tools for speed
- Production: Always manual, conservative approach
- Preview/Staging: Mix of both with extra caution

### 4. Risk Assessment First

- Evaluate migration risk before starting
- Choose appropriate strategy based on risk level
- Have rollback plan ready
- **Future:** Risk tolerance decreases as Firebase safety net is removed

### 5. Firebase Transition Awareness

- **Phase 1 (Soon):** Extra caution during Firebase sync implementation
- **Phase 2 (Future):** Zero tolerance for migration failures
- **All Agents:** Plan for post-Firebase world where recovery options are limited

---

## Migration Strategies

### Strategy 1: IF NOT EXISTS (80% of cases)

**Use For:** Safe additions that cannot conflict

**Pattern:**

```sql
-- Column additions
ALTER TABLE public.bookings ADD COLUMN IF NOT EXISTS "newColumn" TEXT;

-- Index additions
CREATE INDEX IF NOT EXISTS idx_table_column ON public.table(column);

-- Constraint additions
ALTER TABLE public.table ADD CONSTRAINT IF NOT EXISTS constraint_name UNIQUE(column);
```

**Examples:**

```sql
-- ✅ Safe: Adding new optional columns
ALTER TABLE public.bookings ADD COLUMN IF NOT EXISTS "cancelReason" TEXT;
ALTER TABLE public.bookings ADD COLUMN IF NOT EXISTS "paidOut" BOOLEAN DEFAULT FALSE;

-- ✅ Safe: Adding performance indexes
CREATE INDEX IF NOT EXISTS bookings_userId_idx ON public.bookings("userId");
CREATE INDEX IF NOT EXISTS users_email_idx ON public.users(email);

-- ✅ Safe: Adding constraints
ALTER TABLE public.companies ADD CONSTRAINT IF NOT EXISTS companies_slug_key UNIQUE (slug);
```

**When to Use:**

- Adding new columns (nullable or with defaults)
- Creating indexes on existing columns
- Adding constraints to existing tables
- Any operation that cannot cause data loss

### Strategy 2: Shadow Table Backup (15% of cases)

**Use For:** Potentially destructive operations requiring data transformation

**Pattern:**

```sql
-- 1. Create backup
CREATE TABLE public.table_column_backup AS
SELECT id, "column" FROM public.table WHERE "column" IS NOT NULL;

-- 2. Perform migration
ALTER TABLE public.table RENAME COLUMN "column" TO "column_old";
ALTER TABLE public.table ADD COLUMN "column" NEW_TYPE;

-- 3. Restore/transform data
UPDATE public.table
SET "column" = transform_function(backup."column")
FROM public.table_column_backup backup
WHERE table.id = backup.id;

-- 4. Validate and cleanup
-- Keep backup until confident, then DROP
```

**Example:**

```sql
-- Type change with data preservation
CREATE TABLE public.bookings_cancelReason_backup AS
SELECT id, "cancelReason" FROM public.bookings WHERE "cancelReason" IS NOT NULL;

ALTER TABLE public.bookings RENAME COLUMN "cancelReason" TO "cancelReason_old";
ALTER TABLE public.bookings ADD COLUMN "cancelReason" VARCHAR(500);

UPDATE public.bookings
SET "cancelReason" = LEFT(backup."cancelReason", 500)
FROM public.bookings_cancelReason_backup backup
WHERE bookings.id = backup.id;
```

**When to Use:**

- Type changes that might truncate data (TEXT → VARCHAR(n))
- Complex data transformations
- Restructuring that affects multiple columns
- Any operation where data might be lost or corrupted

### Strategy 3: Full Migration Scripts (5% of cases)

**Use For:** Major schema restructuring requiring coordination

**Pattern:**

- Create comprehensive migration script
- Include backup, migration, validation, rollback steps
- Test on staging environment first
- Have business approval for production execution

**When to Use:**

- Major table restructuring
- Multi-table data migrations
- Schema changes affecting core business logic
- Operations requiring business downtime windows

---

## Risk Assessment Framework

### Low Risk (Strategy 1)

- ✅ Cannot cause data loss
- ✅ Reversible without data impact
- ✅ No business logic changes

### Low Risk (Strategy 2 Alternative - RECOMMENDED)

- ✅ Add new column, keep old as fallback
- ✅ Gradual code updates, no big-bang changes
- ✅ Easy rollback if issues discovered
- ✅ No data loss risk

### Medium Risk (Strategy 2)

- ⚠️ Potential for data transformation issues
- ⚠️ Requires backup and validation
- ⚠️ May affect application performance temporarily

### High Risk (Strategy 3)

- 🚨 Can cause data loss if failed
- 🚨 May require application downtime
- 🚨 Affects core business operations

---

## Environment-Specific Guidelines

### Development

- May use Prisma migrate dev for rapid iteration
- Quick resets acceptable for experimentation
- Focus on speed over safety

### Preview/Staging

- Mix of automated and manual approaches
- Extra validation steps required
- Mirror production constraints where possible

### Production

- **MANDATORY:** Manual SQL review and approval
- **MANDATORY:** Backup verification before execution
- **MANDATORY:** Rollback plan documented and tested
- **NEVER:** Use automated migration tools

---

## Execution Workflow

### Pre-Migration

1. **Risk Assessment:** Evaluate using framework above
2. **Strategy Selection:** Choose appropriate approach
3. **Backup Verification:** Ensure recent backups exist
4. **Rollback Plan:** Document recovery steps
5. **Business Approval:** Get sign-off for production changes

### During Migration

1. **Dry Run:** Test on staging/preview first
2. **Backup Creation:** Create additional backups if needed
3. **SQL Review:** Human review of all SQL to be executed
4. **Execution:** Run in production with monitoring
5. **Validation:** Verify data integrity and application functionality

### Post-Migration

1. **Monitoring:** Watch for errors/performance issues
2. **Cleanup:** Remove temporary objects (shadow tables, old columns)
3. **Documentation:** Record what was done and any issues encountered
4. **Communication:** Notify team of successful migration

---

## Tool Selection Guidelines

### Prisma Usage

- **Development:** `prisma migrate dev` - fast iteration
- **Production:** ❌ NEVER - too risky
- **Schema Definition:** ✅ OK - for type safety and documentation

### Direct SQL (Our Primary Tool)

- **All Environments:** ✅ Preferred for safety
- **Pattern:** Always include `IF NOT EXISTS` checks
- **Review:** Mandatory human review before execution

### Alternative ORMs (Future Consideration)

- **Drizzle:** Good alternative - generates reviewable SQL
- **Kysely:** Maximum safety - no automatic schema management
- **pg (node-postgres):** Direct control - what we do now

---

## Recovery Procedures

### Immediate Rollback (< 5 minutes)

1. If migration fails immediately, use shadow table to restore
2. Drop partially migrated objects
3. Restore from backup table

### Database Reset (Last Resort)

**⚠️ DEPRECATING:** This option will become unavailable as we transition away from Firebase.

**Current Status:**

1. **Only possible because we currently shadow Firebase data**
2. Reset PostgreSQL database
3. Re-run complete data import from Firebase
4. Restore application to working state

**Future Status (Post-Firebase):**

- ❌ **UNAVAILABLE** - No external data source for reset
- ⚠️ **Prevention becomes critical** - migrations must succeed first time
- ✅ **Enhanced backup strategies required**

### Business Continuity

**Current (Firebase-dependent):**

- Firebase shadow as ultimate backup
- Document RTO (Recovery Time Objective) for each migration type
- Maintain runbooks for common recovery scenarios

**Future (Post-Firebase transition):**

- **Enhanced shadow table strategies** for all migrations
- **Multiple backup points** throughout migration process
- **Comprehensive testing** before production execution
- **Business approval required** for any migration with > LOW risk
- **Zero tolerance** for migration failures

---

## Success Metrics

### Migration Quality

- ✅ Zero data loss incidents
- ✅ < 5 minute rollback capability
- ✅ 100% manual review completion rate
- ✅ Zero production incidents from migrations

### Process Quality

- ✅ All migrations documented
- ✅ Risk assessments completed
- ✅ Business approval obtained
- ✅ Post-migration monitoring completed

---

## Migration Examples

### Example 1: Safe Column Addition (Low Risk)

```sql
-- Risk: LOW
-- Strategy: IF NOT EXISTS
ALTER TABLE public.bookings ADD COLUMN IF NOT EXISTS "handoverConfirmed" BOOLEAN DEFAULT FALSE;
```

### Example 2: Safe Column Evolution with Gradual Migration (Low Risk)

**Safer Approach: Add new column, keep old column, update references gradually**

**Pre-migration Checklist:**

- ✅ Searched codebase for all `handoverConfirmed` references
- ✅ Cataloged: 3 API routes, 2 React components, 1 utility function
- ✅ User chose: Gradual migration with backward compatibility
- ✅ Added new column `handoverConfirmedAt` to schema
- ✅ Types regenerated with `prisma generate`

**Migration Process:**

1. Add new column (no data migration yet)
2. Update code references one by one (API routes first)
3. Test each component after update
4. Only drop old column after all references updated and verified

```sql
-- Risk: LOW (no destructive operations)
-- Strategy: Gradual Migration with Backward Compatibility

-- Step 1: Add new timestamp column (old column remains untouched)
ALTER TABLE public.bookings ADD COLUMN IF NOT EXISTS "handoverConfirmedAt" TIMESTAMPTZ;

-- Step 2: Migrate existing TRUE values to timestamps (optional - can be done gradually)
UPDATE public.bookings
SET "handoverConfirmedAt" = NOW()
WHERE "handoverConfirmed" = TRUE;

-- Step 3: Update application code gradually
-- Update API routes first, then components, then utilities
-- Keep old column as fallback during transition

-- Step 4: Final cleanup (only after all code updated and tested)
-- ALTER TABLE public.bookings DROP COLUMN "handoverConfirmed";
```

**Migration Timeline:**

- **Week 1**: Update API routes to use `handoverConfirmedAt`
- **Week 2**: Update React components
- **Week 3**: Update utility functions and tests
- **Week 4**: Drop old column after verification

**How CREATE TABLE AS SELECT works:**

- `CREATE TABLE table_name AS SELECT ...` creates a new table with columns matching the SELECT query
- The new table automatically gets the same column types as the source
- It's a single atomic operation - much faster than CREATE + INSERT
- Perfect for backups since it preserves exact data types and constraints

### Example 3: Major Schema Change (High Risk)

```sql
-- Risk: HIGH
-- Strategy: Full Migration Script
-- Requires: Business approval, maintenance window, full testing

-- Comprehensive migration with multiple backup points
-- Rollback procedures documented
-- Business impact assessment completed
```

---

## Future Improvements

### Tooling Enhancements

- Automated risk assessment scripts
- Migration template generators
- Shadow table management utilities

### Process Improvements

- Migration review checklists
- Automated validation scripts
- Better rollback automation

### Monitoring Improvements

- Migration success/failure tracking
- Performance impact monitoring
- Automated health checks post-migration

---

## Database Dump & Testing

### Database Dump Strategy

**ALWAYS create a full database dump before any migration:**

```bash
# Production dump (with compression)
pg_dump --host=$DB_HOST --username=$DB_USER --dbname=$DB_NAME \
  --format=custom --compress=9 --verbose \
  --file="backup_$(date +%Y%m%d_%H%M%S).dump"

# Alternative: Plain SQL format (more readable for review)
pg_dump --host=$DB_HOST --username=$DB_USER --dbname=$DB_NAME \
  --format=plain --no-owner --no-privileges \
  --file="backup_$(date +%Y%m%d_%H%M%S).sql"
```

**Dump Requirements:**

- ✅ **Full database dump** - not just schema, include all data
- ✅ **Compressed format** - for storage efficiency and faster restore
- ✅ **Timestamped filename** - clear versioning for multiple backups
- ✅ **Verified integrity** - test restore capability before proceeding
- ✅ **Off-server storage** - store in separate location from database server

**Verification Commands:**

```bash
# Check dump file integrity
pg_restore --list backup_20241201_143000.dump > /dev/null

# Test restore to temporary database
createdb temp_restore_test
pg_restore --dbname=temp_restore_test --verbose backup_20241201_143000.dump
# Verify data integrity, then drop temp database
dropdb temp_restore_test
```

### Migration Testing Strategy

**Three-Phase Testing Approach:**

#### Phase 1: Development Testing

```bash
# 1. Create isolated test database
createdb flexbike_migration_test

# 2. Restore from production backup
pg_restore --dbname=flexbike_migration_test backup.dump

# 3. Run migration scripts
psql flexbike_migration_test < migration_script.sql

# 4. Run application tests against migrated database
npm test -- --testPathPattern=migration
```

#### Phase 2: Staging Environment Testing

```bash
# 1. Deploy migration to staging
# 2. Run full application test suite
npm run test:e2e

# 3. Perform manual business logic testing
# - Create/update/delete operations
# - Complex queries and relationships
# - Performance under load

# 4. Verify rollback procedures work
```

#### Phase 3: Production Testing

```bash
# 1. Pre-migration validation
# Verify backup integrity
# Confirm monitoring systems active
# Notify stakeholders of maintenance window

# 2. Post-migration validation
# Run automated health checks
# Manual verification of critical business flows
# Performance monitoring for 24-48 hours

# 3. Rollback readiness
# Keep backup available for 7 days post-migration
# Document any issues encountered
# Update runbooks with lessons learned
```

**Critical Testing Checklist:**

- ✅ **Data Integrity**: Row counts, foreign key relationships, constraints
- ✅ **Application Functionality**: All CRUD operations work correctly
- ✅ **Performance**: Query performance meets SLAs
- ✅ **Business Logic**: Complex operations (transfers, calculations, workflows)
- ✅ **Edge Cases**: Null values, boundary conditions, error scenarios
- ✅ **Rollback**: Can successfully revert if issues discovered

**Automated Testing Requirements:**

```typescript
// Migration E2E Test - Real Prisma calls to validate migration success
describe("Migration: handoverConfirmed to handoverConfirmedAt", () => {
  it("should preserve existing boolean values as timestamps", async () => {
    // Query all bookings with timestamps (migrated from boolean)
    const bookingsWithTimestamps = await prisma.booking.findMany({
      where: { handoverConfirmedAt: { not: null } },
      select: {
        id: true,
        handoverConfirmedAt: true,
        status: true,
      },
    });

    // Verify migration created valid timestamps
    expect(bookingsWithTimestamps.length).toBeGreaterThan(0);
    bookingsWithTimestamps.forEach((booking) => {
      expect(booking.handoverConfirmedAt).toBeInstanceOf(Date);
      // Ensure timestamp is reasonable (not in far future/past)
      expect(booking.handoverConfirmedAt!.getTime()).toBeGreaterThan(0);
    });
  });

  it("should maintain referential integrity", async () => {
    // Test foreign key relationships still work
    const bookings = await prisma.bookings.findMany({
      include: {
        user: { select: { id: true, email: true } },
        product: { select: { id: true, name: true } },
      },
      take: 10, // Sample first 10 bookings
    });

    // Verify relationships are intact
    bookings.forEach((booking) => {
      expect(booking.user).toBeDefined();
      expect(booking.product).toBeDefined();
      expect(booking.user.email).toBeTruthy();
    });
  });

  it("should allow new timestamp-based operations", async () => {
    // Test that we can now query by timestamp ranges
    const recentHandovers = await prisma.bookings.findMany({
      where: {
        handOverConfirmed: true, // New timestamp field will be non-null
        // Note: Using old field name since migration adds new column
      },
      select: { id: true, handOverConfirmed: true },
    });

    // Should be able to filter by boolean values still
    expect(Array.isArray(recentHandovers)).toBe(true);
  });

  it("should validate complex business logic still works", async () => {
    // Test a complex query that combines multiple tables and conditions
    const activeBookings = await prisma.bookings.findMany({
      where: {
        status: "confirmed", // Using actual enum value from schema
        handOverConfirmed: true,
        startDate: { lte: new Date() },
        endDate: { gte: new Date() },
      },
      include: {
        user: { select: { email: true, firstName: true } },
        product: { select: { name: true } },
      },
      orderBy: { createdAt: "desc" },
    });

    // Verify complex multi-table queries work
    activeBookings.forEach((booking) => {
      expect(booking.status).toBe("confirmed");
      expect(booking.handOverConfirmed).toBe(true);
      expect(booking.user.email).toBeTruthy();
      expect(booking.product.name).toBeTruthy();
    });
  });
});
```

---

## Conclusion

This migration strategy has proven effective through multiple recovery scenarios. The conservative, manual approach prioritizes business continuity over development convenience.

**Critical Future Context:** As we transition away from Firebase, this strategy becomes even more vital. The ability to reset/reimport data will disappear, making migration success absolutely critical. Future agents must adhere strictly to these guidelines.

**Remember:**

- **Current:** Safety first, automation second
- **Future (Post-Firebase):** Safety ONLY, zero automation in production
- Manual control prevents automated disasters
- Every migration must succeed on first attempt

---

## API Development & Database Interactions

### Database Access Patterns

**Always use Prisma Client for database operations:**

```typescript
// ✅ Correct: Use Prisma with generated types
import { prisma } from "@/server/db/prisma";
import { UserCreateSchema } from "@/generated/zod";

export async function createUser(input: unknown) {
  // Validate input with Zod
  const data = UserCreateSchema.parse(input);

  // Use Prisma with type safety
  return await prisma.user.create({ data });
}
```

**Never use raw SQL in application code:**

```typescript
// ❌ Wrong: Raw SQL bypasses type safety
const user = await sql`SELECT * FROM users WHERE id = ${id}`;

// ❌ Wrong: Direct pg client usage
const client = new Client();
await client.query("SELECT * FROM users");
```

### API Route Structure

**Standard API pattern:**

```typescript
// api/users/route.ts
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/server/db/prisma";
import { UserCreateSchema, UserUpdateSchema } from "@/generated/zod";

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();

    // Validate input
    const data = UserCreateSchema.parse(body);

    // Database operation
    const user = await prisma.user.create({ data });

    // Validate output (optional but recommended)
    const validatedUser = UserSchema.parse(user);

    return NextResponse.json(validatedUser);
  } catch (error) {
    // Zod validation errors are automatically typed
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Validation failed", details: error.errors },
        { status: 400 }
      );
    }

    // Other errors
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

### Error Handling with Types

**Leverage generated error types:**

```typescript
import { PrismaClientKnownRequestError } from "@prisma/client/runtime/library";

export async function handleDatabaseOperation() {
  try {
    return await prisma.user.findUnique({ where: { id } });
  } catch (error) {
    if (error instanceof PrismaClientKnownRequestError) {
      switch (error.code) {
        case "P2025": // Not found
          throw new NotFoundError("User not found");
        case "P2002": // Unique constraint violation
          throw new ValidationError("User already exists");
        default:
          throw new DatabaseError("Database operation failed");
      }
    }
    throw error;
  }
}
```

### Transaction Patterns

**Use Prisma transactions for multi-step operations:**

```typescript
export async function transferBooking(
  fromUserId: string,
  toUserId: string,
  bookingId: string
) {
  return await prisma.$transaction(async (tx) => {
    // Validate booking exists and belongs to fromUser
    const booking = await tx.booking.findFirst({
      where: { id: bookingId, userId: fromUserId },
    });

    if (!booking) {
      throw new Error("Booking not found or access denied");
    }

    // Update booking ownership
    const updatedBooking = await tx.booking.update({
      where: { id: bookingId },
      data: { userId: toUserId },
    });

    // Log the transfer
    await tx.auditLog.create({
      data: {
        action: "BOOKING_TRANSFER",
        entityId: bookingId,
        fromUserId,
        toUserId,
      },
    });

    return updatedBooking;
  });
}
```

### Performance Considerations

**Use appropriate query patterns:**

```typescript
// ✅ Good: Select only needed fields
const users = await prisma.user.findMany({
  select: { id: true, name: true, email: true },
  where: { active: true },
});

// ✅ Good: Use includes for relations
const bookings = await prisma.booking.findMany({
  include: {
    user: { select: { name: true, email: true } },
    product: { select: { name: true, price: true } },
  },
});

// ❌ Bad: Over-fetching
const allUsers = await prisma.user.findMany(); // Gets ALL fields

// ❌ Bad: N+1 queries
const bookings = await prisma.booking.findMany();
for (const booking of bookings) {
  const user = await prisma.user.findUnique({ where: { id: booking.userId } });
}
```

---

## Developer Environment Setup

### Manual Migrations as Setup Scripts

**Save all manual migration SQL as setup scripts:**

```bash
# Create setup scripts directory
mkdir -p scripts/setup

# Save each migration as a timestamped SQL file
# scripts/setup/001_initial_schema.sql
# scripts/setup/002_add_user_roles.sql
# scripts/setup/003_add_booking_columns.sql
```

**New developer setup process:**

```bash
# 1. Fresh PostgreSQL database
createdb flexbike_dev

# 2. Run setup scripts in order
psql flexbike_dev < scripts/setup/001_initial_schema.sql
psql flexbike_dev < scripts/setup/002_add_user_roles.sql
psql flexbike_dev < scripts/setup/003_add_booking_columns.sql

# 3. Generate Prisma client and Zod schemas
npx prisma generate

# 4. Seed with test data (if needed)
npm run db:seed
```

### Automated Setup Script

**Create a master setup script:**

```bash
#!/bin/bash
# scripts/setup-database.sh

echo "Setting up Flexbike database..."

# Create database
createdb flexbike_dev

# Run all setup scripts in order
for script in scripts/setup/*.sql; do
  echo "Running $script..."
  psql flexbike_dev < "$script"
done

# Generate Prisma client and Zod schemas
npx prisma generate

echo "Database setup complete!"
```

---

## Type Safety & Validation

### Always Use Generated Types

**Import and use generated types everywhere:**

```typescript
// ✅ Correct: Use generated types
import type { User, Booking } from "@prisma/client";
import { UserCreateSchema, BookingUpdateSchema } from "@/generated/zod";

function processUser(user: User) {
  // Type-safe operations
}

export async function updateBooking(id: string, data: unknown) {
  const validatedData = BookingUpdateSchema.parse(data);
  return prisma.booking.update({ where: { id }, data: validatedData });
}
```

**Also use Zod types for function definitions:**

```typescript
import type { z } from "zod";

// Type function parameters with Zod inference
function createUser(data: z.infer<typeof UserCreateSchema>) {
  // data is fully typed and validated
}

export async function processBookingUpdate(
  bookingId: string,
  updateData: z.infer<typeof BookingUpdateSchema>
) {
  // Both parameters are fully typed
}
```

### Function Definitions with Zod Prisma Types

**Use generated Zod schemas to type function parameters and ensure type safety:**

```typescript
import {
  UserCreateSchema,
  BookingCreateSchema,
  UserUpdateSchema,
} from "@/generated/zod";
import type { z } from "zod";

// Type-safe function parameters using Zod inference
function createUser(data: z.infer<typeof UserCreateSchema>) {
  // data is fully typed: { email: string, firstName?: string, ... }
  return prisma.user.create({ data });
}

// Validate and type function parameters
export async function processBookingUpdate(
  bookingId: string,
  updateData: z.infer<typeof BookingUpdateSchema>
) {
  // updateData is fully validated and typed
  return prisma.booking.update({
    where: { id: bookingId },
    data: updateData,
  });
}

// Use in API route handlers
export async function POST(request: NextRequest) {
  const body = await request.json();

  // Type-safe validation and processing
  const userData = UserCreateSchema.parse(body);
  const user = await createUser(userData);

  return NextResponse.json(user);
}

// Complex business logic functions
export async function transferBookingOwnership(
  bookingId: string,
  newOwnerData: z.infer<typeof UserCreateSchema>,
  transferReason?: string
) {
  return await prisma.$transaction(async (tx) => {
    // Validate new owner data
    const validatedOwnerData = UserCreateSchema.parse(newOwnerData);

    // Create new owner if needed
    let newOwner = await tx.user.findUnique({
      where: { email: validatedOwnerData.email },
    });

    if (!newOwner) {
      newOwner = await tx.user.create({ data: validatedOwnerData });
    }

    // Transfer booking
    const updatedBooking = await tx.booking.update({
      where: { id: bookingId },
      data: { userId: newOwner.id },
    });

    // Log transfer with validated reason
    if (transferReason) {
      await tx.auditLog.create({
        data: {
          action: "BOOKING_TRANSFER",
          entityId: bookingId,
          details: transferReason,
        },
      });
    }

    return updatedBooking;
  });
}
```

### API Input/Output Validation

**Validate all external data:**

```typescript
// API routes should always validate input
export async function POST(request: NextRequest) {
  const body = await request.json();

  // Parse and validate input
  const bookingData = BookingCreateSchema.parse(body);

  // Type is now guaranteed correct
  const booking = await prisma.booking.create({ data: bookingData });

  // Validate output if needed
  return NextResponse.json(BookingSchema.parse(booking));
}
```

### Custom Validation Rules

**Add business logic validation:**

```typescript
import { z } from "zod";

export const BookingCreateSchema = z
  .object({
    userId: z.string(),
    productId: z.string(),
    startDate: z.date(),
    endDate: z.date(),
  })
  .refine((data) => data.endDate > data.startDate, {
    message: "End date must be after start date",
    path: ["endDate"],
  });
```

---

## Conclusion

This database management strategy integrates Prisma schema as the source of truth while maintaining our conservative, production-safe migration approach.

**Key Principles:**

- **Schema First:** Always update `schema.prisma` before database changes
- **Type Safety:** Use generated Prisma + Zod types everywhere (including function definitions)
- **Validation:** Zod validation for all inputs, outputs, and function parameters
- **Safe Migrations:** Manual review and conservative execution
- **API Standards:** Consistent patterns for database interactions

**Critical Future Context:** As we transition away from Firebase, this strategy becomes even more vital. The ability to reset/reimport data will disappear, making migration success absolutely critical.

**Remember:**

- **Schema is Truth** - All database structure, types, and validation come from `schema.prisma`
- **Safety First** - Manual control prevents automated disasters
- **Types Everywhere** - Generated types and validation in all APIs

**This document will be updated as we progress through the Firebase transition phases.** 🎯
