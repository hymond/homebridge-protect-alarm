# Changelog

## 1.2.0 — 2026-07-25

Maintenance release. No behavioural or config changes.

### Changed

- **Removed the `axios` dependency** in favour of Node's built-in `https`
  module. The plugin now has **zero runtime dependencies** and installs as a
  self-contained file. Request behaviour (JSON, 15s timeout, self-signed-cert
  handling, and non-2xx error reporting) is unchanged. One minor difference:
  the native client does not follow 3xx redirects, which the integration API
  does not return.
- **Raised minimum versions** to match: Homebridge `^1.8.0 || ^2.0.0` (declares
  Homebridge v2 support explicitly) and Node `>=18.15.0`.

## 1.1.0 — 2026-07-23

Fork release based on upstream 1.0.0, adding HomeKit alarm-triggered state.
Drop-in replacement: same platform alias and config keys.

### Added

- **Alarm Triggered state.** `armMode.status === "breach"` now maps to
  `SecuritySystemCurrentState = ALARM_TRIGGERED (4)`, so a real Protect alarm
  raises a HomeKit notification and can drive automations without anyone needing
  Protect access. Previously a breach read as *not armed* and the tile showed
  Off during an alarm.
- **`pollIntervalWhenArmed`** config option (default 2 seconds), used in place
  of `pollInterval` whenever the system is armed or arming, so breaches are
  detected quickly without polling aggressively while disarmed.
- Info-level logging when a breach is detected and when it clears, including
  `breachTriggerEventId` for correlation with the Protect event log.
- Momentary-trigger handling for a breach that begins and ends between two
  polls: reported as triggered and held briefly so HomeKit observes the edge.
  Cleared immediately on disarm.

### Changed

- `SecuritySystemTargetState` continues to report the armed mode derived from
  `armProfileId` while breached — it never follows the current state to 4,
  which is not a valid target value.
- Characteristic updates are now diffed against the last pushed value, so polls
  only emit real changes.

### Robustness

- Unknown `armMode.status` values degrade to upstream behaviour (treated as
  disarmed) with a single warning rather than throwing.
- Missing `breachDetectedAt` / `breachEventId` falls back to de-duplicating on
  status transitions, with a single warning.
- Whether Protect clears `breach` automatically or holds it until disarm is
  **unverified on hardware**; the state machine handles both and does not
  depend on a manual disarm. See the README.

## 1.0.0 — 2026-06-10

Initial public release.

- Native HomeKit Security System accessory for the UniFi Protect alarm
- Single UniFi OS **API-key** authentication (no username/password)
- Automatic arm-profile discovery by name (`Away` / `Night`, configurable)
- Two-way state sync via NVR polling, including which mode is active when
  armed externally (keyfob, Protect app, automations)
- Exit-delay ("arming") treated as armed to prevent HomeKit state flicker
- Configurable poll interval; advanced explicit profile-ID overrides
