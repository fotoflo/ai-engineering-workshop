# Backup System Documentation

## Overview

Flexbike maintains comprehensive backups of both Firebase Firestore data and Supabase PostgreSQL databases. Backups are stored in the `/backups` directory and include automated scripts for creating, managing, and restoring backups.

## Backup Directory Structure

```
backups/
├── firebase-export-metadata.json          # Metadata for Firebase exports
├── firebase-export-{timestamp}/           # Firebase export directories
│   └── firestore_export/
│       ├── all_namespaces/
│       │   └── all_kinds/
│       │       ├── all_namespaces_all_kinds.export_metadata
│       │       ├── output-0                     # Sharded export files
│       │       ├── output-1
│       │       ├── output-2
│       │       └── output-3
│       └── firestore_export.overall_export_metadata
└── 2025-11-04T12:06:35_69092/             # Full Firebase exports
    ├── firebase-export-metadata.json
    ├── firestore_export/                   # Firestore data
    │   └── all_namespaces/all_kinds/
    │       ├── all_namespaces_all_kinds.export_metadata
    │       ├── output-0
    │       ├── output-1
    │       ├── output-2
    │       ├── output-3
    │       └── output-4
    └── storage_export/                     # Firebase Storage files
        ├── blobs/                          # Actual file data
        ├── buckets.json                    # Bucket metadata
        └── metadata/                       # File metadata
```

## Backup Types

### 1. Firebase Exports

**Purpose**: Complete backup of Firebase Firestore and Storage data

**Contents**:
- **Firestore Data**: All documents, collections, and subcollections
- **Storage Files**: All uploaded files and their metadata
- **Metadata**: Export timestamps, versions, and integrity information

**Naming Convention**: `firebase-export-{timestamp}/`

**Example Files**:
```
firebase-export-1764907993389umnUhl/
firebase-export-1766139936431oQh76a/
firebase-export-1766139959793XPktPP/
```

### 2. Full Firebase Backups

**Purpose**: Complete Firebase project backup including Storage

**Contents**:
- All Firestore data (same as above)
- All Firebase Storage files
- Bucket configurations
- File metadata and permissions

**Naming Convention**: `{YYYY-MM-DD}T{time}_{random}/`

**Example**: `2025-11-04T12:06:35_69092/`

### 3. Database Dumps

**Purpose**: PostgreSQL database backups

**Format**: PostgreSQL custom format dumps (.dump files)

**Created by**: `scripts/dump-database.js`

**Storage**: Also stored in `/backups` directory

## Backup Scripts

### Firebase Data Export

**Script**: `scripts/download-production-firebase.js`

**Purpose**: Download production Firebase data to emulator

**Usage**:
```bash
node scripts/download-production-firebase.js
```

**Features**:
- Downloads all production Firestore data
- Initializes local Firebase emulator
- Clears existing emulator data
- Imports production data for local testing

### Database Backup

**Script**: `scripts/dump-database.js`

**Purpose**: Create PostgreSQL database dumps

**Usage**:
```bash
node scripts/dump-database.js
```

**Interactive Prompts**:
- Database environment (dev/preview/prod)
- Environment file selection
- Automatic backup file naming

**Output**: `backups/database-dump-{env}-{timestamp}.dump`

### Automated Backup Commands

```bash
# Firebase export to backups directory
firebase emulators:export backups/firebase-export-$(date +%s)

# Database dump with environment selection
npm run db:dump

# Full backup (Firebase + Database)
npm run backup:all
```

## Restoration Procedures

### Firebase Data Restoration

**To Emulator**:
```bash
# Start emulator
firebase emulators:start --import=backups/firebase-export-1764907993389umnUhl

# Or import to running emulator
firebase emulators:export backups/firebase-export-1764907993389umnUhl
```

**To Production** (Emergency Only):
```bash
# This is a destructive operation - requires explicit confirmation
firebase firestore:delete --all-collections
firebase firestore:restore backups/firebase-export-1764907993389umnUhl
```

### Database Restoration

**Using pg_restore**:
```bash
# Restore to PostgreSQL database
pg_restore --host=localhost --port=5432 --username=postgres \
  --dbname=flexbike_dev --no-owner --clean \
  backups/database-dump-dev-2025-01-07T10-30-00.dump
```

