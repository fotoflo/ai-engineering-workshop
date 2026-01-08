# AI Engineering Workshop

A comprehensive guide and toolkit for building AI agents and skills libraries in complex, production environments.

## 🎯 Overview

This workshop provides real-world patterns, practices, and tools for engineering AI systems that work effectively in complex environments. Learn how to build maintainable agent architectures, create reusable skills libraries, and establish robust development workflows.

## 🏗️ What's Inside

### Agent Development (`AGENTS.md`)

Complete guide for building AI agents that can:
- Navigate complex codebases effectively
- Follow consistent coding standards
- Maintain data integrity and safety
- Work with modern development workflows
- Integrate with existing toolchains

### Skills Library (`.agent/skills/`)

A collection of production-ready skills that demonstrate best practices:

- **Algorithmic Art** - Generative art with p5.js
- **Brand Guidelines** - Consistent design systems
- **Canvas Design** - Visual design tools
- **Document Co-authoring** - Collaborative documentation workflows
- **Frontend Design** - Modern UI component libraries
- **Internal Communications** - Team communication templates
- **MCP Builder** - Model Context Protocol server development
- **PDF/PPTX/DOCX** - Document manipulation toolkits
- **Web Artifacts Builder** - Multi-component web artifacts
- **Webapp Testing** - Playwright-based testing utilities
- **Spreadsheet Tools** - Excel/CSV manipulation

Each skill includes:
- Complete documentation
- Reference implementations
- Validation tools
- Usage examples

### Cursor Rules (`.cursor/`)

Development standards and workflows for AI-assisted coding:

- **Coding Standards** - TypeScript/React patterns, DRY principles
- **Database Safety** - Zero data loss procedures
- **API Development** - Next.js API patterns
- **Testing Standards** - Comprehensive testing strategies
- **Git Workflow** - Conventional commits and deployment
- **Feature Planning** - Deep clarification protocols

### Documentation (`docs/`)

Real-world documentation patterns covering:

- **Architecture** - System design and data flow
- **Database** - Migration strategies and sync systems
- **API** - Endpoint design and debugging
- **Features** - Feature specifications and proposals
- **Guides** - Development workflows and troubleshooting
- **Integrations** - Third-party service integrations

## 🚀 Getting Started

### For Agent Developers

1. **Read the Agent Guide**
   ```bash
   # Start here for understanding agent architecture
   cat AGENTS.md
   ```

2. **Explore Skills**
   ```bash
   # Browse available skills
   ls .agent/skills/
   
   # Read a skill's documentation
   cat .agent/skills/frontend-design/SKILL.md
   ```

3. **Review Cursor Rules**
   ```bash
   # Understand development standards
   ls .cursor/rules/
   ```

### For Skills Library Developers

1. **Study Existing Skills**
   - Review `.agent/skills/` for implementation patterns
   - Each skill includes a `SKILL.md` with complete documentation
   - Check `LICENSE.txt` files for usage rights

2. **Create Your Own Skill**
   - Use `.agent/skills/skill-creator/` as a template
   - Follow the patterns established in existing skills
   - Include comprehensive documentation

3. **Validate Your Skill**
   - Use validation tools in `skill-creator/scripts/`
   - Ensure your skill follows the established patterns

## 📚 Key Concepts

### Agent Architecture

Agents in complex environments need:
- **Clear instructions** - Comprehensive, structured guidance
- **Safety protocols** - Never commit/build/deploy without permission
- **Data integrity** - Zero data loss guarantees
- **Type safety** - Full TypeScript coverage
- **Testing** - Comprehensive test coverage

### Skills Library Design

Effective skills should:
- **Be self-contained** - Include all necessary resources
- **Have clear documentation** - Explain when and how to use
- **Include examples** - Show real-world usage patterns
- **Be reusable** - Work across different projects
- **Follow standards** - Consistent structure and naming

### Development Workflow

Production-ready workflows include:
- **DRY principles** - Don't repeat yourself
- **Conventional commits** - Clear, structured commit messages
- **Safety checks** - Never auto-commit or auto-deploy
- **Testing** - Unit, integration, and E2E tests
- **Documentation** - Keep docs in sync with code

## 🛠️ Best Practices

### Code Quality

- **Clear variable names** - Never use `data`, `result`, `temp`
- **Extract reusable logic** - Functions max 50 lines
- **Type safety first** - Full TypeScript coverage
- **Component patterns** - Functional components with hooks

### Database Operations

- **Never reset without backups** - Zero data loss guarantee
- **Validate transformations** - Test before sync
- **Manual review** - Required for production migrations
- **Schema-first** - Always update Prisma schema first

### API Development

- **Validate all inputs** - Use Zod schemas
- **Consistent error handling** - Proper error types
- **Type-safe responses** - Use generated types
- **Transaction safety** - Use transactions for multi-step operations

## 📖 Documentation Structure

```
docs/
├── architecture/     # System design and patterns
├── database/         # Schema, migrations, sync
├── api/              # Endpoint design and examples
├── features/         # Feature specifications
├── guides/           # Development workflows
└── integrations/     # Third-party services
```

## 🔧 Tools & Technologies

This workshop demonstrates patterns for:
- **Next.js** - App Router, API routes
- **TypeScript** - Type-safe development
- **Prisma** - Database ORM
- **Zod** - Schema validation
- **React** - Component patterns
- **Testing** - Jest, React Testing Library, Playwright

## 🤝 Contributing

This is a workshop repository showcasing real-world patterns. Feel free to:
- Use these patterns in your own projects
- Adapt the skills for your needs
- Share improvements and feedback
- Create your own skills following these patterns

## 📝 License

Skills include their own license files. Check individual `LICENSE.txt` files in each skill directory.

## 🎓 Learning Path

1. **Start with AGENTS.md** - Understand agent architecture
2. **Explore .cursor/rules/** - Learn development standards
3. **Study .agent/skills/** - See skills library patterns
4. **Review docs/** - Understand documentation structure
5. **Build your own** - Create skills following these patterns

## 🔗 Related Resources

- [Cursor Rules Documentation](.cursor/rules/)
- [Agent Instructions](AGENTS.md)
- [Skills Library](.agent/skills/)
- [Development Guides](docs/guides/)

---

**Built for complex environments. Designed for production. Ready for scale.**
