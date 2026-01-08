<!-- de78d2d5-0259-44fd-b792-ebaa332dcd5f fcab5f3f-c868-481f-a0ad-229c58ca7140 -->
# Import bookings/conversations/messages fully via API

## Scope

Load all bookings, conversations, and messages from Firebase via API endpoints with high success by applying transforms, Zod validation, and safe fallbacks. Use the existing API runner pattern. No dev server restarts.

## Key Decisions (confirmed)

- Placeholder userId: `9nePyM3UVseZlSFIfVoTnsoUWEI3` (vendor bookings and conversation fallbacks).
- Placeholder productId: `fPLjsLsfqpCe476PXUXG` (when referenced product is missing).
- Placeholder companyId: `Uq7KTn71fpVBjfdhIXs4` (when referenced company is missing/invalid).
- Re-import users and connect them to companies if present.
- Use relative internal API calls; avoid hardcoded host/port.
- Do not restart the dev server.
- Reuse existing conversation/message bulk-import endpoints and transforms.
- Use our API runner for both small and full imports; factor to a reusable module.

## Architecture (Reusability for bulk and single upserts)

- Service layer per entity: `src/server/services/imports/{entity}.ts`
- Expose `upsertOne(data)` and `upsertMany(list)` that encapsulate Prisma + validation.
- Consume entity transforms from `src/lib/transforms/*`.
- Endpoints
- Bulk: keep existing `src/app/api/admin/{entity}/bulk-import/route.ts`, refactor to call service `upsertMany`.
- Single upsert (CRUD-ready): add `src/app/api/admin/{entity}/upsert/route.ts` to call `upsertOne` (idempotent), ready for incremental sync and admin UI.
- API runner
- Extract shared helpers from `full-api-import.ts` into `src/lib/api/importRunner.ts` with `importViaApi(entity, items, { limit, concurrency })` used by both small and full runs.

## Changes

### 0) Users import (re-run)

- `src/lib/transforms/users.ts`
- Resolve `companyId` from `companyIDs`/`companyID`; if multiple, pick first existing; else null (log).
- `src/server/services/imports/users.ts`
- Implement `upsertOne`, `upsertMany` using Prisma and relaxed Zod on ids.
- `src/app/api/admin/users/bulk-import/route.ts`
- Call `upsertMany`.
- `src/app/api/admin/users/upsert/route.ts`
- Call `upsertOne` (for incremental sync/CRUD).

### 1) Products import

- Keep slug uniqueness in API; transforms do not append id fragments.
- Add `src/server/services/imports/products.ts` and `upsert` endpoints as above.

### 2) Bookings import

- `src/lib/transforms/bookings.ts`
- Defaults: `deposit = 0`, `requestedDelivery = false` when null/undefined.
- Vendor bookings missing `userId` → placeholder user.
- `src/app/api/admin/bookings/bulk-import/route.ts`
- If product missing → `fPLjsLsfqpCe476PXUXG`.
- If company missing/invalid → `Uq7KTn71fpVBjfdhIXs4`.
- Update mapped values before validation; batch with summaries.
- Add `src/server/services/imports/bookings.ts` and single upsert route.

### 3) Conversations import (reuse existing)

- `src/lib/transforms/conversations.ts`
- Resolve `userId`/`companyId` from multiple shapes; fallback placeholders; normalize timestamps.
- `src/app/api/admin/conversations/bulk-import/route.ts`
- Keep; refactor to call `src/server/services/imports/conversations.ts`.
- Add `src/app/api/admin/conversations/upsert/route.ts` for single upsert.

### 4) Messages import (reuse existing)

- `src/lib/transforms/messages.ts`
- Normalize timestamps/booleans; ensure `conversationId` exists (assume conversation imported first).
- `src/app/api/admin/messages/bulk-import/route.ts`
- Keep; refactor to call `src/server/services/imports/messages.ts`.
- Add `src/app/api/admin/messages/upsert/route.ts` for single upsert.

### 5) Post-Import Processing

- Orchestrator endpoint (e.g., `src/app/api/cron/post-import/route.ts`):
- Call `/api/admin/locations/backfill?mode=smart` (idempotent).
- Trigger counts updaters (product counts per location; review aggregates).
- Wire into end of full import runner.

### 6) Small Import (API runner)

- `scripts/firebase-to-supabase/small-api-import.ts`:
- Uses `src/lib/api/importRunner.ts` to import with `limit=100` (configurable) in order: products → bookings → conversations → messages.
- Pre-step: ensure users and companies are already imported (as confirmed working).
- After completion: call post-import orchestrator; print verification counts.

### 7) Full Import (Gated)

- Use the same `importRunner` with `limit = Infinity`.
- Gate behind small-import success criteria (error rate < 1%, no hard failures).
- Add npm scripts `import:small` and `import:full`.

### Incremental Sync & CRUD Readiness

- Firebase onUpdate triggers (external) will POST to single upsert routes for the changed entity.
- Admin/user UIs will use the same single upsert routes.
- All routes are idempotent and re-use the same service layer and transforms.

### Stability & DX

- Use relative `fetch('/api/...')` inside app code.
- Logs concise; sample errors and aggregate counts.

## Execution Order

Pre-step) Small import; proceed only if success criteria met.
0) Re-import users (connect to companies if present).
1) Products.
2) Bookings (apply placeholders).
3) Conversations (apply placeholders).
4) Messages (ensure conversation exists or create if we add auto-create later).
5) Post-import processing (backfill locations + update counts).

## Verification

- Summary script prints counts for bookings, conversations, messages.
- Spot-check `/indonesia/bali/canggu/bikes` renders products.
- Sample conversations/messages and verify relational integrity.

## Rollback & Safety

- No schema changes; guarded by Zod and try/catch.
- Upserts idempotent; safe to re-run.
- Never write to Firebase.

### To-dos

- [ ] Change companyId to let and update mapped data when null/placeholder in bookings bulk-import
- [ ] Use relative fetch in fetchInitialBikes to avoid dev port mismatch
- [ ] Restart dev server and verify bikes page and search API work