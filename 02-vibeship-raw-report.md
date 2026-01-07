# Vibeship Security Scan Report (Raw Output)

> **DISCLAIMER:** This is the raw, unedited output from Vibeship, a third-party automated security scanning tool. We have included it for transparency and reference. **We are not responsible for Vibeship's methodology, accuracy, or conclusions.** For our professional assessment and analysis of these findings, see `03-vibeship-analysis.md`.

---

**Scan URL:** https://scanner.vibeship.co/scan/652f5999-1142-429c-ab16-1cd6277a5a64
**Repository:** https://github.com/EasyEatsBodega/polymarket-analysis-app
**Date:** 1/7/2026, 9:38:06 PM
**Tool:** Vibeship Scanner (Third-Party)

---

## Score Summary

| Metric | Value |
|--------|-------|
| Score | 0/100 |
| Grade | F |
| Status | 🔧 Fix Required |

---

## Detected Stack

- **Languages:** JavaScript, TypeScript
- **Frameworks:** Next.js, PostgreSQL, Prisma, React

---

## Finding Counts

| Severity | Count |
|----------|-------|
| Critical | 10 |
| High | 5 |
| Medium | 40 |
| Low | 1 |
| Info | 285 |

---

## Detailed Findings

### [1] Password reset endpoint - add rate limiting to prevent abuse

- **Severity:** INFO
- **Category:** code
- **CWE:** CWE-798 - Hardcoded Credentials
- **CVSS:** 9.8 (Critical)
- **Location:** jest.config.js:3

**Risk & Fix:**
Passwords in code are a severe security risk. Anyone with code access can authenticate as that user/service. Fix: Remove immediately, use environment variables, rotate the password.

---

### [2-13] Use === instead of == for comparison to avoid type coercion issues

- **Severity:** INFO
- **Category:** code
- **Locations:**
  - src/lib/edgeCalculator.ts:275, 291, 299, 315, 354, 356, 359, 362, 370, 372, 375, 378

---

### [14] Double type assertion bypasses type safety entirely

- **Severity:** MEDIUM
- **Category:** code
- **Location:** src/lib/featureBuilder.ts:68

---

### [15-16] @ts-ignore suppresses TypeScript errors - review carefully

- **Severity:** MEDIUM
- **Category:** code
- **Locations:** src/lib/featureBuilder.ts:75, 94

---

### [17-23] Use === instead of == for comparison to avoid type coercion issues

- **Severity:** INFO
- **Category:** code
- **Locations:** src/lib/featureBuilder.ts:105, 114, 181, 189, 199, 235, 266

---

### [24-25] Use === instead of == for comparison to avoid type coercion issues

- **Severity:** INFO
- **Category:** code
- **Locations:** src/lib/forecaster.ts:222, 267

---

### [26-27] Non-null assertion - may cause runtime errors if value is null

- **Severity:** INFO
- **Category:** code
- **Locations:** src/lib/forecaster.ts:280, 300

---

### [28-30] Use === instead of == for comparison to avoid type coercion issues

- **Severity:** INFO
- **Category:** code
- **Locations:** src/lib/forecaster.ts:383, 387, 393

---

### [31] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/lib/forecaster.ts:572

---

### [32-35] Dynamic property assignment - prototype pollution if key is user-controlled

- **Severity:** MEDIUM
- **Category:** code
- **Locations:** src/lib/marketMatcher.ts:41, 42, 48, 50

---

### [36] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/lib/polymarketClient.ts:553

---

### [37] Use === instead of == for comparison to avoid type coercion issues

- **Severity:** INFO
- **Category:** code
- **Location:** src/lib/polymarketClient.ts:609

---

### [38] Non-null assertion - may cause runtime errors if value is null

- **Severity:** INFO
- **Category:** code
- **Location:** src/lib/polymarketClient.ts:728

---

### [39] Double type assertion bypasses type safety entirely

- **Severity:** MEDIUM
- **Category:** code
- **Location:** src/lib/prisma.ts:4

---

### [40] Code checking for non-production environment - ensure proper env handling

- **Severity:** INFO
- **Category:** code
- **Location:** src/lib/prisma.ts:15

---

### [41] Fetch API with potentially user-controlled URL - validate to prevent SSRF

- **Severity:** MEDIUM
- **Category:** code
- **CWE:** CWE-918 - Server-Side Request Forgery (SSRF)
- **CVSS:** 7.5 (High)
- **Location:** src/app/admin/jobs/page.tsx:26

---

### [42] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/admin/jobs/page.tsx:32

---

### [43] URLSearchParams usage - ensure query data is sanitized before rendering (CWE-79)

