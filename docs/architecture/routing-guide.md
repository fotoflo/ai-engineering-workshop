### Routing Plan: Adopt /country/region/city URL structure

This document proposes how to transition Flexbike’s location routing from /country/city to a three-level structure: /country/region/city. Examples:

- usa/newyork/manhattan
- indonesia/sumatra/mentawais
- thailand/chaingmai/old-city

The plan covers URLs, DB/schema, transforms, validation, APIs, utilities, components, pages, redirects, SEO, tests, and a migration strategy.

## Goals

- Align URLs with how users search, improving SEO and CTR.
- Leverage existing normalization in transforms (regex, gazetteer, LLM) to infer region deterministically.

## Booking Flows & Routing System

Flexbike implements a sophisticated routing system with 4 distinct booking flows, each with different fee structures and user experiences. The system ensures proper URL canonicalization, fee handling, and user experience consistency.

### Core Architecture

#### URL Patterns & Fee Logic

- **Direct Flow (No Fee):** Company-scoped URLs for direct access (`isDirectBooking: true`)
- **Canonical Flow (With Fee):** Location-based URLs for SEO and marketplace discovery (`isDirectBooking: false`)
- **Legacy Redirects:** Old book-now URLs redirect to appropriate flows
- **Fee Determination:** `feeRate: isDirectBooking ? 0 : 0.15` (0% for direct, 15% for canonical)

#### Key Components

- `BikeDetailDateSelector`: Handles date selection with flow-aware URL generation
- `BookingOverviewClient`: Main booking page with flow-specific behavior
- `buildBookingUrl()` & `buildProductUrl()`: URL builders with `preferCompanyScoped` logic
- Catchall route (`[...slug]/page.tsx`): Handles all URL patterns and redirects

### 1. Direct Booking Flow (No Booking Fee)

**Purpose:** Direct company access, no platform fees
**Fee Rate:** 0% (controlled by `isDirectBooking: true`)

#### URL Patterns:

- **Company Page:** `/{companySlug}` (e.g., `/green-island-motors-thailand`)
- **Bike Detail:** `/{companySlug}/{bikeSlug}` (e.g., `/green-island-motors-thailand/yamaha-filano-fPLjsL`)
- **Booking Page:** `/{companySlug}/{bikeSlug}/book` (with query params)

#### Implementation Details:

- `BikeDetailPage` receives `isDirectBooking={true}` from catchall route (2-segment paths)
- `BikeDetailDateSelector` uses `isDirectBooking` flag for fee calculation (0% fee)
- `BookingOverviewClient` generates company-scoped booking URLs
- Bike links from booking pages use direct URLs: `/{companySlug}/{bikeSlug}`

### 2. Canonical Booking Flow (With Booking Fee)

**Purpose:** SEO-optimized marketplace discovery with platform fees
**Fee Rate:** 15% (controlled by `isDirectBooking: false`)

#### URL Patterns:

- **Company Page:** `/{country}/{region}/{city}/{companySlug}` (e.g., `/thailand/northern-thailand/chiang-mai/green-island-motors-thailand`)
- **Bike Detail:** `/{country}/{region}/{city}/{companySlug}/{bikeSlug}` (e.g., `/thailand/northern-thailand/chiang-mai/green-island-motors-thailand/yamaha-filano-fPLjsL`)
- **Booking Page:** `/{country}/{region}/{city}/{companySlug}/{bikeSlug}/book` (with query params)

#### Implementation Details:

- `BikePage` component passes `isDirectBooking={false}` to `BikeDetailPage` (5-segment paths)
- Fee calculation uses 15% booking fee in `BikeDetailDateSelector`
- `BookingOverviewClient` generates canonical booking URLs via `buildBookingUrl()`
- Bike links use canonical URLs: `/{country}/{region}/{city}/{companySlug}/{bikeSlug}`

### 3. /book-now/company-slug (Legacy Redirect)

**Purpose:** Handle old book-now URLs
**Behavior:** Server-side redirect to direct company page

#### Implementation:

- Server component redirects to `/{companySlug}`
- No booking UI - immediate redirect to direct flow
- Preserves SEO with proper metadata (noindex for redirect pages)

### 4. /book-now/{id}-{companycompressedslug} (Legacy Redirect)

