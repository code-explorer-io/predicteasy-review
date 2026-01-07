# Remediation Checklist

**Project:** polymarket-analysis-app
**Date:** January 7, 2026

Use this checklist to track remediation progress. Each item includes verification steps.

---

## Critical Priority (Fix Immediately)

### [ ] 1. Add .env to .gitignore

**File to modify:** `.gitignore`

**Action:** Add these lines to the end of the file:
```
# Environment variables
.env
.env.*
.env.local
.env.development
.env.production
```

**Verification:**
1. Create a test file called `.env.test` in the project root
2. Run `git status`
3. ✅ The file should NOT appear in the list of untracked files
4. Delete the test file

**Time estimate:** 5 minutes

---

### [ ] 2. Create Authentication Middleware

**File to create:** `src/middleware.ts`

**Action:** Create the file with this content:
```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isAdminRoute = createRouteMatcher(['/admin(.*)'])
const isProtectedApi = createRouteMatcher([
  '/api/admin(.*)',
  '/api/config(.*)',
  '/api/watchlist(.*)',
  '/api/releases(.*)',
])

export default clerkMiddleware(async (auth, req) => {
  if (isAdminRoute(req) || isProtectedApi(req)) {
    await auth.protect()
  }
})

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)'],
}
```

**Verification:**
1. Start the development server
2. Open an incognito/private browser window
3. Navigate to `/admin`
4. ✅ You should be redirected to the login page
5. Navigate to `/api/admin/jobs`
6. ✅ You should receive a 401 Unauthorized response

**Time estimate:** 30 minutes

---

### [ ] 3. Fix Cron Secret Verification

**File to modify:** `src/app/api/jobs/ingest-netflix/route.ts` (and similar job routes)

**Action:** Replace the `verifyCronSecret` function:
```typescript
function verifyCronSecret(request: NextRequest): boolean {
  const cronSecret = process.env.CRON_SECRET;

  if (!cronSecret) {
    console.error('CRITICAL: CRON_SECRET not configured');
    return false; // Fail closed - deny access if not configured
  }

  const authHeader = request.headers.get('authorization');
  return authHeader === `Bearer ${cronSecret}`;
}
```

**Verification:**
1. Temporarily remove `CRON_SECRET` from your `.env` file
2. Try to access `/api/jobs/ingest-netflix` in the browser
3. ✅ You should receive a 401 Unauthorized response
4. Restore the `CRON_SECRET` to your `.env` file

**Files to update with this pattern:**
- [ ] `src/app/api/jobs/ingest-netflix/route.ts`
- [ ] `src/app/api/jobs/ingest-signals/route.ts`
- [ ] `src/app/api/jobs/generate-forecasts/route.ts`
- [ ] `src/app/api/jobs/sync-polymarket/route.ts`
- [ ] `src/app/api/jobs/refresh-markets/route.ts`
- [ ] `src/app/api/jobs/snapshot-prices/route.ts`
- [ ] `src/app/api/jobs/scan-insiders/route.ts`
- [ ] `src/app/api/jobs/ingest-flixpatrol/route.ts`
- [ ] `src/app/api/jobs/discover-releases/route.ts`
- [ ] `src/app/api/jobs/ingest-awards/route.ts`
- [ ] `src/app/api/jobs/ingest-pacing/route.ts`

**Time estimate:** 15-30 minutes

---

### [ ] 4. Fix Config API Authentication

**File to modify:** `src/app/api/config/route.ts`

**Action:** Replace the authentication check in the PUT handler:
```typescript
import { auth } from '@clerk/nextjs/server'

export async function PUT(request: NextRequest) {
  // Require authentication
  const { userId } = await auth()
  if (!userId) {
    return NextResponse.json(
      { success: false, error: 'Unauthorized' },
      { status: 401 }
    );
  }

  // ... rest of the handler
}
```

**Verification:**
1. Log out of the application
2. Use curl or Postman to send a PUT request to `/api/config`
3. ✅ You should receive a 401 Unauthorized response

**Time estimate:** 15 minutes

---

### [ ] 5. Add Authentication to Mutation Endpoints

