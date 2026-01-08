# Company User Login with Supabase Auth - Implementation Plan

## Overview

Implement authentication system for company users (vendors) to login with phone/WhatsApp/OTP and Google, while maintaining existing Firebase data connections and admin functionality.

## Current State Analysis

- **Admin Auth**: Already implemented with Supabase + Google OAuth for @flexbike.app emails
- **Database**: Users table has `companyId`, `companyIds`, `phoneNumber`, `whatsappNumber`, `email` fields
- **Routing**: Admin routes exist at `/admin/companies/{companySlug}` and `/admin/users/{userId}/bookings`

## Requirements

1. Allow company users to login with phone/WhatsApp + OTP or Google
2. Connect existing Firebase users to Supabase auth when contact info matches
3. @flexbike.app users get godmode automatically
4. Company users redirect to `/admin/companies/{companySlug}` after login
5. Non-company users redirect to `/users/{userId}/bookings` after login

## Implementation Phases

### Phase 1: Core Authentication Infrastructure

#### 1.1 Create Company Auth Library (`src/lib/company-auth.ts`)

- `getCompanyUser()` - Get authenticated company user (server-side)
- `getCompanyUserClient()` - Get authenticated company user (client-side)
- `ensureCompanyUser()` - Create/update user in DB and connect to existing data
- `requireCompanyUser()` - Require company user authentication
- `isGodModeUser()` - Check if user should have godmode (flexbike.app emails)

**Key Logic:**

```typescript
// Check if contact info matches existing user
const existingUser = await prisma.users.findFirst({
  where: {
    OR: [
      { phoneNumber: authUser.phone },
      { whatsappNumber: authUser.phone },
      { email: authUser.email },
    ],
  },
});

// Connect auth user to existing company data
if (existingUser?.companyId || existingUser?.companyIds) {
  // Link auth user to existing company user
}
```

#### 1.2 Create Login Page (`src/app/login/page.tsx`)

- Support phone/WhatsApp input with country code picker
- Support Google OAuth
- Clean, professional UI matching existing admin login design
- Handle OTP verification flow

#### 1.3 Create Auth Callback Route (`src/app/auth/callback/route.ts`)

- Handle OAuth callbacks (Google)
- Handle OTP verification
- Ensure company user creation/connection
- Handle post-login redirects based on user type

#### 1.4 Update Supabase Auth Configuration

- Enable phone authentication in Supabase dashboard
- Configure WhatsApp OTP via WATI integration
- Set up Google OAuth
- Configure auth hooks for user creation

### Phase 2: User Data Connection & Routing

#### 2.1 Create User Connection Logic

- **ID Strategy**: Current `user.id` is already the Firebase auth ID - no additional field needed
- **Contact Matching**: When user logs in with phone/email, find existing user by contact info
- **Firebase ID Preservation**: Existing `user.id` fields are Firebase auth IDs - maintain as business primary key
- **Auth Linking**: Connect Supabase auth user to existing company data without changing database user IDs
- **Company Resolution**: Handle single companyId vs multiple companyIds arrays
- **Godmode Assignment**: Auto-assign godmode to flexbike.app emails
- **Database Updates**: Track auth connections via `updatedBySource` field ('firebase' → 'supabase')

**Firebase ↔ Supabase ID Resolution:**

```typescript
// When Supabase user logs in, find existing user by contact info
// NEVER change the database user.id - it's already the Firebase auth ID (source of truth)
const existingUser = await prisma.users.findFirst({
  where: {
    OR: [
      { phoneNumber: supabaseUser.phone },
      { whatsappNumber: supabaseUser.phone },
      { email: supabaseUser.email },
    ],
  },
});

// If found, mark as connected to Supabase (user.id is already the Firebase ID)
if (existingUser) {
  await prisma.users.update({
    where: { id: existingUser.id },
    data: {
      updatedBySource: "supabase",
      updatedAt: new Date(),
    },
  });
  // The existing user.id (Firebase auth ID) remains the business primary key
  // Supabase auth user now has access to this business data
}
```

#### 2.2 Create Post-Login Redirect Logic

