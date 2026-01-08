# Deploy to Production Command

**Command**: `/deploy-production`

**Purpose**: Safe production deployment workflow with pre-deployment checks and rollback procedures.

---

## 🎯 Deployment Workflow

### Automated Safe Deployment

```bash
# Recommended: Runs smoke tests first, then deploys if tests pass
npm run deploy:prod

# Test deployment workflow without actually deploying
npm run deploy:prod:test
```

### Manual Deployment Steps

```bash
# 1. Run smoke tests on development
npm run smoke:pages:dev

# 2. If tests pass, deploy to production
npm run deploy:prod:force  # Bypasses smoke test check

# 3. Alternative manual deployment
vercel --prod --yes
```

---

## 📋 Pre-Deployment Checklist

### ⚠️ **Critical Requirements**

- [ ] **Database migrations applied?**

  - Run: `npm run db:deploy`
  - Verify: Check migration status in database

- [ ] **Environment variables synced?**

  - Run: `npm run envpull`
  - Verify: Check Vercel environment variables match local `.env.local`

- [ ] **Tests passing locally?**

  - Run: `npm run test`
  - Verify: All tests pass, no failures

- [ ] **Build successful locally?**
  - Run: `npm run build`
  - Verify: Build completes without errors

### 🔍 **Additional Checks**

- [ ] **Smoke tests pass on development**

  - Run: `npm run smoke:pages:dev`
  - Verify: All critical pages load correctly

- [ ] **No console errors in DevTools**

  - Manual check: Open browser, check console

- [ ] **TypeScript compilation clean**

  - Run: `npx tsc --noEmit`
  - Verify: No TypeScript errors

- [ ] **ESLint passes**
  - Run: `npm run lint`
  - Verify: No linting errors

---

## 🚀 Environment Details

### Production Environment

- **URL**: `https://flexbike.app`
- **Database**: Production Supabase instance
- **Firebase**: Production Firebase project
- **CDN**: Vercel global CDN

### Preview Environment

- **URL**: `https://preview.flexbike.app` or Vercel preview URL
- **Database**: Preview Supabase instance
- **Firebase**: Shared production Firebase instance
- **Auto-deploy**: On every git push

### Development Environment

- **URL**: `http://localhost:3000`
- **Database**: Local/development Supabase instance
- **Firebase**: Development Firebase project

---

## 🔄 Post-Deployment Verification

### Immediate Checks (First 5 minutes)

- [ ] **Site loads correctly**

  - Visit: `https://flexbike.app`
  - Verify: Page loads, no 500 errors

- [ ] **Critical functionality works**

  - Search functionality
  - Bike listing pages
  - Basic booking flow start

- [ ] **Admin interface accessible**
  - Visit: `https://flexbike.app/admin`
  - Verify: Login works, basic navigation

### Extended Monitoring (First 24 hours)

- [ ] **Performance monitoring**

  - Check Vercel analytics
  - Monitor response times

- [ ] **Error monitoring**

  - Check server logs
  - Monitor for JavaScript errors

- [ ] **Database connectivity**
  - Verify booking creation works
  - Check data synchronization

---

## 🛡️ Rollback Procedures

### Emergency Rollback (< 5 minutes)

If critical issues detected immediately after deployment:

```bash
# 1. Identify the problematic deployment
vercel ls flexbike

# 2. Rollback to previous deployment
vercel rollback [deployment-id]

# 3. Verify rollback successful
curl https://flexbike.app/api/health
```

### Database Rollback (Complex)

If database migration issues:

```bash
# 1. Stop accepting new requests (maintenance mode)
# 2. Create backup of current state
# 3. Revert database migration
npm run db:rollback
# 4. Rollback application deployment
vercel rollback
# 5. Verify system stability
```

### Partial Rollback Options

- **Application rollback**: Keep database changes, rollback only frontend
- **Database rollback**: Keep frontend deployment, rollback only database changes
- **Feature flag rollback**: Disable problematic features via environment variables

---

## 🔧 Deployment Scripts

### Available Commands

```bash
# Safe production deployment (recommended)
npm run deploy:prod          # Smoke tests → Deploy (if tests pass)
npm run deploy:prod:test     # Test deployment workflow only

# Manual deployment options
npm run deploy:prod:force    # Deploy without smoke tests
vercel --prod --yes          # Direct Vercel deployment

# Environment management
npm run envpull              # Pull env vars from Vercel
npm run envpush              # Push env vars to Vercel

# Database deployment
npm run db:deploy            # Deploy database migrations
npm run db:rollback          # Rollback database migrations
```

### Vercel Integration

- **Auto-preview**: Every git push creates preview deployment
- **Production**: Manual deployment via `vercel --prod --yes`
- **Environment sync**: `npm run envpull` keeps local env vars in sync

---

## 📊 Monitoring & Alerts

### Post-Deployment Monitoring

1. **Application Performance**

   - Vercel dashboard metrics
   - Core Web Vitals scores
   - Page load times

2. **Error Tracking**

   - Vercel function logs
   - Browser console errors
   - API error rates

3. **Business Metrics**
   - Booking conversion rates
   - Search functionality
   - User engagement

### Alert Thresholds

- **Response time**: > 3 seconds (warning), > 10 seconds (critical)
- **Error rate**: > 5% (warning), > 15% (critical)
- **Uptime**: < 99.9% (warning), < 99% (critical)

---

## 🔗 Related Commands

### Testing Commands

- **Smoke Test**: `/smoke-test` - Pre-deployment smoke testing
- **Checkout Test**: `/checkout-test` - Checkout flow validation
- **Refactor Test**: `/refactor-test` - Post-refactoring validation

### Development Commands

- **Create Migration**: `/create-migration` - Database migration workflow
- **Firebase Sync**: `/firebase-sync` - Data synchronization workflow

---

## ✅ Deployment Success Criteria

**Deployment is successful when:**

- ✅ Smoke tests pass on development
- ✅ All pre-deployment checks complete
- ✅ Vercel deployment succeeds without errors
- ✅ Production site loads correctly
- ✅ Critical functionality verified working
- ✅ No immediate error spikes
- ✅ Monitoring systems show normal operation

**Rollback is required when:**

- ❌ Site returns 500 errors
- ❌ Critical functionality broken
- ❌ Database connectivity issues
- ❌ Performance degradation > 50%
- ❌ Security vulnerabilities introduced

---

## 📝 Emergency Contacts

- **Critical Issues**: Stop operations, contact engineering lead immediately
- **Performance Issues**: Monitor Vercel dashboard, prepare rollback plan
- **Data Issues**: Check Firebase sync status, consider data rollback
- **Security Issues**: Isolate affected systems, prepare full rollback

---

## 🔒 Security Considerations

- **Environment Variables**: Never commit secrets, use Vercel env management
- **Database Access**: Production database requires proper authentication
- **API Security**: Rate limiting and authentication active in production
- **CDN Security**: Vercel handles SSL termination and bot protection

---

## 📈 Continuous Improvement

### Post-Mortem Process

After each deployment (successful or failed):

1. **Document what happened**
2. **Identify improvement opportunities**
3. **Update deployment checklists**
4. **Review monitoring thresholds**
5. **Update rollback procedures**

### Deployment Metrics

Track and improve:

- **Deployment frequency**
- **Mean time to deploy**
- **Rollback frequency**
- **Post-deployment issue detection time**
- **Time to recovery**
