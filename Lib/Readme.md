# `/lib`

Cross-cutting utilities shared across API routes, the detection engine,
and the frontend. If more than one part of the app needs it, it lives
here rather than being duplicated.

## Files

| File | Purpose |
|---|---|
| `db.js` | Cached Mongoose connection helper (`connectDB()`), reused across hot-reloaded serverless-style route handlers so we don't open a new connection per request. |
| `session.js` | Signs and verifies the JWT stored in the `sentinelx_session` httpOnly cookie; exposes `SESSION_COOKIE_NAME` and helpers to read/create/clear the session. |
| `authGuard.js` | `requireAuth([...roles])` the server-side gate every protected API route calls first. Re-validates the session against the database (not just the JWT payload) and enforces role checks. Also defines the standard `apiSuccess()` / `apiError()` response envelope. |
| `password.js` | bcrypt hashing (cost factor 12) and password-strength validation. |
| `rateLimit.js` | Lightweight in-memory sliding-window rate limiter used on auth endpoints. Good enough for a local/single-instance app; swap for Redis if this ever runs across multiple instances. |
| `riskScoring.js` | Transparent, explainable risk scoring (`RISK_WEIGHTS`) every point added to an alert's score is tied to a named factor, never a black box. |
| `exclusions.js` | Pure function `isExcluded(rule, { sourceIP, username, timestamp })` that checks a triggered detection against a rule's configured false-positive exclusions, including overnight time windows. Fully unit-tested in `/tests`. |
| `defaultRules.js` | The seed data for `DetectionRule` documents default thresholds, time windows, severities, and MITRE mappings for all seven detectors. |
| `utils.js` | Small dependency-free helpers: ID generation, a minimal user-agent parser used for device fingerprinting/off-hours logic. |
| `apiClient.js` | Client-side (`"use client"`) fetch wrapper used by pages/components to call the API and normalize error handling (`ApiError`). |
| `pdfReport.js` | Client-side (`"use client"`) PDF generation for incident reports, built with `jsPDF` + `jspdf-autotable`. |
| `simulateRouteFactory.js` | A factory that wraps a simulator function into a standard `POST` route handler enforces `ADMIN`/`SECURITY_ANALYST`-only access and writes an audit log entry for every simulation run. Used by all `/api/simulate/*` routes so they don't each reimplement the same boilerplate. |

## Conventions

- Files that must run in the browser are explicitly marked `"use client"`
  at the top (`apiClient.js`, `pdfReport.js`); everything else assumes a
  Node.js server environment.
- Functions here favor being **pure and testable** where possible
  `exclusions.js`, `riskScoring.js`, `password.js`, and `utils.js` have no
  database dependency, which is exactly why they're the files covered by
  `/tests`.
