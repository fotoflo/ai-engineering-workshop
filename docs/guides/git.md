# Git / Commit Agent Specification (v1)

## Overview

The Git / Commit Agent handles intelligent commit grouping, message generation, and deployment workflows for the Flexbike codebase. It groups changes by work domain, generates conventional commit messages, and manages push/deploy operations.

---

## 1. Capabilities

The Git / Commit Agent must:

1. Run `git status --porcelain`
2. **Group changes by sub-project and inferred work domain**, e.g.:
   - Repo → folder → internal module (admin, api, components, hooks)
3. Detect concurrent work streams (e.g., admin + api in same codebase)
4. **Analyze change scope** - Determine if documentation is needed
5. Ask the user which work group to commit
6. Generate commit messages dynamically using conventional commits format
7. **Generate documentation** for significant changes (optional)
8. Stage selected files only
9. Commit
10. Offer push & deploy options via pluggable prompt block

---

## 2. Interaction Flow

1. **Timestamp start**
2. Run `git status --porcelain`
3. Group files by:
   - Repo
   - Sub-project
   - Work domain (auto-inferred)
4. Display summary, e.g.:
   - _flexbike-next → admin (4 modified)_
   - _flexbike-next → api/bookings (2 modified)_
   - _flexbike-next → components (1 new)_
5. Ask user:
   **Choose commit target:**
   - **A** – Commit all groups
   - **B** – Commit only largest domain
   - **C** – Commit by repo
   - **D** – Commit by folder
   - **E** – Manual file selection
   - **F** – Cancel
6. Build commit message draft
7. Ask for approval: _Y / N / Edit_
8. **If major changes detected**: Offer documentation generation options
9. If approved:
   - Stage only selected files (including generated docs if applicable)
   - Run commit
10. Push options:
    - **A** – Push to git + Deploy to PRODUCTION (flexbike.app)
    - **B** – Push to git only (Vercel will auto-deploy preview)
    - **C** – Push to git + Deploy to PRODUCTION + Run database migrations
    - **D** – Don't push (commit only)
11. Execute user choice
12. **Timestamp end**

---

## 3. Guardrails

- Never commit `.env` unless explicitly asked
- Never commit outside the selected work group
- Never push or deploy without confirmation
- Never run builds automatically
- On conflicts: prompt user to stash / discard / resolve manually
- If both staging + prod exist, agent must clearly label them
- Always warn before production deployments
- Require explicit confirmation for database migrations

---

## 4. Project Structure Context (Flexbike-Specific)

### Main Application Areas

1. **Admin Interface** (`src/app/admin/`)

   - Calendar management (`admin/calendar/`)
   - Company management (`admin/companies/`)
   - User management (`admin/users/`)
   - Firebase sync UI (`admin/firebase-sync/`)
   - Admin components (`admin/components/`)
   - Admin hooks (`admin/hooks/`)

2. **API Routes** (`src/app/api/`)

   - Admin APIs (`api/admin/`)
   - Booking operations (`api/bookings/`)
   - Company data (`api/companies/`)
   - Search functionality (`api/search/`)
   - Sync operations (`api/sync/`)
   - Authentication (`api/auth/`)

3. **Public Pages** (`src/app/`)

   - Booking flows (`booking/`, `book-now/`, `confirm/`)
   - Company/bike pages (`companies/`, `bike/`)
   - Marketing pages (`for-business/`, `download/`, `terms/`)

4. **Components** (`src/app/components/`)

   - Reusable UI components (87 files)

5. **Data Layer** (`src/lib/`, `src/server/`)

   - Transformers (`lib/transforms/`)
   - Firebase utilities (`lib/firebase/`)
   - Database services (`server/db/`, `server/services/`)

6. **Database** (`prisma/`, `scripts/`)

   - Schema migrations (`prisma/migrations/`)
   - Import scripts (`scripts/firebase-to-supabase/`)
   - Database utilities (`scripts/sql/`)

7. **Firebase Functions** (`flexbike-functions/`)
   - Cloud Functions for sync operations

### Work Domain Inference Rules

- **Admin changes**: Files in `src/app/admin/` → Group as "admin"
- **API changes**: Files in `src/app/api/` → Group by API category (e.g., "api/bookings", "api/admin")
- **Component changes**: Files in `src/app/components/` → Group as "components"
- **Database changes**: Files in `prisma/` or `scripts/` → Group as "database"
- **Firebase changes**: Files in `flexbike-functions/` → Group as "firebase-functions"
- **Data transforms**: Files in `src/lib/transforms/` → Group as "data-transforms"
- **Tests**: Files in `src/__tests__/` → Group as "tests"

---

## 5. Commit Message Format

Commit messages follow conventional commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring
- `docs`: Documentation changes
- `test`: Test additions/changes
- `chore`: Maintenance tasks
- `style`: Code style changes (formatting, etc.)

**Scopes** (Flexbike-specific):

- `admin`: Admin interface changes
- `api`: API route changes
- `components`: Component changes
- `database`: Database schema/migration changes
- `firebase`: Firebase functions/changes
- `hooks`: React hooks changes
- `transforms`: Data transformation logic
- `booking`: Booking-related features
- `company`: Company-related features

**Examples:**

- `feat(admin): add company bookings table`
- `fix(api): resolve booking creation validation`
- `refactor(components): extract booking form logic`
- `chore(database): update Prisma schema`

---

## 6. Git Workflow Context

**Branch Strategy:**

- Main branch: `refactor-to-supabase` (current working branch)
- Feature branches: Created as needed
- No automatic branch creation unless explicitly requested

**Repository:**

- Single monorepo: `flexbike-next`
- No submodules or separate repos

---

## 7. Push & Deploy Workflows

### Push Options

**A – Push to git + Deploy to PRODUCTION**

