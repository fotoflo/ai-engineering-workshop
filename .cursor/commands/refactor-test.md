# Refactoring Testing Command

**Command**: `/refactor-test`

**Purpose**: Comprehensive testing checklist for validating refactored code maintains functionality.

---

## 🎯 Quick Start Testing

### Admin Product Detail Page (`/admin/companies/[companySlug]/[productSlug]`)

#### Revenue Calculations (useProductRevenue hook)
- [ ] **Total Revenue Display**
  - Verify revenue displays correctly in ProductHeader stats
  - Test with products that have multiple bookings
  - Test with products that have no bookings (should show 0)
  - Test with bookings that have different price fields (totalPrice, price, totalCents)
  - Test with bookings that need date-based calculation

- [ ] **Average Revenue Per Booking**
  - Verify calculation is correct
  - Test edge cases (single booking, no bookings)

- [ ] **Average Occupancy**
  - Verify percentage calculation is accurate
  - Test with bookings spanning different date ranges
  - Test with overlapping bookings
  - Test with no bookings (should show 0%)

#### Pricing Section (ProductPricingSection component)
- [ ] **Deposit Input**
  - Enter deposit amount
  - Clear deposit (set to null)
  - Verify autosave works
  - Test with different currencies

- [ ] **Rate Tabs (Daily/Weekly/Monthly)**
  - Switch between tabs
  - Enter values for each rate type
  - Verify values persist when switching tabs
  - Test autosave for each rate type
  - Clear values (set to null)

#### Vehicle Information Section (VehicleInfoSection component)
- [ ] **Basic Vehicle Info**
  - Edit vehicle type, color, condition
  - Verify autosave works

- [ ] **Additional Information**
  - Edit size and displacement
  - Test electric vs non-electric (displacement should hide for electric)
  - Verify autosave

- [ ] **Specifications Collapsible**
  - Open/close collapsible
  - Edit transmission, fuel type, seats, doors, luggage, engine size, fuel efficiency
  - Verify all fields save correctly

- [ ] **Features Checkboxes**
  - Toggle surf rack, keyless, ABS, phone holder
  - Verify all checkboxes save independently

- [ ] **Electric Vehicle Details**
  - Toggle "Electric Vehicle" checkbox
  - Verify electric details section appears/disappears
  - Edit electric range, top speed, power
  - Test fuel level vs battery health (should show appropriate field)

- [ ] **Maintenance & Status Collapsible**
  - Edit availability status dropdown
  - Edit fuel level (non-electric) or battery health (electric)
  - Edit last/next maintenance dates
  - Verify date formatting and saving

- [ ] **Instructions Collapsible**
  - Edit pickup instructions
  - Edit return instructions
  - Verify textarea saves correctly

#### Copy From Dialog (CopyFromProductDialog component)
- [ ] **Search Functionality**
  - Open dialog (if button exists)
  - Search for vehicles by name
  - Search for vehicles by slug
  - Verify search results display correctly
  - Test with no results

- [ ] **Vehicle Selection**
  - Select a vehicle from search results
  - Verify vehicle preview displays (image, title, company)
  - Click "Copy from this bike" button
  - Verify data is copied to form
  - Verify dialog closes after copy

- [ ] **Data Copying**
  - Verify all copied fields are correct:
    - Description, category, tags
    - Currency, pricing (daily/weekly/monthly), deposit
    - All vehicle fields
  - Verify existing vehicle data is preserved where appropriate

#### QR/Status Section (ProductQRStatusSection component)
- [ ] **QR Code Display**
  - Verify QR code renders correctly
  - Verify QR code contains correct URL

- [ ] **URL Display & Copy**
  - Verify status page URL displays correctly
  - Click copy button
  - Verify URL is copied to clipboard
  - Verify alert message appears

- [ ] **Actions**
  - Click "Open Public Status Page" link
  - Verify opens in new tab with correct URL
  - Click "Download QR Code" button
  - Verify QR code downloads as PNG file
  - Verify filename is correct (`qr-{slug}.png`)

#### General Admin Product Page
- [ ] **Autosave**
  - Make changes to any field
  - Verify "Saving..." status appears
  - Verify "Saved" status appears after save
  - Test rapid changes (debouncing should work)

- [ ] **Error Handling**
  - Test with network errors
  - Verify error messages display correctly
  - Test with invalid data

---

## Bike Status Page (`/[companySlug]/[productSlug]/status`)

### Status Badge (BikeStatusBadge component)
- [ ] **Status Display**
  - Test with "in_shop" status (green badge)
  - Test with "out" status (blue badge)
  - Test with "blocked" status (yellow badge)
  - Test with "missing" status (red badge)
  - Verify emoji and label display correctly

### Status Guidance (getStatusGuidance helper)
- [ ] **Contextual Messages**
  - Verify correct message displays for each status
  - Test message appears in guidance card

### Helper Functions
- [ ] **WhatsApp URL Building**
  - Test with valid phone number
  - Test with phone number containing special characters
  - Test with no phone number (should be null)
  - Verify message format is correct

- [ ] **Map Query Building**
  - Test with coordinates (should use coordinates)
  - Test with address (should use address)
  - Test with only city/country (should use location string)
  - Verify map displays correctly

- [ ] **Role Checking**
  - Test as ADMIN (should see admin features)
  - Test as VENDOR (should see vendor features)
  - Test as VENDOR_OWNER (should see vendor features)
  - Test as regular user (should not see admin features)
  - Verify "canManageBike" logic works correctly

