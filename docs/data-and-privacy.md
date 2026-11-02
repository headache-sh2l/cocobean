# Data and Privacy

> Status: 🟡 Proposed. "100% on-device" is a promise that must be enforced by the
> manifest, the storage layout, and the export design — not just the marketing copy.

## 1. Threat model

| Threat | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Lost/stolen unlocked device | Medium | High | Android FBE + optional app lock + optional `FLAG_SECURE` |
| Shoulder surfing | Medium | Medium | App lock, screenshot protection |
| Malicious app with root | Low | High | Out of scope; document |
| Cloud backup extraction | Medium | High | `allowBackup=false`, exclude from backup |
| Export file shared insecurely | Medium | High | Encrypt by default, warn on share |
| Analytics/ad SDK exfiltration | High if added | High | No SDKs, no `INTERNET` permission |
| Crash reporter leaking data | Medium | High | No crash reporter; local logs only |
| Notification content on lock screen | High | Medium | Sensitive content hidden by default; user can enable previews |
| Forensic recovery after delete | Low | Medium | Document limits; FBE; no claim of secure erase |

## 2. Storage map

| Data | Location | Backup | Encryption |
|---|---|---|---|
| User records | Room DB (`/data/data/<pkg>/databases/cocobean.db`) | Excluded | Android FBE; optional SQLCipher |
| Preferences | DataStore (`/data/data/<pkg>/files/datastore/`) | Excluded | FBE |
| Photos | `/data/data/<pkg>/files/photos/` | Excluded | FBE |
| Exports | User-chosen via SAF | User-controlled | AES-256-GCM by default |
| Content DB | APK assets | N/A | Read-only |
| Logs | None in release | N/A | — |
| Cache | `/data/data/<pkg>/cache/` | Excluded | FBE |

## 3. Manifest and permissions

```xml
<application
    android:allowBackup="false"
    android:fullBackupContent="false"
    android:dataExtractionRules="@xml/data_extraction_rules"
    ... />
```

- `android:allowBackup="false"` and `dataExtractionRules` excluding everything.
- No `INTERNET` permission.
- No `ACCESS_NETWORK_STATE`.
- No location, contacts, SMS, call log, or external storage permissions.
- `POST_NOTIFICATIONS` only.
- `READ_CALENDAR` / `WRITE_CALENDAR` only if calendar integration ships.
- `USE_BIOMETRIC` for app lock.
- `SCHEDULE_EXACT_ALARM` only if exact appointment reminders ship.
- `RECEIVE_BOOT_COMPLETED` for rescheduling.
- `VIBRATE` optional.

**CI check:** fail the build if `INTERNET` or any analytics SDK appears in the merged
manifest.

## 4. Room schema

### 4.1. Conventions

- Table names: plural snake_case (`journeys`, `food_entries`).
- IDs: `TEXT` UUIDv7 for user data; `TEXT` stable keys for content.
- Dates: `INTEGER` epochDay; timestamps: `INTEGER` epochMillis.
- Booleans: `INTEGER` 0/1.
- Enums: `TEXT` with a `TypeConverter`; unknown values map to a safe default.
- Every user table has `created_at`, `updated_at`, and optional `deleted_at`.
- Foreign keys with `ON DELETE CASCADE` only where the parent owns the child; otherwise
  `RESTRICT` and handle in a transaction.

### 4.2. Tables

