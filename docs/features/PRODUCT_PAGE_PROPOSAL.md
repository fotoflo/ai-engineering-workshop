# 🚀 Admin Product Management Page Proposal

## Overview

**Approach**: Add an **edit mode** to the existing public product page (`BikeDetailPage.tsx`) instead of building a separate admin page. This maintains UI consistency and reduces development effort.

**URL Pattern**:

- Public: `/indonesia/bali/canggu/bollu-garage-2/yamaha-fazzio-4`
- Admin Edit: `/indonesia/bali/canggu/bollu-garage-2/yamaha-fazzio-4?edit=true`
- Or: `/admin/products/[productId]` → redirects to public URL with `?edit=true`

**How It Works**:
When an admin user visits the product page with `?edit=true` query parameter (or is detected as admin), the page switches to edit mode where all fields become editable inline. The same beautiful UI is maintained, just with editable inputs instead of static text.

**Benefits**:

- ✅ Reuse existing UI/components (faster development)
- ✅ Admins see exactly what customers see (consistency)
- ✅ Less code to maintain (one page, not two)
- ✅ Familiar interface for admins
- ✅ Estimated 50% faster than building separate page

---

## 🎨 Design Philosophy

### Core Principles

1. **Reuse Existing UI**: Leverage the beautiful existing `BikeDetailPage` design
2. **Edit Mode Toggle**: Seamless switch between view and edit modes
3. **Inline Editing**: Edit fields in place without leaving the page
4. **Progressive Enhancement**: Edit mode adds functionality without breaking public view
5. **Consistent Experience**: Admins see the same page customers see, just editable

---

## 📐 Edit Mode Implementation

### Architecture

**Current**: `BikeDetailPage.tsx` (Server Component) → Renders static product page
**New**: `BikeDetailPage.tsx` → Wraps content in `BikeDetailEditWrapper.tsx` (Client Component)

### How Edit Mode Works

1. **URL Pattern**: `/indonesia/bali/canggu/bollu-garage-2/yamaha-fazzio-4?edit=true`
2. **Detection**: Client component checks `searchParams.edit === 'true'` AND admin auth
3. **State**: Local state manages edit mode and form data
4. **Transformation**: Conditional rendering - show inputs in edit mode, text in view mode
5. **Persistence**: Save button calls API, then updates URL to remove `?edit=true`

### Component Structure

```
BikeDetailPage.tsx (Server Component)
└── BikeDetailEditWrapper.tsx (Client Component)
    ├── EditModeHeader.tsx (Admin bar with save/cancel)
    ├── EditableTitle.tsx (h1 → input when editing)
    ├── EditablePricing.tsx (price display → inputs)
    ├── EditableDescription.tsx (text → textarea)
    ├── EditableGallery.tsx (images → upload/delete)
    ├── EditableSpecs.tsx (specs → form fields)
    └── ... (all other sections become editable)
```

### 1. Edit Mode Header (Floating Bar)

```
┌─────────────────────────────────────────────────┐
│  🔒 Admin Edit Mode                              │
│  [💾 Save Changes] [❌ Cancel] [👁️ View Public] │
│  Product ID: abc123 [Copy]                      │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Sticky Bar**: Fixed at top when scrolling
- **Save Button**: Disabled until changes detected, shows loading state
- **Cancel Button**: Discards changes, exits edit mode
- **View Public**: Removes `?edit=true`, shows public view
- **Product ID**: Always visible in edit mode
- **Change Indicator**: Shows "Unsaved changes" badge if modified

---

### 2. Status & Quick Info Bar

```
┌─────────────────────────────────────────────────┐
│  Status: ✅ Available  |  Blocked: ❌ No        │
│  Type: VEHICLE  |  Category: Motorcycle        │
│  Created: Jan 15, 2024  |  Updated: Jan 20, 2024 │
│  Total Bookings: 47  |  Rating: 4.8 (23 reviews) │
└─────────────────────────────────────────────────┘
```

**Key Elements:**

- **Availability Status**: `products.isAvailable` (toggle)
- **Blocked Status**: `products.blocked` with `blockedReason`, `blockedBy`, `blockedAt`
- **Product Type**: `products.type` (VEHICLE/ACCESSORY/SERVICE)
- **Category**: `products.category`
- **Timestamps**: `createdAt`, `updatedAt`
- **Metrics**: `totalBookings`, `rating`, `reviewCount`

---

### 3. Image Management Section

```
┌─────────────────────────────────────────────────┐
│  📸 Product Images                               │
│  ─────────────────────────────────────────────── │
│  [Image 1] [Image 2] [Image 3] [+ Add Image]     │
│                                                  │
│  • Drag to reorder                              │
│  • Click to edit/delete                          │
│  • Set primary image                             │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Image Gallery**: Display `products.images` (JSON array)
- **Upload**: Add new images
- **Reorder**: Drag and drop to change order
- **Delete**: Remove images
- **Primary**: Mark first image as primary
- **Preview**: Lightbox view

