# Product Requirements Document: Admin Calendar View

## 1. Overview

Create an authenticated administrative calendar interface that provides a centralized, real-time view of all bookings across all shops in the Flexbike platform, with filtering capabilities and timezone support.

---

## 2. Goals & Objectives

- **Primary Goal**: Provide operations team with a unified, real-time view of all bookings
- **Secondary Goals**:
  - Enable quick filtering by shop/company
  - Maintain secure access with proper authentication
  - Display bookings in user's local timezone
  - Leverage existing real-time WebSocket infrastructure
  - Create foundation for future admin tools

---

## 3. User Stories

1. **As an operations manager**, I want to see all bookings in a calendar view so I can understand overall platform utilization
2. **As an operations manager**, I want to filter by shop so I can focus on specific company performance
3. **As an operations manager**, I want to authenticate securely so unauthorized users cannot access sensitive booking data
4. **As an operations manager**, I want to see key booking details (dates, customer, bike, status) at a glance
5. **As an operations manager**, I want to see bookings update in real-time when changes occur
6. **As an operations manager working across timezones**, I want to view bookings in my local time or switch timezones

---

## 4. Functional Requirements

### 4.1 Authentication

- **REQ-1**: Use **Supabase Auth** with email/password for secure authentication
- **REQ-2**: Support single admin user for MVP (expandable to multiple users)
- **REQ-3**: Session managed by Supabase with automatic token refresh
- **REQ-4**: Failed authentication attempts should show clear error message
- **REQ-5**: Route protection via Next.js middleware
- **REQ-6**: Optional: `ADMIN_PASSWORD` env var as development bypass or initial setup password

### 4.2 Calendar Display

- **REQ-7**: Use **react-big-calendar** library for calendar UI
- **REQ-8**: Display all bookings in calendar format (month view as default)
- **REQ-9**: Show key booking information:
  - Booking ID
  - Customer name/contact
  - Company/shop name
  - Bike model/name
  - Booking status (pending, confirmed, completed, cancelled, declined)
  - Start and end dates
  - Total price
- **REQ-10**: Color-code bookings by status for quick visual scanning
- **REQ-11**: Handle multi-day bookings appropriately (span across days)
- **REQ-12**: **Real-time updates** - bookings update automatically when changes occur via WebSocket

### 4.3 Filtering & Search

- **REQ-13**: Provide dropdown to filter bookings by company/shop
- **REQ-14**: "All Companies" option to show unfiltered view
- **REQ-15**: Filter state should persist during session
- **REQ-16**: Display count of visible bookings

### 4.4 Navigation & Views

- **REQ-17**: Allow navigation between months (previous/next buttons)
- **REQ-18**: Provide "Today" button to jump to current date
- **REQ-19**: Display current month/year clearly
- **REQ-20**: Click on booking navigates to existing confirm-booking page (under `/admin/confirm-booking/[bookingId]/[token]`)

### 4.5 Timezone Handling

- **REQ-21**: Display bookings in **user's local timezone** by default
- **REQ-22**: Provide timezone selector dropdown for switching between timezones
- **REQ-23**: Common timezones to include:
  - Local (browser default)
  - Bangkok (UTC+7)
  - Singapore (UTC+8)
  - Tokyo (UTC+9)
  - London (UTC+0)
  - New York (UTC-5/-4)
- **REQ-24**: Clearly indicate which timezone is currently selected
- **REQ-25**: Persist timezone preference in session storage

### 4.6 Data Loading

- **REQ-26**: Show loading state while fetching initial bookings
- **REQ-27**: Handle errors gracefully with user-friendly messages
- **REQ-28**: Extend existing WebSocket connection to broadcast booking updates
- **REQ-29**: Ensure all booking mutations trigger WebSocket events

---

## 5. Technical Requirements

### 5.1 Architecture

- **TECH-1**: Built using Next.js 15 App Router at `/admin/calendar`
- **TECH-2**: Move existing confirm-booking page to `/admin/confirm-booking/[bookingId]/[token]`
- **TECH-3**: Use Supabase/Prisma for data fetching
- **TECH-4**: Client-side rendering with "use client" directive
- **TECH-5**: Responsive design (mobile-friendly with list view fallback)

### 5.2 Authentication Implementation

- **TECH-6**: **Supabase Auth** with email/password
- **TECH-7**: Create admin authentication using Supabase's built-in auth (consider adding `is_admin` role in user metadata)
- **TECH-8**: Protect `/admin/*` routes with Next.js middleware checking Supabase session
- **TECH-9**: Optional dev bypass: Check `ADMIN_PASSWORD` env var for quick access during development