**Purpose:** Handle encoded legacy URLs with database IDs
**Behavior:** Parse ID, lookup company, redirect to direct company page

#### Implementation:

- Middleware extracts company slug from URL pattern `/{id}-{companySlug}`
- Redirects to `/{companySlug}` without API call for performance
- Handles URL-encoded company names with special characters

### Critical Implementation Details

#### Flow Detection & Fee Logic

```typescript
// In useDateSelection hook (BikeDetailDateSelector)
feeRate: isDirectBooking ? 0 : 0.15 // 0% for direct, 15% for canonical

// Component prop flow
BikeDetailPage → BikeDetailDateSelector (passes isDirectBooking)
BookingOverviewClient → ImprovedBookingPickupDropoffInfo → BikeDetailDateSelector
```

#### URL Generation Logic

```typescript
// BikeDetailDateSelector.handleContinue
if (isDirectBooking && companySlug && bikeSlug) {
  // Direct booking: company-scoped URL
  bookingUrl = `/${companySlug}/${bikeSlug}/book`;
} else {
  // Canonical booking: use buildBookingUrl
  bookingUrl = buildBookingUrl({
    country, region, city, category, bikeSlug,
    path: "/book", searchParams: objParams,
    companySlug, preferCompanyScoped: Boolean(companySlug)
  });
}

// BookingOverviewClient.handleDateChange
if (isDirectBooking && companySlug && bikeSlug) {
  // Direct: company-scoped URL
  url = `/${companySlug}/${bikeSlug}/book${query}`;
} else {
  // Canonical: full location-based URL
  url = buildBookingUrl({...});
}
```

#### Component Props Flow

- `BikeDetailPage` → `BikeDetailDateSelector` (passes `isDirectBooking`)
- `BookingOverviewClient` → `ImprovedBookingPickupDropoffInfo` → `BikeDetailDateSelector`
- All date selectors must receive and respect `isDirectBooking` flag

#### Category-Scoped URL Handling

- URLs like `/country/region/city/bikes/bike-slug/book` should redirect to company-scoped
- Catchall route checks if bike has company association
- Redirects to canonical company URL if company exists: `buildCanonicalPath([country, region, city, companySlug, bikeSlug, "book"])`

#### Dialog Management

- `BikeDetailDateSelector` must close dialog after `onDateChange` callback
- Both callback and internal navigation paths need `setIsOpen(false)`

#### Bike Link Generation

```typescript
// In BookingOverviewClient - bike links from booking pages
if (isDirectBooking && companySlug && bikeSlug) {
  bikeUrl = `/${companySlug}/${bikeSlug}`; // Direct
} else {
  bikeUrl = buildProductUrl({...}); // Canonical
}
```

### Common Issues & Solutions

#### Issue: Canonical bookings redirect to direct URLs

**Root Cause:** `BikeDetailDateSelector` not checking `isDirectBooking` flag
**Solution:** Add `isDirectBooking` check in URL generation logic

#### Issue: Date changes don't close dialog

**Root Cause:** `setIsModalOpen(false)` only called in internal navigation path
**Solution:** Add `setIsOpen(false)` in `onDateChange` callback path

#### Issue: Category-scoped booking URLs (/bikes/ in path)

**Root Cause:** Bikes accessed via category listings generate category URLs
**Solution:** Redirect category-scoped booking URLs to company-scoped when bike has company

#### Issue: Bike links from booking pages use wrong URLs

**Root Cause:** `BookingOverviewClient` always uses `buildProductUrl` with `preferCompanyScoped`
**Solution:** Check `isDirectBooking` and use direct URLs for direct bookings

#### Issue: Instant booking filtering in database queries

**Root Cause:** Some queries filter by `instantBookingEnabled: true` unnecessarily
**Solution:** Remove instant booking filters for general bike availability queries

### Testing Scenarios

1. **Direct Flow Navigation:**

   - Navigate to `/company-slug/bike-slug` → `BikeDetailPage` with `isDirectBooking={true}`
   - Click "Book Now" → date selector with 0% fee
   - Select dates → redirect to `/company-slug/bike-slug/book`
   - Change dates on booking page → stay in direct flow