---

### 4. Product Details (Editable Form)

```
┌─────────────────────────────────────────────────┐
│  📋 Basic Information                            │
│  ─────────────────────────────────────────────── │
│  Title: [Honda CRF250L Adventure        ] [Save] │
│  Description: [Rich text editor...      ]        │
│  Tags: [adventure, off-road, 250cc] [+ Add]    │
│  Area: [Chiang Mai                    ]         │
│  Category: [Motorcycle              ▼]          │
└─────────────────────────────────────────────────┘
```

**Editable Fields:**

- **Title**: `products.title` (VarChar 255)
- **Description**: `products.description` (rich text)
- **Tags**: `products.tags` (String array) - add/remove tags
- **Area**: `products.area` (VarChar 255)
- **Category**: `products.category` (VarChar 255)
- **Search Priority**: `products.searchPriority` (Int)

---

### 5. Pricing Section

```
┌─────────────────────────────────────────────────┐
│  💰 Pricing                                      │
│  ─────────────────────────────────────────────── │
│  Currency: [THB ▼]                              │
│  Daily Rate:   [800.00] THB/day                 │
│  Weekly Rate:  [5,000.00] THB/week              │
│  Monthly Rate: [18,000.00] THB/month            │
│  Deposit:      [5,000.00] THB                   │
│  ─────────────────────────────────────────────── │
│  [Calculate from Daily] [Save Pricing]          │
└─────────────────────────────────────────────────┘
```

**Fields:**

- **Currency**: `products.currency` (enum: IDR, VND, THB, USD, etc.)
- **Daily**: `products.pricePerDay` (Decimal)
- **Weekly**: `products.weeklyPrice` (Decimal, optional)
- **Monthly**: `products.monthlyPrice` (Decimal, optional)
- **Deposit**: `products.deposit` (Decimal, optional)
- **Helper**: Auto-calculate weekly/monthly from daily

---

### 6. Vehicle Specifications (if VEHICLE type)

```
┌─────────────────────────────────────────────────┐
│  🏍️ Vehicle Details                              │
│  ─────────────────────────────────────────────── │
│  Model: [CRF250L]  Year: [2023]  Plate: [ABC123]│
│  Vehicle Type: [Motorcycle ▼]                   │
│  Engine: [250] cc  Fuel: [Petrol ▼]             │
│  Transmission: [Manual ▼]  ABS: [✅ Yes]        │
│  ─────────────────────────────────────────────── │
│  Features:                                       │
│  ☑ Surf Rack  ☑ Keyless  ☑ Phone Holder        │
│  ☐ Electric  ☐ Luggage Rack                     │
│  ─────────────────────────────────────────────── │
│  Condition: [Excellent ▼]                        │
│  Fuel Level: [85%]  Battery Health: [92%]       │
│  Last Maintenance: [2024-01-10]                 │
│  Next Maintenance: [2024-04-10]                 │
└─────────────────────────────────────────────────┘
```

**Fields from `vehicle_products`:**

- **Basic**: `model`, `year`, `plate`, `vehicleType`
- **Engine**: `displacement`, `fuelType`, `transmission`, `engineSize`
- **Dimensions**: `size`, `seats`, `doors`, `luggage`
- **Features**: `surfRack`, `keyless`, `abs`, `phoneHolder`, `isElectric`
- **Electric**: `electricRange`, `electricTopSpeed`, `electricPower` (if electric)
- **Condition**: `condition`, `fuelLevel`, `batteryHealth`
- **Maintenance**: `lastMaintenanceDate`, `nextMaintenanceDate`
- **Insurance**: `insuranceInfo` (JSON editor)
- **Instructions**: `pickupInstructions`, `returnInstructions` (text areas)