### 5.3 Data Model

- **TECH-10**: Fetch from `bookings` table with joins to:
  - `companies` (shop information)
  - `bikes` (bike details)
  - `users` (customer information)
- **TECH-11**: Query optimization for date ranges (only fetch visible month ± 1 month buffer)
- **TECH-12**: Use existing booking data structure

### 5.4 Real-Time Updates

- **TECH-13**: Extend existing WebSocket implementation from `/confirm-booking` page
- **TECH-14**: Subscribe to booking updates for all companies (or filtered company)
- **TECH-15**: Update calendar view when booking status changes (confirmed, declined, cancelled)
- **TECH-16**: Update calendar when new bookings created
- **TECH-17**: Handle connection drops gracefully with reconnection logic

### 5.5 UI Components

- **TECH-18**: Use **react-big-calendar** for calendar rendering
- **TECH-19**: Use existing component library where possible
- **TECH-20**: Tailwind CSS for styling
- **TECH-21**: Leverage existing booking card/display components

### 5.6 Timezone Handling

- **TECH-22**: Use `date-fns-tz` or similar library for timezone conversions
- **TECH-23**: Store booking dates in UTC in database (existing behavior)
- **TECH-24**: Convert to selected timezone on display only
- **TECH-25**: Pass timezone context to all date formatting functions

### 5.7 Environment Variables

```bash
# Supabase Auth (likely already configured)
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...

# Optional: Development bypass or initial admin password
ADMIN_PASSWORD=<secure-password-here>

# WebSocket (if needed)
NEXT_PUBLIC_WS_URL=...
```

---

## 6. UI/UX Specifications

### 6.1 Login Screen (Supabase Auth)

- Centered login form
- Email input field
- Password input field (type="password")
- "Login" button
- Error message display area
- Minimal branding (Flexbike logo)
- Optional "Forgot password?" link (for future)

### 6.2 Calendar View Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Flexbike Admin Calendar                         [Logout]    │
├─────────────────────────────────────────────────────────────┤
│ Filter: [All Companies ▼]  Timezone: [Local Time ▼]        │
│                                    [← October 2025 →][Today]│
├─────────────────────────────────────────────────────────────┤
│ Sun | Mon | Tue | Wed | Thu | Fri | Sat                     │
├─────────────────────────────────────────────────────────────┤
│  1  │  2  │  3  │  4  │  5  │  6  │  7                      │
│     │[Bk1]│     │[Bk2]│[Bk2]│     │    🔴 Live             │
│     │     │     │[Bk3]│[Bk3]│[Bk3]│                         │
└─────────────────────────────────────────────────────────────┘
```

### 6.3 Booking Display

- **Card format** showing:
  - Company name (bold)
  - Bike name
  - Customer name
  - Status badge (color-coded)
  - Time range in selected timezone
- **Click behavior**: Navigate to `/admin/confirm-booking/[bookingId]/[token]` page
- **Real-time indicator**: Small dot or pulse animation when WebSocket connected
- **Color scheme**:
  - Pending: Yellow/Amber (`#F59E0B`)
  - Confirmed: Blue (`#3B82F6`)
  - Completed: Green (`#10B981`)
  - Cancelled: Gray (`#6B7280`)
  - Declined: Red (`#EF4444`)

### 6.4 Timezone Selector

- Dropdown with common timezones
- Show offset (e.g., "Bangkok (UTC+7)")
- Default to browser's local timezone
- Persist selection across page refreshes

### 6.5 Responsive Behavior