2. **Canonical Flow Navigation:**

   - Navigate to `/country/region/city/company-slug/bike-slug` → `BikeDetailPage` with `isDirectBooking={false}`
   - Click "Book Now" → date selector with 15% fee
   - Select dates → redirect to `/country/region/city/company-slug/bike-slug/book`
   - Change dates on booking page → stay in canonical flow

3. **Legacy Redirects:**

   - `/book-now/company-slug` → redirect to `/company-slug`
   - `/book-now/id-company-slug` → redirect to `/company-slug`

4. **Category URL Handling:**
   - Access `/country/region/city/bikes/bike-slug` → if bike has company, redirect to canonical URL
   - Access category-scoped booking URL → redirect to company-scoped booking URL

### Implementation Checklist

- [ ] `BikeDetailDateSelector` checks `isDirectBooking` for URL generation
- [ ] `BikeDetailDateSelector` closes dialog in both navigation paths
- [ ] `BookingOverviewClient` passes `isDirectBooking` to all child components
- [ ] `BookingOverviewClient` uses correct URLs for bike links based on flow
- [ ] Catchall route redirects category-scoped booking URLs to company-scoped
- [ ] Middleware handles legacy book-now URL parsing
- [ ] Server component handles simple book-now redirects
- [ ] All booking components respect `isDirectBooking` for fee calculation
- [ ] Test all 4 flow types end-to-end
- [ ] Verify no category-scoped booking URLs remain accessible

Notes:

- No legacy redirects or backward compatibility needed (pre‑launch). We will flip to the new structure directly and update all internal links accordingly.

## Current State (as of branch refactor-to-supabase)

- Additional static pages implemented:

  - `/map` - Interactive map page showing bike locations and rental companies
  - `/bali-bike-delivery` - Marketing page for bike delivery services in Bali
  - `/free-bike` - Promotions page for free bike rental offers

- Dynamic routes exist at `src/app/[country]/[city]/...` and product/company pages under those trees.
- Validation: `src/lib/data/validation.ts` validates `country`, `city`, and `category`.
- Data access:
  - Cities and countries: `src/lib/data/cities.ts`, `src/lib/data/countries.ts`.
  - Cities API: `src/app/api/cities/[country]/route.ts`.
  - Location suggestions: `src/app/api/search/locations/route.ts`.
  - Geo: `src/app/api/geo/route.ts`.
- URL helpers: `src/app/utils/routing.ts`, `src/lib/seo.ts`.
- Transforms: `scripts/firebase-to-supabase/transforms/companies.transform.ts` already extracts `admin1`, `admin2`, `area`, `normalized_city` and persists to `companies` and `company_locations`.
- DB (Prisma) relevant fields:
  - `Company`: `country`, `countrySlug`, `city`, `citySlug`, `area` (no `region` yet).
  - `CompanyLocation`: `admin1`, `admin2`, `area`, `normalizedCity`.

## Proposed URL Model

- Region becomes a first‑order path segment between country and city.
- Canonical paths:
  - Region landing: `/{country}/{region}`
  - City landing: `/{country}/{region}/{city}`
  - Category listing (country): `/{country}/{segment}` → unchanged for country‑level listings
  - Category listing (city): `/{country}/{region}/{city}/{segment}` (adds region)
  - Company detail: `/{country}/{region}/{city}/companies/{companySlug}` (adds region)
  - Product detail: `/{country}/{region}/{city}/{categoryOrCompany}/{slug}` (adds region)

Notes:

- `segment` remains one of: `search`, `scooters`, `motorcycles`, `cars`, `electric-...` per `src/app/utils/routing.ts`.

## Region Definition and Slugs

- Region corresponds to administrative level 1 (state/province/island etc.).
- Source fields: `CompanyLocation.admin1` (primary), fallback to deterministic rules by country when `admin1` missing.
- Region slug: `slugify(admin1)` using the same slugify logic as cities/countries.
- Company denormalization: add `region` and `regionSlug` to `Company` for fast lookups and indexes.

## Database Changes (Prisma)

- Add fields to `Company`:
  - `region String?`
  - `regionSlug String?`
  - Indexes: `@@index([countrySlug, regionSlug])`, `@@index([countrySlug, regionSlug, citySlug])`.
- Optional: Add `regionSlug` to `CompanyLocation` for auditing/joins (can be computed at read time; not strictly required).

Migration outline (SQL):