---

### 7. Availability & Booking Rules

```
┌─────────────────────────────────────────────────┐
│  📅 Availability Settings                       │
│  ─────────────────────────────────────────────── │
│  Available: [✅ Yes] [Toggle]                   │
│  Blocked: [❌ No] [Block] [Reason: ___________] │
│  ─────────────────────────────────────────────── │
│  Collection Available: [✅ Yes]                  │
│  Delivery Available: [❌ No]                    │
│  Instant Booking: [✅ Yes]                      │
│  ─────────────────────────────────────────────── │
│  Minimum Booking Days: [1]                      │
│  Minimum Delivery Days: [0]                     │
└─────────────────────────────────────────────────┘
```

**Settings:**

- **Available**: `products.isAvailable` (Boolean toggle)
- **Blocked**: `products.blocked` with reason (`blockedReason`, `blockedBy`, `blockedAt`)
- **Collection**: `products.collectionAvailable` (default true)
- **Delivery**: `products.deliveryAvailable` (default false)
- **Instant Book**: `products.instantBook` (Boolean)
- **Minimum Days**: `products.minimumBookingDays` (default 1), `minimumDeliveryDays` (default 0)

---

### 8. Related Bookings Table

```
┌─────────────────────────────────────────────────┐
│  📋 Bookings (47 total)                         │
│  ─────────────────────────────────────────────── │
│  [Filter: All ▼] [Search: ________] [Export]   │
│  ─────────────────────────────────────────────── │
│  Customer    | Dates      | Status    | Total   │
│  ─────────────────────────────────────────────── │
│  John Doe    | Jan 20-25  | Confirmed | THB 4K  │
│  Jane Smith  | Jan 15-18  | Completed | THB 3K  │
│  ...                                              │
│  ─────────────────────────────────────────────── │
│  [View All Bookings] [Create New Booking]      │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Table View**: List of `bookings` filtered by `productId`
- **Columns**: Customer name, dates, status, total price, actions
- **Filters**: Status, date range, payment status
- **Search**: By customer name, rider name, booking ID
- **Actions**: View details, edit, cancel
- **Quick Actions**: Create booking, export CSV

---

### 9. Reviews Section

```
┌─────────────────────────────────────────────────┐
│  ⭐ Reviews (23 reviews, 4.8 avg)                │
│  ─────────────────────────────────────────────── │
│  [Filter: All Ratings ▼] [Sort: Newest ▼]      │
│  ─────────────────────────────────────────────── │
│  ⭐⭐⭐⭐⭐ John Doe - "Great bike!"              │
│  Jan 15, 2024 • Booking #abc123                 │
│  [View] [Edit] [Delete]                         │
│  ─────────────────────────────────────────────── │
│  ⭐⭐⭐⭐ Jane Smith - "Good condition..."        │
│  ...                                              │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Review List**: From `reviews` table filtered by `productId`
- **Display**: `riderName`, `rating`, `text`, `createdAt`
- **Link**: To booking (`bookingId`)
- **Actions**: View, edit, delete (admin only)
- **Stats**: Average rating, rating distribution

---

### 10. Analytics & Metrics

```
┌─────────────────────────────────────────────────┐
│  📊 Product Performance                         │
│  ─────────────────────────────────────────────── │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Bookings │ │ Revenue  │ │ Occupancy │       │
│  │   47     │ │ THB 180K │ │   68%    │       │
│  └──────────┘ └──────────┘ └──────────┘       │
│  ─────────────────────────────────────────────── │
│  [Occupancy Chart - Last 30 Days]              │
│  [Revenue Chart - Last 30 Days]                │
└─────────────────────────────────────────────────┘
```

**Metrics:**

- **Total Bookings**: Count from `bookings` table
- **Total Revenue**: Sum of `bookings.totalPrice`
- **Occupancy Rate**: Calculate from booking dates
- **Average Rating**: `products.rating` or `averageReviewScore`
- **Charts**: Occupancy over time, revenue trends

---

### 11. Location & Coordinates

```
┌─────────────────────────────────────────────────┐
│  📍 Location                                    │
│  ─────────────────────────────────────────────── │
│  Coordinates: [JSON Editor]                    │
│  { "lat": 18.7883, "lng": 98.9853 }            │
│  ─────────────────────────────────────────────── │
│  [Map View] [Geocode Address]                  │
│  Geohash: [w4gq8x]                             │
└─────────────────────────────────────────────────┘
```

