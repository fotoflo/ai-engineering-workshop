# Checkout Flow Testing Command

**Command**: `/checkout-test`

**Purpose**: Comprehensive testing checklist for the CheckOutFlow component and booking creation process.

---

## 🎯 Quick Start Testing

### Access the Component
1. Navigate to: `/admin/companies/[companySlug]/[productSlug]/status` or `/[companySlug]/[productSlug]/status`
2. Click "Check Out" button on a bike with status "in_shop"
3. The CheckOutFlow dialog should open

---

## 📋 Step-by-Step Feature Testing

### **Step 1: Renter Information** (`renter`)

#### ✅ Basic Functionality
- [ ] Dialog opens with "Renter" step visible
- [ ] "Full Name" input field is present
- [ ] "WhatsApp Number" input field is present
- [ ] "Cancel" button works and closes dialog
- [ ] "Next" button is present

#### ✅ Validation
- [ ] Try clicking "Next" with empty fields → Should show error messages
- [ ] Enter only name → Error on WhatsApp field
- [ ] Enter only WhatsApp → Error on name field
- [ ] Enter both fields → Should proceed to next step
- [ ] Error messages display correctly (red border, text below field)

#### ✅ Navigation
- [ ] "Next" button advances to "returnDate" step
- [ ] "Cancel" button closes dialog and calls `onCancel`

---

### **Step 2: Return Date** (`returnDate`)

#### ✅ Basic Functionality
- [ ] Shows checkout date and return date fields
- [ ] Shows checkout time and return time fields
- [ ] "Change dates" button opens date picker dialog
- [ ] Date picker dialog is full-screen
- [ ] Calendar shows 2 months (stacked on mobile, side-by-side on desktop)

#### ✅ Date Selection
- [ ] Can select start date in calendar
- [ ] Can select end date in calendar
- [ ] "Apply dates" button closes dialog and updates dates
- [ ] "Clear" button resets date selection
- [ ] Selected dates display correctly in input fields
- [ ] If return date < checkout date, return date auto-adjusts

#### ✅ Time Selection
- [ ] Time inputs accept 30-minute increments
- [ ] Default times are set (if not provided)
- [ ] Can change checkout time
- [ ] Can change return time

#### ✅ Rental Summary
- [ ] Shows rental length (e.g., "1 week + 3 days" or "5 days")
- [ ] Updates when dates change
- [ ] Handles edge cases (same day = "1 day")

#### ✅ Conflict Detection
- [ ] Shows "Checking availability..." while loading
- [ ] Detects conflicts with existing bookings
- [ ] Shows conflict warning with booking details
- [ ] "Next" button is disabled when conflicts exist
- [ ] Can proceed when no conflicts
- [ ] Conflict check happens automatically when return date changes

#### ✅ Navigation
- [ ] "Back" button returns to "renter" step
- [ ] "Next" button advances to "fulfillment" step (if no conflicts)

---

### **Step 3: Fulfillment** (`fulfillment`)

#### ✅ Pickup/Delivery Selection
- [ ] Radio buttons for "Pickup / Collection" and "Delivery"
- [ ] Default is "Pickup"
- [ ] Can switch between pickup and delivery
- [ ] UI updates based on selection

#### ✅ Pickup Mode
- [ ] Shows "Special instructions" card
- [ ] Textarea for customer notes
- [ ] Can enter pickup instructions
- [ ] Instructions are saved

#### ✅ Delivery Mode
- [ ] Shows delivery address input with map
- [ ] Can enter delivery address
- [ ] Map component loads and displays location
- [ ] Can change delivery address
- [ ] Delivery notes field is available
- [ ] Address validation works (if implemented)

#### ✅ Navigation
- [ ] "Back" button returns to "returnDate" step
- [ ] "Next" button creates booking and advances to "invoice" step

---

### **Step 4: Invoice** (`invoice`)

#### ✅ Booking Creation
- [ ] Booking is created in "draft" status
- [ ] Booking ID is saved to localStorage
- [ ] Invoice step loads with booking data
- [ ] Price is pre-calculated from bike pricing

#### ✅ Invoice Display
- [ ] Shows rider name
- [ ] Shows WhatsApp number
- [ ] Shows start date
- [ ] Shows return date
- [ ] Shows delivery address (if delivery)
- [ ] Shows customer note (if provided)
- [ ] Shows subtotal
- [ ] Shows delivery fee
- [ ] Shows deposit (if applicable)
- [ ] Shows total price
- [ ] Currency displays correctly

