# `/tests`

A Node.js-native test suite (`node:test` + `node:assert/strict`) no
extra test framework or dependency needed. Covers the pure,
side-effect-free logic that doesn't require a MongoDB connection.

## Files

| File | Covers |
|---|---|
| `exclusions.test.mjs` | `lib/exclusions.js` false-positive exclusion matching by source IP, username, and time window, including overnight windows that wrap past midnight (e.g. `00:00`–`03:00`). |
| `riskScoring.test.mjs` | `lib/riskScoring.js` risk score computation and normalization against the named `RISK_WEIGHTS` factors. |
| `password.test.mjs` | `lib/password.js` password hashing/verification and password-strength validation. |
| `utils.test.mjs` | `lib/utils.js` the lightweight user-agent parser and other small pure helpers. |

## Running

```bash
npm test
```

which runs `node --test tests/*.test.mjs`.

## What isn't covered here

Anything that needs MongoDB the full detector pipeline, alert creation,
correlation/deduplication isn't unit tested against a real database.
It's instead verified end-to-end by hand using the Attack Simulator
against a running instance; see the step-by-step walkthrough in the main
README's [Testing](../README.md#12-testing) section.
