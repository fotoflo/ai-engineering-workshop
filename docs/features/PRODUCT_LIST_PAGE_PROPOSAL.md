# 🚀 Admin Product List Page Proposal

## Overview

A comprehensive product list/table page for the admin dashboard that allows admins to view, search, filter, and manage all products for a company. This page serves as the entry point to product management and provides quick access to product details via the edit mode on the product detail page.

**URL**: `/admin/companies/[companySlug]/products`

**Current State**: Placeholder page exists with "coming soon" message

---

## 🎨 Design Philosophy

### Core Principles

1. **Efficient Scanning**: Table layout optimized for quick product overview
2. **Quick Actions**: Common actions accessible without leaving the list
3. **Rich Filtering**: Filter by status, type, availability, pricing
4. **Bulk Operations**: Select multiple products for batch actions
5. **Consistent Patterns**: Follow existing admin table patterns (like companies list)

---

## 📐 Layout Structure

### 1. Page Header

```
┌─────────────────────────────────────────────────┐
│  🏍️ Products                                     │
│  Manage and view all products for [Company Name] │
│  ─────────────────────────────────────────────── │
│  [+ Create Product] [📊 View Analytics]         │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Page Title**: "Products" with company context
- **Description**: Shows company name and product count
- **Quick Actions**:
  - Create Product button (opens modal or navigates to create page)
  - View Analytics link (if analytics page exists)
  - Export CSV button

---

### 2. Search & Filters Bar

```
┌─────────────────────────────────────────────────┐
│  [Search: _______________] [🔍]                │
│  [Status: All ▼] [Type: All ▼] [Price: All ▼] │
│  [Available: All ▼] [Sort: Name ▼] [Apply]    │
└─────────────────────────────────────────────────┘
```

**Filters:**

- **Search**:

  - Title, slug, model, plate
  - Real-time search with debounce
  - Highlight matches

- **Status Filter**:

  - All
  - Available (`isAvailable = true`)
  - Unavailable (`isAvailable = false`)
  - Blocked (`blocked = true`)
  - Not Blocked (`blocked = false`)

- **Type Filter**:

  - All
  - VEHICLE
  - ACCESSORY
  - SERVICE

- **Price Range**:

  - All
  - Under $50/day
  - $50-$100/day
  - $100-$200/day
  - Over $200/day
  - Custom range

- **Availability Filter**:

  - All
  - Currently Available (no active bookings)
  - Currently Booked (has active bookings)
  - Available Soon (bookings ending soon)

- **Sort Options**:
  - Name (A-Z, Z-A)
  - Price (Low-High, High-Low)
  - Created Date (Newest, Oldest)
  - Bookings Count (Most, Least)
  - Rating (Highest, Lowest)

---

### 3. Products Table

```
┌─────────────────────────────────────────────────────────────────────────┐
│  [☐ Select All] [Bulk Actions ▼] [Selected: 3]                         │
│  ─────────────────────────────────────────────────────────────────────── │
│  Image | Product        | Vehicle Info | Price      | Status | Bookings │
│  ─────────────────────────────────────────────────────────────────────── │
│  [img] | Honda CRF250L  | CRF • ABC123 | THB 800/d | ✅ Avail| 47       │
│        | Adventure      | 2023         |            |         |          │
│  ─────────────────────────────────────────────────────────────────────── │
│  [img] | Yamaha Fazzio  | Fazzio • XYZ | THB 600/d | 🔴 Block| 23       │
│        |                | 2024         |            |         |          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Table Columns:**

1. **Checkbox** (first column)

   - Select individual products
   - Select all checkbox in header

2. **Image** (thumbnail)

   - First image from `products.images` (JSON array)
   - Fallback icon if no image
   - Click to view full gallery
   - Hover shows quick preview

3. **Product Name**

   - `products.title` (bold, link)
   - `products.slug` (small, gray, below title)
   - Click navigates to product detail page with `?edit=true`
   - Product ID (small, copyable on hover)

