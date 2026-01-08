# KPI Metrics Calculation System Documentation

## Overview

The KPI metrics system in FlexBike provides comprehensive business intelligence for rental companies through four key metric cards: Revenue, Occupancy, Products & Bookings, and Active Users. These calculations provide real-time insights into company performance across multiple dimensions.

## Key Components

### 1. Revenue Calculation (`src/app/admin/hooks/useCompanyData.ts`)

#### Algorithm Overview
The revenue calculation aggregates booking values across all company bookings, using sophisticated pricing logic to ensure accuracy.

```typescript
const totalRevenue = bookings.reduce((sum, booking) => {
  let amount = 0;

  // Primary calculation: Use product rates and rental duration
  if (booking.startDate && booking.endDate && booking.product) {
    const days = Math.ceil((endDate.getTime() - startDate.getTime()) / (1000 * 60 * 60 * 24));

    const breakdown = computePricingBreakdown({
      days,
      daily: Number(booking.product.pricePerDay || 0),
      weekly: booking.product.weeklyPrice,
      monthly: booking.product.monthlyPrice,
      deliveryFee: Number(booking.deliveryCharge || 0),
      feeRate: 0.15, // 15% booking fee
    });
    amount = breakdown.total;
  } else {
    // Fallback: Use stored price data
    amount = booking.totalPrice ?? booking.price ?? 0;
  }

  // Data quality checks
  if (!Number.isFinite(amount) || amount > 1e15) {
    return sum; // Skip garbage values
  }

  return sum + amount;
}, 0);
```

#### Advanced Pricing Engine (`src/app/utils/pricing.ts`)

The system uses dynamic programming to optimize pricing across multiple rental periods:

**Algorithm Features:**
- **Multi-tier Pricing**: Supports daily, weekly, and monthly rates
- **Optimal Rate Selection**: Automatically chooses the most cost-effective combination
- **Booking Fee Calculation**: Adds 15% platform fee to all bookings
- **Delivery Fee Integration**: Includes delivery/pickup charges
- **Precision Handling**: Works in cents to prevent rounding errors

**Dynamic Programming Approach:**
```typescript
// DP array tracks minimum cost for each day count
const dp = new Array<number>(days + 1).fill(Number.POSITIVE_INFINITY);
dp[0] = 0;

// Evaluate month (30 days), week (7 days), and day (1 day) options
for (let t = 1; t <= days; t++) {
  // Prefer larger units on ties for better pricing
  if (t >= 30 && monthlyRate && dp[t - 30] + monthlyCost <= dp[t]) {
    dp[t] = dp[t - 30] + monthlyCost;
    choice[t] = 30;
  }
  // ... similar logic for weekly and daily rates
}
```

#### Currency Handling
- **Multi-currency Support**: Handles USD, IDR, THB, and other currencies
- **Primary Currency Detection**: Analyzes booking currencies to determine company's primary currency
- **Smart Formatting**: Special handling for Rupiah (IDR) with abbreviated notation (k/M)

### 2. Product Bookings & Dormant Products Calculation

#### Current Status Tracking
Products are categorized into "currently booked" vs "currently dormant" based on real-time occupancy:

```typescript
// Products currently booked = products with any occupancy in visible range
if (occupancyData.occupied > 0) {
  productsCurrentlyBooked++;
}

const productsCurrentlyDormant = totalAvailableProducts - productsCurrentlyBooked;
```

**Key Metrics:**
- **Currently Booked**: Products with at least one day occupied in the selected time period
- **Currently Dormant**: Available products with zero occupancy in the selected time period
- **Total Available**: All products marked as available for booking

#### Real-time Updates
These metrics update dynamically as users interact with the calendar:
- Date range changes trigger recalculation
- Status filter changes affect occupancy calculations
- New bookings appear immediately in the metrics

### 3. Active Users Calculation

#### Simple Count Aggregation
```typescript
activeUsers: company._count.users
```

**Data Source**: Direct count from the database relationship
- **Source**: `company._count.users` (Prisma-generated count)
- **Scope**: All registered users associated with the company
- **Real-time**: Updates when user registrations occur