**Using Prisma**:
```bash
# Reset and restore (development only)
npx prisma migrate reset --force
npx prisma db push
```

## Backup Scheduling

### Development Environment
- **Frequency**: Daily automated backups
- **Retention**: 7 days
- **Purpose**: Code changes and testing recovery

### Preview Environment
- **Frequency**: Before deployments
- **Retention**: 30 days
- **Purpose**: Deployment safety and rollback capability

### Production Environment
- **Frequency**: Hourly automated + daily full backup
- **Retention**: 90 days (with offsite copies)
- **Purpose**: Business continuity and disaster recovery

## Security Considerations

### Access Control
- **Backup files**: Encrypted at rest
- **Access**: Limited to infrastructure team
- **Audit logging**: All backup and restore operations logged

### Data Protection
- **Sensitive data**: Anonymized in development backups
- **PII**: Redacted from non-production environments
- **Encryption**: All backups encrypted with project keys

### Compliance
- **Retention policies**: Enforced automatically
- **Deletion**: Secure deletion of expired backups
- **Access reviews**: Quarterly access permission reviews

## Monitoring and Alerts

### Backup Health Checks
- **Success notifications**: Slack alerts for backup completion
- **Failure alerts**: Immediate alerts for backup failures
- **Size monitoring**: Alerts for unusual backup size changes
- **Integrity checks**: Automated verification of backup integrity

### Dashboard Metrics
- **Backup age**: Time since last successful backup
- **Success rate**: Percentage of successful backups
- **Restore testing**: Regular automated restore validation
- **Storage usage**: Backup storage consumption tracking

## Emergency Procedures

### Data Loss Recovery
1. **Assess scope**: Determine what data was lost
2. **Identify backup**: Find appropriate backup point
3. **Test restore**: Restore to staging environment first
4. **Execute restore**: Perform production restore with rollback plan
5. **Verify integrity**: Validate restored data
6. **Update stakeholders**: Communicate recovery status

### Rollback Plan
- **Time limit**: 1-hour window for production rollbacks
- **Verification**: Automated checks before rollback execution
- **Communication**: Stakeholder notification requirements
- **Documentation**: Post-mortem and lessons learned

## Best Practices

### Backup Verification
- **Test restores**: Monthly restore testing to staging
- **Integrity checks**: Automated checksum verification
- **Sample validation**: Spot-checks of backup contents
- **Performance testing**: Restore time and performance validation

### Storage Management
- **Compression**: All backups compressed to save space
- **Deduplication**: Avoid storing duplicate data
- **Lifecycle policies**: Automatic deletion of expired backups
- **Cost optimization**: Balance retention with storage costs

### Documentation Updates
- **Change tracking**: Update backup docs when processes change
- **Runbook maintenance**: Keep restoration procedures current
- **Team training**: Regular backup and restore training
- **Contact updates**: Maintain current emergency contact information

## Glob Patterns for Backup Files

The following glob patterns match all backup-related files:

```
# Firebase export metadata
backups/firebase-export-metadata.json

# Firebase export directories
backups/firebase-export-*/

# Full Firebase backups
backups/20*/firebase-export-metadata.json
backups/20*/firestore_export/**/*
backups/20*/storage_export/**/*

# Database dumps
backups/database-dump-*.dump
backups/database-dump-*.sql

# Backup scripts and utilities
scripts/dump-database.js
scripts/download-production-firebase.js
scripts/import/extract-emulator-data.js
scripts/import/export-new-records-for-restore.js
scripts/restore-database.js
scripts/import-export-data.js
```

## Related Documentation

- **Database Operations**: `docs/database/migration-guide.md`
- **Firebase Sync**: `docs/database/firebase-supabase-sync.md`
- **Deployment Safety**: `.cursor/commands/deploy-production.md`
- **Data Recovery**: `scripts/recovery/recover-booking-pitr.js`

---

**Last Updated**: January 2025
**Backup System**: Firebase + Supabase with automated procedures
**Retention Policy**: 7 days (dev), 30 days (preview), 90 days (prod)