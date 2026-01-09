## WhatsApp Floating Button — Implementation Notes for Agents

This document explains how the global WhatsApp button works, how its message is composed, and how pages can provide context (bike/shop/title) without extra queries.

### Overview

- Component: `src/app/components/WhatsAppFloatingButton.tsx` (client)
- Included globally in `src/app/layout.tsx` so it renders on every page.
- Optional helper: `src/app/components/EmitBikeContext.tsx` (client) — lets any page inject the current bike title and company name into the button message without additional fetches.
- Env var: `NEXT_PUBLIC_WA_NUMBER` (documented in `example.env`), used as the destination number for `wa.me` links.

### Message composition

The button builds a `wa.me` link with a pre-filled message. It prefers structured data in this order and gracefully falls back:

1. City

- From the URL path `/country/city/...` (preferred)
- From `?location=City, Country` search param
- From GeoIP via `/api/geo` (Vercel geo headers)

2. Dates

- From URL `?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD` (or `checkIn/checkOut`)
- Else from the global booking draft saved by the home/search UI
  - The draft emits a `window` event `bookingDraft:update` whenever dates change
- Dates are rendered in long form: `Tuesday, September 3, 2025`

3. Shop / Bike info (for bike/shop pages)

- Preferred: page injects context using `EmitBikeContext`:
  - Example: `<EmitBikeContext title={item.title} companyName={item.company?.name} />`
  - This sets `window.__BIKE_CTX__ = { title, companyName }` and fires `bikeContext:update`
- Fallbacks if missing: infer shop name from the 3rd path segment when it’s not a category; infer bike title by prettifying the last slug, removing any trailing ID-like token.

### Final message structure

```
Hi, I want to rent a {bikeType} in {City}{ optional: " with {Shop}" } from {Start Long} to {End Long}.

Here's the bike I'm looking at — {Bike Title}
Link: {current page URL}
```

Notes:

- `{bikeType}` is mapped from categories: bikes/scooters/motorcycles/cars (and electric variants).
- The bike line only appears if a title is available (injected or derived).
- One blank line precedes the bike line; the link is on its own line.

### Files and key points

- `src/app/components/WhatsAppFloatingButton.tsx`

  - Client component, uses `usePathname`/`useSearchParams`
  - Reads city from route/search, dates from URL or global draft, and optional bike context from `window.__BIKE_CTX__`
  - Listens for:
    - `bookingDraft:update` (dates)
    - `bikeContext:update` (title/company)
  - Uses `NEXT_PUBLIC_WA_NUMBER` with a default fallback number

- `src/app/components/EmitBikeContext.tsx`

  - Tiny client shim to publish `{ title, companyName }` to the button
  - Sets `window.__BIKE_CTX__` and dispatches `bikeContext:update`

- `src/app/layout.tsx`

  - Imports and renders `<WhatsAppFloatingButton />` once globally

- `example.env`
  - Adds `NEXT_PUBLIC_WA_NUMBER=` (international format, no `+`)

### How to integrate on a bike detail page

1. Ensure the page has the bike title and, if available, company name.
2. Render the emitter near the top of the page (client-safe location):

```tsx
<EmitBikeContext
  title={item.title}
  companyName={item.company?.name || undefined}
/>
```

3. Nothing else required — the global button will pick it up.

### Styling

Tailwind classes are applied for a round floating action button:

```
fixed bottom-6 right-6 z-50 bg-[#25D366] hover:bg-[#128C7E] text-white
shadow-xl hover:shadow-2xl transition-shadow rounded-full w-14 h-14
flex items-center justify-center
```

### Testing checklist

- Home page
  - Pick delivery & pickup dates → message shows long-form dates
  - Searching city populates city in message
- City browse or search pages
  - City resolves from path or `?location=`
- Bike detail page
  - `EmitBikeContext` present → exact title in message
  - If omitted → slug-derived title without trailing ID
  - Link line equals current URL
- Env
  - `NEXT_PUBLIC_WA_NUMBER` set; falls back to default if unset

### Maintenance tips

- If message format needs tweaks, edit only `WhatsAppFloatingButton.tsx`.
- If a new page type should provide a title/shop name, render `EmitBikeContext` with the relevant props.
- Be mindful that this runs client-side; avoid server-only APIs in these components.
