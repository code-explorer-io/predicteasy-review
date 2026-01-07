# PredictEasy Code Review

A friendly code review of the [PredictEasy Polymarket Analysis App](https://github.com/EasyEatsBodega/polymarket-analysis-app).

---

## Overview

Nice work on this — solid concept and good data pipeline architecture. Below are some suggestions to tighten things up before scaling.

---

## Priority Issues

### 1. Strengthen Authentication on Job Endpoints

**Location:** `/src/app/api/jobs/*`

The job endpoints that trigger expensive operations (data ingestion, forecast generation) have a bypass that skips security in development mode:

```typescript
if (!cronSecret || process.env.NODE_ENV === 'development') {
  return true;  // Skips auth entirely in dev
}
```

**Risk:** If `CRON_SECRET` isn't set properly in production, anyone could spam these endpoints.

**Fix:** Remove the development bypass and protect all job endpoints with proper authentication (e.g., Clerk middleware).

---

### 2. Add a .env.example File

**Problem:** There's no documentation on what environment variables are needed. This makes it easy to:
- Accidentally commit secrets while debugging
- Deploy with missing keys and get mysterious failures

**Fix:** Create a `.env.example` file listing all required variables:

```
TMDB_API_KEY=
DATABASE_URL=
CRON_SECRET=
ADMIN_API_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
```

---

### 3. Add Input Validation on APIs

**Location:** `/src/app/api/insider-finder/route.ts`, `/src/app/api/titles/route.ts`

Query parameters like `timeframe` and `search` aren't validated before being used in database queries:

```typescript
const timeframe = parseInt(searchParams.get('timeframe') || '30'); // No bounds check
```

**Risk:** Someone could pass `timeframe=1000000` and cause huge database scans.

**Fix:** Add bounds checking:
- `timeframe`: 1-365 days
- `search`: max 50 characters
- `categories`: validate against allowed values

---

### 4. Improve Error Handling for Users

**Problem:** Most components fail silently or show "Check console for details" — not user-friendly.

**Locations:**
- `/src/components/OpportunityGrid.tsx` — no error state
- `/src/app/admin/page.tsx` — "Check console" messages

**Fix:** Add proper error states and loading indicators so users know what's happening.

---

### 5. Add Timeout Protection on Long Jobs

**Location:** `/src/app/api/jobs/generate-forecasts/route.ts`

Jobs that process thousands of records could hang indefinitely. No transaction rollback if something fails mid-way.

**Fix:**
- Add request timeouts (55s for Vercel functions)
- Wrap database operations in transactions
- Log full errors instead of truncating to 100 lines

---

## Quick Wins

| Issue | Effort | Impact |
|-------|--------|--------|
| Remove dev auth bypass | 5 mins | High |
| Add .env.example | 10 mins | Medium |
| Add timeframe bounds check | 10 mins | Medium |
| Add loading spinners | 30 mins | Medium |

---

## Summary

The core functionality is solid. These fixes are mostly about hardening the app for production use — preventing abuse, improving user experience, and making it easier for others to set up.

Happy to help implement any of these or walk through them on a call.

— Code Explorer
