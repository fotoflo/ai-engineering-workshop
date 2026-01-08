# Admin Calendar Testing Guide

## ✅ Setup Checklist

### 1. Environment Variables

Ensure these are set in your `.env.local`:

```bash
# Supabase (should already be configured)
NEXT_PUBLIC_SUPABASE_URL=https://lupzygkeadzuvlvavrrf.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...

# Admin domain for authentication
ADMIN_EMAIL_DOMAIN=flexbike.app
```

### 2. Google OAuth in Supabase

- ✅ Google OAuth credentials added to Supabase Dashboard
- ✅ Redirect URLs configured:
  - `http://localhost:3000/admin/calendar`
  - `https://preview.flexbike.app/admin/calendar`
  - `https://booking.flexbike.app/admin/calendar`
  - `https://flexbike.app/admin/calendar`

### 3. Database

- ✅ `UserRole` enum added with values: `CUSTOMER`, `VENDOR`, `VENDOR_OWNER`, `ADMIN`, `SUPER_ADMIN`
- ✅ `role` field added to `users` table (default: `CUSTOMER`)
- ✅ Indexes on `email` and `role` columns

---

## 🧪 Test Cases

### Test 1: Authentication Flow

#### 1.1 Unauthenticated Access

- [ ] Navigate to `http://localhost:3000/admin/calendar`
- [ ] **Expected**: Redirect to `/admin/login` with `?redirect=/admin/calendar` param
- [ ] **Verify**: Not able to access calendar without login

#### 1.2 Google OAuth Login

- [ ] Click "Sign in with Google" button
- [ ] **Expected**: Google OAuth popup appears
- [ ] Sign in with a `@flexbike.app` email
- [ ] **Expected**: Redirect back to `/admin/calendar` after successful login

#### 1.3 Non-Admin Email Rejection

- [ ] Try signing in with a non-@flexbike.app email (e.g., `@gmail.com`)
- [ ] **Expected**: 403 Forbidden error from middleware
- [ ] **Verify**: Only @flexbike.app emails can access admin routes

#### 1.4 Session Persistence

- [ ] After logging in, refresh the page
- [ ] **Expected**: Stay logged in, no redirect to login page
- [ ] Close browser and reopen
- [ ] **Expected**: Still logged in (session persists)

#### 1.5 Sign Out

- [ ] Click "Sign out" button in top navigation
- [ ] **Expected**: Redirect to `/admin/login`
- [ ] Try accessing `/admin/calendar` again
- [ ] **Expected**: Redirect to login page

---

### Test 2: Calendar Display

#### 2.1 Initial Load

- [ ] Log in and access `/admin/calendar`
- [ ] **Expected**: Calendar displays with current month
- [ ] **Verify**: All bookings for visible date range are shown
- [ ] **Verify**: "Live" indicator shows WebSocket connection status

#### 2.2 Booking Display

- [ ] **Verify**: Each booking shows:
  - Company name
  - Bike name
  - Customer name
  - Color-coded by status
- [ ] **Verify**: Multi-day bookings span across multiple days
- [ ] **Verify**: Status colors match the legend:
  - Draft: Gray
  - Requested/Pending: Amber
  - Accepted/Confirmed: Blue
  - Completed: Green
  - Cancelled: Gray
  - Rejected: Red

#### 2.3 Calendar Navigation

- [ ] Click "Today" button
- [ ] **Expected**: Calendar jumps to current date
- [ ] Click "Next" arrow
- [ ] **Expected**: Calendar advances to next month
- [ ] Click "Previous" arrow
- [ ] **Expected**: Calendar goes back one month

#### 2.4 View Switching

- [ ] Switch to "Week" view
- [ ] **Expected**: Shows current week with bookings
- [ ] Switch to "Day" view
- [ ] **Expected**: Shows single day with bookings
- [ ] Switch to "Agenda" view
- [ ] **Expected**: Shows list of bookings chronologically
- [ ] Switch back to "Month" view
- [ ] **Expected**: Default calendar grid view

---

### Test 3: Filtering & Timezone

#### 3.1 Company Filter

- [ ] Open "Filter by Shop" dropdown
- [ ] **Expected**: See list of all companies with bookings
- [ ] Select a specific company
- [ ] **Expected**: Calendar only shows bookings for that company
- [ ] **Verify**: Booking count updates
- [ ] Switch back to "All Shops"
- [ ] **Expected**: All bookings visible again

#### 3.2 Timezone Selector

- [ ] Default timezone should be "Local Time"
- [ ] Note the start/end times of a booking
- [ ] Change timezone to "Bangkok (UTC+7)"
- [ ] **Expected**: Booking times update to Bangkok timezone
- [ ] Change to "Singapore (UTC+8)"
- [ ] **Expected**: Booking times shift by 1 hour
- [ ] Change to "New York (UTC-5/-4)"
- [ ] **Expected**: Booking times shift significantly
- [ ] **Verify**: Bookings remain on correct days (no date shifts)

#### 3.3 Filter Persistence

- [ ] Filter by a specific company
- [ ] Select a specific timezone
- [ ] Refresh the page
- [ ] **Expected**: Filters reset to defaults (by design)
  - _Note: We store in session, but don't persist across refreshes for MVP_

---

### Test 4: Booking Details

#### 4.1 Click on Booking

