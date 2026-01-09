# Database Documentation

Welcome to Flexbike's comprehensive database documentation. This section covers everything from schema design and migrations to API patterns and real-time synchronization.

## 📋 Overview

Flexbike uses **Supabase PostgreSQL** as the primary database with **Prisma ORM** for type-safe database operations. The system maintains **zero data loss** through careful migration strategies and **bidirectional synchronization** with Firebase for legacy compatibility.

### Key Components

| Component             | Purpose                             | Technology                    |
| --------------------- | ----------------------------------- | ----------------------------- |
| **Primary Database**  | Main data storage                   | Supabase PostgreSQL           |
| **Schema Management** | Type-safe database operations       | Prisma ORM                    |
| **Real-time Sync**    | Firebase ↔ Supabase synchronization | Cloud Functions               |
| **Data Validation**   | Runtime type checking               | Zod schemas                   |
| **Migration Safety**  | Production-safe schema changes      | Manual + automated strategies |

### Architecture

```
Firebase Firestore (Legacy)
    ↕️ Real-time sync
Supabase PostgreSQL (Primary)
    ↕️ Prisma Client
Application Code (TypeScript)
    ↕️ Zod Validation
API Endpoints
```

## 🚀 Quick Start

### Development Setup

```bash
# 1. Install dependencies
npm install

# 2. Set up environment variables
vercel env pull
# should pull the correct .env.local, database running on localhost

# 3. Generate Prisma client and types
npx prisma generate

# 4. Run database migrations
npx prisma db push

# 5. Seed initial data (optional)
npm run db:seed
```

### Basic Database Operations

```typescript
import { prisma } from "@/server/db/prisma";

// Create a booking
const booking = await prisma.booking.create({
  data: {
    userId: "user-123",
    productId: "product-456",
    startDate: new Date("2024-01-01"),
    endDate: new Date("2024-01-05"),
  },
});

// Query with relations
const bookings = await prisma.booking.findMany({
  include: {
    user: true,
    product: true,
  },
});
```

## 📚 Documentation Sections

### Schema & Types

- **[`schema.md`](schema.md)** - Complete schema reference with all models, fields, and relationships
- **[`query-patterns.md`](query-patterns.md)** - Common query patterns and optimization techniques

### Migrations & Safety

- **[`migration-guide.md`](migration-guide.md)** - Safe migration strategies and production deployment
- **`.cursor/rules/database-safety/RULE.md`** - Database operation standards and guardrails

### Synchronization

- **[`firebase-supabase-sync.md`](firebase-supabase-sync.md)** - Real-time sync system documentation
- **Admin Interface** - Manual sync operations at `/admin/firebase-sync`

### API Integration

- **[`../api/README.md`](../api/README.md)** - API patterns and database interactions
- **Type Safety** - Zod validation and Prisma integration

## 🔑 Key Principles

### 1. Zero Data Loss Guarantee

- All migrations must preserve existing data
- Backup strategies for any destructive operations
- Recovery procedures documented and tested

### 2. Type Safety First

- Full TypeScript coverage for database operations
- Auto-generated types from Prisma schema
- Runtime validation with Zod schemas

### 3. Manual Control for Production

- Human review required for all production migrations
- Clear audit trail of changes
- Rollback procedures always available

### 4. Firebase Compatibility

- Bidirectional sync maintains legacy compatibility
- Gradual transition away from Firebase
- Data integrity across both systems

## 🛠️ Common Tasks

### Adding a New Feature

```bash
# 1. Update schema.prisma
# 2. Run validation and type generation
npx prisma validate && npx prisma generate

# 3. Update application code with new types
# 4. Test changes locally
npm run test

# 5. Create migration for production
npx prisma migrate dev --create-only
```

### Database Migration

```bash
# 1. Review migration SQL carefully
# 2. Backup production database
# 3. Run migration in staging first
npm run db:deploy:staging

# 4. Deploy to production
npm run db:deploy
```

### Data Synchronization

