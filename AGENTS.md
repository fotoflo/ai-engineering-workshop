# Flexbike Agent Instructions

Welcome, AI agent! This document contains essential instructions for working effectively with the Flexbike codebase. Read this before making any changes.

## 🎯 Project Overview

**Flexbike** is a Next.js-based motorcycle rental marketplace that enables users to browse and book bikes from verified rental companies. The application uses Firebase for real-time data and Supabase for structured storage.

**Tech Stack:**

- **Frontend:** Next.js 16.1 (App Router), React, Tailwind CSS, Proxy, (middleware is deprecated)
- **Database:** Supabase PostgreSQL with Prisma ORM
- **Real-time:** Supabase WebSocket channels
- **Legacy Support** Firebase Firestore with bidirectional sync
- **Deployment:** Vercel with preview/production environments

## 📁 Directory Structure

### Core Application (`src/`)

- `src/app/` - Next.js App Router pages, layouts, and API routes
- `src/app/components/` - Reusable React components
- `src/app/hooks/` - Custom React hooks for data fetching
- `src/lib/` - Business logic, utilities, and transformers
- `src/server/` - Server-side utilities and database operations

### Configuration & Scripts

- `prisma/` - Database schema and migrations
- `scripts/` - Data import, testing, and maintenance scripts
- `flexbike-functions/` - Firebase Cloud Functions

### Documentation

- `docs/` - Human & AI -readable documentation (architecture, features, guides)
- `.cursor/rules/` - Persistent coding standards and patterns
- `.cursor/commands/` - Reusable workflow templates

#### 📖 Documentation Index

**Architecture & System Design:**

- `docs/architecture/architecture.md` - High-level system overview
- `docs/architecture/data-flow.md` - Data flow and state management
- `docs/architecture/routing.md` - Next.js routing patterns
- `docs/architecture/websockets.md` - Real-time WebSocket implementation

**Database & Data Management:**

- `docs/database/database-overview.md` - Database overview and navigation
- `docs/database/schema.md` - Complete schema reference
- `docs/database/migration-guide.md` - Safe migration procedures
- `docs/database/firebase-supabase-sync.md` - Real-time sync system
- `docs/database/query-patterns.md` - Common query patterns

**API Documentation:**

- `docs/api/examples.md` - API usage examples and patterns
- `docs/api/authentication.md` - Authentication and authorization
- `docs/api/search.md` - Search API and functionality
- `docs/api/examples.md` - Usage examples and integration
- `docs/api/debugging.md` - API debugging and troubleshooting

**Features & Business Logic:**

- `docs/documentation-index.md` - Complete documentation index (includes features)
- `docs/features/bookings.md` - Booking system specification
- `docs/features/checkout-flow.md` - Checkout process documentation
- `docs/features/admin-calendar.md` - Admin calendar interface

**Development Guides:**

- `docs/guides/getting-started.md` - New developer setup
- `docs/guides/testing.md` - Testing strategies and procedures
- `docs/guides/refactoring.md` - Safe code refactoring
- `docs/guides/git-workflow.md` - Git practices and workflows
- `docs/guides/troubleshooting.md` - Common issues and solutions

#### 🎯 Cursor Rules Index

**Core Development Standards:**

- `.cursor/rules/coding-standards.mdc` - TypeScript/React patterns, DRY principles, naming conventions
- `.cursor/rules/git-commits.mdc` - Conventional commit formatting, branch strategy, push/deploy workflows
- `.cursor/rules/development-workflow.mdc` - Critical safety protocols for commits, builds, deployments

**Database & Data Operations:**

- `.cursor/rules/database-safety.mdc` - Zero data loss procedures, migration strategies, backup requirements
- **Applies to:** `prisma/`, `src/lib/transforms/`, `src/server/db/`, database migration scripts

**API Development:**

- `.cursor/rules/api-routes.mdc` - Next.js API patterns, validation, error handling, database interactions
- **Applies to:** `src/app/api/`, `src/server/`, `src/lib/api/`

**Testing & Quality:**

- `.cursor/rules/testing.mdc` - Jest/React Testing Library patterns, coverage requirements, mocking strategies
- **Applies to:** `**/*.test.*`, `**/*.spec.*`, `**/__tests__/**/*`, `src/__tests__/**/*`, `tests/`, `e2e/`

