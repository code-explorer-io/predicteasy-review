# Security Analysis Report: polymarket-analysis-app

**Repository:** https://github.com/EasyEatsBodega/polymarket-analysis-app
**Analysis Date:** 2026-01-07
**Analyst:** Claude Security Agent

---

## Executive Summary

I analyzed the PredictEasy/polymarket-analysis-app repository and found **multiple critical and high severity security vulnerabilities** that require immediate attention. The application is a Next.js 16 analytics platform using Clerk authentication, Prisma ORM, and PostgreSQL, but has significant gaps in authorization enforcement.

| Severity | Count |
|----------|-------|
| Critical | 5 |
| High | 2 |
| Medium | 2 |
| Low | 1 |

---

## Critical Vulnerabilities

### 1. Missing .env in .gitignore

**Severity:** CRITICAL
**Location:** `.gitignore`
**CWE:** CWE-312 (Cleartext Storage of Sensitive Information)

The `.gitignore` file does not include `.env` files, meaning environment secrets could be accidentally committed to the repository.

**Current .gitignore (missing entries):**
```
# dependencies
/node_modules
/.pnp
...
# No .env patterns!
```

**Impact:** Database credentials, API keys (TMDB, Clerk), CRON_SECRET, and ADMIN_API_KEY could be exposed publicly if accidentally committed.

**Remediation:** Add `.env*` patterns to `.gitignore` immediately:
```
.env
.env.*
.env.local
.env.production
```

---

### 2. Missing Authentication on Admin API Routes

**Severity:** CRITICAL
**Location:** `src/app/api/admin/jobs/route.ts`
**CWE:** CWE-306 (Missing Authentication for Critical Function)

The admin jobs API has **no authentication whatsoever**:

```typescript
export async function GET() {
  // NO AUTH CHECK - Anyone can access job history
  try {
    const jobs = await prisma.jobRun.findMany({
      orderBy: { startedAt: 'desc' },
      take: 50,
    });

    return NextResponse.json({
      success: true,
      jobs,
    });
  }
  // ...
}
```

**Impact:** Any unauthenticated user can view job run history, error messages, and internal system details by accessing `/api/admin/jobs`.

**Remediation:** Add Clerk authentication:
```typescript
import { auth } from '@clerk/nextjs/server'

export async function GET() {
  const { userId } = await auth()
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  // ... rest of handler
}
```

---

### 3. Missing Authentication on Data Modification Endpoints

**Severity:** CRITICAL
**Locations:**
- `src/app/api/watchlist/route.ts` - POST (add to watchlist)
- `src/app/api/watchlist/[titleId]/route.ts` - DELETE (remove from watchlist)
- `src/app/api/releases/[id]/route.ts` - PATCH, DELETE (modify/delete releases)

All these mutation endpoints have **no authentication**:

```typescript
// src/app/api/watchlist/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { titleId, pinnedBy } = body;

    // NO AUTH CHECK - Anyone can add to watchlist
    if (!titleId) {
      return NextResponse.json(
        { success: false, error: 'titleId is required' },
        { status: 400 }
      );
    }
    // Creates record without verifying user identity
  }
}
```

**Impact:** Attackers can modify application data without authorization - adding/removing watchlist items, updating release candidates, and deleting records.

**Remediation:** Add authentication to all mutation endpoints using Clerk's `auth()` helper.

---

### 4. Weak Cron Secret Protection with Dev Bypass

**Severity:** CRITICAL
**Location:** `src/app/api/jobs/ingest-netflix/route.ts:20-28`
**CWE:** CWE-287 (Improper Authentication)

```typescript
function verifyCronSecret(request: NextRequest): boolean {
  const authHeader = request.headers.get('authorization');
  const cronSecret = process.env.CRON_SECRET;

  // Skip verification in development or if no secret is set
  if (!cronSecret || process.env.NODE_ENV === 'development') {
    return true;  // BYPASSES ALL AUTH
  }

  return authHeader === `Bearer ${cronSecret}`;
}
```

**Impact:** If `CRON_SECRET` is not set in production (or misconfigured), anyone can trigger all job endpoints:
- `/api/jobs/ingest-netflix`
- `/api/jobs/ingest-signals`
- `/api/jobs/generate-forecasts`
- `/api/jobs/sync-polymarket`
- And 10+ other job endpoints

**Remediation:**
1. Fail closed - require CRON_SECRET in production
2. Add startup validation to ensure required env vars are set