### 4. Occupancy Calculation System (`src/app/admin/calendar/utils.ts`)

#### `calculateOccupancy()` Function

```typescript
export function calculateOccupancy(
  productBookings: Booking[],
  visibleDays: Date[],
  includedStatuses: string[] = [
    "draft", "requested", "accepted", "confirmed", "paid",
    "pending_payment", "review_required", "completed", "OFFLINE_BOOKING"
  ]
): { occupied: number; percentage: number }
```

**Parameters:**
- `productBookings`: Array of bookings for a specific product
- `visibleDays`: Array of dates in the current calendar view window
- `includedStatuses`: Booking statuses to include in occupancy calculation (configurable)

**Algorithm:**
1. **Total Available Days**: Count of days in the visible calendar range
2. **Occupied Days Tracking**: Uses a Set to track unique occupied days (prevents double-counting overlapping bookings)
3. **Date Normalization**: Converts all dates to UTC start-of-day for consistent comparison
4. **Inclusive Booking Period**: Considers bookings to occupy all days from start date to end date inclusive
5. **Status Filtering**: Only includes bookings with statuses in the `includedStatuses` array
6. **Overlap Handling**: Checks if booking periods overlap with the visible date range

**Return Value:**
- `occupied`: Number of unique days occupied by bookings
- `percentage`: Occupancy rate (rounded to nearest integer)

### 2. Average Occupancy Aggregation (`src/app/components/CompanyBookingCalendar.tsx`)

```typescript
// Calculate global occupancy and product stats
const averageOccupancy = totalAvailableProducts > 0
  ? Math.round(totalOccupancyPercentage / totalAvailableProducts)
  : 0;
```

**Process:**
1. **Product Iteration**: Loops through all available products for the company
2. **Individual Calculation**: Calls `calculateOccupancy()` for each product
3. **Summation**: Accumulates total occupancy percentages across all products
4. **Averaging**: Divides by number of available products, rounds to nearest integer
5. **Real-time Updates**: Recalculates whenever calendar view changes, date range changes, or status filters change

### 3. Dynamic Time Period Calculation

The system calculates occupancy over the **currently visible calendar range**, not fixed periods. This provides:

- **Real-time accuracy**: Occupancy reflects the exact time period being viewed
- **Flexible analysis**: Users can zoom in/out of different time periods
- **Performance optimization**: Only calculates for visible days, not entire date ranges

### 4. Status-Based Filtering

Users can configure which booking statuses contribute to occupancy calculations via a multi-select dropdown:

```typescript
const defaultStatuses = [
  "draft", "requested", "accepted", "confirmed", "paid",
  "pending_payment", "review_required", "completed", "OFFLINE_BOOKING"
];
```

**Benefits:**
- **Conservative vs Aggressive**: Include draft bookings for potential occupancy or exclude for confirmed-only
- **Custom Analysis**: Different stakeholders can view occupancy based on their operational definitions
- **Real-time Adjustment**: Filter changes immediately update all occupancy calculations

## Key Features

### 1. Accurate Date Handling
- **UTC Normalization**: Prevents timezone-related calculation errors
- **Inclusive Periods**: Properly counts start and end dates as occupied
- **Overlap Prevention**: Uses Set data structure to avoid double-counting overlapping bookings

### 2. Real-time Performance
- **Memoized Calculations**: useEffect dependencies ensure efficient recalculation
- **Lazy Evaluation**: Only calculates when necessary (view changes, filters change)
- **Scalable Architecture**: Handles large numbers of products and bookings efficiently

### 3. Configurable Metrics
- **Status Filtering**: Choose which booking statuses count toward occupancy
- **Time Period Flexibility**: Calculates over any visible calendar range
- **Product-Level Granularity**: Individual product occupancy plus company averages

### 4. Visual Integration
- **Color Coding**: Occupancy percentages map to intuitive color schemes (red → green)
- **KPI Cards**: Real-time display in company dashboard
- **Calendar Visualization**: Visual representation in booking calendar

## Usage Examples