**Specification Writing:**

- `.cursor/rules/specification-writing.mdc` - Standards for writing specs, agent files, documentation, and spec processes
- **Applies to:** `agents/**/*.md`, `docs/**/*.md`, `.cursor/**/*.md`, `**/*.spec.*`

#### ⚡ Cursor Commands Index

**Testing Workflows:**

- `.cursor/commands/smoke-test.md` - Automated smoke testing for deployment validation
- `.cursor/commands/checkout-test.md` - Complete checkout flow testing checklist
- `.cursor/commands/refactor-test.md` - Post-refactoring validation checklist

**Development Operations:**

- `.cursor/commands/deploy-production.md` - Safe production deployment with pre-checks and rollback procedures

#### 🔍 Quick Reference by Task

**Starting New Feature:**

- Read: `docs/documentation-index.md` (features section), `docs/planning/proposals/`
- Follow: `.cursor/rules/coding-standards.mdc`, `.cursor/rules/git-commits.mdc`
- Test: `.cursor/commands/checkout-test.md` (for checkout features)

**Database Schema Changes:**

- Read: `docs/database/migration-guide.md`, `docs/database/schema.md`
- Follow: `.cursor/rules/database-safety.mdc`
- Validate: `docs/database/query-patterns.md`

**API Development:**

- Read: `docs/api/examples.md`, `docs/api/search-debugging.md`
- Follow: `.cursor/rules/api-routes.mdc`
- Test: `docs/guides/api-testing.md`, `.cursor/commands/smoke-test.md`

**Bug Fixes & Testing:**

- Read: `docs/guides/testing.md`, `docs/guides/troubleshooting.md`
- Follow: `.cursor/rules/testing.mdc`
- Validate: `.cursor/commands/smoke-test.md`

**Production Deployment:**

- Read: `.cursor/commands/deploy-production.md`
- Follow: `.cursor/rules/development-workflow.mdc`
- Pre-check: `.cursor/commands/smoke-test.md`

**Documentation & Specifications:**

- Read: `.cursor/rules/specification-writing.mdc`
- Follow: `docs/documentation-index.md` for organization
- Reference: `docs/guides/` for specific topics

## 🔧 Working Standards

### Code Style

<ALWAYS: DRY/>

