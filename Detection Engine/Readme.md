# `/detection-engine`

The rule-based detection logic at the heart of SentinelX. Every file here
is a pure-ish detector: given a newly-recorded `IdentityEvent` (and the
relevant recent history from MongoDB), it decides whether a pattern of
concern has occurred and, if so, hands back evidence for an alert.

## Files

| File | Detects |
|---|---|
| `passwordSpray.js` | Many failed logins against many distinct users from a single source IP within a short window classic credential-spraying behavior. |
| `mfaAbuse.js` | Repeated MFA resets, or an MFA reset immediately followed by a successful login (a common account-takeover pattern). |
| `privilegeEscalation.js` | A user's role changed to a more privileged one (severity scales up sharply if the new role is `ADMIN`). |
| `unusualAdminLogin.js` | An admin account logging in off-hours, from a new IP, a new device, or a new location. |
| `impossibleTravel.js` | Two successful logins for the same user from different countries too close together in time to be physically possible. |
| `newDevice.js` | A login from a device outside the user's established baseline. |
| `accountTakeover.js` | A correlation detector fires when several weaker signals (new IP, new device, failed logins, a successful login, an MFA change) stack up for the same user within one window. |
| `common.js` | Shared helpers used across detectors: time-window queries, off-hours checks, baseline comparisons, and evidence-formatting utilities. |
| `index.js` | The entry point exports the ordered list of detectors and the function `services/eventPipeline.js` calls to run all *enabled* rules against a new event. |

## How a detector runs

1. `services/eventPipeline.js` calls into `index.js` after every event is
   stored.
2. Only detection rules currently marked **enabled** (`DetectionRule`
   documents) are run.
3. Each detector queries just the recent events it actually needs (e.g.
   failed logins from the same IP in the last N minutes) no full-collection
   scans.
4. Before raising anything, the detector's result is checked against that
   rule's configured exclusions (`lib/exclusions.js`) so tuned-out false
   positives stay suppressed.
5. If the pattern still matches, a transparent risk score
   (`lib/riskScoring.js`) is computed and an alert is created or, if one
   already exists for the same `correlationKey`, the new evidence is
   merged into it instead of creating a duplicate.

## Design notes

- **No black-box ML.** Every detector is a named, explainable rule with a
  configurable threshold and time window (see `DetectionRule` in
  `/models`) intentional, per the project's offensive-security /
  transparency requirements.
- Detectors are side-effect-light: they read events and return a
  decision + evidence object. Persisting the alert, firing a
  notification, and writing the audit log entry are the pipeline's job,
  not the detector's keeping each detector easy to unit-test in
  isolation.
- Thresholds, time windows, and enable/disable state all live in the
  `DetectionRule` documents seeded by `lib/defaultRules.js`, not as
  constants inside these files an admin can retune detection sensitivity
  from the UI without touching code.