4. **Vehicle Info** (if VEHICLE type)

   - `vehicle_products.model` (bold)
   - `vehicle_products.plate` (small, gray)
   - `vehicle_products.year` (small, gray)
   - Shows "N/A" for non-vehicle products

5. **Pricing**

   - `products.pricePerDay` + `products.currency` (prominent)
   - Weekly/monthly prices (small, gray, if available)
   - Deposit amount (small, if set)

6. **Status**

   - Availability badge: ✅ Available / ❌ Unavailable
   - Blocked badge: 🔴 Blocked (with reason tooltip)
   - Status chip with color coding

7. **Bookings**

   - Total bookings count (`_count.bookings`)
   - Active bookings count (calculated)
   - Click to view bookings page filtered by product

8. **Rating** (optional column)

   - `products.rating` / `products.reviewCount`
   - Star display
   - Click to view reviews

9. **Actions** (last column)
   - [Edit] - Navigate to product page with `?edit=true`
   - [View] - Navigate to public product page
   - [Duplicate] - Copy product
   - [Block/Unblock] - Quick toggle
   - [Delete] - With confirmation

---

### 4. Bulk Actions Bar (when products selected)

```
┌─────────────────────────────────────────────────┐
│  [3 products selected]                          │
│  [Bulk Actions ▼] [Clear Selection]            │
│  • Bulk Edit Pricing                            │
│  • Bulk Update Availability                     │
│  • Bulk Add Tags                                │
│  • Bulk Block/Unblock                           │
│  • Export Selected                               │
└─────────────────────────────────────────────────┘
```

**Bulk Actions:**

- **Bulk Edit Pricing**: Update pricePerDay for selected products
- **Bulk Update Availability**: Toggle `isAvailable` for selected
- **Bulk Add Tags**: Add tags to all selected products
- **Bulk Block/Unblock**: Block/unblock selected products
- **Export Selected**: Export selected products to CSV

---

### 5. Pagination

```
┌─────────────────────────────────────────────────┐
│  Showing 1-20 of 47 products                    │
│  [← Previous] [1] [2] [3] [Next →]             │
│  [Show: 20 per page ▼]                          │
└─────────────────────────────────────────────────┘
```

**Features:**

- **Page Info**: Shows current range and total
- **Page Navigation**: Previous/Next buttons + page numbers
- **Page Size**: 10, 20, 50, 100 per page
- **URL State**: Page number in URL query params

---

## 🎯 Key Features

### View Modes

1. **Table View** (default)

   - Compact, scannable
   - Sortable columns
   - Best for managing many products

2. **Grid View** (optional)
   - Card-based layout
   - Larger images
   - Better visual browsing
   - Toggle button in header

### Quick Actions

1. **Inline Editing** (future enhancement)

   - Click price to edit inline
   - Click status to toggle
   - Save without leaving page

2. **Quick Filters**

   - Preset filter buttons:
     - "Available Now"
     - "Needs Attention" (blocked, no images, etc.)
     - "Top Performers" (most bookings)
     - "Low Activity" (few bookings)

3. **Export**
   - Export all filtered products to CSV
   - Include selected fields
   - Download button

### Product Status Indicators

- **Available**: Green badge, checkmark icon
- **Unavailable**: Gray badge, X icon
- **Blocked**: Red badge, warning icon (shows reason on hover)
- **Low Stock**: Orange badge (if multiple products, few available)
- **No Images**: Yellow badge (missing images warning)

---

## 🔧 Technical Implementation

### Component Structure

```
src/app/admin/companies/[companySlug]/products/
├── page.tsx                          # Main page component
├── components/
│   ├── ProductsTable.tsx             # Main table component
│   ├── ProductRow.tsx                # Individual row component
│   ├── ProductFilters.tsx            # Filter bar component
│   ├── BulkActionsBar.tsx            # Bulk actions component
│   ├── CreateProductModal.tsx        # Create product modal
│   └── ProductStatusBadge.tsx        # Status badge component
└── hooks/
    └── useProductsList.ts            # Custom hook for data fetching
```