**DRY (Don't Repeat Yourself) is the #1 principle** - Never write the same code twice. Extract common patterns into reusable functions, components, hooks, and utilities.

#### DRY Fundamentals

**What DRY Means:**

- **Eliminate duplication** - If you write the same logic twice, extract it
- **Single source of truth** - Each piece of knowledge should exist in one place
- **Maintainability** - Changes in one place update everywhere

**Why DRY Matters:**

- **Bug prevention** - Fix a bug once, not in multiple places
- **Faster development** - Reuse existing solutions
- **Easier testing** - Test logic once, not everywhere it's duplicated
- **Code clarity** - Less code to understand and maintain

**DRY in Practice:**

```typescript
// ❌ BAD: Duplicated booking calculation logic
function calculateTotal(price: number, days: number) {
  return price * days * 1.1; // 10% fee
}

function calculateMonthlyTotal(price: number, months: number) {
  return price * months * 30 * 1.1; // Duplicate logic!
}

// ✅ GOOD: Single reusable function
function calculateBookingTotal(
  price: number,
  duration: number,
  durationUnit: "days" | "months"
) {
  const days = durationUnit === "months" ? duration * 30 : duration;
  return price * days * 1.1; // Single source of truth
}
```

**Common DRY Patterns:**

- **Utility functions** for repeated calculations
- **Custom hooks** for shared stateful logic
- **Shared components** for common UI patterns
- **Constants** for repeated values
- **Base classes/mixins** for shared behavior

#### DRY in Flexbike Codebase

**API Response Patterns:**

```typescript
// ❌ BAD: Duplicated API error handling
export async function getBookings() {
  try {
    const response = await fetch("/api/bookings");
    if (!response.ok) {
      throw new Error("Failed to fetch bookings");
    }
    return response.json();
  } catch (error) {
    console.error("API Error:", error);
    throw error;
  }
}

export async function getCompanies() {
  try {
    const response = await fetch("/api/companies");
    if (!response.ok) {
      throw new Error("Failed to fetch companies"); // Duplicate!
    }
    return response.json();
  } catch (error) {
    console.error("API Error:", error); // Duplicate!
    throw error;
  }
}

// ✅ GOOD: Shared API utility
export async function apiRequest<T>(endpoint: string): Promise<T> {
  try {
    const response = await fetch(endpoint);
    if (!response.ok) {
      throw new Error(`Failed to fetch ${endpoint}`);
    }
    return response.json();
  } catch (error) {
    console.error(`API Error for ${endpoint}:`, error);
    throw error;
  }
}

// Usage - DRY and consistent
export const getBookings = () => apiRequest<Booking[]>("/api/bookings");
export const getCompanies = () => apiRequest<Company[]>("/api/companies");
```

**Component Patterns:**

```typescript
// ❌ BAD: Duplicated form validation UI
function LoginForm() {
  return (
    <form>
      <input type="email" />
      <span className="text-red-500">Email is required</span> {/* Duplicate styling */}
      <input type="password" />
      <span className="text-red-500">Password is required</span> {/* Duplicate styling */}
    </form>
  );
}

function SignupForm() {
  return (
    <form>
      <input type="email" />
      <span className="text-red-500">Email is required</span> {/* Duplicate styling */}
      <input type="text" />
      <span className="text-red-500">Name is required</span> {/* Duplicate styling */}
    </form>
  );
}

// ✅ GOOD: Shared error component
function FormError({ message }: { message: string }) {
  return <span className="text-red-500 text-sm">{message}</span>;
}

function LoginForm() {
  return (
    <form>
      <input type="email" />
      <FormError message="Email is required" />
      <input type="password" />
      <FormError message="Password is required" />
    </form>
  );
}
```

**Database Query Patterns:**

```typescript
// ❌ BAD: Duplicated user queries
export async function getUserById(id: string) {
  return prisma.user.findUnique({
    where: { id },
    select: { id: true, email: true, firstName: true, lastName: true }, // Duplicate field selection
  });
}

export async function getUserProfile(id: string) {
  return prisma.user.findUnique({
    where: { id },
    select: { id: true, email: true, firstName: true, lastName: true }, // Duplicate field selection
  });
}

// ✅ GOOD: Shared user selector
const USER_BASIC_FIELDS = {
  id: true,
  email: true,
  firstName: true,
  lastName: true,
} as const;

export async function getUserById(id: string) {
  return prisma.user.findUnique({
    where: { id },
    select: USER_BASIC_FIELDS,
  });
}

export async function getUserProfile(id: string) {
  return prisma.user.findUnique({
    where: { id },
    select: USER_BASIC_FIELDS,
  });
}
```

#### DRY Detection & Refactoring

**When to Extract (Ask yourself):**

1. **Writing the same code twice?** → Extract to utility function
2. **Copy-pasting code blocks?** → Create shared component
3. **Repeating API patterns?** → Build generic API client
4. **Duplicating validation logic?** → Create validation schema
5. **Repeating styling patterns?** → Extract to design system

**Refactoring Steps:**

1. **Identify duplication** - Search codebase for similar patterns
2. **Extract common logic** - Create reusable function/component
3. **Update all usages** - Replace duplicated code with new abstraction
4. **Test thoroughly** - Ensure no regressions
5. **Document the abstraction** - Make it discoverable for future use

**DRY Benefits in Flexbike:**

- **Booking calculations** → Shared utility prevents pricing bugs
- **API error handling** → Consistent user experience
- **Form validation** → Unified validation logic
- **Database queries** → Optimized, consistent data access
- **UI components** → Consistent design system

**Remember:** When in doubt, extract it out! The Flexbike codebase thrives on DRY principles.

- **Variable names:** Clear and descriptive (never `data`, `result`, `temp`)
- **Functions:** Extract reusable logic, max 50 lines
- **Components:** Functional components with hooks
- **Types:** Full TypeScript coverage for all new code
- **Schema:** Always stored in `/schema.prisma`

### File Organization

- **Components:** `PascalCase.tsx` in `src/app/components/`
- **Hooks:** `use*.ts` in `src/app/hooks/`
- **Utilities:** `kebab-case.ts` in `src/lib/` or `src/utils/`
- **API routes:** `route.ts` in `src/app/api/*/`

### Git Workflow

- **Commits:** Conventional format (`feat(scope): description`)
- **Branches:** Feature branches from `main`
- **PRs:** Keep under 400 LOC for easier review

## 🚨 Critical Rules

### Data Integrity (ALWAYS APPLY)

- **Never** run destructive database operations without backups
- **Always** validate data transformations before sync
- **Zero data loss** guarantee for all operations
- **Manual review** required for production migrations

### Environment Safety

- **Never** commit `.env.local` or secrets
- **Always** test in preview environment first
- **Warn** before production deployments
- **Confirm** database operations explicitly

### Code Quality

- **No console errors** in browser DevTools
- **TypeScript errors** must be resolved
- **ESLint rules** must pass
- **Tests** should cover critical paths

## 🔄 Common Workflows

### Adding New Features

1. Check existing patterns in similar features **KEEP CODE DRY**
2. Create components in appropriate directories
3. Add data fetching with custom hooks
4. Update routing and navigation
5. Add tests for critical functionality

### Database Changes

1. Update `prisma/schema.prisma`
2. Run `npx prisma validate && npx prisma generate`
3. Create migration script with rollback plan
4. Test in development environment
5. Manual review before production deployment

### Component Refactoring

1. Identify reusable patterns
2. Extract to shared components
3. Update all import references
4. Maintain existing prop interfaces
5. Test all usage locations

## 🗄️ Data Management

### Firebase ↔ Supabase Sync

- **Real-time:** Cloud Functions handle automatic sync
- **Manual:** Admin interface at `/admin/firebase-sync`
- **Transformers:** Bidirectional conversion in `src/lib/transforms/`
- **Validation:** Zod schemas ensure data integrity

### Migration Safety

- **Backups:** Always create before destructive operations
- **Validation:** Test transformations with sample data
- **Rollback:** Admin interface supports reversion
- **Monitoring:** Check sync logs for errors

## 🧪 Testing Strategy

### Unit Tests

- Place in `__tests__/` alongside source files
- Test critical business logic and utilities
- Mock external dependencies (Firebase, APIs)

### Integration Tests

- Test complete user flows
- Verify database operations
- Check API endpoints

### E2E Tests

- Critical path validation
- Cross-browser compatibility
- Mobile responsiveness

## 🚀 Deployment

### Preview Environment

- Automatic on git push to feature branches
- Uses preview Supabase database
- Shares production Firebase instance
- URL: `preview.flexbike.app`

### Production Environment

- Manual deployment via `pnpm deploy:prod`
- Uses production Supabase and Firebase
- Requires explicit confirmation
- URL: `flexbike.app`

### Deployment Checklist

- [ ] Database migrations applied
- [ ] Environment variables synced
- [ ] Tests passing
- [ ] Build successful locally

## 📋 Quick Reference

### Most Important Files

- `prisma/schema.prisma` - Database schema
- `src/lib/transforms/` - Data conversion logic
- `.cursor/rules/` - Coding standards
- `docs/` - Feature documentation

### Common Commands

- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm test` - Run test suite
- `npx prisma studio` - Database browser

### Emergency Contacts

- **Data Issues:** Stop operations, contact engineering lead
- **Production Problems:** Check monitoring, prepare rollback
- **Sync Failures:** Use `/admin/firebase-sync` for manual intervention

---

**Remember:** When in doubt, search the codebase semantically, read existing patterns, and prioritize user experience. Keep changes focused and well-tested.

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:

- Invoke: Bash("openskills read <skill-name>")
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:

- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
  </usage>

<available_skills>

<skill>
<name>algorithmic-art</name>
<description>Creating algorithmic art using p5.js with seeded randomness and interactive parameter exploration. Use this when users request creating art using code, generative art, algorithmic art, flow fields, or particle systems. Create original algorithmic art rather than copying existing artists' work to avoid copyright violations.</description>
<location>project</location>
</skill>

<skill>
<name>brand-guidelines</name>
<description>Applies Anthropic's official brand colors and typography to any sort of artifact that may benefit from having Anthropic's look-and-feel. Use it when brand colors or style guidelines, visual formatting, or company design standards apply.</description>
<location>project</location>
</skill>

<skill>
<name>canvas-design</name>
<description>Create beautiful visual art in .png and .pdf documents using design philosophy. You should use this skill when the user asks to create a poster, piece of art, design, or other static piece. Create original visual designs, never copying existing artists' work to avoid copyright violations.</description>
<location>project</location>
</skill>

<skill>
<name>doc-coauthoring</name>
<description>Guide users through a structured workflow for co-authoring documentation. Use when user wants to write documentation, proposals, technical specs, decision docs, or similar structured content. This workflow helps users efficiently transfer context, refine content through iteration, and verify the doc works for readers. Trigger when user mentions writing docs, creating proposals, drafting specs, or similar documentation tasks.</description>
<location>project</location>
</skill>

<skill>
<name>docx</name>
<description>"Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. When Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"</description>
<location>project</location>
</skill>

<skill>
<name>frontend-design</name>
<description>Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.</description>
<location>project</location>
</skill>

<skill>
<name>internal-comms</name>
<description>A set of resources to help me write all kinds of internal communications, using the formats that my company likes to use. Claude should use this skill whenever asked to write some sort of internal communications (status reports, leadership updates, 3P updates, company newsletters, FAQs, incident reports, project updates, etc.).</description>
<location>project</location>
</skill>

<skill>
<name>mcp-builder</name>
<description>Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).</description>
<location>project</location>
</skill>

<skill>
<name>pdf</name>
<description>Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms. When Claude needs to fill in a PDF form or programmatically process, generate, or analyze PDF documents at scale.</description>
<location>project</location>
</skill>

<skill>
<name>pptx</name>
<description>"Presentation creation, editing, and analysis. When Claude needs to work with presentations (.pptx files) for: (1) Creating new presentations, (2) Modifying or editing content, (3) Working with layouts, (4) Adding comments or speaker notes, or any other presentation tasks"</description>
<location>project</location>
</skill>

<skill>
<name>skill-creator</name>
<description>Guide for creating effective skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.</description>
<location>project</location>
</skill>

<skill>
<name>slack-gif-creator</name>
<description>Knowledge and utilities for creating animated GIFs optimized for Slack. Provides constraints, validation tools, and animation concepts. Use when users request animated GIFs for Slack like "make me a GIF of X doing Y for Slack."</description>
<location>project</location>
</skill>

<skill>
<name>template</name>
<description>Replace with description of the skill and when Claude should use it.</description>
<location>project</location>
</skill>

<skill>
<name>theme-factory</name>
<description>Toolkit for styling artifacts with a theme. These artifacts can be slides, docs, reportings, HTML landing pages, etc. There are 10 pre-set themes with colors/fonts that you can apply to any artifact that has been creating, or can generate a new theme on-the-fly.</description>
<location>project</location>
</skill>

<skill>
<name>web-artifacts-builder</name>
<description>Suite of tools for creating elaborate, multi-component claude.ai HTML artifacts using modern frontend web technologies (React, Tailwind CSS, shadcn/ui). Use for complex artifacts requiring state management, routing, or shadcn/ui components - not for simple single-file HTML/JSX artifacts.</description>
<location>project</location>
</skill>

<skill>
<name>webapp-testing</name>
<description>Toolkit for interacting with and testing local web applications using Playwright. Supports verifying frontend functionality, debugging UI behavior, capturing browser screenshots, and viewing browser logs.</description>
<location>project</location>
</skill>

<skill>
<name>xlsx</name>
<description>"Comprehensive spreadsheet creation, editing, and analysis with support for formulas, formatting, data analysis, and visualization. When Claude needs to work with spreadsheets (.xlsx, .xlsm, .csv, .tsv, etc) for: (1) Creating new spreadsheets with formulas and formatting, (2) Reading or analyzing data, (3) Modify existing spreadsheets while preserving formulas, (4) Data analysis and visualization in spreadsheets, or (5) Recalculating formulas"</description>
<location>project</location>
</skill>

</available_skills>

<!-- SKILLS_TABLE_END -->

</skills_system>
