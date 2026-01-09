# Sitemap Feature – Product Specification

**Owner:** Product Management  
**Stakeholders:** SEO team, Engineering, Marketing

---

## 1. Purpose & Goals

Search engines need a complete and up-to-date map of our public pages so that potential customers discover Flexbike content quickly. Our sitemap should:

1. Enumerate **all crawl-worthy URLs** (static + dynamic) with correct canonical form.
2. Update automatically when new bikes, companies, or pages are added.
3. Follow [sitemaps.org](https://www.sitemaps.org/) XML schema and aim to keep each sitemap chunk < **5 MB** or **5 000 URLs** (our current implementation threshold).
4. Expose the file at `/sitemap.xml` and be referenced from `robots.txt`.

## 2. Scope

### In-scope

- Static marketing pages (`/`, `/for-business`, `/download`, etc.).
- Dynamic marketplace pages:
  - Company pages → `/book-now/{companyId}`
  - Bike detail pages → `/bike/{bikeId}`
- Change frequency (`<changefreq>`) & priority (`<priority>`) tags.
- Nightly regeneration (build-time or on-demand ISR).

### Out-of-scope

- Auth-gated pages (booking details, profile, etc.).
- Language/locale variants (future work).

## 3. Functional Requirements

1. `GET /sitemap.xml` returns valid XML within 200 ms.
2. Each URL node contains:
   - `<loc>` – absolute canonical URL.
   - `<lastmod>` – ISO-8601 timestamp of last meaningful content change.
   - `<changefreq>` – `daily` for dynamic inventory, `weekly` for static marketing pages.
   - `<priority>` – 1.0 for home, 0.8 for company/bike pages, 0.5 for others.
3. File size stays within limits; if exceeded, create sitemap index and split.
4. `public/robots.txt` must include: `Sitemap: https://flexbike.example.com/sitemap.xml`.
5. Route is **cache-controlled** (`Cache-Control: s-maxage=86400`) for CDN.

## 4. Non-Functional Requirements

- **Reliability:** Failure to generate sitemap must not crash the site; fallback to last known good.
- **Security:** Only public data is exposed – no PII.
- **Performance:** Handle up to 100 k bike pages without noticeable delay.

## 5. Acceptance Criteria

- [ ] Lighthouse SEO audit shows "Sitemap detected".
- [ ] Google Search Console accepts sitemap with zero errors.
- [ ] cURL returns HTTP 200 and valid XML (`xmllint --noout`).
- [ ] Automated Jest test covers XML generation for sample data.

## 6. Engineering Notes

- Data sources: Firestore collections `companies`, `bikes`.
- Suggested approach: **Edge route** that streams XML or pre-generated file deployed each build.
- Pagination: Use `LIMIT` when querying large collections to avoid OOM.

## 7. Open Questions

1. Do we need locale support (e.g., `/id/`, `/en/`) in this version?
2. Should we include blog posts once the CMS is integrated?

---

_Last updated: 2025-07-22_