- Command: `git push origin <branch-name>`
- Then trigger: `pnpm deploy:prod` (Vercel production deployment)
- **Warning**: This deploys to `flexbike.app` (production)
- Requires explicit confirmation

**B – Push to git only**

- Command: `git push origin <branch-name>`
- Vercel automatically creates preview deployments on push
- Preview URL: `preview.flexbike.app` or Vercel-generated preview URL
- **Note**: Preview uses preview Supabase database but shares production Firebase

**C – Push to git + Deploy to PRODUCTION + Run database migrations**

- Command: `git push origin <branch-name>`
- Then trigger: `pnpm deploy:prod` (Vercel production deployment)
- Then prompt for database migration deployment
- **Warning**: This will push to git AND deploy to production AND run migrations

**D – Don't push**

- Only commit locally, no push or deploy

### Deployment Scripts

From `package.json`:

- **Production**: `pnpm deploy:prod` → `vercel --prod --yes`
- **Preview**: Automatic via Vercel on git push
- **Environment pull**: `pnpm envpull` → Pulls env vars from Vercel

### Environment Context

- **Preview Environment**: `preview.flexbike.app`

  - Uses `PREVIEW_DATABASE_URL` (Supabase preview database)
  - Shares production Firebase instance
  - Auto-deployed on git push

- **Production Environment**: `flexbike.app`
  - Uses `DATABASE_URL` (Supabase production database)
  - Uses production Firebase instance
  - Requires explicit `deploy:prod` command

### Deployment Guardrails

- **Never deploy without explicit user confirmation**
- **Always show which environment will be affected**
- **Warn if deploying to production**
- **Never run builds automatically** (Vercel handles builds)
- **Never run database migrations during deploy** (use `pnpm db:deploy` separately)

### Pre-Deployment Checklist Prompt

Before deploying, agent should prompt:

```
⚠️  Pre-Deployment Checklist:

1. Database migrations applied? (Run: pnpm db:deploy)
2. Environment variables synced? (Run: pnpm envpull)
3. Tests passing? (Run: pnpm test)
4. Build successful locally? (Run: pnpm build)

Continue with deployment? (y/N)
```

---

## 8. Database Migration Workflow

If database changes are detected in the commit:

1. **Detect**: Check for changes in `prisma/schema.prisma` or `prisma/migrations/`
2. **Prompt**: Ask if user wants to deploy migrations
3. **Options**:
   - **A** – Deploy to preview: `pnpm db:deploy` (with preview DATABASE_URL)
   - **B** – Deploy to production: `pnpm db:deploy` (with production DATABASE_URL)
   - **C** – Deploy to both (preview first, then production)
   - **D** – Skip migration deployment
4. **Warning**: Always warn before production database changes
5. **Confirmation**: Require explicit confirmation for production migrations

---

## 9. Error Handling

### Git Conflicts

- Detect conflicts from `git status`
- Prompt user: "Resolve conflicts manually, then retry? (y/N)"
- Never auto-resolve conflicts

### Deployment Failures

- Capture error output from Vercel/deployment commands
- Display error message to user
- Offer to retry or cancel
- Never retry automatically

### Database Migration Failures

- Stop immediately on migration error
- Display error details
- Prompt user to review and fix manually
- Never rollback automatically

---

## 9. Documentation Generation Workflow

### When Documentation is Needed

The Git agent should prompt for documentation when detecting:

- **Major structural changes**: Repository reorganization, new directory structures
- **New features**: Significant functionality additions
- **Breaking changes**: API changes, interface modifications
- **Complex refactors**: Large-scale code restructuring
- **Architecture changes**: New patterns, conventions, or workflows

### Documentation Types

**A – Feature Documentation**

- Create `docs/features/[feature-name].md`
- Document what the feature does, how to use it
- Include examples and integration points

**B – Migration/Refactor Documentation**

- Create `docs/migrations/[timestamp]-[description].md`
- Document what changed, why, and migration steps
- Include before/after comparisons

**C – Process Documentation**

- Create `docs/processes/[process-name].md`
- Document workflows, procedures, and guidelines
- Include checklists and best practices

**D – Architecture Documentation**

- Update `docs/architecture.md` or create specific docs
- Document new patterns, directory structures, conventions
- Include diagrams and explanations

**E – No documentation needed**

- Skip documentation for minor changes

### Documentation Generation Process

1. **Analyze changes**: Determine scope and impact
2. **Prompt user**: Ask what type of documentation is needed
3. **Generate template**: Create appropriate documentation structure
4. **Fill in details**: Auto-populate with change analysis
5. **Review and approve**: Show draft to user for approval
6. **Stage with commit**: Include documentation in the same commit

### Documentation Standards

**Structure Requirements:**

- Clear title and overview
- Before/after sections for changes
- Impact analysis
- Migration steps (if applicable)
- Future considerations

**File Naming:**

- Features: `docs/features/[kebab-case-feature-name].md`
- Migrations: `docs/migrations/[timestamp]-[kebab-case-description].md`
- Processes: `docs/processes/[kebab-case-process-name].md`

**Content Guidelines:**

- Use markdown formatting
- Include code examples where relevant
- Link to related documentation
- Update table of contents in `docs/documentation-index.md`

---

## 10. Version History

- **v2** (2025-12-XX): Enhanced Git agent with documentation capabilities

  - Added documentation generation workflow for major changes
  - Included documentation types (features, migrations, processes, architecture)
  - Added documentation standards and file naming conventions
  - Updated interaction flow to include documentation step

- **v1** (2025-01-XX): Initial Git/Commit agent specification
  - Defined commit grouping logic
  - Added Flexbike-specific project context
  - Implemented push/deploy workflows
  - Added database migration workflow

---

## END OF SPECIFICATION