- [ ] Click on any booking in the calendar
- [ ] **Expected**: New tab opens with `/admin/confirm-booking/[bookingId]/[token]`
- [ ] **Verify**: Booking details page loads successfully
- [ ] **Verify**: Can see full booking information
- [ ] **Verify**: Can update booking status (if authorized)

#### 4.2 Token Validation

- [ ] Note the URL of a booking detail page
- [ ] Modify the token in the URL
- [ ] **Expected**: 404 Not Found (invalid token)
- [ ] **Verify**: Token security is working

---

### Test 5: Real-Time Updates (WebSocket)

#### 5.1 WebSocket Connection

- [ ] Open `/admin/calendar`
- [ ] **Verify**: Green "Live" indicator appears in top right
- [ ] Check browser console
- [ ] **Expected**: See log "Calendar WebSocket: Channel status: SUBSCRIBED"

#### 5.2 Real-Time Booking Updates

**Setup**: Open two browser windows side-by-side

- Window A: Admin calendar at `/admin/calendar`
- Window B: Booking detail page (via confirm-booking link)

**Steps**:

- [ ] In Window B, update a booking status (e.g., from "requested" to "confirmed")
- [ ] **Expected**: Window A (calendar) automatically refreshes
- [ ] **Verify**: Booking color updates to reflect new status
- [ ] **Verify**: No manual refresh needed

#### 5.3 WebSocket Reconnection

- [ ] Turn off WiFi/network
- [ ] **Verify**: "Live" indicator disappears or turns gray
- [ ] Turn network back on
- [ ] **Expected**: "Live" indicator returns (auto-reconnect)
- [ ] **Verify**: Calendar still receives updates

---

### Test 6: Responsive Design

#### 6.1 Desktop (1920x1080)

- [ ] **Verify**: Full calendar grid visible
- [ ] **Verify**: Filters side-by-side horizontally
- [ ] **Verify**: All navigation controls accessible

#### 6.2 Tablet (768x1024)

- [ ] **Verify**: Calendar still usable
- [ ] **Verify**: Filters stack or remain side-by-side
- [ ] **Verify**: Bookings readable

#### 6.3 Mobile (375x667)

- [ ] **Verify**: Calendar switches to "Agenda" view (better for mobile)
- [ ] **Verify**: Filters stack vertically
- [ ] **Verify**: Navigation simplified
- [ ] **Verify**: Can click bookings to see details

---

### Test 7: Performance

#### 7.1 Large Dataset

- [ ] Filter by "All Shops" (maximum bookings)
- [ ] Navigate through multiple months
- [ ] **Expected**: Calendar loads within 3 seconds
- [ ] **Verify**: No lag when switching views
- [ ] **Verify**: No memory leaks (check browser dev tools)

#### 7.2 API Rate Limiting

- [ ] Rapidly switch between companies
- [ ] **Expected**: Debounced API calls (not one per keystroke)
- [ ] **Verify**: Calendar doesn't hang or crash

---

### Test 8: Error Handling

#### 8.1 Network Failure

- [ ] Turn off network while calendar is open
- [ ] Try to change filters
- [ ] **Expected**: Error message displayed
- [ ] **Verify**: Calendar doesn't crash

#### 8.2 Invalid Data

- [ ] Access `/admin/calendar` when database is empty
- [ ] **Expected**: Calendar shows "0 bookings" gracefully
- [ ] **Verify**: No console errors

#### 8.3 Expired Session

- [ ] Manually delete Supabase auth cookies
- [ ] Try to interact with calendar
- [ ] **Expected**: Redirect to login page
- [ ] **Verify**: No data exposure

---

## 🐛 Known Issues / Future Enhancements

### Current Limitations:

- ❌ Cannot create/edit bookings from calendar (read-only)
- ❌ No export functionality (CSV, PDF)
- ❌ No search by customer name or booking ID
- ❌ Filter preferences don't persist across refreshes
- ❌ Mobile view doesn't have custom optimizations (uses default agenda)

### Planned Enhancements:

- 📅 Drag-and-drop to reschedule bookings
- 🔍 Advanced search and filtering
- 📊 Analytics overlay (occupancy rates, revenue)
- 📤 Export reports
- 👥 Multi-admin support with different permission levels
- 🔔 Browser notifications for new bookings

---

## 🚀 Deployment Checklist

Before deploying to production:

- [ ] All tests above passing
- [ ] Google OAuth configured in Supabase for production domain
- [ ] Environment variables set in Vercel/hosting platform
- [ ] Confirmed `ADMIN_EMAIL_DOMAIN` is set correctly
- [ ] Tested on production domain with real @flexbike.app email
- [ ] Verified WebSocket works in production (check SUPABASE_SERVICE_ROLE_KEY)
- [ ] Confirmed no sensitive data logged to console in production
- [ ] Middleware protecting all `/admin/*` routes
- [ ] SSL/HTTPS enabled (required for OAuth)

---

## 📝 Notes

- **Database sync**: The calendar fetches bookings within ±1 month of the visible date range for performance
- **Timezone handling**: All dates stored in UTC in database, converted on display
- **WebSocket**: Uses Supabase Realtime with `admin:bookings` channel
- **Token security**: Uses same token derivation as confirm-booking page (`deriveConfirmToken`)

---

**Last Updated**: 2025-01-08
**Version**: 1.0.0