- `ALTER TABLE companies ADD COLUMN region VARCHAR(255);`
- `ALTER TABLE companies ADD COLUMN region_slug VARCHAR(255);`
- `CREATE INDEX idx_companies_country_region ON companies (country_slug, region_slug);`
- `CREATE INDEX idx_companies_country_region_city ON companies (country_slug, region_slug, city_slug);`

Backfill strategy:

- Prefer deterministic parser first (existing `normalizeAddress`); if confidence < 0.8 or `admin1/normalized_city` missing, resolve via `llmNormalizeBatch` on `companyAddress` values.
- Use LLM outputs to populate:
  - `region = admin1`, `regionSlug = slugify(admin1)`
  - `city = normalized_city`, `citySlug = slugify(normalized_city)` (only if missing)
  - Keep `area`, `admin2`, `postal_code` for `company_locations` audit row
- Batch parameters:
  - `maxItemsPerCall` ≈ 20; `maxCharsPerCall` ≈ 9000 characters per request (tunable via env)
  - Chunk inputs by both limits; preserve order to map results back
- Error handling:
  - If LLM returns null/invalid, retain deterministic results; as a last resort, leave `region/regionSlug` null and log for manual review

## Transforms Updates

- File: `scripts/firebase-to-supabase/transforms/companies.transform.ts`
  - When creating company rows, set:
    - `region = norm.admin1 || undefined`
    - `regionSlug = helpers.slugify(norm.admin1)` when present
  - Continue populating `city`, `country`, `area`, and `company_locations` as today.
  - Ensure `citySlug` and `countrySlug` stay consistent; add `regionSlug` where helpful in the location row for debug.

Rationale: We already normalize admin fields via deterministic and LLM paths; this extends the existing pipeline.

Input contract:

- Transform INPUTS do not change. We only enrich outputs with region/regionSlug based on normalized fields.

LLM normalization batching plan:

- Enhance `scripts/firebase-to-supabase/geo/llmNormalize.ts` with a batch API to save cost during backfills:

```
export async function llmNormalizeBatch(
  addresses: string[],
  opts?: { maxItemsPerCall?: number; maxCharsPerCall?: number }
): Promise<Array<LlmNormalized | null>>
```

- Behavior:
  - Chunk by count (default 20) and cumulative character length (default ~9000 chars) to avoid prompt clipping.
  - Single response returns a JSON array of LlmSchema objects in the same order as inputs.
  - Keep existing `llmNormalizeAddress(address)` implemented via `llmNormalizeBatch([address])`.
- Calling pattern in batch migration scripts:
  - Run deterministic parser first; collect low‑confidence or missing fields.
  - Feed unresolved addresses to `llmNormalizeBatch` in chunks; map results back by index.

### City→Region Cache (DB)

- Instead of caching full address normalizations, persist a reusable mapping of city→region per country for repeated use across companies and pages.

Schema (Prisma):

```
model CityRegionCache {
  id           String   @id @default(cuid())
  countrySlug  String
  citySlug     String   // normalized/kebab-case city identifier
  // Values from normalization
  region       String?
  regionSlug   String?
  admin1       String?
  confidence   Decimal? @db.Decimal(3,2)
  // Meta
  source       String?  // "deterministic" | "llm"
  version      Int      @default(1)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @default(now()) @updatedAt

  @@unique([countrySlug, citySlug])
  @@index([countrySlug])
  @@map("city_region_cache")
}
```

Lookup/write flow:

- Key by `(countrySlug, citySlug)` where `citySlug = slugify(normalized_city)`.
- On enrichment or route validation, check cache first; if hit with adequate confidence, use `regionSlug`.
- If miss/low confidence:
  - Call LLM with a minimal prompt based on city name and country (not full address) to obtain `admin1`.
  - Save mapping with `source = "llm"` and `confidence`.
- Deterministic parser hits should also populate the cache with `source = "deterministic"`.

## Data Layer and Validation

- New data accessors in `src/lib/data`:
  - `regions.ts`
    - `getRegionsByCountry(countrySlug): Promise<{ slug: string; name: string | null; countrySlug: string }[]>` → `distinct: [regionSlug]` from `companies`.
    - `getRegionBySlug(countrySlug, regionSlug)` → validates region existence.
  - Update `cities.ts`:
    - Add `getCitiesByRegion(countrySlug, regionSlug)` → `where: { countrySlug, regionSlug }`, `distinct: [citySlug]`.
    - Add `getCityBySlug(countrySlug, regionSlug, citySlug)`.
  - Update `validation.ts` to accept `region`:
    - Validate `country` → `region` → `city` hierarchy.
    - Keep `category` validation unchanged.

