# Flexbike Project Architecture & Structure

## Overview

Flexbike is a modern Next.js-based motorcycle rental marketplace application that enables users to browse and book motorcycles from trusted rental shops. The application has been recently refactored from a Firebase-only architecture to a hybrid Firebase + Supabase system with real-time bidirectional data synchronization.

## Technology Stack

### Frontend
- **Next.js 15** with App Router
- **React 18** with TypeScript
- **Tailwind CSS** for styling
- **shadcn/ui** component library

### Backend & Data
- **Supabase** (PostgreSQL) as primary database
- **Firebase Firestore** for offline bookings and real-time features
- **Prisma ORM** for database operations
- **Firebase Cloud Functions** for data synchronization

### Development & Deployment
- **pnpm** for package management
- **ESLint** for code linting
- **Jest** for testing
- **Playwright** for E2E testing

## Project Structure

### Root Level Directories

```
/Users/fotoflo/dev/flexbike/flexbike-next/
├── src/                          # Main application code
├── prisma/                       # Database schema and migrations
├── flexbike-functions/           # Firebase Cloud Functions
├── agents/docs/                  # Documentation
├── scripts/                      # Database and utility scripts
├── tests/                        # Test suites
├── public/                       # Static assets
├── firebase-export-*/            # Firebase data exports (untracked)
├── *.js                          # Utility and migration scripts
└── *.md                          # Documentation files
```

### Source Code Structure (`src/`)

#### App Router Structure (`src/app/`)
```
src/app/
├── [...slug]/                    # Catch-all dynamic routes
├── actions/                      # Server actions
├── admin/                        # Admin interface (52 files)
│   ├── calendar/                 # Admin calendar views
│   ├── companies/                # Company management
│   ├── components/               # Admin-specific components
│   ├── users/                    # User management
│   └── firebase-sync/            # Data synchronization UI
├── api/                          # API routes (128 endpoints)
│   ├── admin/                    # Admin-only endpoints
│   ├── auth/                     # Authentication
│   ├── bookings/                 # Booking operations
│   ├── companies/                 # Company data
│   ├── search/                   # Search functionality
│   └── sync/                     # Data synchronization
├── components/                   # Reusable UI components (87 files)
├── hooks/                        # React hooks (31 files)
├── utils/                        # Utility functions
└── providers/                    # React context providers
```

#### Library Code (`src/lib/`)
```
src/lib/
├── transforms/                   # Data transformation logic (9 files)
├── firebase/                     # Firebase utilities
├── stripe.ts                     # Payment integration
├── supabaseClient.ts             # Supabase client
├── bookingService.ts             # Booking business logic
├── dataTransformers.ts           # Legacy transformers
└── utils.ts                      # General utilities
```

#### Server-Side Code (`src/server/`)
```
src/server/
├── db/                           # Database utilities
├── geo/                          # Geolocation services
├── services/                     # Business logic services
└── supabase.ts                   # Server-side Supabase client
```

## Key Components & Their Purposes

### Admin Interface (`src/app/admin/`)
The admin interface provides comprehensive management tools:

- **Calendar Management**: Visual booking calendar with drag-and-drop
- **Company Management**: CRUD operations for rental companies
- **User Management**: User administration and booking history
- **Data Synchronization**: Manual sync controls and rollback tools

### API Routes (`src/app/api/`)
128 API endpoints organized by functionality:

- **Admin APIs**: Company, booking, and user management
- **Public APIs**: Search, booking creation, company data
- **Sync APIs**: Firebase ↔ Supabase synchronization
- **Utility APIs**: Sitemaps, health checks, webhooks

### Data Transformation Layer (`src/lib/transforms/`)
Critical bidirectional data transformers:

- `bookings.ts` (1188 lines): Complex booking data transformation
- `companies.ts`: Company data mapping
- `products.ts`: Motorcycle/product data
- `users.ts`: User profile transformation
- `conversations.ts`: Chat/message data
- `utils.ts`: Shared transformation utilities