**Fields:**

- **Coordinates**: `products.coordinates` (JSON) - lat/lng
- **Geohash**: `products.geohash` (VarChar 255)
- **Map**: Visual map with marker
- **Geocode**: Convert address to coordinates

---

### 12. Internal Admin Notes

```
┌─────────────────────────────────────────────────┐
│  📝 Admin Notes                                 │
│  ─────────────────────────────────────────────── │
│  [Rich text editor for internal notes]          │
│  • Maintenance reminders                        │
│  • Issues to address                            │
│  • Special instructions                         │
│  ─────────────────────────────────────────────── │
│  Note: These notes are only visible to admins   │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Internal Notes**: Admin-only field (may need to add to schema)
- **Rich Text**: Formatting support
- **History**: Track note changes
- **Visibility**: Admin-only, not public

---

### 13. Audit Trail / Change History

```
┌─────────────────────────────────────────────────┐
│  📜 Change History                              │
│  ─────────────────────────────────────────────── │
│  Jan 20, 2024 - Admin User                     │
│  • Updated pricePerDay: 750 → 800               │
│  • Changed isAvailable: false → true            │
│  ─────────────────────────────────────────────── │
│  Jan 15, 2024 - Admin User                     │
│  • Created product                              │
│  ─────────────────────────────────────────────── │
│  [View Full History]                            │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Track Changes**: Field-level change tracking
- **Who/When**: Admin user and timestamp
- **Before/After**: Show old and new values
- **History Table**: Full audit log (may need `history` table)

---

## 🎯 Key Admin Features

### Data Management

1. **Full CRUD Operations**

   - Create new product
   - Edit all fields
   - Delete (with confirmation)
   - Duplicate product

2. **Bulk Operations**

   - Bulk edit pricing
   - Bulk update availability
   - Bulk add tags
   - Export to CSV

3. **Quick Actions**
   - Toggle availability
   - Block/unblock product
   - Set as featured
   - Archive product

### Booking Management

1. **View Related Bookings**

   - Filter by status, date range
   - Search by customer
   - Quick status updates

2. **Create Booking**

   - Quick booking form
   - Link to existing customer
   - Set dates and pricing

3. **Booking Actions**
   - Edit booking details
   - Cancel booking
   - Update status
   - Add internal notes

### Analytics & Reporting

1. **Product Metrics**

   - Booking count and trends
   - Revenue analytics
   - Occupancy rates
   - Rating trends

2. **Performance Charts**
   - Occupancy over time
   - Revenue charts
   - Booking status distribution

### Status Management

1. **Availability Control**

   - Toggle `isAvailable`
   - Block with reason
   - Set maintenance mode

2. **Visibility Control**
   - Hide from public
   - Set search priority
   - Control in listings

---

## 🔧 Technical Implementation

### Component Structure

```
src/app/components/pages/
├── BikeDetailPage.tsx                    # Existing server component (modified)
└── BikeDetailEditWrapper.tsx            # NEW: Client wrapper for edit mode
    ├── EditModeHeader.tsx               # Admin bar with save/cancel
    ├── EditableTitle.tsx                # Title → input
    ├── EditablePricing.tsx               # Pricing → inputs
    ├── EditableDescription.tsx          # Description → textarea
    ├── EditableGallery.tsx              # Gallery → upload manager
    ├── EditableSpecs.tsx                 # Specs → form
    ├── EditableAvailability.tsx         # Availability → toggles
    └── EditableLocation.tsx             # Location → map editor
```

### Implementation Steps

1. **Modify BikeDetailPage.tsx**

   - Pass `searchParams` to wrapper
   - Wrap content in `BikeDetailEditWrapper`
   - Keep server-side data fetching

2. **Create BikeDetailEditWrapper.tsx** (Client Component)

   - Check `searchParams.edit === 'true'`
   - Verify admin authentication
   - Manage edit state and form data
   - Handle save/cancel

3. **Create Editable Components**

   - Each section gets an editable version
   - Conditional rendering based on edit mode
   - Maintain same styling and layout

4. **API Integration**
   - Use existing `/api/admin/products/[productId]` endpoint
   - PATCH for updates
   - Handle errors gracefully

