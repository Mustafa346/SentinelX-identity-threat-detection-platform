# `/components`

Shared React client components used across the app's pages — layout
chrome, auth context, and small presentational pieces. All the actual
route logic (data fetching, page-specific UI) lives in `/app`; this
folder is for the pieces reused across multiple routes.

## Structure

```
components/
  AuthProvider.jsx   Client-side auth/session context
  layout/             AppShell, Sidebar, Topbar
  ui/                 Shared presentational components (badges, etc)
```

## Files

| File | Purpose |
|---|---|
| `AuthProvider.jsx` | A React context (`useAuth()`) that fetches the current session (`GET /api/auth/me`) on mount, exposes `{ user, loading, refresh }`, and is used by every protected page to know who's logged in and what role they have — without every page re-implementing that fetch. |
| `layout/AppShell.jsx` | The wrapper every protected page renders inside. Redirects to `/login` if there's no session, and redirects to `/dashboard` if the current user's role isn't in that page's `allowedRoles`, so route-level access control is enforced consistently in one place on the client (in addition to the server-side checks in `lib/authGuard.js`). |
| `layout/Sidebar.jsx` | The main navigation sidebar — links to Dashboard, Alerts, Events, Detections, Simulator, MITRE, Playbooks, Reports, Users, Audit Logs, Settings, etc, with active-route highlighting and role-aware visibility. |
| `layout/Topbar.jsx` | Top navigation bar: global search, notifications dropdown, and the current user's account menu. |
| `ui/Badges.jsx` | Small reusable badge components mapping severity (`CRITICAL`/`HIGH`/`MEDIUM`/`LOW`/`INFO`) and alert status (`NEW`/`IN_REVIEW`/`TRUE_POSITIVE`/etc) to consistent color classes, used everywhere alerts and events are listed. |

## Conventions

- Everything here is a client component (`"use client"`) — server-only
  logic belongs in `/lib` or the `app/api` route handlers instead.
- New shared, purely-presentational pieces (badges, cards, tables) should
  go in `ui/`; anything that's part of the app's persistent frame
  (navigation, header) belongs in `layout/`.