```typescript
function getPostLoginRedirect(user: CompanyUser) {
  if (user.godMode) {
    return "/admin/calendar"; // Godmode users go to admin
  }

  if (user.companyId) {
    return `/admin/companies/${user.companySlug}`;
  }

  return `/users/${user.id}/bookings`;
}
```

#### 2.3 Create User Bookings Page (`src/app/users/[userId]/bookings/page.tsx`)

- Display list of user's bookings
- Basic booking management (view, contact host, etc.)
- Match existing admin booking table design
- Ensure user can only see their own bookings

### Phase 3: Authentication Middleware & Protection

#### 3.1 Create Auth Middleware (`src/middleware.ts`)

- Protect admin routes (existing)
- Protect company routes (`/admin/companies/*`)
- Protect user routes (`/users/*`)
- Handle redirects for unauthenticated users

#### 3.2 Update Admin Layout (`src/app/admin/layout.tsx`)

- Add company user support alongside admin users
- Update navigation based on user type
- Show different menus for company users vs admins

#### 3.3 Create Company User Layout (`src/app/admin/companies/[companySlug]/layout.tsx`)

- Existing company layout can be reused
- Ensure proper authentication checks

### Phase 4: API Routes & Data Access

#### 4.1 Create Company User API Routes

- `GET /api/auth/company-user` - Get current company user
- `POST /api/auth/link-existing-user` - Link auth user to existing data
- `GET /api/users/[userId]/bookings` - Get user's bookings

#### 4.2 Update Existing Admin APIs

- Ensure company user permissions work alongside admin permissions
- Add company user checks where needed

#### 4.3 Create User Permission Helpers

```typescript
export function canAccessCompany(user: CompanyUser, companyId: string) {
  if (user.godMode) return true;
  return user.companyId === companyId || user.companyIds?.includes(companyId);
}

export function canAccessBooking(user: CompanyUser, booking: Booking) {
  if (user.godMode) return true;
  return booking.userId === user.id || booking.companyId === user.companyId;
}
```

### Phase 5: UI/UX Implementation

#### 5.1 Phone/WhatsApp Login Component

- Country code selector
- Phone number input with validation
- WhatsApp OTP delivery (primary method)
- OTP input component
- Resend OTP functionality via WhatsApp

#### 5.2 Google Login Integration

- Reusable Google OAuth button
- Handle OAuth flow for company users
- Error handling and user feedback

#### 5.3 User Dashboard (`src/app/users/[userId]/page.tsx`)

- Simple user profile/overview
- Link to bookings
- Basic account settings

#### 5.4 Mobile Responsiveness

- Ensure login flows work on mobile
- Test OTP WhatsApp delivery via WATI
- Optimize user booking list for mobile

### Phase 6: Testing & Migration

#### 6.1 Testing Strategy

- Unit tests for auth logic
- Integration tests for user connection
- E2E tests for login flows
- Test existing admin functionality still works

#### 6.2 Data Migration Verification

- Test that existing Firebase users connect properly
- Verify company associations work
- Check booking access permissions

#### 6.3 Rollout Plan

- Deploy behind feature flag initially
- Test with select company users
- Monitor error rates and user feedback
- Gradual rollout to all company users

## Database Schema Considerations

- Users table already supports multiple companies via `companyIds` JSON field
- Existing `companyId` field for single company relationships
- `updatedBySource` field tracks auth system ('firebase' | 'supabase')
- **Current `user.id` is already the Firebase auth ID** - no additional firebaseId field needed
- No schema changes needed - existing user.id serves as both auth ID and business primary key

## Security Considerations

- Phone number verification prevents unauthorized access
- Company data isolation (users only see their companies/bookings)
- Admin routes properly protected
- Rate limiting on OTP requests
- Secure token handling

## Success Metrics

- Company user login success rate >95%
- Existing Firebase users successfully connected >98%
- Admin functionality unaffected
- User booking access works correctly
- Mobile login experience satisfactory

## Dependencies

- Supabase Auth (already configured)
- WhatsApp API via WATI for OTP delivery (needs setup)
- Google OAuth (already configured for admin)
- Existing Prisma database schema
