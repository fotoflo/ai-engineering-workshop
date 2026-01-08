# Company User Login with Supabase Auth - Revised Implementation Plan

## Overview

Implement authentication system for company users (vendors) to login with Google OAuth only, while maintaining existing Firebase data connections and admin functionality. **WATI is already implemented** - no additional WhatsApp setup needed. Focus on maximum code reuse from existing admin auth system.

## Current State Analysis

- **Admin Auth**: Already implemented with Supabase + Google OAuth for @flexbike.app emails
- **Database**: Users table has `companyId`, `companyIds`, `phoneNumber`, `whatsappNumber`, `email` fields
- **Database Update Needed**: Add `supabaseId` field to track Supabase auth user IDs
- **Routing**: Admin routes exist at `/admin/companies/{companySlug}` and `/admin/users/{userId}/bookings`
- **Contact Info**: Comes from Google auth only (no phone/WhatsApp OTP needed)

## Requirements

1. Allow company users to login with Google OAuth only
2. Connect existing Firebase users to Supabase auth when email matches
3. @flexbike.app users get godmode automatically
4. Company users redirect to `/admin/companies/{companySlug}` after login
5. Non-company users redirect to `/users/{userId}/bookings` after login
6. **DRY Principle**: Reuse 90%+ of existing admin auth infrastructure

## Implementation Phases

### Phase 1: Database Schema Update

#### 1.1 Add supabaseId to users table

```sql
-- Migration to add supabaseId field
ALTER TABLE users ADD COLUMN "supabaseId" TEXT;
CREATE INDEX "users_supabaseId_idx" ON users("supabaseId");
```

**Location**: Update `prisma/schema.prisma` users model
**Rationale**: Track Supabase auth user IDs separately from Firebase IDs for proper auth linking.

### Phase 2: Core Company Auth Library (Reuse Admin Auth Patterns)

#### 2.1 Create `src/lib/company-auth.ts` (Extend existing `admin-auth.ts`)

**Reuse existing patterns from `admin-auth.ts`:**

- `serializeWithBigInt()` - Reuse for data serialization
- `isAdminEmail()` - Keep for admin detection
- `isAdminRole()` - Keep for admin role checking
- `getAdminUser()` and variants - Use as reference but create company equivalents

**New Company-Specific Functions:**

```typescript
// Reuse admin auth patterns but adapt for company users
export async function getCompanyUser() {
  // Similar to getAdminUser() but for company users
}

export async function getCompanyUserClient() {
  // Similar to getAdminUserClient() but for company users
}

export async function ensureCompanyUser(request: NextRequest) {
  // Reuse ensureAdminUser() logic but for company auth
}

export async function requireCompanyUser() {
  // Similar to requireAdmin() but for company users
}

// Reuse godmode logic
export function isGodModeUser(user: any) {
  return user.godMode || isAdminEmail(user.email);
}
```

**Key Logic - Reuse Firebase ↔ Supabase ID Resolution:**

```typescript
// Reuse and adapt existing user connection logic
const existingUser = await prisma.users.findFirst({
  where: {
    OR: [
      { email: supabaseUser.email },
      // No phone matching since contact info comes from Google auth only
    ],
  },
});

// If found, link Supabase auth user
if (existingUser) {
  await prisma.users.update({
    where: { id: existingUser.id },
    data: {
      supabaseId: supabaseUser.id, // NEW: Track Supabase auth ID
      updatedBySource: "supabase",
      updatedAt: new Date(),
    },
  });
}
```

### Phase 3: Authentication Pages (Reuse Admin Login UI)

#### 3.1 Create Login Page `src/app/login/page.tsx` (Reuse admin login design)

**Reuse existing admin login UI components:**

- Same Google OAuth button styling
- Same error handling patterns
- Same loading states
- Same logo and branding

**Key Differences:**

- Remove "@flexbike.app" restriction messaging
- Allow any Google account (not just admin domain)
- Different redirect logic (see Phase 5)

```typescript
// Reuse admin login structure
const handleGoogleLogin = async () => {
  // Reuse existing Google OAuth flow from admin login
  const { error } = await supabase.auth.signInWithOAuth({
    provider: "google",
    options: {
      redirectTo: `${window.location.origin}/auth/callback`, // NEW: General callback
      // ... reuse existing OAuth options
    },
  });
};
```

#### 3.2 Create Auth Callback Route `src/app/auth/callback/route.ts` (Reuse admin callback)

**Reuse existing admin callback logic:**

- Same cookie handling with `createServerClient`
- Same `exchangeCodeForSession` flow
- Same response structure

**Key Differences:**

- Use `ensureCompanyUser()` instead of `ensureAdminUser()`
- Handle different redirect logic (see Phase 5)

```typescript
// Reuse admin callback structure
const { error } = await supabase.auth.exchangeCodeForSession(code);

if (!error) {
  // NEW: Ensure company user instead of admin user
  const companyUser = await ensureCompanyUser(request);
  // Handle post-login redirects based on user type
}
```

