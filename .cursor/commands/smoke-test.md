# Smoke Testing Command

**Command**: `/smoke-test`

**Purpose**: Run automated smoke tests to verify critical pages work after deployment.

---

## 🎯 Quick Start

### Run Smoke Tests

```bash
# Test development environment (default)
npm run smoke:pages

# Test different environments
npm run smoke:pages:dev       # Development (alias)
npm run smoke:pages:staging   # Staging environment
npm run smoke:pages:preview   # Vercel preview URL
npm run smoke:pages:prod      # Production (may get 403)

# Test custom URL
BASE_URL=https://your-custom-url.com npm run smoke:pages
```

### Update Expected Content

```bash
# After legitimate page changes, update smoke.json
npm run smoke:update
```

---

## 📋 What Gets Tested

### Critical Pages

1. **Home page** (`/`)

   - HTTP 200 status
   - Page title and content load

2. **Search results page** (`/indonesia/bali/canggu`)

   - HTTP 200 status
   - Dynamic content loading
   - Search functionality

3. **Company/Product pages**
   - Only tested if static links found in HTML
   - Follows: search results → first company → first product

### Content Validation

Tests validate against expected values in `smoke.json`:

- **Open Graph (og:) meta tags**
- **H1 headings**
- **Page titles**
- **OG images and descriptions**

---

## 🔧 Integration Options

### Development-First Deployment (Recommended)

Production deployments automatically run smoke tests:

```bash
npm run deploy:prod      # Tests dev first, then deploys if tests pass
npm run deploy:prod:test # Test deployment workflow without deploying
```

**Workflow:**

1. 🧪 Run smoke tests on development
2. ✅ Tests pass → Deploy to production
3. ❌ Tests fail → Abort deployment

### Vercel Bot Protection

**⚠️ Important**: Production (`flexbike.app`) has Vercel bot protection that returns HTTP 403 for automated requests. This is expected security behavior.

### Internal Health Check

For production monitoring, use the internal API endpoint:

```bash
curl "https://flexbike.app/api/smoke?internal=true"
# Returns: {"success": true, "message": "Internal Vercel health check passed"}
```

---

## 📊 Test Output

### Success Example

```
🏠 Testing home page...
✅ Home page OK
   📄 Title: "Book Motorbike Rentals Worldwide | Flexbike"
   🖼️  OG Image: http://localhost:3000/assets/home-hero-bali-2.png

🔍 Testing search results page...
✅ Search page OK
   📄 Title: "Book Motorbike Rentals Worldwide | Flexbike"
   🖼️  OG Image: http://localhost:3000/assets/home-hero-bali-2.png
```

### Configuration

- **Timeout**: 30 seconds per page
- **Exit codes**: 0 (success), 1 (failure)
- **Environment**: `BASE_URL` variable controls target

---

## 🔗 Related Commands

### Testing Commands

- **Checkout Test**: `/checkout-test` - Checkout flow testing checklist
- **Refactor Test**: `/refactor-test` - Refactoring testing checklist

### Deployment Commands

- **Deploy Production**: `/deploy-production` - Full production deployment
- **Create Migration**: `/create-migration` - Database migration workflow

### CI/CD Integration

- GitHub Actions workflow runs on deployments
- Vercel deployment hooks for health checks
- External monitoring services integration

---

## ✅ Success Criteria

**All smoke tests pass:**

- ✅ HTTP 200 status for all critical pages
- ✅ Expected content loads correctly
- ✅ No server errors or crashes
- ✅ Open Graph meta tags present
- ✅ Page titles and headings render
- ✅ Dynamic content loading works
- ✅ Navigation links functional

**Integration works:**

- ✅ Deployment pipeline respects test results
- ✅ Production environment accessible (when bot protection allows)
- ✅ Internal health check endpoint responds
- ✅ Monitoring services can access health checks

---

## 🐛 Troubleshooting

### Common Issues

**403 Forbidden on Production:**

- Expected due to Vercel bot protection
- Use internal health check endpoint instead
- Configure monitoring services properly

**Content Validation Failures:**

- Run `npm run smoke:update` after legitimate changes
- Check `smoke.json` for expected values
- Verify page content matches expectations

**Timeout Errors:**

- Increase timeout in test configuration
- Check server performance
- Verify network connectivity

**Dynamic Content Issues:**

- Smoke tests expect dynamic content to load
- JavaScript execution is normal behavior
- Failures indicate server-side rendering issues

---

## 📝 Implementation Notes

### Test Architecture

- Uses Cheerio for HTML parsing
- Follows navigation flow automatically
- Validates structured content expectations
- Provides detailed error reporting

### Configuration Files

- `smoke.json`: Expected content validation
- `scripts/smoke/smoke-pages.ts`: Test implementation
- Package.json scripts for different environments

### Security Considerations

- Bot protection prevents automated testing on production
- Internal endpoints provide alternative health checks
- Environment-specific testing strategies