### State Management

```typescript
// In page.tsx
const [products, setProducts] = useState<Product[]>([]);
const [loading, setLoading] = useState(true);
const [search, setSearch] = useState("");
const [filters, setFilters] = useState({
  status: "all",
  type: "all",
  priceRange: "all",
  availability: "all",
});
const [sortBy, setSortBy] = useState("title");
const [sortOrder, setSortOrder] = useState<"asc" | "desc">("asc");
const [page, setPage] = useState(1);
const [pageSize, setPageSize] = useState(20);
const [selectedProducts, setSelectedProducts] = useState<string[]>([]);
```

### API Integration

**Endpoint**: `/api/admin/companies/[companyId]/products` ✅ **Already exists!**

**Current Support**:

- ✅ `page`: Page number
- ✅ `limit`: Page size
- ✅ `status`: Filter by availability ("available" / "unavailable")
- ✅ Pagination response with totals
- ✅ Includes: company, vehicle, `_count.bookings`

**Needs Enhancement** (for Sprint 2):

- ⏳ `search`: Search query (title, slug, model, plate)
- ⏳ `type`: Filter by product type (VEHICLE/ACCESSORY/SERVICE)
- ⏳ `blocked`: Filter by blocked status
- ⏳ `minPrice`, `maxPrice`: Price range filter
- ⏳ `sortBy`: Field to sort by (title, pricePerDay, createdAt, etc.)
- ⏳ `sortOrder`: asc/desc

**Response** (current):