```typescript
function verifyCronSecret(request: NextRequest): boolean {
  const cronSecret = process.env.CRON_SECRET;

  if (!cronSecret) {
    console.error('CRON_SECRET not configured');
    return false; // Fail closed
  }

  const authHeader = request.headers.get('authorization');
  return authHeader === `Bearer ${cronSecret}`;
}
```

---

### 5. Admin Config API Key Bypass in Development

**Severity:** CRITICAL
**Location:** `src/app/api/config/route.ts:52-56`
**CWE:** CWE-287 (Improper Authentication)

```typescript
export async function PUT(request: NextRequest) {
  try {
    // Note: In production, this should be protected by Clerk auth
    // For now, check for admin API key
    const apiKey = request.headers.get('x-api-key');
    if (apiKey !== process.env.ADMIN_API_KEY && process.env.NODE_ENV !== 'development') {
      return NextResponse.json(
        { success: false, error: 'Unauthorized' },
        { status: 401 }
      );
    }

    // Can modify app configuration including:
    // - momentumWeights
    // - breakoutThreshold
  }
}
```

**Impact:**
- Development environments have no authentication for modifying system configuration
- If ADMIN_API_KEY is not set, anyone can modify app config in any environment

**Remediation:** Use proper Clerk authentication with role-based access control.

---

## High Severity Vulnerabilities

### 6. Missing Next.js Middleware for Route Protection

**Severity:** HIGH
**Location:** No `middleware.ts` file exists
**CWE:** CWE-862 (Missing Authorization)

