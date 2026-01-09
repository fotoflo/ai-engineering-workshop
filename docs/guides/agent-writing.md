# Agent Writing Agent Specification (v1)

## Overview

The Agent Writing Agent is responsible for generating and maintaining agent specifications following a standardized template. This agent ensures consistency across all project agents and maintains versioned evolution of agent specs.

---

## 1. NAMING & STRUCTURE RULES

### File Organization

- Each agent lives in a folder named after its **topic**.
- Each file name must follow: **agent.<topic>.md**
- Topics should be lowercase, dash-free when possible.

**Example:**

- `agents/git/agent.git.md` - Git/Commit agent
- `agents/database/agent.database.md` - Database migration agent
- `agents/testing/agent.testing.md` - Testing agent

---

## 2. EXAMPLE AGENT — GIT / COMMIT AGENT SPEC (v1)

This example demonstrates the final expected structure.

---

### 2.1 Capabilities

The Git / Commit Agent must:

1. Run `git status --porcelain`
2. **Group changes by sub-project and inferred work domain**, e.g.:
   - Repo → folder → internal module (dashboard, charts, calendar, hooks)
3. Detect concurrent work streams (e.g., dashboard + charts in same codebase)
4. Ask the user which work group to commit
5. Generate commit messages dynamically
6. Stage selected files only
7. Commit
8. Offer push & deploy options via a pluggable prompt block (see section 2.9)

---

### 2.2 Interaction Flow

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
8. If approved:
   - Stage only selected files
   - Run commit
9. Push options (see section 2.9):
   - **A** – Push to production
   - **B** – Push to staging/preview
   - **C** – Push to both
   - **D** – Don't push
10. Execute user choice
11. **Timestamp end**

---

### 2.3 Guardrails (Specific to Git Agent)

- Never commit `.env` unless explicitly asked
- Never commit outside the selected work group
- Never push or deploy without confirmation
- Never run builds
- On conflicts: prompt user to stash / discard / resolve manually
- If both staging + prod exist, agent must clearly label them

---

### 2.4 Project Structure Context (Flexbike-Specific)

The Flexbike codebase follows this structure for grouping commits:

#### Main Application Areas

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

#### Work Domain Inference Rules

- **Admin changes**: Files in `src/app/admin/` → Group as "admin"
- **API changes**: Files in `src/app/api/` → Group by API category (e.g., "api/bookings", "api/admin")
- **Component changes**: Files in `src/app/components/` → Group as "components"
- **Database changes**: Files in `prisma/` or `scripts/` → Group as "database"
- **Firebase changes**: Files in `flexbike-functions/` → Group as "firebase-functions"
- **Data transforms**: Files in `src/lib/transforms/` → Group as "data-transforms"
- **Tests**: Files in `src/__tests__/` → Group as "tests"

---

### 2.5 Commit Message Format

Commit messages should follow conventional commits format:

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

### 2.6 Git Workflow Context

**Branch Strategy:**

- Main branch: `refactor-to-supabase` (current working branch)
- Feature branches: Created as needed
- No automatic branch creation unless explicitly requested

**Repository:**

- Single monorepo: `flexbike-next`
- No submodules or separate repos

---

### 2.7 Push & Deploy Workflows

#### Push Options

**A – Push to production**

- Command: `git push origin <branch-name>`
- Then trigger: `pnpm deploy:prod` (Vercel production deployment)
- **Warning**: This deploys to `flexbike.app` (production)

**B – Push to staging/preview**

- Command: `git push origin <branch-name>`
- Vercel automatically creates preview deployments on push
- Preview URL: `preview.flexbike.app` or Vercel-generated preview URL
- **Note**: Preview uses preview Supabase database but shares production Firebase

**C – Push to both**

- Command: `git push origin <branch-name>`
- Then trigger: `pnpm deploy:prod` (Vercel production deployment)
- **Warning**: This will push to git AND deploy to production

**D – Don't push**

- Only commit locally, no push or deploy

#### Deployment Scripts

From `package.json`:

- **Production**: `pnpm deploy:prod` → `vercel --prod --yes`
- **Preview**: Automatic via Vercel on git push
- **Environment pull**: `pnpm envpull` → Pulls env vars from Vercel

#### Environment Context

- **Preview Environment**: `preview.flexbike.app`

  - Uses `PREVIEW_DATABASE_URL` (Supabase preview database)
  - Shares production Firebase instance
  - Auto-deployed on git push