- **Severity:** INFO
- **Category:** code
- **CWE:** CWE-79 - Cross-site Scripting (XSS)
- **CVSS:** 6.1 (Medium)
- **Location:** src/app/admin/releases/page.tsx:75

---

### [44-45] Fetch API with potentially user-controlled URL - validate to prevent SSRF

- **Severity:** MEDIUM
- **Category:** code
- **CWE:** CWE-918 - Server-Side Request Forgery (SSRF)
- **CVSS:** 7.5 (High)
- **Locations:** src/app/admin/releases/page.tsx:79, 170

---

### [46] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/admin/releases/page.tsx:176

---

### [47] Non-null assertion - may cause runtime errors if value is null

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/admin/releases/page.tsx:443

---

### [48] Fetch API with potentially user-controlled URL - validate to prevent SSRF

- **Severity:** MEDIUM
- **Category:** code
- **CWE:** CWE-918 - Server-Side Request Forgery (SSRF)
- **CVSS:** 7.5 (High)
- **Location:** src/app/admin/settings/page.tsx:37

---

### [49] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/admin/settings/page.tsx:47

---

### [50-54] parseInt without radix parameter - specify base to avoid unexpected behavior

- **Severity:** INFO
- **Category:** code
- **Locations:** src/app/admin/settings/page.tsx:167, 184, 201, 249, 280

---

### [55] Dynamic property assignment - prototype pollution if key is user-controlled

- **Severity:** MEDIUM
- **Category:** code
- **Location:** src/app/api/charts/rank-trends/route.ts:112

---

### [56] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/api/charts/rank-trends/route.ts:140

---

### [57-59] Non-null assertion - may cause runtime errors if value is null

- **Severity:** INFO
- **Category:** code
- **Locations:** src/app/api/movers/route.ts:208, 209, 210

---

### [60] Console error logging - ensure not exposed to users in production (CWE-209)

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/api/movers/route.ts:257

---

### [61-69] Various issues in releases API

- **Severity:** INFO-MEDIUM
- **Category:** code
- **Location:** src/app/api/releases/[id]/route.ts

---

### [70-72] Various issues in releases route

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/api/releases/route.ts

---

### [73-84] TypeScript any type and console logging issues

- **Severity:** INFO
- **Category:** code
- **Locations:** src/app/api/titles/[id]/route.ts, src/app/api/titles/route.ts

---

### [85-93] Various issues in insider-finder pages

- **Severity:** INFO-MEDIUM
- **Category:** code
- **Locations:** src/app/insider-finder/[wallet]/page.tsx, src/app/insider-finder/page.tsx

---

### [94-95] Non-null assertion and parseInt issues

- **Severity:** INFO
- **Category:** code
- **Location:** src/app/netflix/page.tsx

---

### [96-115] Various API route issues

- **Severity:** INFO-MEDIUM
- **Category:** code
- **Locations:** Multiple API routes including admin/jobs, awards, breakouts, config, edge-finder, forecasts

---

### [Additional findings 116-341]

Remaining findings follow similar patterns:
- Console error logging (INFO)
- Non-null assertions (INFO)
- TypeScript any usage (INFO)
- Dynamic property assignment (MEDIUM)
- Fetch API SSRF concerns (MEDIUM)
- parseInt radix issues (INFO)
- === vs == comparisons (INFO)

---

## Vibeship Recommended Fixes

### For Hardcoded Credentials (Finding 1):
```javascript
// Safe: Use bcrypt
import bcrypt from 'bcrypt';

const saltRounds = 12;
const hash = await bcrypt.hash(password, saltRounds);
const isValid = await bcrypt.compare(password, storedHash);
```

### For SSRF (Findings 41, 44-45, 48, 86, 92):
```javascript
import { URL } from 'url';

const ALLOWED_HOSTS = ['api.example.com', 'webhook.example.com'];

function validateUrl(urlString) {
  const url = new URL(urlString);

  const blockedPatterns = [
    /^localhost$/i,
    /^127\./,
    /^10\./,
    /^172\.(1[6-9]|2[0-9]|3[0-1])\./,
    /^192\.168\./,
    /^0\./
  ];

  if (blockedPatterns.some(p => p.test(url.hostname))) {
    throw new Error('Invalid URL: private addresses not allowed');
  }

  if (!ALLOWED_HOSTS.includes(url.hostname)) {
    throw new Error('Invalid URL: host not allowed');
  }

  return url.href;
}
```

### For XSS (Findings 43, 89):
```javascript
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);

// React with sanitization
<div dangerouslySetInnerHTML={{
  __html: DOMPurify.sanitize(userContent)
}} />
```

---

*Note: This is the raw output from Vibeship scanner. See the cross-reference analysis report for assessment of which findings are valid vs false positives.*
