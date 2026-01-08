# Admin Calendar - Quick Start Guide

## 🚀 Get Started in 5 Minutes

### Step 1: Complete Google OAuth Setup

You're setting this up now! Once complete:

1. **Google Cloud Console**:

   - ✅ OAuth Client created: "Flexbike Admin Auth - Supabase"
   - ✅ Authorized JavaScript Origins added
   - ✅ Authorized Redirect URI: `https://lupzygkeadzuvlvavrrf.supabase.co/auth/v1/callback`

2. **Supabase Dashboard** (Authentication > Providers > Google):

   - ✅ Paste Google Client ID
   - ✅ Paste Google Client Secret
   - ✅ Enable Google provider

3. **Supabase Dashboard** (Authentication > URL Configuration):
   - Add these Redirect URLs:
     ```
     http://localhost:3000/admin/calendar
     https://preview.flexbike.app/admin/calendar
     https://booking.flexbike.app/admin/calendar
     https://flexbike.app/admin/calendar
     ```

---

### Step 2: Set Environment Variable

Add to your `.env.local`:

```bash
ADMIN_EMAIL_DOMAIN=flexbike.app
```

---

### Step 3: Start the Dev Server

```bash
pnpm dev
```

---

### Step 4: Test Login

1. Open browser to: `http://localhost:3000/admin/calendar`
2. You'll be redirected to login page
3. Click "Sign in with Google"
4. Sign in with your `@flexbike.app` email
5. Authorize the application
6. You'll be redirected to the calendar

---

### Step 5: Explore the Calendar

- **View bookings**: See all bookings in calendar format
- **Filter by shop**: Use dropdown to filter by company
- **Change timezone**: Switch between Local, Bangkok, Singapore, Tokyo, London, NYC
- **Click booking**: Opens detailed booking page in new tab
- **Check live updates**: Look for green "Live" indicator (WebSocket connected)

---

## ✅ Verify Everything Works

### Quick Checklist:

- [ ] Can log in with @flexbike.app email
- [ ] Cannot log in with non-@flexbike.app email
- [ ] Calendar shows bookings
- [ ] Company filter works
- [ ] Timezone selector updates times
- [ ] Clicking booking opens detail page
- [ ] "Live" indicator shows (green dot with "Live" text)
- [ ] Can sign out successfully

---

## 🎨 What You Should See

### Login Page

```
┌─────────────────────────────────┐
│      [Flexbike Logo]            │
│                                 │
│      Admin Portal               │
│  Sign in with @flexbike.app     │
│                                 │
│  [🔵 Sign in with Google]      │
│                                 │
│  Only authorized @flexbike.app  │
│  accounts can access            │
└─────────────────────────────────┘
```

### Calendar Page

```
┌─────────────────────────────────────────────────────────────┐
│ [Logo] Admin                          [User Avatar] [Logout] │
├─────────────────────────────────────────────────────────────┤
│ Booking Calendar                                             │
│ View and manage all bookings across shops                    │
│                                                              │
│ Filter: [All Shops ▼]  Timezone: [Local Time ▼]  🟢 Live   │
│                                                              │
│ Status Legend: [Draft] [Pending] [Confirmed] [Completed]... │
│                                                              │
│ ┌─ January 2025 ──────────────────────────────────────┐    │
│ │ Sun │ Mon │ Tue │ Wed │ Thu │ Fri │ Sat │            │    │
│ │ ... │ ... │[Bk]│[Bk]│[Bk]│ ... │ ... │            │    │
│ └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🐛 Troubleshooting

### "Unauthorized: Admin access required"

- ❌ You're using a non-@flexbike.app email
- ✅ Solution: Use an @flexbike.app email for testing

### "Redirect loop" or "Can't log in"

- ❌ Google OAuth not configured correctly
- ✅ Solution: Check Supabase Dashboard > Authentication > Providers > Google

### "No bookings showing"

- ❌ No bookings in database, or date range doesn't match
- ✅ Solution: Navigate to months with bookings, or create test bookings

### "Live indicator not showing"

- ❌ WebSocket connection issue
- ✅ Solution: Check browser console for errors, verify `SUPABASE_SERVICE_ROLE_KEY` is set

### "Calendar not loading"

- ❌ Missing npm packages
- ✅ Solution: Run `pnpm install` to ensure all packages installed

---

## 📚 Next Steps

1. **Read Testing Guide**: See [`testing.md`](testing.md) for comprehensive test cases
2. **Review Implementation**: See [`implementation.md`](implementation.md) for technical details
3. **Test Real-Time**: Open two windows and test WebSocket updates
4. **Deploy to Preview**: Test on preview environment before production
5. **Add More Admins**: Sign in with other @flexbike.app emails

---

## 🎉 Success!

If you can:

- ✅ Log in with Google OAuth
- ✅ See bookings on the calendar
- ✅ Filter by company
- ✅ Switch timezones
- ✅ See the "Live" indicator

**Congratulations!** The admin calendar is working correctly. 🎊

---

## 📞 Need Help?

Check these files:

- **Testing**: [`testing.md`](testing.md) (28 test cases)
- **Implementation**: [`implementation.md`](implementation.md) (technical details)
- **PRD**: [`prd.md`](prd.md) (requirements & decisions)

Browser console logs will help debug issues - look for messages starting with:

- `Admin WebSocket:`
- `Calendar WebSocket:`
- `Broadcasting booking update:`

---

**Ready to go!** 🚀

Start the dev server and navigate to: `http://localhost:3000/admin/calendar`