#### ✅ Inline Editing
- [ ] Click rider name → Can edit inline
- [ ] Click WhatsApp → Can edit inline
- [ ] Click start date → Date picker opens
- [ ] Click return date → Date picker opens
- [ ] Changes save on blur/Enter
- [ ] Changes update booking via API
- [ ] Error handling if update fails

#### ✅ Price Adjustments
- [ ] Can edit subtotal manually
- [ ] Can edit delivery fee manually
- [ ] Price calculations update correctly
- [ ] Discount shows if manual price < calculated price
- [ ] Total updates when prices change

#### ✅ Payment Status
- [ ] Shows payment status
- [ ] "Mark as Paid" button appears if not paid
- [ ] Clicking "Mark as Paid" updates booking status
- [ ] Payment status updates to "Paid" after marking
- [ ] Green checkmark appears when paid

#### ✅ WhatsApp Link
- [ ] "Send Booking Link via WhatsApp" button present
- [ ] Opens WhatsApp with booking link
- [ ] Link format is correct

#### ✅ Navigation
- [ ] "Back" button returns to "fulfillment" step
- [ ] "Next" button advances to "identity" step

---

### **Step 5: Identity** (`identity`)

#### ✅ Photo Capture
- [ ] "Take Photo" button for renter photo
- [ ] "Upload" button for renter photo
- [ ] "Take Photo" button for passport photo
- [ ] "Upload" button for passport photo
- [ ] Camera opens on mobile when "Take Photo" clicked
- [ ] File picker opens when "Upload" clicked

#### ✅ Photo Display
- [ ] Renter photo preview shows after capture/upload
- [ ] Passport photo preview shows after capture/upload
- [ ] Preview images are correct size/format
- [ ] "X" button removes photo
- [ ] Can replace photo

#### ✅ Validation
- [ ] "Next" button disabled until both photos captured
- [ ] Error message if trying to proceed without photos
- [ ] Both photos required to advance

#### ✅ Navigation
- [ ] "Back" button returns to "invoice" step
- [ ] "Next" button advances to "condition" step (if photos present)

---

### **Step 6: Condition** (`condition`)

#### ✅ Photo Capture
- [ ] "Take Photo" button for bike photos
- [ ] "Upload" button for bike photos
- [ ] Can capture/upload multiple photos
- [ ] Photo counter shows number of photos

#### ✅ Photo Display
- [ ] All bike photos display in grid
- [ ] Each photo has remove button
- [ ] Can remove individual photos
- [ ] Can add more photos

#### ✅ Validation
- [ ] "Next" button disabled if no photos
- [ ] Error message if trying to proceed without photos
- [ ] At least one photo required

#### ✅ Navigation
- [ ] "Back" button returns to "identity" step
- [ ] "Next" button advances to "signature" step (if photos present)

---

### **Step 7: Signature** (`signature`)

#### ✅ Signature Canvas
- [ ] Canvas is visible and properly sized
- [ ] Can draw with mouse
- [ ] Can draw with touch (mobile)
- [ ] Signature appears as drawn
- [ ] "Clear" button resets canvas
- [ ] Canvas clears correctly

#### ✅ Validation
- [ ] "Next" button disabled if no signature
- [ ] Error message if trying to proceed without signature
- [ ] Signature required to advance

#### ✅ Navigation
- [ ] "Back" button returns to "condition" step
- [ ] "Next" button advances to "confirm" step (if signature present)

---

### **Step 8: Confirm** (`confirm`)

#### ✅ Summary Display
- [ ] Shows renter name
- [ ] Shows WhatsApp number
- [ ] Shows photo status (✓/✗ for renter, passport, bike count)
- [ ] Shows signature status (✓/✗)

#### ✅ Final Submission
- [ ] "Confirm Checkout" button present
- [ ] Button shows "Saving..." while processing
- [ ] All photos upload to booking
- [ ] Signature uploads to booking
- [ ] Booking status changes to "confirmed"
- [ ] Handover data is saved
- [ ] Product marked as unavailable (if applicable)
- [ ] Success callback fires
- [ ] Dialog closes after completion

#### ✅ Navigation
- [ ] "Back" button returns to "signature" step
- [ ] "Cancel" button closes dialog (loses progress)

---

## 🔄 Advanced Features

### **Draft Booking Resume**
- [ ] Start checkout flow
- [ ] Fill in renter info and dates
- [ ] Advance to invoice step (creates draft)
- [ ] Close dialog
- [ ] Reopen checkout for same bike
- [ ] Should resume from invoice step with saved data
- [ ] Form fields are pre-filled
- [ ] Booking ID is restored