| Table | Key columns | Indices |
|---|---|---|
| `profile` | `id=1` | — |
| `journeys` | `id`, `type`, `status`, `is_primary` | `is_primary`, `type` |
| `pregnancy_details` | `journey_id` | — |
| `edd_history` | `id`, `journey_id`, `changed_at` | `journey_id` |
| `postpartum_details` | `journey_id` | — |
| `children` | `id`, `birth_on` | `birth_on` |
| `cycle_profiles` | `journey_id` | — |
| `cycle_entries` | `id`, `journey_id`, `start_on` | `journey_id, start_on` |
| `ovulation_tests` | `id`, `journey_id`, `tested_on` | `journey_id, tested_on` |
| `daily_logs` | `id`, `journey_id`, `logged_on` | unique `journey_id, logged_on` |
| `food_items` | `id`, `name`, `risk` | FTS on `name`, `aliases` |
| `food_entries` | `id`, `journey_id`, `logged_on` | `journey_id, logged_on` |
| `conditions` | `id`, `journey_id`, `type` | `journey_id` |
| `appointments` | `id`, `journey_id`, `starts_at` | `journey_id, starts_at` |
| `reminders` | `id`, `journey_id`, `next_trigger_at` | `journey_id`, `enabled` |
| `supplements` | `id`, `journey_id` | `journey_id` |
| `hydration_logs` | `id`, `journey_id`, `logged_on` | `journey_id, logged_on` |
| `activity_logs` | `id`, `journey_id`, `logged_on` | `journey_id, logged_on` |
| `milestone_definitions` | `key` | — |
| `milestone_achievements` | `id`, `child_id`, `milestone_key` | `child_id` |
| `growth_measurements` | `id`, `child_id`, `measured_on` | `child_id, measured_on` |
| `feeding_logs` | `id`, `child_id`, `started_at` | `child_id, started_at` |
| `diaper_logs` | `id`, `child_id`, `occurred_at` | `child_id, occurred_at` |
| `sleep_logs` | `id`, `child_id`, `started_at` | `child_id, started_at` |
| `vaccine_definitions` | `key` | `region` |
| `vaccination_records` | `id`, `child_id`, `vaccine_key` | `child_id` |
| `mood_checkins` | `id`, `journey_id`, `logged_on` | `journey_id, logged_on` |
| `epds_assessments` | `id`, `journey_id`, `taken_on` | `journey_id, taken_on` |
| `emergency_contacts` | `id`, `profile_id` | — |
| `hospital_bag_items` | `id`, `journey_id` | `journey_id` |
| `export_records` | `id`, `created_at` | — |

### 4.3. Migrations

- Every schema change ships a `Migration` and a migration test.
- No `fallbackToDestructiveMigration()` in release.
- If a migration cannot be written, block the upgrade and prompt an export first.
- Content tables are replaced atomically; user tables are never dropped.

## 5. Encryption

### 5.1. Baseline

- Android file-based encryption (FBE) protects data at rest on a locked device.
- App-private storage prevents other apps from reading the DB.
- This is the minimum; document it honestly.

### 5.2. Optional hardening

| Option | Pros | Cons |
|---|---|---|
| SQLCipher for Room | Strong at-rest encryption independent of FBE | Adds ~5 MB, key management, migration complexity |
| App lock (biometric + PIN) | Protects an unlocked device | UX friction; PIN storage must be hashed |
| `FLAG_SECURE` | Blocks screenshots/recents preview | User cannot screenshot their own data |
| Encrypted export | Protects shared backups | User must remember password |

**Recommendation:** ship app lock + encrypted export in MVP; defer SQLCipher until there
is a clear threat requirement. Revisit in `open-questions.md` Q-08.

### 5.3. Key management

- Use Android Keystore for any app-generated keys.
- Never hardcode keys.
- If the user sets an export password, derive a key with PBKDF2-HMAC-SHA256
  (≥ 600,000 iterations) or Argon2id, with a random salt.
- Store only a verifier, never the password.

## 6. Export / import specification

### 6.1. Envelope

```json
{
  "format": "cocobean.export",
  "schemaVersion": 1,
  "appVersion": "1.0.0 (1)",
  "contentVersion": "1.0.0",
  "exportedAt": "2026-11-01T10:15:30Z",
  "deviceLocale": "en-US",
  "recordCounts": { "journeys": 2, "dailyLogs": 140 },
  "checksum": "sha256:...",
  "encrypted": false,
  "payload": { "...": "..." }
}
```