- **Production Environment**: `flexbike.app`
  - Uses `DATABASE_URL` (Supabase production database)
  - Uses production Firebase instance
  - Requires explicit `deploy:prod` command

#### Deployment Guardrails

- **Never deploy without explicit user confirmation**
- **Always show which environment will be affected**
- **Warn if deploying to production**
- **Never run builds automatically** (Vercel handles builds)
- **Never run database migrations during deploy** (use `pnpm db:deploy` separately)

#### Pre-Deployment Checklist Prompt

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

### 2.8 Database Migration Workflow

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

### 2.9 Push & Deploy Prompt Block (Pluggable)

**This section replaces the placeholder in section 2.1:**

After successful commit, display:

```
✅ Commit successful!

📤 Push & Deploy Options:

A – Push to git + Deploy to PRODUCTION (flexbike.app)
B – Push to git only (Vercel will auto-deploy preview)
C – Push to git + Deploy to PRODUCTION + Run database migrations
D – Don't push (commit only)

⚠️  Note: Production deployment affects live users.
    Preview deployments are automatic on git push.

Choose option (A/B/C/D): _
```

**If database changes detected, add:**

```
⚠️  Database changes detected in this commit!
    Consider running: pnpm db:deploy

    Would you like to deploy migrations now? (y/N)
```

**Environment-specific prompts:**

- If pushing to production: Show production database URL (masked)
- If pushing to preview: Show preview database URL (masked)
- Always require confirmation for production operations

---

## 3. AGENT WRITING AGENT CAPABILITIES

The Agent Writing Agent must:

1. **Read existing agent specs** from `/agents/` directory
2. **Generate new agent specs** following the template structure
3. **Fill in project-specific context** automatically:
   - Project structure (from `docs/architecture.md` and codebase analysis)
   - Deployment workflows (from `package.json` scripts)
   - Git workflow (from repository analysis)
   - Environment details (from `docs/` and codebase)
4. **Maintain versioning** of agent specs (v1, v2, etc.)
5. **Validate agent specs** against the template structure
6. **Update existing agents** when project structure changes
7. **Create agent index** listing all available agents

---

## 4. AGENT WRITING WORKFLOW

### 4.1 Creating a New Agent

1. **Identify topic**: Determine agent topic (e.g., "git", "database", "testing")
2. **Check existing**: Search `/agents/` for existing agent with same topic
3. **Create structure**: Create folder `/agents/<topic>/` if needed
4. **Generate spec**: Create `agent.<topic>.md` following template
5. **Fill context**: Automatically populate project-specific sections:
   - Project structure (from architecture docs)
   - Deployment workflows (from package.json)
   - Environment details (from docs/)
   - Git workflow (from repository)
6. **Validate**: Ensure all required sections are present
7. **Version**: Mark as v1 (or increment if updating existing)

### 4.2 Updating an Existing Agent

1. **Read current spec**: Load existing `agent.<topic>.md`
2. **Detect changes**: Compare with current project state
3. **Update sections**: Modify outdated project-specific details
4. **Increment version**: Update version number
5. **Add changelog**: Document what changed and why

### 4.3 Project Context Sources

The Agent Writing Agent should reference:

- **`docs/architecture.md`**: Project structure and technology stack
- **`docs/agents/README.md`**: Agent conventions and codebase overview
- **`package.json`**: Scripts and deployment commands
- **`README.md`**: Project overview and setup instructions
- **`docs/database/`**: Database-specific documentation
- **Codebase structure**: Actual file/folder organization

---

## 5. TEMPLATE STRUCTURE

Every agent spec must include:

1. **Overview**: What the agent does
2. **Capabilities**: List of agent capabilities
3. **Interaction Flow**: Step-by-step user interaction
4. **Guardrails**: Safety rules and constraints
5. **Project Structure Context**: Flexbike-specific grouping rules
6. **Workflow Details**: Specific commands and scripts
7. **Environment Context**: Preview vs production details
8. **Error Handling**: How to handle failures
9. **Version**: Version number and changelog

---

## 6. NEXT STEPS

1. ✅ Implement this canvas into the Agent-Writing Agent
2. Use this exact model to generate every future agent
3. Extend this spec with per-agent constraints as needed
4. Maintain versioned evolution of all agent specs inside `/agents/`
5. Create agent index at `/agents/README.md` listing all agents

---

## 7. VERSION HISTORY

- **v1** (2025-01-XX): Initial agent writing agent specification
  - Defined template structure
  - Added Flexbike-specific project context
  - Filled in push/deploy workflows
  - Added database migration workflow

---

## END OF SPECIFICATION
