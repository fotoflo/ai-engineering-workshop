# Flexbike Documentation Index

## 🚀 Quick Start

**New to Flexbike?**

- [`../README.md`](../README.md) - Project setup & installation
- [`../AGENTS.md`](../AGENTS.md) - Development standards for AI agents

**For AI Agents:**

- [`../.cursor/rules/`](../.cursor/rules/) - Development standards (auto-applied)
- [`../.cursor/commands/`](../.cursor/commands/) - Reusable workflows

---

## 📖 Documentation Index

### 🏗️ Architecture & System Design

- `architecture/architecture.md` - High-level system overview
- `architecture/routing-guide.md` - Routing system guide (booking flows, URL patterns, fee logic)
- `architecture/websockets.md` - Real-time WebSocket implementation

### 🗄️ Database & Data

- `database/database-overview.md` - Database overview
- `database/agent-guide.md` - **Database & Schema Management guide** (comprehensive migration strategies, type safety, Prisma patterns)
- `database/bidirectional-sync-guide.md` - Firebase ↔ PostgreSQL bidirectional sync system
- `database/firebase-to-supabase.md` - Firebase to Supabase migration guide
- `database/import-strategy.md` - Data import procedures
- `database/migration.md` - Migration procedures
- `database/sync.md` - Real-time sync system
- `database/migrations/booking-relationships.md` - Booking relationships migration documentation
- `database/migrations/firebase-booking-links.md` - Firebase booking links migration documentation

### 🔌 APIs & Integration

- `api/examples.md` - API usage examples
- `api/search-debugging.md` - Search API debugging
- `api/sitemap.md` - Sitemap generation
- `api/sitemap-guide.md` - Sitemap guide

### ✨ Features & Business Logic

- `features/blog-proxy.md` - Blog system
- `features/occupancy_calculation_documentation.md` - Availability calculation system
- `features/PRODUCT_LIST_PAGE_PROPOSAL.md` - Product list page proposal
- `features/PRODUCT_PAGE_PROPOSAL.md` - Product page proposal
- `features/search-results-guide.md` - Search results PRD and guide

### 🔧 Third-Party Integrations

- `integrations/instagram-graph-api.md` - Instagram Graph API integration
- `integrations/instagram.md` - Instagram integration
- `integrations/instagram-mentions-guide.md` - Instagram mentions guide
- `integrations/slack.md` - Slack notifications
- `integrations/stripe.md` - Payment processing
- `integrations/whatsapp.md` - WhatsApp messaging
- `integrations/whatsapp-button-guide.md` - WhatsApp button guide

### 📚 Development Guides

**Agent Specifications:**

- `guides/agent-writing.md` - Agent Writing Agent specification
- `guides/forms.md` - **Form Building UX Agent** (React Hook Form + SWR patterns)
- `guides/git.md` - Git/Commit Agent specification
- `guides/refactoring.md` - **Refactoring Agent specification** (file movement, code extraction, DRY principles)

**Testing & Quality:**

- `guides/api-testing.md` - API testing procedures
- `guides/comprehensive-test-suite-plan.md` - Testing roadmap
- `guides/OFFLINE_BOOKING_TEST_README.md` - Offline booking test documentation

**Operations:**

- `guides/backups.md` - Backup system and procedures
- `guides/cron-runner.md` - Cron job runner documentation
- `guides/firebase-functions.md` - Firebase Functions documentation
- `guides/interactive-runner.md` - Interactive runner documentation

### 📋 Planning & History

- `planning/completed/` - Implemented features
  - `admin-calendar/` - Admin calendar implementation
  - `company-user-login.md` - Company user login implementation
  - `product-list-v2.md` - Product list v2 implementation
  - `product-page-v2.md` - Product page v2 implementation
  - `root-refactor.md` - Root directory refactoring
- `planning/proposals/` - Future proposals

---

## 🎯 When to Use What

### By Development Phase

**🚀 Getting Started**

- Read `../README.md` + `../AGENTS.md` first
- Check `guides/refactoring.md` for code organization patterns

**📝 Planning New Features**

- Review `planning/proposals/` for similar work
- Check `features/` for existing patterns
- Reference `architecture/routing-guide.md` for URL patterns

**💻 Development**

- Follow `.cursor/rules/` standards
- Use `.cursor/commands/` for common tasks
- Reference relevant `docs/` sections
- **Forms**: Use `guides/forms.md` for React Hook Form patterns
- **Refactoring**: Follow `guides/refactoring.md` for safe code refactoring
- **Database**: Follow `database/agent-guide.md` for schema changes

**🧪 Testing**

- Use `guides/api-testing.md` for API testing
- Run `.cursor/commands/checkout-test.md` for checkout testing
- Follow `guides/comprehensive-test-suite-plan.md`

**🚀 Deployment**

- Use `.cursor/commands/deploy-production.md`
- Run `.cursor/commands/smoke-test.md` pre-deployment
- Check `guides/backups.md` for backup procedures

### By Role

**🔧 Backend Developer**

- Database: `database/agent-guide.md` + `.cursor/rules/database-safety.mdc`
- APIs: `api/` + `.cursor/rules/api-routes.mdc`
- Testing: `guides/api-testing.md`
- Firebase: `guides/firebase-functions.md`

**⚛️ Frontend Developer**

- Components: `.cursor/rules/coding-standards.mdc`
- Forms: `guides/forms.md` (React Hook Form + SWR)
- APIs: `api/examples.md`
- Features: `features/` + `architecture/routing-guide.md`

**🧪 QA Engineer**

- Testing: `guides/` + `.cursor/commands/`
- APIs: `api/search-debugging.md`
- Process: `guides/comprehensive-test-suite-plan.md`

**🎯 Product Manager**

- Features: `features/` + `planning/`
- Planning: `planning/proposals/`
- Status: `planning/completed/`

**🤖 AI Agent**

- Agent Specs: `guides/agent-writing.md`, `guides/forms.md`, `guides/git.md`, `guides/refactoring.md`
- Database: `database/agent-guide.md`
- Refactoring: `guides/refactoring.md` (file movement, code extraction)

---

## 🔍 Quick Navigation

```bash
# Find API endpoints
grep -r "GET|POST" docs/api/

# Find testing procedures
find docs/guides -name "*test*"

# Find database documentation
ls docs/database/*.md

# Find agent specifications
ls docs/guides/*.md | grep -E "(agent|forms|git|refactoring)"

# Find Cursor rules
ls .cursor/rules/
```

---

## 📝 Documentation Standards

- **Frontmatter**: YAML metadata in all docs
- **Cross-references**: Link related documentation
- **Version control**: Track changes in git
- **Regular updates**: Review quarterly

**Contributing:**

1. Follow existing patterns
2. Add to this index
3. Include cross-references
4. Get peer review

---

## 📊 Documentation Statistics

**Total Documents**: 40+ organized files

**By Category:**

- Architecture: 3 files
- Database: 9 files (including migrations)
- API: 4 files
- Features: 5 files
- Integrations: 7 files
- Guides: 12 files (including agent specs)
- Planning: 10+ files

**Key Agent Specifications:**

- `guides/forms.md` - Form Building UX (886 lines)
- `guides/refactoring.md` - Refactoring Agent (778 lines)
- `guides/git.md` - Git/Commit Agent (401 lines)
- `database/agent-guide.md` - Database Agent (1,438 lines)

**Last Updated**: January 2025