The app uses Clerk for authentication but has **no middleware.ts** to protect routes. This means:
- Admin pages at `/admin/*` are accessible to anyone
- API routes have no centralized auth enforcement
- Each route must implement its own auth (which most don't)

**Remediation:** Create `src/middleware.ts`:

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

---

### 7. Vulnerable xlsx Dependency

**Severity:** HIGH
**Location:** `package.json`
**CWE:** CWE-1395 (Dependency on Vulnerable Third-Party Component)

```
npm audit report:

xlsx  *
Severity: high
Prototype Pollution in sheetJS - https://github.com/advisories/GHSA-4r6h-8v6p-xvw6
SheetJS Regular Expression Denial of Service (ReDoS) - https://github.com/advisories/GHSA-5pgg-2g8v-p4x9
No fix available
node_modules/xlsx

1 high severity vulnerability
```

The `xlsx` library is used in `src/jobs/ingestNetflixWeekly.ts` to parse Netflix Top 10 data files:

```typescript
import * as XLSX from 'xlsx';

async function downloadAndParseXLSX<T>(url: string): Promise<T[]> {
  const response = await axios.get(url, {
    responseType: 'arraybuffer',
    timeout: 60000,
  });

  const workbook = XLSX.read(response.data, { type: 'buffer' });
  // ...
}
```

**Impact:**
- Prototype Pollution could allow attackers to modify object prototypes
- ReDoS could cause denial of service

**Mitigating Factors:** The data source is Netflix's official tudum.com domain, reducing supply chain risk.

**Remediation Options:**
1. Replace with `exceljs` or `xlsx-parse-json`
2. Run xlsx parsing in an isolated serverless function
3. Accept risk with monitoring (given trusted data source)

---

## Medium Severity Issues

### 8. Verbose Error Messages Expose Internal Details

**Severity:** MEDIUM
**Location:** Multiple API routes
**CWE:** CWE-209 (Information Exposure Through an Error Message)

Multiple API routes return raw error messages to clients:

```typescript
// Example from src/app/api/titles/route.ts
catch (error) {
  console.error('Error fetching titles:', error);
  return NextResponse.json(
    {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
    },
    { status: 500 }
  );
}
```

**Impact:** Internal error details (database errors, file paths, stack traces) could reveal system information to attackers.

**Remediation:** Log detailed errors server-side, return generic messages to clients:

```typescript
catch (error) {
  console.error('Error fetching titles:', error);
  return NextResponse.json(
    { success: false, error: 'An internal error occurred' },
    { status: 500 }
  );
}
```

---

### 9. No Rate Limiting on Public APIs

**Severity:** MEDIUM
**Location:** All API endpoints
**CWE:** CWE-770 (Allocation of Resources Without Limits)

All API endpoints lack rate limiting:
- `/api/titles` - paginated title listing
- `/api/markets` - market data
- `/api/edge-finder` - computational edge calculations
- `/api/insider-finder/[wallet]` - wallet lookups
- `/api/forecasts` - forecast data

**Impact:**
- Endpoints are vulnerable to abuse and scraping
- Computational endpoints like `/api/edge-finder` could be used for DoS
- Database could be overwhelmed with requests

**Remediation:** Implement rate limiting via:
- Vercel Edge Config rate limiting
- Upstash Redis rate limiter
- `@upstash/ratelimit` package

---

## Low Severity Issues

### 10. Insufficient Input Validation on Pagination

**Severity:** LOW
**Location:** `src/app/api/titles/route.ts:20-21`
**CWE:** CWE-20 (Improper Input Validation)

```typescript
const page = Math.max(1, parseInt(searchParams.get('page') || '1', 10));
const pageSize = Math.min(50, parseInt(searchParams.get('pageSize') || '20', 10));
```

While bounded, `parseInt` could return `NaN` for invalid input (e.g., `?page=abc`), which would cause unexpected behavior with `Math.max()`.

**Remediation:** Add explicit NaN checks:

```typescript
const rawPage = parseInt(searchParams.get('page') || '1', 10);
const page = Number.isNaN(rawPage) ? 1 : Math.max(1, rawPage);

const rawPageSize = parseInt(searchParams.get('pageSize') || '20', 10);
const pageSize = Number.isNaN(rawPageSize) ? 20 : Math.min(50, Math.max(1, rawPageSize));
```

---

## Positive Security Findings

| Area | Finding |
|------|---------|
| SQL Injection | ✅ Prisma ORM used exclusively - no raw SQL queries found |
| XSS | ✅ No `dangerouslySetInnerHTML` or `eval()` usage |
| Command Injection | ✅ No `child_process` or shell execution |
| HTTPS | ✅ Vercel provides automatic HTTPS |
| Type Safety | ✅ TypeScript provides compile-time safety |
| CSRF | ✅ Next.js App Router provides built-in CSRF protection for mutations |
| Dependencies | ✅ Only 1 vulnerable package (xlsx) |

---

## Remediation Priority

| Priority | Issue | Effort | Impact |
|----------|-------|--------|--------|
| P0 | Add `.env*` to `.gitignore` | 5 min | Prevents credential exposure |
| P0 | Create `middleware.ts` for Clerk auth | 30 min | Protects all admin routes |
| P0 | Add auth to admin/mutation endpoints | 2 hrs | Prevents unauthorized data modification |
| P1 | Require `CRON_SECRET` in production | 15 min | Prevents unauthorized job execution |
| P1 | Fix config API authentication | 30 min | Prevents config tampering |
| P2 | Replace or sandbox `xlsx` library | 2 hrs | Mitigates prototype pollution |
| P2 | Add rate limiting | 1-2 hrs | Prevents abuse/DoS |
| P3 | Sanitize error messages | 1 hr | Reduces information disclosure |
| P3 | Fix pagination validation | 15 min | Improves input handling |

---

## Recommended Immediate Actions

### 1. Add to `.gitignore`:
```
# Environment variables
.env
.env.*
.env.local
.env.development
.env.production
```

### 2. Create `src/middleware.ts`:
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

### 3. Fix cron verification to fail closed:
```typescript
function verifyCronSecret(request: NextRequest): boolean {
  const cronSecret = process.env.CRON_SECRET;

  if (!cronSecret) {
    console.error('CRITICAL: CRON_SECRET not configured');
    return false;
  }

  const authHeader = request.headers.get('authorization');
  return authHeader === `Bearer ${cronSecret}`;
}
```

---

## Files Reviewed

- `.gitignore`
- `package.json`
- `vercel.json`
- `prisma/schema.prisma`
- `src/app/layout.tsx`
- `src/app/admin/layout.tsx`
- `src/app/api/admin/jobs/route.ts`
- `src/app/api/config/route.ts`
- `src/app/api/watchlist/route.ts`
- `src/app/api/watchlist/[titleId]/route.ts`
- `src/app/api/releases/[id]/route.ts`
- `src/app/api/titles/route.ts`
- `src/app/api/titles/[id]/route.ts`
- `src/app/api/markets/route.ts`
- `src/app/api/insider-finder/[wallet]/route.ts`
- `src/app/api/edge-finder/route.ts`
- `src/app/api/jobs/ingest-netflix/route.ts`
- `src/jobs/ingestNetflixWeekly.ts`
- `src/jobs/ingestFlixPatrol.ts`
- `src/lib/prisma.ts`
- `src/lib/tmdbClient.ts`