```bash
# Check sync status
curl http://localhost:3000/api/admin/firebase-sync/status

# Manual sync operation
curl -X POST http://localhost:3000/api/admin/firebase-sync/sync

# Access admin interface
open http://localhost:3000/admin/firebase-sync
```

## 📊 Database Statistics

### Current Schema

- **19,743 records** total (including location data)
- **495 companies** with rental locations
- **1,897 products** (motorcycles/scooters)
- **4,417 bookings** and reservations
- **1,645 conversations** between users
- **9,608 messages** in conversations
- **184 reviews** and ratings

### Performance Metrics

- **Average query time**: < 50ms for common operations
- **Connection pooling**: Enabled via Supabase
- **Indexing strategy**: Foreign keys and commonly filtered fields indexed
- **Caching layer**: Redis for session and temporary data

## 🔍 Monitoring & Health Checks

### Database Health

```bash
# Check connection
npx prisma db execute --file scripts/health-check.sql

# View active connections
npx prisma db execute --query "SELECT * FROM pg_stat_activity;"

# Monitor slow queries
npx prisma db execute --query "SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

### Sync Health

```bash
# Check Firebase sync status
curl http://localhost:3000/api/admin/firebase-sync/health

# View recent sync operations
curl http://localhost:3000/api/admin/firebase-sync/logs

# Manual sync trigger
curl -X POST http://localhost:3000/api/admin/firebase-sync/manual-sync
```

## 🚨 Emergency Procedures

### Data Loss Incident

1. **Stop all write operations** immediately
2. **Assess scope of data loss**
3. **Restore from backup** if available
4. **Contact engineering team** for recovery
5. **Document incident** for post-mortem

### Sync Failure

1. **Check Firebase connectivity**
2. **Verify Cloud Function logs**
3. **Review recent schema changes**
4. **Use admin interface** for manual intervention
5. **Monitor for automatic recovery**

### Performance Degradation

1. **Check query performance** with `EXPLAIN ANALYZE`
2. **Review recent migrations** for indexing issues
3. **Monitor connection pool usage**
4. **Scale database resources** if needed
5. **Optimize slow queries**

## 📈 Future Roadmap

### Short Term (3 months)

- Complete Firebase transition
- Implement advanced caching strategies
- Add automated backup verification

### Medium Term (6 months)

- Database sharding for scale
- Advanced analytics and reporting
- Real-time notification system

### Long Term (1 year)

- Multi-region database deployment
- Advanced query optimization
- Machine learning integration

## 🔗 Related Documentation

### Development Workflow

- **[`.cursor/rules/git-commits/RULE.md`](../.cursor/rules/git-commits/RULE.md)** - Commit standards for database changes
- **[`.cursor/rules/testing/RULE.md`](../.cursor/rules/testing/RULE.md)** - Database testing patterns
- **[`../guides/testing.md`](../guides/testing.md)** - Testing database operations

### API Development

- **[`../api/README.md`](../api/README.md)** - API patterns using database
- **[`../api/search.md`](../api/search.md)** - Search API with database queries
- **[`../api/authentication.md`](../api/authentication.md)** - Auth patterns with database

### Deployment & Operations

- **[`.cursor/commands/create-migration.md`](../.cursor/commands/create-migration.md)** - Migration workflow
- **[`.cursor/commands/deploy-production.md`](../.cursor/commands/deploy-production.md)** - Production deployment
- **[`.cursor/commands/firebase-sync.md`](../.cursor/commands/firebase-sync.md)** - Sync operations

### Architecture

- **[`../architecture/overview.md`](../architecture/overview.md)** - System architecture
- **[`../architecture/data-flow.md`](../architecture/data-flow.md)** - Data flow patterns
- **[`../architecture/routing.md`](../architecture/routing.md)** - API routing with database

---

**Need Help?**

- Check **[migration troubleshooting](migration-guide.md#troubleshooting)** for common issues
- Review **[API examples](../api/examples.md)** for database usage patterns
- Access **[admin interface](../../src/app/admin/firebase-sync/)** for sync operations