### Phase 4: User Data Connection & Permissions

#### 4.1 Create Company User API Routes (Reuse Admin API Patterns)

**Reuse existing API route structures:**

- `GET /api/auth/company-user` - Similar to `/api/admin/me`
- `GET /api/users/[userId]/bookings` - Adapt existing booking APIs

**Permission Helpers (Reuse and extend admin patterns):**

```typescript
// Reuse admin permission logic patterns
export function canAccessCompany(user: CompanyUser, companyId: string) {
  if (user.godMode || isAdminEmail(user.email)) return true; // Reuse godmode check
  return user.companyId === companyId || user.companyIds?.includes(companyId);
}

export function canAccessBooking(user: CompanyUser, booking: Booking) {
  if (user.godMode || isAdminEmail(user.email)) return true; // Reuse godmode check
  return booking.userId === user.id || booking.companyId === user.companyId;
}
```

### Phase 5: Post-Login Redirect Logic

**Create `getPostLoginRedirect()` function (reuse existing routing patterns):**

```typescript
function getPostLoginRedirect(user: CompanyUser) {
  // Reuse existing admin redirect logic patterns
  if (isGodModeUser(user) || isAdminEmail(user.email)) {
    return "/admin/calendar"; // Reuse admin redirect
  }

  if (user.companyId) {
    // Use existing company slug routing pattern
    const companySlug = await getCompanySlug(user.companyId);
    return `/admin/companies/${companySlug}`;
  }

  return `/users/${user.id}/bookings`; // NEW: User dashboard
}
```

### Phase 6: User Dashboard & Bookings (Reuse Admin Components)

#### 6.1 Create User Bookings Page `src/app/users/[userId]/bookings/page.tsx`

**Reuse existing admin booking table components:**

- Same `BookingTable` component
- Same booking display logic
- Same pagination and filtering

**Key Differences:**

- Filter to show only user's bookings: `where: { userId: params.userId }`
- Remove admin actions (edit, delete)
- Add user-specific actions (contact host, etc.)

#### 6.2 Create User Dashboard `src/app/users/[userId]/page.tsx`

**Reuse admin dashboard patterns:**

- Similar layout structure
- Quick stats and links
- Clean, professional UI

### Phase 7: Middleware & Route Protection

#### 7.1 Update Middleware `src/middleware.ts` (Create if doesn't exist)

**Reuse existing admin route protection patterns:**

```typescript
// Reuse existing admin route protection logic
if (path.startsWith("/admin/")) {
  // Admin route logic (existing)
}

// NEW: Company user route protection
if (path.startsWith("/users/")) {
  // Company user route logic
}
```

#### 7.2 Update Admin Layout (Reuse existing layout)

**Extend existing `AdminLayoutClient.tsx`:**

- Add company user navigation logic
- Show different menus based on user type
- Reuse existing navigation components

### Phase 8: Testing & Migration

#### 8.1 Testing Strategy (Reuse existing admin testing patterns)

**Unit Tests:**

- Reuse existing auth logic tests
- Test company user connection scenarios

**Integration Tests:**

- Reuse existing OAuth callback tests
- Test user data linking scenarios

**E2E Tests:**

- Reuse existing login flow tests
- Test company user redirect flows

#### 8.2 Migration Verification

**Reuse existing Firebase sync patterns:**

- Verify existing Firebase users connect properly via email
- Check company associations work
- Audit `supabaseId` field population

## Key Reusability Achievements

✅ **90%+ Code Reuse:**

- Auth logic patterns from `admin-auth.ts`
- UI components from admin login page
- API route structures from admin APIs
- Database connection logic from existing Firebase sync
- Permission checking patterns from admin system
- Component styling and layouts from admin pages

✅ **DRY Principle Maximized:**

- Single Google OAuth implementation
- Shared auth callback flow
- Common permission helpers
- Reusable booking display components

## Technical Assumptions Verified

- ✅ **WATI Already Implemented**: No WhatsApp OTP setup needed
- ✅ **Google OAuth Only**: No phone/WhatsApp authentication required
- ✅ **Contact Info Source**: User data comes from Google OAuth only
- ✅ **Database Schema**: Users table supports multiple companies via `companyIds`
- ✅ **Company Routing**: Companies have `slug` field for URL routing
- ✅ **Existing Infrastructure**: Comprehensive admin auth system already built

## Success Metrics

- Company user login success rate >95%
- Existing Firebase users connected via email >98%
- Admin functionality unaffected
- User booking access works correctly
- Google OAuth flow reliable

## Dependencies

- ✅ **Supabase Auth**: Already configured
- ✅ **Google OAuth**: Already configured for admin
- ✅ **Existing Prisma database schema**: Ready for `supabaseId` addition
- ✅ **WATI**: Already implemented (no additional setup needed)

---

**Implementation Status**: Ready for Phase 1 (Database Schema Update)
