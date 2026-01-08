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
- `architecture/data-flow.md` - Data flow patterns
- `architecture/routing.md` - Next.js routing
- `architecture/deployment.md` - Infrastructure & deployment
- `architecture/websockets.md` - Real-time WebSocket implementation

### 🗄️ Database & Data

- `database/README.md` - Database overview
- `database/schema.md` - Schema reference
- `database/migration-guide.md` - Migration procedures
- `database/firebase-supabase-sync.md` - Real-time sync
- `database/query-patterns.md` - Common query patterns

### 🔌 APIs & Integration

- `api/README.md` - API overview
- `api/authentication.md` - Auth & security
- `api/search.md` - Search functionality
- `api/sitemap.md` - Sitemap generation
- `api/examples.md` - Usage examples
- `api/debugging.md` - Troubleshooting

### ✨ Features & Business Logic

- `features/README.md` - Feature overview
- `features/bookings.md` - Booking system
- `features/checkout-flow.md` - Checkout process
- `features/offline-booking.md` - Offline functionality
- `features/occupancy-calculation.md` - Availability system
- `features/admin-calendar.md` - Admin calendar
- `features/blog-proxy.md` - Blog system

### 🔧 Third-Party Integrations

- `integrations/stripe.md` - Payment processing
- `integrations/instagram.md` - Social media
- `integrations/whatsapp.md` - Messaging
- `integrations/slack.md` - Notifications

### 📚 Development Guides

- `guides/getting-started.md` - New developer setup
- `guides/testing.md` - Testing strategies
- `guides/refactoring.md` - Safe refactoring
- `guides/git-workflow.md` - Git practices
- `guides/troubleshooting.md` - Common issues
- `guides/api-testing.md` - API testing
- `guides/comprehensive-test-suite-plan.md` - Testing roadmap
- `guides/backups.md` - Backup system and procedures

### 📋 Planning & History

- `planning/completed/` - Implemented features
- `planning/proposals/` - Future proposals

---

## 🎯 When to Use What

### By Development Phase

**🚀 Getting Started**

- Read `../README.md` + `../AGENTS.md` first
- Check `guides/getting-started.md` for setup

**📝 Planning New Features**

- Review `planning/proposals/` for similar work
- Check `features/` for existing patterns

**💻 Development**

- Follow `.cursor/rules/` standards
- Use `.cursor/commands/` for common tasks
- Reference relevant `docs/` sections

**🧪 Testing**

- Use `guides/testing.md` + `guides/api-testing.md`
- Run `.cursor/commands/checkout-test.md` for checkout testing
- Follow `guides/comprehensive-test-suite-plan.md`

**🚀 Deployment**

- Use `.cursor/commands/deploy-production.md`
- Run `.cursor/commands/smoke-test.md` pre-deployment
- Check `architecture/deployment.md`

### By Role

**🔧 Backend Developer**

- Database: `database/` + `.cursor/rules/database-safety.mdc`
- APIs: `api/` + `.cursor/rules/api-routes.mdc`
- Testing: `guides/testing.md` + `guides/api-testing.md`

**⚛️ Frontend Developer**

- Components: `.cursor/rules/coding-standards.mdc`
- APIs: `api/examples.md` + `api/debugging.md`
- Features: `features/` + `features/checkout-flow.md`

**🧪 QA Engineer**

- Testing: `guides/` + `.cursor/commands/`
- APIs: `api/debugging.md`
- Process: `guides/comprehensive-test-suite-plan.md`

**🎯 Product Manager**

- Features: `features/` + `planning/`
- Planning: `planning/proposals/`
- Status: `planning/completed/`

---

## 🔍 Quick Navigation

```bash
# Find API endpoints
grep -r "GET|POST" docs/api/

# Find testing procedures
find docs/guides -name "*test*"

# Find database schemas
ls docs/database/*schema*

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

**Total Documents**: 36+ organized files
**Last Updated**: January 2025