**Files to modify:**
- [ ] `src/app/api/watchlist/route.ts` (POST handler)
- [ ] `src/app/api/watchlist/[titleId]/route.ts` (DELETE handler)
- [ ] `src/app/api/releases/[id]/route.ts` (PATCH and DELETE handlers)
- [ ] `src/app/api/releases/route.ts` (POST handler if exists)

**Action:** Add auth check to each mutation handler:
```typescript
import { auth } from '@clerk/nextjs/server'

export async function POST(request: NextRequest) {
  const { userId } = await auth()
  if (!userId) {
    return NextResponse.json(
      { success: false, error: 'Unauthorized' },
      { status: 401 }
    );
  }

  // ... rest of existing handler
}
```

**Verification:**
1. Log out of the application
2. Try to add an item to the watchlist via the UI
3. ✅ The action should fail or redirect to login
4. Log in and try again
5. ✅ The action should succeed

**Time estimate:** 1-2 hours

---

## High Priority (Fix This Week)

### [ ] 6. Add Rate Limiting

**Action:** Install and configure rate limiting:

```bash
npm install @upstash/ratelimit @upstash/redis
```

Create `src/lib/ratelimit.ts`:
```typescript
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

export const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'), // 10 requests per 10 seconds
})
```

Add to API routes:
```typescript
import { ratelimit } from '@/lib/ratelimit'

export async function GET(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1'
  const { success } = await ratelimit.limit(ip)

  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { status: 429 }
    )
  }

  // ... rest of handler
}
```

**Verification:**
1. Write a script that makes 20 rapid requests to `/api/titles`
2. ✅ Some requests should return 429 Too Many Requests

**Time estimate:** 1-2 hours

---

### [ ] 7. Replace xlsx Library (Optional - Risk Acceptance Alternative)

**Option A: Replace the library**
```bash
npm uninstall xlsx
npm install exceljs
```

Update `src/jobs/ingestNetflixWeekly.ts` to use exceljs instead.

**Option B: Accept the risk**
- Complete the Risk Acceptance Form (document 08)
- Document the mitigating factor: data source is Netflix's official domain

**Time estimate:** 2 hours for replacement, 15 minutes for risk acceptance

---

## Medium Priority (Fix When Convenient)

### [ ] 8. Sanitize Error Messages

**Files to modify:** All API routes in `src/app/api/`

**Action:** Replace error responses:

Before:
```typescript
catch (error) {
  return NextResponse.json({
    success: false,
    error: error instanceof Error ? error.message : 'Unknown error',
  }, { status: 500 });
}
```

After:
```typescript
catch (error) {
  console.error('API Error:', error); // Keep detailed logging server-side
  return NextResponse.json({
    success: false,
    error: 'An internal error occurred',
  }, { status: 500 });
}
```

**Verification:**
1. Temporarily break something in an API route
2. Call the endpoint
3. ✅ The response should say "An internal error occurred" not the actual error

**Time estimate:** 1 hour

---

### [ ] 9. Fix Pagination NaN Handling

**File to modify:** `src/app/api/titles/route.ts` (and similar paginated endpoints)

**Action:** Add NaN checks:
```typescript
const rawPage = parseInt(searchParams.get('page') || '1', 10);
const page = Number.isNaN(rawPage) ? 1 : Math.max(1, rawPage);

const rawPageSize = parseInt(searchParams.get('pageSize') || '20', 10);
const pageSize = Number.isNaN(rawPageSize) ? 20 : Math.min(50, Math.max(1, rawPageSize));
```

**Verification:**
1. Call `/api/titles?page=abc`
2. ✅ Should return page 1, not an error

**Time estimate:** 15 minutes

---

## Sign-Off

| Item | Fixed By | Date | Verified By | Date |
|------|----------|------|-------------|------|
| 1. .gitignore | | | | |
| 2. Middleware | | | | |
| 3. Cron secrets | | | | |
| 4. Config API | | | | |
| 5. Mutation auth | | | | |
| 6. Rate limiting | | | | |
| 7. xlsx library | | | | |
| 8. Error messages | | | | |
| 9. Pagination | | | | |

---

**All critical items completed:** [ ] Yes / [ ] No

**Date remediation completed:** _______________

**Verified by:** _______________