### State Management

```typescript
// In BikeDetailEditWrapper.tsx
const [isEditMode, setIsEditMode] = useState(searchParams.edit === "true");
const [formData, setFormData] = useState(initialProductData);
const [hasChanges, setHasChanges] = useState(false);
const [isSaving, setIsSaving] = useState(false);

// Track changes
useEffect(() => {
  const changed =
    JSON.stringify(formData) !== JSON.stringify(initialProductData);
  setHasChanges(changed);
}, [formData]);

// Save handler
const handleSave = async () => {
  setIsSaving(true);
  try {
    await fetch(`/api/admin/products/${productId}`, {
      method: "PATCH",
      body: JSON.stringify(formData),
    });
    router.push(currentPath); // Remove ?edit=true
  } catch (error) {
    // Show error toast
  } finally {
    setIsSaving(false);
  }
};
```

### API Endpoints

- **PATCH**: `/api/admin/products/[productId]` - Update product (existing)
- **GET**: Product data already fetched in server component
- **POST**: `/api/admin/products/[productId]/images` - Upload images (if needed)

---

## 📱 Responsive Design

### Desktop (> 1024px)

- Two-column layout (form + sidebar)
- Full table views
- Inline editing
- Side-by-side comparisons

### Tablet (768px - 1024px)

- Stacked sections
- Collapsible panels
- Touch-friendly controls

### Mobile (< 768px)

- Single column
- Accordion sections
- Bottom sheet modals
- Simplified tables

---

## 🎨 Admin Design System

### Colors