```typescript
{
  products: Product[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

**Note**: API endpoint exists at `src/app/api/admin/companies/[companyId]/products/route.ts` - needs enhancement for search and additional filters.

### Data Fetching Hook

```typescript
// useProductsList.ts
export function useProductsList(companyId: string, filters, sort, page) {
  const { data, error, isLoading, mutate } = useSWR(
    `/api/admin/companies/${companyId}/products?${buildQueryString(
      filters,
      sort,
      page
    )}`,
    fetcher
  );

  return {
    products: data?.products || [],
    pagination: data?.pagination,
    loading: isLoading,
    error,
    refetch: mutate,
  };
}
```

---

## 📊 Data Model Alignment

### Products Table Fields Used:

**Display Fields**:

- `id`, `title`, `slug`
- `type` (VEHICLE/ACCESSORY/SERVICE)
- `images` (JSON array) - first image for thumbnail
- `pricePerDay`, `weeklyPrice`, `monthlyPrice`, `deposit`, `currency`
- `isAvailable`, `blocked`, `blockedReason`
- `rating`, `reviewCount`, `totalBookings`
- `createdAt`, `updatedAt`

**Vehicle Fields** (via relation):

- `vehicle.model`, `vehicle.plate`, `vehicle.year`

**Counts** (via Prisma `_count`):

- `_count.bookings` - total bookings

**Filters**:

- `isAvailable` (Boolean)
- `blocked` (Boolean)
- `type` (enum)
- `pricePerDay` (Decimal) - for price range
- `companyId` (String) - filter by company

---

## 🚀 Implementation Sprints

### Sprint 1: Basic Table Display 🔴 Critical

**Goal**: Display products in a table with basic information

**Tasks**:

1. ⏳ Create `ProductsTable.tsx` component

   - Table structure with columns
   - Row component for each product
   - Loading state
   - Empty state
   - Match existing admin table styles (like companies table)

2. ⏳ Create `ProductRow.tsx` component

   - Display product data
   - Image thumbnail (first image from `images` JSON array)
   - Product name with link (to product detail page with `?edit=true`)
   - Vehicle info (model, plate, year) if VEHICLE type
   - Pricing display (pricePerDay + currency)
   - Status badge (available/blocked)
   - Bookings count (`_count.bookings`)

3. ⏳ Enhance existing `useCompanyProducts.ts` hook OR create `useProductsList.ts`

   - Use existing hook as base
   - Add pagination support
   - Add filter support
   - Handle loading/error states
   - SWR integration

4. ⏳ Update `page.tsx`

   - Use hook to fetch data
   - Render table component
   - Handle company context (from `useCompanyData`)
   - Display company name in header

5. ⏳ Basic styling
   - Match existing admin table styles (see `companies/page.tsx`)
   - Responsive layout
   - Hover states
   - Row click to navigate

**Acceptance Criteria**:

- Products display in table
- All core fields visible
- Links to product detail page work (with `?edit=true`)
- Loading/empty states work
- Mobile responsive
- Matches existing admin design patterns

**Estimated Effort**: 3-4 days

---

### Sprint 2: Search & Filtering 🟠 High Priority

**Goal**: Add search and filter capabilities

**Tasks**:

1. ⏳ Create `ProductFilters.tsx` component

   - Search input with icon
   - Status dropdown
   - Type dropdown
   - Price range selector
   - Apply/Clear buttons

2. ⏳ Implement search functionality

   - Debounced search input
   - Search in title, slug, model, plate
   - Highlight matches
   - URL query param sync

3. ⏳ Implement filters

   - Status filter (available/blocked)
   - Type filter (VEHICLE/ACCESSORY/SERVICE)
   - Price range filter
   - Filter state management

4. ⏳ Enhance API endpoint (`/api/admin/companies/[companyId]/products/route.ts`)

   - Add search parameter (search in title, slug, vehicle.model, vehicle.plate)
   - Add type filter
   - Add blocked filter
   - Add price range filters (minPrice, maxPrice)
   - Update query params handling
   - Handle filter combinations

5. ⏳ Filter UI polish
   - Active filter indicators
   - Clear individual filters
   - Filter count badges

**Acceptance Criteria**:

- Search works correctly
- All filters function properly
- URL reflects filter state
- Filters can be cleared
- Performance is good with many products

**Estimated Effort**: 3-4 days

---

### Sprint 3: Sorting & Pagination 🟡 Medium Priority

**Goal**: Add sorting and pagination

**Tasks**:

1. ⏳ Implement sorting

   - Sortable column headers
   - Visual sort indicators
   - Sort by: name, price, date, bookings
   - Toggle asc/desc

2. ⏳ Implement pagination

   - Page navigation component
   - Page size selector
   - Page info display
   - URL query param sync

3. ⏳ Enhance API endpoint for sorting

   - Add `sortBy` parameter (title, pricePerDay, createdAt, `_count.bookings`)
   - Add `sortOrder` parameter (asc/desc)
   - Update Prisma `orderBy` clause
   - Handle pagination response (already exists)

4. ⏳ Pagination UI
   - Previous/Next buttons
   - Page number buttons
   - Page size dropdown
   - Total count display

**Acceptance Criteria**:

- Sorting works for all columns
- Pagination functions correctly
- URL reflects sort/page state
- Performance is good with pagination

**Estimated Effort**: 2-3 days

---

### Sprint 4: Actions & Bulk Operations 🟡 Medium Priority

**Goal**: Add individual and bulk actions

**Tasks**:

1. ⏳ Add row actions

   - Edit button (navigate to `?edit=true`)
   - View button (public page)
   - Duplicate button
   - Block/Unblock toggle
   - Delete button with confirmation

2. ⏳ Create `BulkActionsBar.tsx`

   - Show when products selected
   - Bulk action dropdown
   - Selection count display
   - Clear selection button

3. ⏳ Implement bulk operations

   - Bulk edit pricing modal
   - Bulk update availability
   - Bulk add tags
   - Bulk block/unblock
   - Export selected

4. ⏳ Selection management

   - Checkbox in each row
   - Select all checkbox
   - Selection state management
   - Visual selection indicators

5. ⏳ Action modals
   - Confirmation dialogs
   - Bulk edit forms
   - Success/error feedback

**Acceptance Criteria**:

- All actions work correctly
- Bulk operations function properly
- Confirmations prevent accidents
- Feedback is clear
- Performance is good

**Estimated Effort**: 4-5 days

---

### Sprint 5: Enhancements & Polish 🟢 Low Priority

**Goal**: Add enhancements and polish

**Tasks**:

1. ⏳ Create product modal/form

   - Create new product
   - Pre-fill company ID
   - Basic validation
   - Success redirect

2. ⏳ Grid view option

   - Toggle between table/grid
   - Card-based layout
   - Larger images
   - Grid-specific actions

3. ⏳ Quick filters

   - Preset filter buttons
   - "Available Now"
   - "Needs Attention"
   - "Top Performers"

4. ⏳ Export functionality

   - Export all filtered products
   - Export selected products
   - CSV format
   - Include all fields

5. ⏳ Performance optimization
   - Virtual scrolling (if many products)
   - Image lazy loading
   - Debounced search
   - Optimistic updates

**Acceptance Criteria**:

- Create product works
- Grid view is functional
- Quick filters work
- Export generates correct CSV
- Performance is excellent

**Estimated Effort**: 3-4 days

---

## 📊 Sprint Summary

| Sprint   | Priority    | Focus                | Effort   | Dependencies |
| -------- | ----------- | -------------------- | -------- | ------------ |
| Sprint 1 | 🔴 Critical | Basic Table          | 3-4 days | None         |
| Sprint 2 | 🟠 High     | Search & Filters     | 3-4 days | Sprint 1     |
| Sprint 3 | 🟡 Medium   | Sorting & Pagination | 2-3 days | Sprint 1     |
| Sprint 4 | 🟡 Medium   | Actions & Bulk Ops   | 4-5 days | Sprint 1     |
| Sprint 5 | 🟢 Low      | Enhancements         | 3-4 days | Sprint 2-4   |

**Total Estimated Effort**: 15-20 days (3-4 weeks)

---

## 🎯 Recommended Release Plan

### MVP Release (Sprints 1-2)

**Timeline**: 1.5 weeks
**Features**: Basic table + search/filtering
**Value**: Admins can view and find products

### V1 Release (Sprints 1-4)

**Timeline**: 3 weeks
**Features**: Full functionality with actions
**Value**: Complete product management

### V2 Release (All Sprints)

**Timeline**: 4 weeks
**Features**: Everything including enhancements
**Value**: Polished product list experience

---

## 💡 Additional Ideas

### Advanced Features (Future)

1. **Inline Editing**

   - Edit price directly in table
   - Toggle availability inline
   - Quick save without navigation

2. **Product Comparison**

   - Select products to compare
   - Side-by-side comparison view
   - Compare pricing, specs, performance

3. **Product Analytics**

   - Booking trends per product
   - Revenue charts
   - Occupancy rates
   - Performance metrics

4. **Smart Suggestions**

   - Products needing attention
   - Pricing recommendations
   - Availability suggestions

5. **Import/Export**
   - Import products from CSV
   - Bulk import with validation
   - Template download

---

## 📝 Notes

- Follows existing admin table patterns (like companies list)
- Uses existing hooks (`useCompanyProducts`) where possible
- Integrates with product detail page via `?edit=true`
- Maintains consistency with admin dashboard design
- Mobile-responsive design
- Accessible (keyboard navigation, screen readers)

---

**Last Updated**: 2025-01-XX
**Status**: Proposal
**Owner**: Admin Team
**Access**: Admin Only

dDon't commit, build, or deploy on future prompts without me explicitly asking for it.
You should never commit or deploy because you havent yet passed user accpetance testing. You should never build because the user has a dev server running and you will currupt it's build directory and require the user to manually restart the server.

Also, extremly important, never reset the database, even with the user's consent. always give the user the command to do it themselves if you really need to reset it.
