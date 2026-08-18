# `/scripts`

Standalone Node.js scripts run outside the Next.js server, for local
database setup.

## Files

| File | Purpose |
|---|---|
| `seed.mjs` | Wipes and rebuilds the database with realistic demo data: seeded user accounts across IT, Finance, HR, Security, Engineering, and Management departments (see [Demo Credentials](../README.md#5-demo-credentials)), the default detection rules (`lib/defaultRules.js`), incident response playbooks, and roughly 14 days of baseline `IdentityEvent` activity plus a handful of pre-triaged true/false-positive alerts — so the dashboard, charts, and detection quality metrics are meaningful the moment you run the app for the first time. |

## Usage

```bash
npm run seed
```

To re-seed events, rules, and playbooks **without** wiping existing user
accounts:

```bash
npm run seed -- --keep-users
```

Requires `.env.local` to be present with a valid `MONGODB_URI` (copy it
from `.env.example` first — see the main README's
[Installation](../README.md#4-installation) section).
