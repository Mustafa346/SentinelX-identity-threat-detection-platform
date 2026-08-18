# `/models`

Mongoose schemas for every collection SentinelX persists to MongoDB. These
are the single source of truth for shape and validation API routes,
the detection engine, and the seed script all import from here rather
than trusting raw request bodies.

## Files

| Model | Purpose |
|---|---|
| `User.js` | Accounts (`ROLES`: `ADMIN` / `SECURITY_ANALYST` / `EMPLOYEE`, `DEPARTMENTS`), plus each user's behavioral **baseline** (known IPs, known devices, known countries, normal login hours) that several detectors compare new events against. |
| `IdentityEvent.js` | The core event log every login, logout, MFA action, role change, new device/IP/location, admin login, lockout, and more. Defines the full `EVENT_TYPES` enum. This is what the detection engine reads and what powers the Events page. |
| `Alert.js` | A security alert raised by the detection engine: severity, detection type, linked user/IP, evidence, MITRE mapping, investigation status, analyst notes, and a `correlationKey` used to deduplicate repeat triggers into one alert. |
| `DetectionRule.js` | Configurable rule definitions (`DETECTION_TYPES` enum) threshold, time window, severity, MITRE technique, enabled/disabled flag, and per-rule exclusions. Nothing about detection behavior is hard-coded in the UI; it all comes from documents of this shape. |
| `IncidentReport.js` | A frozen snapshot (alert + evidence + notes + playbook steps) captured at the moment a PDF incident report is generated, so the report always reflects what the analyst saw at the time. |
| `Playbook.js` | Incident response playbooks ordered response steps per detection type, shown on an alert's investigation page. |
| `AuditLog.js` | Append-only log of sensitive actions (rule changes, user management, report generation, simulations run) for accountability. |
| `Notification.js` | In-app notifications, either targeted at a specific user or broadcast to a set of roles (e.g. all `ADMIN`/`SECURITY_ANALYST` accounts) when a new alert fires. |

## Conventions

- Every schema uses `mongoose.models.X || mongoose.model("X", XSchema)` so
  hot-reloading in `next dev` never throws `OverwriteModelError`.
- Enums (`ROLES`, `DEPARTMENTS`, `EVENT_TYPES`, `DETECTION_TYPES`) are
  exported alongside their schema so other modules (API validation,
  detection engine, seed script) import the same list instead of
  duplicating string literals.
- `{ timestamps: true }` is used wherever `createdAt`/`updatedAt` are
  meaningful for auditing or sorting.