### Revenue Calculation Example
```typescript
// Calculate revenue for a 7-day booking with weekly rate
const breakdown = computePricingBreakdown({
  days: 7,
  daily: 25,      // $25/day
  weekly: 150,    // $150/week (better than 7 × $25 = $175)
  monthly: 500,   // $500/month
  deliveryFee: 20,
  feeRate: 0.15   // 15% booking fee
});
// Result: { total: 195 } = $150 + $20 delivery + $25 fee
```

### Occupancy Calculation Example
```typescript
// Calculate occupancy for a single product over 30 days
const occupancy = calculateOccupancy(
  productBookings,
  generateDateRange(30), // 30 consecutive days
  ['confirmed', 'completed'] // Only confirmed bookings
);
// Result: { occupied: 15, percentage: 50 }
```

### Product Status Example
```typescript
// Calculate current product utilization
const totalProducts = company.products.filter(p => p.isAvailable).length;
const bookedProducts = productsCurrentlyBooked; // From calendar calculation
const dormantProducts = totalProducts - bookedProducts;

// Display: "8/12 products active (4 dormant)"
```

### Company Average Occupancy
```typescript
// Across all company products
const totalPercentage = products.reduce((sum, product) => {
  const occupancy = calculateOccupancy(product.bookings, visibleDays, selectedStatuses);
  return sum + occupancy.percentage;
}, 0);

const averageOccupancy = Math.round(totalPercentage / products.length);
```

## Strengths of This KPI System

### Revenue Calculation
1. **Sophisticated Pricing Engine**: Dynamic programming ensures optimal rate selection
2. **Multi-tier Rate Support**: Handles daily/weekly/monthly pricing intelligently
3. **Accurate Fee Calculation**: Proper booking fee and delivery charge inclusion
4. **Currency Intelligence**: Automatic primary currency detection and formatting
5. **Data Quality**: Robust handling of edge cases and corrupted data

### Product Utilization
1. **Real-time Status**: Live tracking of booked vs dormant products
2. **Dynamic Time Windows**: Metrics reflect current calendar view, not static periods
3. **Occupancy Integration**: Seamlessly connects with occupancy calculations
4. **Visual Clarity**: Clear "booked/total (dormant)" format

### User Metrics
1. **Simple Accuracy**: Direct database counts ensure reliability
2. **Real-time Updates**: Reflects new user registrations immediately
3. **Relational Integrity**: Leverages Prisma's automatic relationship counting

### Occupancy Metrics
1. **Mathematical Accuracy**: Proper handling of date overlaps, inclusive periods, and percentage calculations
2. **Performance Optimized**: Efficient algorithms with minimal recalculation
3. **User Configurable**: Flexible status filtering for different operational needs
4. **Real-time Updates**: Immediate feedback as users interact with calendar
5. **Scalable Design**: Works efficiently with large numbers of products and bookings
6. **Intuitive UX**: Visual color coding and clear percentage displays

## Data Flow Architecture

```
Calendar View Changes → Date Range Updates → Recalculate All KPIs → Update Dashboard
Status Filter Changes → Recalculate Occupancy → Update Booked/Dormant → Update KPIs
Booking Data Changes → Recalculate Revenue & Occupancy → Update KPIs
User Registrations → Update User Count → Update KPIs
Product Changes → Recalculate Utilization → Update KPIs
```

## Integration Points

### Calendar Component (`CompanyBookingCalendar.tsx`)
- Calculates real-time occupancy and product status
- Provides `onGlobalOccupancyChange` callback for KPI updates
- Handles status filtering for occupancy calculations

### Data Hook (`useCompanyData.ts`)
- Aggregates booking data for revenue calculation
- Manages date ranges and filters
- Provides unified data interface to components

### KPI Cards Component (`CompanyKPICards.tsx`)
- Displays formatted metrics with proper currency handling
- Shows trend indicators and descriptions
- Responsive design for mobile and desktop

This comprehensive KPI system provides rental companies with precise, actionable business intelligence across revenue, utilization, and user engagement metrics, enabling data-driven decision making and operational optimization.