## APIs

- New endpoints:
  - `GET /api/regions/[country]` → calls `getRegionsByCountry`.
  - `GET /api/cities/[country]/[region]` → calls `getCitiesByRegion`.
- Updates:
  - `GET /api/search/locations` should return `regionSlug` in suggestions and match against city within a region where possible.
  - `GET /api/geo` should include `regionSlug` when resolvable.

Response shape additions (examples):

```
{
  display: "Canggu, Bali, Indonesia",
  city: "Canggu",
  country: "Indonesia",
  citySlug: "canggu",
  regionSlug: "bali",
  countrySlug: "indonesia",
  distanceKm?: number
}
```

## Routing Utilities

- File: `src/app/utils/routing.ts`
  - Extend `RouteState` with `region?: string`.
  - `encodeRoute`:
    - Country+city path becomes `/${country}/${region}/${city}/${category}`.
    - Country-level remains `/${country}/${segment}`.
  - `decodeRoute`:
    - Parse `[country]/[region]/[city]` for city-level.
  - `segmentFromTags`/`tagsFromSegment` unchanged.

## Components and UX

- Search input: `src/app/components/SearchBar.tsx`

  - Extend `LocationSuggestion` with `regionSlug` and include region in display text.
  - When a city is selected, preserve both `countrySlug` and `regionSlug` for path building.
  - Update calls:
    - Countries chips: unchanged.
    - Cities list: load via `GET /api/cities/[country]/[region]` once a region is selected; or when no region selected, show grouped cities by region with small region badges.
  - Submission (`encodeRoute`): pass `region` if present to build three-level path.
  - Region chips: on country pages (or when a country is selected), fetch regions via `/api/regions/[country]` and render as selectable chips that filter cities and listings.

- Product/Company pages:
  - `src/app/[country]/[city]/[category]/[slug]/page.tsx` → move to `src/app/[country]/[region]/[city]/[category]/[slug]/page.tsx` and include `region` in params and in calls to validators and SEO breadcrumbs. And also find appropraite places to show on the web page.
  - `src/app/[country]/[city]/companies/[slug]/page.tsx` → move to `src/app/[country]/[region]/[city]/companies/[slug]/page.tsx` and include region.

Incremental adoption:

- All internal links should be updated to include region immediately (use `company.regionSlug`).

## Pages and Routing Tree

- Add new route segments:
  - `src/app/[country]/[region]/page.tsx` → region landing that likely redirects to `/{country}/{region}/search`.
  - `src/app/[country]/[region]/[city]/page.tsx` (if needed) for city landing.
  - Move existing city-level category and detail routes under `[region]`.

## Non‑canonical Company Shortcut Route

- Provide a top‑level convenience route `/{companySlug}` that resolves to the canonical company page and performs a redirect or server render.
- Resolution API: `GET /api/resolve-company/[slug]` should return `{ countrySlug, regionSlug, citySlug, slug }`.
- Canonical destination: `/{countrySlug}/{regionSlug}/{citySlug}/companies/{slug}`.
- Implementation options:
  - Keep `src/middleware.ts` redirect logic (update to include `regionSlug`).
  - Or add `src/app/[companySlug]/page.tsx` that fetches and calls `redirect()` server‑side.

## Reserved Routes Centralization

- Move the reserved route list (currently a Set in `src/middleware.ts`) into a shared module, e.g. `src/lib/routing/reserved.ts`:

```
export const RESERVED_PATHS = new Set([
  "book-now",
  "for-business",
  "sitemap",
  "favicon.ico",
  "placeholder-bike.jpg",
  "placeholder-image.jpg",
  "assets",
  "api",
  "_next",
  "public",
]);

export function isReservedTopLevelPath(seg: string): boolean {
  return RESERVED_PATHS.has(seg);
}
```

- Update `middleware.ts` and any other callers to import `isReservedTopLevelPath`.

## SEO