### **Error Handling**
- [ ] Network error during booking creation → Shows error message
- [ ] Network error during photo upload → Shows error message
- [ ] Network error during booking update → Shows error message
- [ ] Invalid date selection → Proper validation
- [ ] API returns error → User-friendly error message

### **State Persistence**
- [ ] Navigate back and forth between steps
- [ ] All form data persists
- [ ] Photos remain after navigation
- [ ] Signature remains after navigation
- [ ] Selected dates remain after navigation

### **Mobile Responsiveness**
- [ ] All steps work on mobile
- [ ] Date picker is usable on mobile
- [ ] Photo capture works on mobile
- [ ] Signature works with touch
- [ ] Buttons are properly sized (min-h-[44px])
- [ ] Text is readable

### **Edge Cases**
- [ ] Bike with no pricing → Handles gracefully
- [ ] Very long rental period → Calculations work
- [ ] Same-day checkout/return → Shows "1 day"
- [ ] Multiple conflicts → All shown
- [ ] Large photos → Uploads successfully
- [ ] Special characters in names → Handles correctly
- [ ] International phone numbers → Formats correctly

---

## 🧪 Integration Testing

### **API Integration**
- [ ] `/api/bikes/[bikeId]` - Fetches bike pricing
- [ ] `/api/admin/bookings` - Creates booking
- [ ] `/api/admin/bookings/[id]` - Updates booking
- [ ] `/api/admin/bookings?productId=...` - Checks conflicts
- [ ] `/api/admin/upload-image` - Uploads photos
- [ ] `/api/admin/products/[id]` - Marks product unavailable

### **LocalStorage**
- [ ] Draft booking ID saved to `checkout_draft_[bikeId]`
- [ ] Draft cleared after successful checkout
- [ ] Draft persists across page refreshes

### **Component Integration**
- [ ] `onComplete` callback fires after successful checkout
- [ ] `onCancel` callback fires when dialog closed
- [ ] Parent component updates after checkout
- [ ] Bike status updates in BikeStatusPage

---

## 🐛 Regression Testing

### **Verify No Breaking Changes**
- [ ] All existing functionality works
- [ ] No console errors
- [ ] No TypeScript errors
- [ ] No runtime errors
- [ ] Performance is acceptable
- [ ] No memory leaks

### **Compare with Original**
- [ ] Same validation rules
- [ ] Same API calls
- [ ] Same data structure
- [ ] Same user experience
- [ ] Same error messages

---

## 📱 Device Testing

### **Desktop**
- [ ] Chrome
- [ ] Firefox
- [ ] Safari
- [ ] Edge

### **Mobile**
- [ ] iOS Safari
- [ ] Android Chrome
- [ ] Touch interactions work
- [ ] Camera access works

---

## ✅ Quick Smoke Test (5 minutes)

1. Open checkout flow
2. Enter renter name and WhatsApp
3. Select dates (no conflicts)
4. Choose pickup
5. Advance to invoice
6. Take/upload renter photo
7. Take/upload passport photo
8. Take bike photo
9. Draw signature
10. Confirm checkout
11. Verify booking created and dialog closes

---

## 🎯 Priority Testing Order

### **Critical (Must Test First)**
1. Step navigation (back/next)
2. Form validation
3. Booking creation
4. Photo uploads
5. Final submission

### **Important (Test Second)**
1. Conflict detection
2. Invoice calculations
3. Inline editing
4. Draft resume
5. Error handling

### **Nice to Have (Test Last)**
1. Edge cases
2. Mobile responsiveness
3. Performance
4. Accessibility

---

## 📝 Notes

- Test with a bike that has pricing configured
- Test with a bike that has existing bookings (for conflicts)
- Test with admin/vendor role (required for checkout)
- Keep browser console open to catch errors
- Test on actual device for camera/signature features

---

## 🔗 Related Commands

- **Smoke Test**: `/smoke-test` - General smoke testing
- **Refactor Test**: `/refactor-test` - Testing after refactoring
- **API Test**: See `scripts/README.md` for API testing scripts

---

## 📊 Success Criteria

**All Critical tests pass:**
- ✅ Checkout flow completes successfully
- ✅ Booking is created in database
- ✅ Photos and signature are uploaded
- ✅ Bike status updates correctly
- ✅ No console errors or TypeScript errors
- ✅ Mobile responsiveness works