# PRD: Unified Search UX, URL Structure, and Canonicals

## Vision

- One consistent search experience across all surfaces. Clean, human-readable, path-based URLs with minimal query strings.
- Any route can be deep-linked and gives the same SearchBar + Results behavior.
- SEO-friendly canonicalization and 301 redirects from legacy/ambiguous paths.

## Primary Entities

- **Location**: countrySlug, citySlug
- **Category (type)**: bikes, scooters, motorcycles, cars, electric-bikes, electric-scooters, electric-motorcycles, electric-cars
- **Company and product**: companySlug, bikeSlug (productSlug)

## Core UX Components

### SearchBar

- Location input with country/city chips; instantly navigates on selection
- Category tags: Scooters, Motorcycles, Cars, Electric (multi-select with path mapping below)
- Always path-based navigation (no bike=&location= noise)
- Active tags visibly highlighted everywhere

### Results (SearchShell + Grid)

- Same presentation across pages; infinite scroll
- Accepts location + tags from path; no divergence between pages

## URL Scheme (path-first; query only for optional faceting like pagination)

### Root

- `/` → marketing homepage with SearchBar (path navigation)

### Global search

- `/search` → generic search surface; no auto-added query params
- If user types only a country (no comma), redirect to `/{country}/search`

### Country

- `/{country}/search` → canonical country landing (SearchBar + Results)
- `/{country}/{segment}` → category pages (segment mapping below)
- Canonical: `/{country}/search` (set rel=canonical to this for country category pages)

### City

- `/{country}/{city}/{segment}` → city category pages (canonical to itself)

### Category segment mapping (path ↔ tag set)

- `bikes` → [] (or multi-type combos fallback)
- `scooters` → ['scooters']
- `motorcycles` → ['motorcycles']
- `cars` → ['cars']
- `electric-bikes` → ['electric']
- `electric-scooters` → ['electric','scooters']
- `electric-motorcycles` → ['electric','motorcycles']
- `electric-cars` → ['electric','cars']
- If both scooters and motorcycles selected (no electric): → `bikes`

### Company/product

- `/{country}/{city}/companies/{companySlug}` → company detail page (canonical)
- `/{country}/{city}/{companySlug}/{bikeSlug}` → bike detail (canonical to itself)
- Shortcuts:
  - `/{companySlug}` → 301 → `/{country}/{city}/companies/{companySlug}`
  - `/{country}/{city}/{companySlug}` → 301 → `/{country}/{city}/companies/{companySlug}`
- Product slug search fallbacks:
  - `/{country}/{city}/bikes/{query}` if query equals a vehicle slug → 302 → `/{country}/{city}/bikes/{bikeSlug}`
  - Legacy `/search?bike=...` → 301 → best path equivalent when resolvable

## Navigation Rules (SearchBar)

### Selecting a city

- Build segment from tags (see mapping) and push `/{country}/{city}/{segment}`

### Selecting only a country

- Push `/{country}/search`

### Changing tags

- Resolve to the correct {segment}; stay on same location (country or city)

### Never push `/search?bike=&location=`

- Only append query (e.g., ?tags=..., cursor) when non-empty and non-canonical

## SEO/Canonicalization

### Country

- All country pages set canonical to `/{country}/search`

### City

- All city category pages canonicalize to their own path (e.g., `/{country}/{city}/scooters`)

### Redirects

- Strip legacy or noisy query-string routes (301 to canonical paths)
- Redirect unknown company short paths if resolvable

## Error/Edge Handling

### Filter out bad suggestions

- Remove "null, indonesia" from typeahead

### Fuzzy country matching

- If user enters fuzzy country text (`/search?location=Indonesian`): fuzzy-match available countries and replace to `/{country}/search`

### Unknown categories

- Fallback to `/{country}/bikes` (no tags)

## Action Plan

### Routing structure (App Router)

#### Country-level catch-all (already added)

- `app/[country]/[...rest]/page.tsx`
  - Parse first segment; map to tags; render SearchBar + SearchShell with location={country}, tags; submit re-routes path-first

#### City-level category routing

- Add `app/[country]/[city]/[...rest]/page.tsx`
  - Same logic: parse first segment to tags; render unified SearchBar + SearchShell
- Keep existing `app/[country]/[city]/bikes/page.tsx` if needed temporarily; eventually consolidate logic into [...rest]

#### Remove conflicting dynamic folders

- Ensure `app/[country]/[category]/` directory is fully removed (not just file) to eliminate "different slug names" errors

#### Country index/alias

- `app/[country]/search/page.tsx` (created) reuses CountryBikesClient (or combine with country [...rest] logic)

#### Company/product

- Keep company detail and product pages as-is
- Middleware keeps handling '/{companySlug}' → 301 → company path
- Ensure `/{country}/{city}/bikes/{query}` resolution check redirects to ``/{country}/{city}/{companySlug}/{bikeSlug}` when query matches a product slug in that city

### SearchBar behavior updates

#### Country chips

- Default on `/search`: push `/{country}/search`

#### Category tag submit

- Compute path segment from tags; push to `/{country}/{city}/{segment}` or `/{country}/{segment}`

#### Keep active tag state

- Via initialTags; reflect selections visually across all pages

### Clean URL enforcement

#### SearchShell canonical

- When not in country/city → `/search` (no bike/location params)

#### On submit from `/search`

- If both bike and location empty, stay at `/search`
- Else route to path-first (country/city/category) where possible; use tags param only for auxiliary filters (optional)

## QA checklist

- `/`, `/search`: no empty query strings; chips work; country redirects to `/{country}/search`
- `/{country}/search` loads; chips render; category routes: `/{country}/scooters`, `/{country}/electric-scooters`, etc.
- `/{country}/{city}/{segment}` loads; tag states visible
- `/{companySlug}` 301 to canonical; `/{country}/{city}/{companySlug}` 301 to `/companies/{companySlug}`
- `/{country}/{city}/bikes/{query}` slug-proof → `/bike/{id}`

## Rollout

1. Remove `app/[country]/[category]/` dir
2. Add `app/[country]/[...rest]/page.tsx` (done) + `app/[country]/[city]/[...rest]/page.tsx`
3. Verify middleware and redirects
4. Regression test all above routes and UX
5. Monitor logs for remaining dynamic name conflicts; fix by renaming/removing leftover dynamic directories

## Result

This yields one SearchBar + Results UX everywhere, with consistent path-based URLs: `/country?/city?/{segment?}/{optional-entity?}`, and clean canonicals and redirects.