- **Primary**: Blue (#3b82f6) - Actions
- **Success**: Green (#10b981) - Available/Active
- **Warning**: Amber (#f59e0b) - Attention needed
- **Error**: Red (#ef4444) - Blocked/Errors
- **Info**: Teal (#08b7b7) - Information

### Typography

- **Headings**: Bold, clear hierarchy
- **Body**: Readable, monospace for IDs/codes
- **Labels**: Medium weight, smaller

### Components

- **Forms**: Consistent input styling
- **Tables**: Sortable, filterable
- **Modals**: For edits and confirmations
- **Toasts**: For success/error messages

---

## 🔍 Data Model Alignment

### Products Table Fields Used:

- All fields editable via forms
- Relations: `company`, `vehicle`, `bookings`

### Vehicle Products Table Fields Used:

- All fields editable if product type is VEHICLE
- Conditional display based on `products.type`

### Bookings Table (for related bookings):

- Filter by `productId`
- Display: customer, dates, status, price
- Actions: view, edit, cancel

### Reviews Table (for product reviews):

- Filter by `productId`
- Display: rating, text, customer, date
- Actions: view, edit, delete

### Companies Table (for context):

- Display company info
- Link to company page
- Company-level settings

---

## 🚀 Implementation Sprints (Prioritized)

### Sprint 1: Edit Mode Infrastructure 🔴 Critical

**Goal**: Add edit mode toggle and basic editing capability

**Tasks**:

1. ⏳ Create `BikeDetailEditWrapper.tsx` client component

   - Detect `?edit=true` query param
   - Check admin authentication
   - Manage edit/view state
   - Handle URL updates

2. ⏳ Create `EditModeHeader.tsx` component

   - Floating admin bar
   - Save/Cancel buttons
   - Product ID display
   - Change indicator
   - View public link

3. ⏳ Modify `BikeDetailPage.tsx`

   - Accept `searchParams` prop
   - Wrap content in `BikeDetailEditWrapper`
   - Pass product data to wrapper

4. ⏳ Create `EditableTitle.tsx` component

   - Display title normally
   - Show input field in edit mode
   - Same styling, conditional rendering

5. ⏳ Create `EditablePricing.tsx` component

   - Display pricing normally
   - Show input fields in edit mode
   - Currency selector
   - Daily/weekly/monthly inputs

6. ⏳ API integration for saving

   - PATCH endpoint call
   - Error handling
   - Success feedback
   - URL update after save

**Acceptance Criteria**:

- Admin can toggle edit mode via `?edit=true`
- Title and pricing can be edited
- Changes save successfully
- Page maintains same look/feel
- Mobile responsive

**Estimated Effort**: 3-4 days

---

### Sprint 2: Core Field Editing 🟠 High Priority

**Goal**: Make all core product fields editable

**Tasks**:

1. ⏳ Create `EditableDescription.tsx`

   - Rich text editor (or textarea)
   - Same styling as display
   - Character count

2. ⏳ Create `EditableTags.tsx`

   - Tag display → tag input component
   - Add/remove tags
   - Autocomplete from existing tags

3. ⏳ Create `EditableAvailability.tsx`

   - Status badge → toggle switches
   - `isAvailable` toggle
   - `blocked` toggle with reason input
   - Show blocked metadata

4. ⏳ Create `EditableGallery.tsx`

   - Gallery display → image manager
   - Upload new images
   - Delete images
   - Reorder (drag & drop)
   - Preview changes

5. ⏳ Form validation

   - Required fields
   - Number validation for pricing
   - URL validation for images
   - Error messages

**Acceptance Criteria**:

- All core fields are editable
- Validation works correctly
- Images can be managed
- Changes persist after save
- UI remains consistent

**Estimated Effort**: 4-5 days

---

### Sprint 3: Vehicle Specifications Editing 🟡 Medium Priority

**Goal**: Edit vehicle-specific fields

**Tasks**:

1. ⏳ Create `EditableSpecs.tsx` component

   - Transform specs display → form
   - Conditional rendering (only if `type === VEHICLE`)
   - Group fields logically:
     - Basic: model, year, plate, vehicleType
     - Engine: displacement, fuelType, transmission
     - Features: checkboxes for surfRack, keyless, abs, etc.
     - Condition: condition, fuelLevel, batteryHealth
     - Maintenance: dates
     - Instructions: pickup/return text areas

2. ⏳ Create `EditableAdvancedSettings.tsx`

   - Collection/delivery toggles
   - Instant booking toggle
   - Minimum days inputs
   - Same location in sidebar

3. ⏳ Create `EditableLocation.tsx`

   - Map display → editable map
   - Coordinates input
   - Geohash display
   - Geocode helper

4. ⏳ Conditional field logic

   - Show/hide electric fields based on `isElectric`
   - Show/hide vehicle specs only for VEHICLE type
   - Dynamic form sections

**Acceptance Criteria**:

- Vehicle specs editable when applicable
- Conditional fields work correctly
- Location editing functional
- Form maintains layout

**Estimated Effort**: 4-5 days

---

### Sprint 4: Additional Sections Editing 🟡 Medium Priority

**Goal**: Make remaining sections editable

**Tasks**:

1. ⏳ Edit company section (if needed)

   - Company info display stays read-only
   - Link to company admin page
   - Maybe quick company edit link

2. ⏳ Edit location section

   - Map becomes editable
   - Coordinates input
   - Address editing

3. ⏳ Edit host section

   - Mostly read-only (company data)
   - Link to company management

4. ⏳ Polish and refinement

   - Consistent edit mode styling
   - Better error handling
   - Loading states
   - Success animations

**Acceptance Criteria**:

- All editable sections work
- Consistent UX throughout
- Error handling is robust
- Performance is good

**Estimated Effort**: 3-4 days

---

### Sprint 5: Admin Enhancements 🟢 Low Priority

**Goal**: Add admin-specific features and polish

**Tasks**:

1. ⏳ Quick actions menu

   - Duplicate product button
   - Block/unblock quick action
   - Link to bookings page
   - Link to analytics

2. ⏳ Admin-only sections (in edit mode)

   - Show internal notes section
   - Show change history
   - Show admin metadata (createdBy, updatedBy)

3. ⏳ Enhanced validation

   - Real-time validation
   - Field-level error messages
   - Prevent invalid saves

4. ⏳ Performance optimization

   - Debounce form inputs
   - Optimistic updates
   - Cache management

**Acceptance Criteria**:

- Quick actions work smoothly
- Admin features are accessible
- Validation is comprehensive
- Performance is excellent

**Estimated Effort**: 3-4 days

---

### Sprint 6: Analytics & Metrics Dashboard 🟢 Low Priority

**Goal**: Product performance insights

**Tasks**:

1. ⏳ Analytics cards

   - Total bookings count
   - Total revenue (sum of booking totals)
   - Average occupancy rate
   - Average rating

2. ⏳ Occupancy chart

   - Line/bar chart showing occupancy over time
   - Date range selector (last 7/30/90 days)
   - Calculate from booking dates

3. ⏳ Revenue chart

   - Revenue over time
   - Group by day/week/month
   - Currency handling

4. ⏳ Booking trends

   - Bookings by status
   - Bookings by month
   - Peak booking periods

5. ⏳ API for analytics
   - Endpoint to calculate metrics
   - Efficient queries
   - Caching if needed

**Acceptance Criteria**:

- Metrics are accurate
- Charts render correctly
- Date ranges work properly
- Performance is good

**Estimated Effort**: 5-6 days

---

### Sprint 7: Advanced Admin Features 🔵 Nice to Have

**Goal**: Enhanced admin productivity tools

**Tasks**:

1. ⏳ Duplicate product

   - Copy all product fields
   - Copy vehicle details
   - Generate new ID and slug
   - Set as draft/unavailable

2. ⏳ Bulk operations

   - Bulk edit pricing (multiple products)
   - Bulk update availability
   - Bulk add/remove tags
   - Selection checkboxes

3. ⏳ Export functionality

   - Export product data to CSV
   - Export bookings to CSV
   - Export with all related data

4. ⏳ Quick actions menu
   - Set as featured
   - Archive product
   - Duplicate
   - Share link

**Acceptance Criteria**:

- Bulk operations work correctly
- Export includes all data
- Quick actions are accessible
- No performance issues

**Estimated Effort**: 4-5 days

---

### Sprint 8: Audit Trail & Admin Notes 🔵 Nice to Have

**Goal**: Track changes and internal notes

**Tasks**:

1. ⏳ Change history component

   - Track field-level changes
   - Show before/after values
   - Display admin user and timestamp
   - Filter by date range
   - Link to `history` table if exists

2. ⏳ Admin notes section

   - Rich text editor for notes
   - Timestamp and author
   - Note history
   - May need new field in schema

3. ⏳ Activity log
   - Recent changes
   - Recent bookings
   - Recent reviews
   - Timeline view

**Acceptance Criteria**:

- All changes are tracked
- Notes can be added and edited
- History is searchable
- Performance is acceptable

**Estimated Effort**: 4-6 days

---

## 📊 Sprint Summary

| Sprint   | Priority    | Focus               | Effort   | Dependencies |
| -------- | ----------- | ------------------- | -------- | ------------ |
| Sprint 1 | 🔴 Critical | Edit Mode Setup     | 3-4 days | None         |
| Sprint 2 | 🟠 High     | Core Field Editing  | 4-5 days | Sprint 1     |
| Sprint 3 | 🟡 Medium   | Vehicle Specs       | 4-5 days | Sprint 2     |
| Sprint 4 | 🟡 Medium   | Additional Sections | 3-4 days | Sprint 2     |
| Sprint 5 | 🟢 Low      | Admin Enhancements  | 3-4 days | Sprint 2     |

**Total Estimated Effort**: 17-22 days (3.5-4.5 weeks)

**Note**: This approach is much faster since we're reusing existing UI!

---

## 🎯 Recommended Release Plan

### MVP Release (Sprint 1)

**Timeline**: 1 week
**Features**: Edit mode toggle + title/pricing editing
**Value**: Admins can make quick edits to products

### V1 Release (Sprints 1-2)

**Timeline**: 2 weeks
**Features**: All core fields editable
**Value**: Complete product editing capability

### V2 Release (Sprints 1-3)

**Timeline**: 3 weeks
**Features**: Full editing including vehicle specs
**Value**: Complete product management

### V3 Release (All Sprints)

**Timeline**: 4-5 weeks
**Features**: Everything including admin enhancements
**Value**: Polished admin editing experience

---

## 🎯 Key Benefits of This Approach

1. **Faster Development**: Reuse existing UI/components
2. **Consistency**: Admins see exactly what customers see
3. **Less Code**: No duplicate components
4. **Better UX**: Familiar interface for admins
5. **Easier Maintenance**: One page to maintain, not two

---

## 📝 Notes

- This is an **admin-only** page (requires authentication)
- Follows existing admin panel patterns
- Uses existing admin components where possible
- Integrates with existing admin API endpoints
- Maintains consistency with other admin pages

---

**Last Updated**: 2025-01-XX
**Status**: Proposal
**Owner**: Admin Team
**Access**: Admin Only