### Firebase Functions (`flexbike-functions/`)
Cloud Functions for real-time synchronization:

- `syncToPostgres.js`: Firebase → Supabase sync
- `syncToFirebase.js`: Supabase → Firebase sync
- Function triggers on Firestore document changes

## Data Flow Architecture

### Bidirectional Synchronization System

```
Firebase Firestore ←→ Firebase Cloud Functions ←→ Supabase PostgreSQL
     ↑                                                       ↑
   Offline Bookings                                      Admin UI
   Mobile Apps                                         Web Interface
```

### Booking Creation Flow

1. **Online Bookings**: Next.js API → Supabase (direct)
2. **Offline Bookings**: Firebase SDK → Firestore → Cloud Function → Next.js Webhook → Supabase

### Data Synchronization

- **Real-time Sync**: Cloud Functions trigger on Firestore changes
- **Batch Sync**: Admin interface for bulk operations
- **Conflict Resolution**: Metadata prevents sync loops
- **Rollback Support**: Admin tools for data recovery

## Database Schema (Prisma)

### Core Entities
- **Companies**: Rental shop information
- **Products**: Available motorcycles/scooters
- **Bookings**: Rental reservations
- **Users**: Customer profiles
- **Conversations**: Chat between users and companies
- **Messages**: Individual chat messages
- **Reviews**: Customer ratings and feedback

### Recent Schema Changes
- Made `userId` nullable in bookings (migration: `make_userid_nullable_in_bookings`)
- Added Stripe customer ID support
- Added company ID and god mode flags
- Enhanced booking status tracking

## Testing Structure

### Unit Tests (`src/__tests__/`)
- Component tests with React Testing Library
- Hook tests for data layer logic
- Utility function tests

### Integration Tests (`src/__tests__/integration/`)
- API endpoint testing
- Data transformation verification
- End-to-end booking flows

### E2E Tests (`e2e/`)
- Playwright-based browser automation
- Critical user journey testing

## Development Scripts & Tools

### Database Operations
- `npm run import:setup`: Complete environment setup
- `npm run import:full`: Bulk data import from Firebase
- `scripts/`: Individual migration and utility scripts

### Firebase Operations
- `pnpm emu`: Start Firebase emulator
- `firebase-export-*/`: Data export snapshots

### Testing & Quality
- `npm test`: Run Jest test suite
- `npm run test:e2e`: Run Playwright tests
- ESLint configuration for code quality

## Recent Changes (Working Tree)

### Modified Files (Core Functionality)
- **Admin Interface**: Calendar, company management, booking tables
- **API Routes**: Booking operations, company data, admin endpoints
- **Data Transforms**: Booking transformation logic, utilities
- **Tests**: Transform verification, booking logic

### New Components
- `CompanyBookingsTable.tsx`: Enhanced booking display
- `BookingCreateForm.tsx`: New booking creation interface
- `CompanyBookingsTab.tsx`: Company-specific booking management

### Database Changes
- Schema updates for nullable user IDs
- Migration scripts for data integrity
- Seed data updates

### Firebase Integration
- Offline booking test scripts
- Synchronization improvements
- Webhook endpoint enhancements

## Deployment & Environment

### Environment Variables
- `DATABASE_URL`: Supabase connection string
- `FIREBASE_*`: Firebase configuration
- `STRIPE_*`: Payment processing
- `SYNC_API_KEY`: Synchronization security

### Build Process
- Next.js static generation for performance
- Prisma client generation
- Tailwind CSS compilation
- Asset optimization

## Security & Best Practices

### Data Protection
- Row Level Security (RLS) in Supabase
- Firebase security rules
- API key authentication for sync operations

### Code Quality
- TypeScript for type safety
- ESLint for code consistency
- Pre-commit hooks for quality checks

---

**Last Updated:** December 2025
**Architecture Status:** ✅ **HYBRID FIREBASE + SUPABASE SYSTEM**