- `payload` contains all user tables, keyed by table name.
- Content tables are **not** exported; they ship with the app.
- Photos are exported as base64 or as a zip sidecar; prefer a zip with `manifest.json`
  and `photos/`.
- `checksum` covers the canonical JSON of `payload`.

### 6.2. Encryption

- Default: encrypted with a user password.
- AES-256-GCM, random 12-byte nonce, PBKDF2/Argon2 key derivation.
- File extension: `.cocobean` (encrypted) or `.cocobean.json` (plain, with a warning).
- Never write the password to disk or logs.

### 6.3. Import

1. Pick a file via SAF.
2. Detect format and schema version.
3. If encrypted, prompt for password; verify with a test vector.
4. Validate schema and checksums.
5. Show a **preview**: counts, date range, journeys, children.
6. Choose mode:
   - **Replace all** (wipe current data after a confirmation and an automatic backup).
   - **Merge** (match by ID; on conflict, keep the newer `updated_at` or ask).
7. Run in a single transaction; roll back on any error.
8. Reschedule reminders and refresh the UI.
9. Show a summary: imported, skipped, conflicts.

### 6.4. Compatibility

- Importing a newer `schemaVersion` is refused with a clear message.
- Importing an older version runs migrations.
- Unknown fields are ignored, not fatal.
- Missing optional fields get safe defaults.

## 7. Deletion

| Action | Behavior |
|---|---|
| Delete a log | Hard delete after confirmation; undo snackbar for 5 seconds. |
| Delete a journey | Soft delete; 30-day recovery in Settings; reminders cancelled immediately. |
| Delete a child | Requires deleting/reassigning logs; soft delete; recovery window. |
| Delete a photo | Hard delete; remove the file. |
| Delete everything | Type "DELETE"; offer export first; wipe DB, DataStore, photos, exports in app storage; cancel all work. |

- Deletion is local and immediate; there is no server copy.
- Do not claim "secure erase" on flash storage; say "removed from the app."
- After "delete everything," the app returns to onboarding.

## 8. Photos

- Use the Android Photo Picker (no storage permission).
- Copy the selected image into app-private storage immediately.
- Downscale to a reasonable max (e.g., 2048px), strip EXIF (including location).
- Store a relative path, never an absolute `content://` URI.
- Handle missing files with a placeholder, not a crash.
- Photos are included in export as a zip sidecar.

## 9. App lock

- Optional, off by default.
- Biometric prompt with device-credential fallback.
- Lock on background after a configurable timeout (default 60s).
- Lock immediately on cold start.
- Do not lock during an active phone call or emergency flow.
- If biometrics fail repeatedly, fall back to device credential; never lock the user out
  of their own data permanently.

## 10. Logging and diagnostics

- Release builds: no health data in logs.
- Debug builds: structured logs, never committed.
- No crash reporting SDK.
- A local "Diagnostics" screen may show app version, DB version, content version, and
  reminder status — no personal data.

## 11. Play Store data safety

- Data collected: **none**.
- Data shared: **none**.
- Data processed ephemerally: **none**.
- Data encrypted in transit: N/A (no network).
- Users can request deletion: in-app "Delete everything."
- Health data: yes, stored locally; declare it accurately.

## 12. Privacy policy outline

1. What we store (health logs, dates, photos).
2. Where it is stored (on your device only).
3. What we do not do (no account, no cloud, no analytics, no ads, no sharing).
4. Your controls (export, delete, app lock).
5. Children's privacy (the app is for parents; no child accounts).
6. Contact.

## 13. Open questions

- SQLCipher at launch? (Recommend: no, revisit.)
- Encrypted export by default? (Recommend: yes.)
- Screenshot protection default on or off? (Recommend: off, with a toggle.)
- 30-day soft-delete recovery? (Recommend: yes for journeys, no for individual logs.)
