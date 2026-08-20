# `/app`

The Next.js App Router root — every page and every API route in
SentinelX lives here. One folder per URL segment; a page's UI is its
`page.js`, an API endpoint is its `route.js`.

## Structure

```
app/
  layout.js            Root layout (fonts, AuthProvider, global shell)
  page.js               Redirects "/" to /dashboard or /login
  globals.css           Tailwind + design tokens (sx-* badge/utility classes)
  favicon.ico

  login/, register/     Public auth pages
  dashboard/             SOC dashboard — live stats, charts, alert feed
  alerts/                Alert queue + alerts/[id] investigation page
  events/                Identity event log, filterable/paginated
  detections/            Detection rule configuration (Admin)
  simulator/              Attack Simulator UI (8 scenarios)
  mitre/                  MITRE ATT&CK mapping view
  false-positives/        False-positive tuning + detection quality metrics
  playbooks/              Incident response playbooks
  reports/                Generated incident reports
  users/                  User management (Admin)
  audit-logs/             Audit log viewer (Admin)
  settings/               App settings, demo data reset

  api/                    All REST-style route handlers (see below)
```

## Pages

Every page folder above (`dashboard/`, `alerts/`, `events/`, etc.) renders
inside `components/layout/AppShell.jsx`, which enforces that a session
exists and that the logged-in user's role is allowed on that route.
Pages fetch data client-side through `lib/apiClient.js` and hit the
matching endpoint under `app/api/`.

## API Routes (`/app/api`)

```
api/
  auth/           register, login, logout, me
  events/         identity event log (filterable, paginated)
  alerts/         alert queue, alerts/[id] get + patch
  detections/     detection rule list, detections/[id] patch
  exceptions/     false-positive exclusion rules
  simulate/       one route per attack scenario (password-spray,
                  mfa-abuse, privilege-escalation, unusual-admin-login,
                  impossible-travel, new-device, new-ip, account-takeover)
  analytics/      overview stats, false-positive/detection-quality metrics
  mitre/          MITRE ATT&CK technique summary with live alert counts
  playbooks/      incident response playbooks
  reports/        incident report list/create, reports/[id] fetch
  users/          user management (Admin), users/[id] patch/delete
  audit-logs/     audit log listing (Admin)
  notifications/  in-app notifications, mark-read
  search/         global search across users, alerts, IPs, events, rules
  admin/          generate-demo-attack, reset-demo (demo/testing helpers)
```

Every route handler follows the same shape: connect to the DB
(`lib/db.js`), authenticate + authorize via `requireAuth([...roles])`
from `lib/authGuard.js`, do the work (often delegating to `/services` or
`/detection-engine`), and return the standard envelope:

```json
{ "success": true, "data": "..." }
{ "success": false, "message": "...", "error": "ERROR_CODE" }
```

Role checks happen **server-side on every route** — the frontend hiding a
button or link is never the only line of defense.

See the main [README's API Overview](../README.md#6-api-overview) for the
full endpoint reference.