- Update `buildCanonicalPath` usage throughout to include region.
- Breadcrumbs should include region between country and city.
- Ensure OpenGraph/Twitter URLs reflect canonical regional paths.
- Sitemap (if present) should emit region paths; if not present yet, plan to add when we implement generation.

## Migration Steps

1. Schema:
   - Add `region` and `regionSlug` to `Company` in `prisma/schema.prisma` and migrate.
   - Regenerate Prisma client and Zod types.
2. Backfill:
   - Write a one-off script to backfill `Company.region/regionSlug` from `CompanyLocation.admin1` with sensible fallbacks.
3. Transforms:
   - Update `companies.transform.ts` to set region fields and re-run if needed for reimport.
4. APIs/Data:
   - Add `/api/regions/[country]`, `/api/cities/[country]/[region]` and new data-layer functions.
   - Update `validation.ts` to validate region.
5. Routing tree:
   - Add `[region]` segment routes and update existing pages.
6. Components:
   - Update `SearchBar.tsx` suggestion shapes and `encodeRoute` usage to include region.
7. Company shortcut route:
   - Ensure `/{companySlug}` resolution includes `regionSlug` and uses centralized reserved‑paths helper.
8. Tests:
   - Update and add tests; ensure green run locally and in CI.

## Tests

- Update Playwright tests in `tests/routing-e2e.spec.ts` to use three-level city paths, e.g. `indonesia/bali/canggu/motorcycles`.
- Update unit tests in `src/__tests__/routing.unit.test.ts` for `encodeRoute`/`decodeRoute` with region.
- Add tests for `/{companySlug}` shortcut resolution and reserved‑path pass‑through.

## Open Questions

- Mostly addressed by LLM normalization with batching:
  - Prefer `admin1` from LLM/deterministic parser as `region`.
  - For city‑states or ambiguous areas, fall back to deterministic country rules; maintain a minimal static mapping list for tricky cases.

## Implementation Notes

- Keep slugs lowercase kebab-case via the existing `slugify`.
- Do not introduce query parameters for routing; keep path-based navigation as today.
- Minimize user-visible churn by landing redirects and internal links together.

If you are implementing this change, start with schema + backfill, then move routing utils and APIs, and finally flip the routes and add redirects. Update internal links in one sweep.

## Action Plan

1. Schema and Location Store

- Add `Region` and `City` models to Prisma (source of truth, not a cache)
- Migrate and regenerate Prisma + Zod

2. Seeding and Enrichment

- Write a seed script to import `src/lib/data/citiesCache.csv` into `regions` and `cities`
- Run deterministic parser; for missing region, call `llmNormalizeBatch(city,country)` in chunks; upsert `regionSlug`

3. Data Layer

- Implement `getRegionsByCountry(countrySlug)` and `getCitiesByRegion(countrySlug, regionSlug)` using DB
- Update `getCityBySlug` to accept region and read from DB
- Keep CSV as dev fallback when DB is empty

4. APIs

- Add `GET /api/regions/[country]`
- Add `GET /api/cities/[country]/[region]`
- Update `/api/search/locations` to include `regionSlug` and prefer DB lookups

5. Validation

- Extend `src/lib/data/validation.ts` to validate `country → region → city`

6. Routing Utilities

- Extend `RouteState` with `region`; update `encodeRoute` and `decodeRoute`

7. Components & Pages

- `SearchBar.tsx`: add region chips; include `regionSlug` in `LocationSuggestion`; submit 3‑segment paths
- Move city-level routes under `[region]` and pass region through to detail pages and SEO breadcrumbs

8. Transforms

- Inputs unchanged; enrich outputs (`region/regionSlug`) using DB location store first, then `llmNormalizeBatch` for misses
- Persist city→region mapping so subsequent documents reuse it

9. Company Shortcut and Reserved Paths

- Centralize reserved paths into `src/lib/routing/reserved.ts`
- Update `middleware.ts` to use it and, once available, include `regionSlug` in company redirects

10. Tests

- Update E2E to 3‑segment paths and region chips; add unit tests for encode/decode and company shortcut

11. SEO

- Ensure canonical URLs and breadcrumbs include region; update sitemaps when generated

12. Launch Checklist

- Update internal links; verify 200s for all major locations; sanity-check LLM costs; monitor logs for unresolved cities