- **Desktop**: Full calendar grid with react-big-calendar
- **Tablet**: Condensed calendar with smaller cards
- **Mobile**: Agenda/list view (react-big-calendar's agenda view)

---

## 7. Security Considerations

- **SEC-1**: Supabase Auth handles secure password hashing and session management
- **SEC-2**: Row Level Security (RLS) policies in Supabase for admin access
- **SEC-3**: Next.js middleware protects all `/admin/*` routes
- **SEC-4**: HTTPS only (enforced by hosting)
- **SEC-5**: Consider rate limiting on login attempts (Supabase built-in)
- **SEC-6**: Personal data (phone numbers, emails) should be partially masked in calendar view, full details on booking detail page
- **SEC-7**: Audit logging (optional for MVP, Supabase provides this)

---

## 8. Out of Scope (MVP)

- ❌ Creating/editing bookings from calendar
- ❌ Multi-user authentication with different roles (single admin for MVP)
- ❌ Export functionality (CSV, PDF)
- ❌ Advanced analytics or reporting
- ❌ Email notifications
- ❌ Booking conflict detection
- ❌ Drag-and-drop rescheduling
- ❌ Password reset flow (can add post-MVP)
- ❌ OAuth providers (Google, etc.)

---

## 9. Future Enhancements (Post-MVP)

1. **Week/Day views** in addition to month view
2. **Booking creation/editing** directly from calendar
3. **Multi-admin support** with different permission levels
4. **Export to Excel/PDF** functionality
5. **Advanced filtering**: by status, date range, bike type, customer
6. **Search**: find specific booking by ID or customer name
7. **Analytics overlay**: occupancy rates, revenue trends
8. **Role-based access control**: read-only vs full admin
9. **Audit log UI**: track who viewed/modified what
10. **Integration with communication tools** (send reminder to customer)
11. **Booking conflict detection** with visual warnings
12. **Revenue reporting** per company/period

---

## 10. Success Metrics

- **Primary**: Admin can successfully authenticate and view all bookings in real-time
- **Secondary**: Admin can filter by company and switch timezones
- **Performance**: Page loads within 3 seconds, WebSocket connects within 1 second
- **Usability**: Team can find specific booking within 30 seconds
- **Reliability**: Real-time updates reflect within 2 seconds of database change

---

## 11. Implementation Decisions (FINALIZED)

1. **Calendar Library**: ✅ **react-big-calendar**
2. **Authentication**: ✅ **Supabase Auth (email/password)** - Better foundation for future admin features
3. **Real-time Updates**: ✅ **Extend existing WebSocket** from confirm-booking page
4. **Booking Details**: ✅ **Move confirm-booking to `/admin/confirm-booking/`** and link from calendar
5. **Mobile Experience**: ✅ **Agenda/list view** for mobile devices (react-big-calendar built-in)
6. **Timezone Handling**: ✅ **User's local time + timezone dropdown selector**

---

## 12. Implementation Phases

### Phase 1: Authentication Setup

1. Set up Supabase Auth for admin users
2. Create admin user in Supabase
3. Create Next.js middleware to protect `/admin/*` routes
4. Build login page at `/admin/login`

### Phase 2: Calendar Base

5. Install and configure react-big-calendar
6. Create `/admin/calendar` page with basic calendar
7. Fetch all bookings from Supabase
8. Display bookings on calendar with color coding

### Phase 3: Filtering & Timezone

9. Add company filter dropdown
10. Add timezone selector
11. Implement timezone conversion for all dates
12. Persist filter/timezone preferences

### Phase 4: Real-time Updates

13. Move confirm-booking to `/admin/confirm-booking/`
14. Extend WebSocket to broadcast all booking changes
15. Subscribe to WebSocket in calendar page
16. Update calendar view on real-time events

### Phase 5: Polish & Testing

17. Responsive design (mobile list view)
18. Loading and error states
19. Click to navigate to booking details
20. Testing across timezones and companies

---

## 13. Acceptance Criteria

- [ ] Admin must authenticate via Supabase Auth to access `/admin/*` routes
- [ ] All bookings from database are displayed in calendar format
- [ ] Calendar updates in real-time when bookings change (via WebSocket)
- [ ] Admin can filter by company using dropdown
- [ ] Admin can switch timezones and dates update accordingly
- [ ] Admin can navigate between months
- [ ] Clicking booking navigates to confirm-booking page
- [ ] Bookings are color-coded by status
- [ ] Calendar is responsive (desktop grid, mobile list)
- [ ] Loading and error states are handled gracefully
- [ ] WebSocket connection status is visible
- [ ] Existing confirm-booking page moved to `/admin/confirm-booking/`

---

## 14. Technical Debt & Notes

- **Supabase Auth Setup**: If not already configured for admin users, need to set up auth tables and policies
- **WebSocket Broadcasting**: Need to audit all booking mutation endpoints to ensure they trigger WebSocket events
- **Timezone Library**: Add `date-fns-tz` or similar to dependencies
- **Mobile Testing**: Test on actual mobile devices for list view usability

---

**Decision Required**: Should we proceed with Supabase Auth implementation, or use simple password first and refactor later?

**My Recommendation**: ✅ **Implement Supabase Auth now** - It's only ~2-3 hours more work and creates a proper foundation for future admin features. You're already on Supabase, and this is the first admin tool, so it's the perfect time to set it up right.

---

Ready to proceed? Please confirm the authentication approach and I'll create the implementation task list! 🚀
