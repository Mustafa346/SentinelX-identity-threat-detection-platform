# `/services`

Business-logic services that sit between the API routes and the raw
Mongoose models — the "what actually happens" layer.

## Files

| File | Purpose |
|---|---|
| `eventPipeline.js` | `recordIdentityEvent(fields)` — the **single entry point** for creating an `IdentityEvent`. Stores the event, then immediately runs it through `detection-engine`'s pipeline (before updating the user's baseline, so the event is judged against what was "normal" up to that point) and only afterwards updates the baseline. Every code path that produces identity telemetry — real login/logout, MFA activity, admin actions, and the attack simulator — goes through this one function, so detection behaves identically whether the traffic is real or simulated. |
| `attackSimulator.js` | Implements the 8 local-only attack scenarios exposed by the Attack Simulator (`/simulator` page, `/api/simulate/*` routes): password spray, MFA abuse, privilege escalation, unusual admin login, impossible travel, new device, new IP, and account takeover. Every simulated event is tagged `isSimulated: true` with a shared `batchId` and written through `eventPipeline.js` like any real event — nothing is faked at the alert level, it's genuinely detected. |
| `auditLog.js` | `writeAuditLog(...)` — appends an `AuditLog` entry for sensitive actions (rule changes, user management, simulations, report generation) and, where relevant, fires an in-app `Notification`. |

## Design notes

- Services are the only layer allowed to orchestrate multiple models in
  one operation (e.g. create an event *and* run detection *and* update a
  baseline). API routes stay thin — they validate/authorize, then call a
  service.
- Keeping the attack simulator's writes flowing through the exact same
  `recordIdentityEvent()` pipeline as real traffic is what makes the
  simulator a meaningful test of the detection engine rather than a
  scripted demo.