### Admin Actions
- [ ] **Check Out Flow**
  - Click "Check Out Bike" when status is "in_shop"
  - Verify CheckOutFlow modal opens
  - Complete checkout flow
  - Verify bike status updates

- [ ] **Check In Flow**
  - Click "Check In Bike" when status is "out"
  - Verify CheckInFlow modal opens
  - Complete check-in flow
  - Verify bike status updates

- [ ] **Payment Status Toggle**
  - Click "Mark Booking as Paid" / "Mark Unpaid"
  - Verify confirmation dialog appears
  - Confirm action
  - Verify payment status updates
  - Verify page refreshes with new data

- [ ] **Edit Bike Button**
  - Click "Edit Bike" button
  - Verify navigates to admin product page

### Public Features
- [ ] **WhatsApp Contact**
  - Click "WhatsApp the shop" button
  - Verify opens WhatsApp with pre-filled message
  - Test message contains bike title and plate/slug

- [ ] **Login CTA**
  - Verify "Log in as admin" button appears for non-admins
  - Click button
  - Verify redirects to login with return URL

---

## Bike Detail Page (Public Product Page)

### Related Bikes (RelatedBikes component)
- [ ] **Display**
  - Verify related bikes section appears
  - Test with company-specific bikes (should show "More adventures with {company}")
  - Test with city-wide bikes (should show "More adventures in {city}")
  - Verify current bike is excluded from list

- [ ] **Bike Cards**
  - Verify all bike cards display correctly
  - Click on a related bike
  - Verify navigation works (canonical vs direct booking URLs)
  - Test with direct booking context

- [ ] **Empty State**
  - Test with no related bikes
  - Verify section doesn't render (returns null)

### Product Reviews (ProductReviews component)
- [ ] **Display**
  - Verify reviews section appears when reviews exist
  - Verify review cards display:
    - Rider name (or "Guest")
    - Star rating (1-5 stars)
    - Review date
    - Review text (if provided)
    - Rating badge

- [ ] **Empty State**
  - Test with no reviews
  - Verify section doesn't render (returns null)

- [ ] **Review Formatting**
  - Verify star display (filled vs empty)
  - Verify date formatting
  - Verify text truncation if needed

---

## Cross-Cutting Tests

### Performance
- [ ] **Page Load Times**
  - Verify pages load quickly
  - Test with slow network (throttle in DevTools)
  - Verify no unnecessary re-renders

### Responsive Design
- [ ] **Mobile View**
  - Test all pages on mobile viewport
  - Verify components are responsive
  - Test touch interactions

- [ ] **Tablet View**
  - Test on tablet-sized viewport
  - Verify layouts adapt correctly

- [ ] **Desktop View**
  - Test on desktop viewport
  - Verify all features accessible

### Browser Compatibility
- [ ] **Chrome/Edge**
- [ ] **Firefox**
- [ ] **Safari**
- [ ] **Mobile browsers (iOS Safari, Chrome Mobile)**

### Error Scenarios
- [ ] **Network Failures**
  - Test with network offline
  - Test with slow network
  - Verify error messages are user-friendly

- [ ] **Invalid Data**
  - Test with missing required fields
  - Test with invalid date formats
  - Test with invalid numbers

- [ ] **Permission Errors**
  - Test accessing admin features without permission
  - Verify appropriate error handling

---

## Regression Tests

### Existing Functionality
- [ ] **Product Creation/Editing**
  - Create new product
  - Edit existing product
  - Verify all fields save correctly

- [ ] **Booking Management**
  - Create booking from product page
  - View bookings table
  - Verify booking status updates

- [ ] **Image Upload**
  - Upload product images
  - Drag and drop images
  - Delete images
  - Reorder images

- [ ] **Product History**
  - Add notes
  - View history
  - Verify notes display correctly

---

## Quick Smoke Test (5 minutes)

1. **Admin Product Page**
   - [ ] Open product detail page
   - [ ] Edit pricing (daily rate)
   - [ ] Edit vehicle info (model, color)
   - [ ] Verify autosave works
   - [ ] Check revenue stats display

2. **Bike Status Page**
   - [ ] Open status page
   - [ ] Verify status badge displays correctly
   - [ ] Test WhatsApp button (if available)
   - [ ] Test admin actions (if logged in)

3. **Bike Detail Page**
   - [ ] Open public product page
   - [ ] Verify related bikes section
   - [ ] Verify reviews section (if reviews exist)
   - [ ] Test booking flow

---

## Notes

- All refactored components should maintain the same functionality as before
- Pay special attention to autosave functionality - it should work seamlessly
- Test edge cases like empty data, null values, and missing fields
- Verify no console errors appear in browser DevTools
- Check that TypeScript types are correct (no type errors)

---

## 🔗 Related Commands

- **Checkout Test**: `/checkout-test` - Checkout flow testing checklist
- **Smoke Test**: `/smoke-test` - General smoke testing workflow
- **Deploy Production**: `/deploy-production` - Production deployment workflow

---

## ✅ Success Criteria

**All refactored code passes:**
- ✅ Existing functionality preserved
- ✅ No console errors or TypeScript errors
- ✅ Autosave works correctly
- ✅ Performance maintained or improved
- ✅ Responsive design intact
- ✅ Cross-browser compatibility maintained