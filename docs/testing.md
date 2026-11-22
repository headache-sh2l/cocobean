# Testing Strategy

> Status: 🟡 Proposed. The app handles dates, health guidance, and reminders. Bugs are not
> cosmetic; they erode trust.
>
> **MVP scope:** MVP tests cover onboarding, EDD/GA, the dashboard, kick counter,
> appointments, and the hospital bag. Reminder, food, export, and postpartum tests are
> post-MVP. See [mvp.md](mvp.md).

## 1. Test pyramid

| Layer | Share | Tools | What |
|---|---|---|---|
| Unit (pure Kotlin) | 60% | JUnit, Truth, property-based | Date math, cycle prediction, risk rules, EPDS, export mapping |
| Data | 20% | Room in-memory, Robolectric | DAOs, migrations, converters, content loading |
| UI | 15% | Compose UI test, Roborazzi | Screens, states, accessibility |
| End-to-end / manual | 5% | Device matrix | Notifications, reboot, timezone, OEM behavior |

## 2. What must be tested

### 2.1. Date and clinical math [MVP]

- [MVP] EDD from LMP, ultrasound, manual, IVF (day 3/5/6).
- [MVP] GA at boundaries: 0w0d, 13w6d, 14w0d, 27w6d, 28w0d, 37w0d, 42w0d.
- [MVP] Leap years, month ends, DST transitions.
- [MVP] EDD change recalculates GA and preserves logs.
- [Post-MVP] Cycle average with outliers, irregular cycles, PCOS flag.
- [Post-MVP] Fertile window with default and custom luteal phase.
- [Post-MVP] Two-week wait boundaries.
- [Post-MVP] EPDS scoring, including item 10 precedence.
- [Post-MVP] Food risk with condition modifiers.
- [MVP] Unit conversion round-trips.

### 2.2. State machine [MVP]

- Every transition in `domain-model.md` §4.
- Loss transition cancels reminders and never creates postpartum.
- Manual stage change preserves data.
- Primary journey uniqueness.
- Deleting a journey cascades correctly and respects soft delete.

### 2.3. Data [Post-MVP]

- DAO queries: today's log, week range, search, upcoming appointments.
- FTS search with aliases and typos.
- Migration from every prior schema version.
- Export → import round-trip preserves all records and photos.
- Import rejects newer schema, corrupt checksum, and partial data.
- Content version replacement keeps user snapshots.

### 2.4. Notifications [Post-MVP]

- Next-trigger calculation across quiet hours, DST, timezone, month boundaries.
- Coalescing and minimum spacing.
- Cancel on journey end.
- Reschedule on boot/time change (Robolectric or instrumented).
- Permission-denied state is surfaced.

### 2.5. UI and accessibility [MVP]

- Every screen: loading, empty, error, content.
- 200% font scaling: no clipping (screenshot tests).
- TalkBack semantics: labels, roles, focus order.
- Risk badges are not color-only.
- Dark mode contrast.
- Sensitive-mode copy (loss, NICU, infertility).

## 3. Test doubles

```kotlin
class TestClock(var now: Instant) : Clock { override fun instant() = now }
class FakeJourneyRepository : JourneyRepository { ... }
class FakeReminderScheduler : ReminderScheduler { ... }
class FakeContentRepository : ContentRepository { ... }
```

- No `Thread.sleep`; use `TestDispatcher` and `runTest`.
- No real network (there is none).
- No real notifications in unit tests.

## 4. Fixtures

- A `TestData` object with:
  - a planning journey with 6 cycles,
  - a pregnancy at 24w3d,
  - a postpartum journey with a 6-week-old,
  - a loss journey,
  - a twin pregnancy,
  - a preterm child.
- Golden files for export JSON.
- Golden images for fruit illustrations and key screens.

## 5. Property-based tests

Use for date invariants:

- `GA(edd, edd) == 0w0d`.
- `GA(edd, edd - 280 days) == 40w0d`.
- Adding a day never decreases GA.
- EDD change is monotonic in the audit log.
- Export → import → export is byte-identical (canonical JSON).

## 6. Database tests

- In-memory Room for DAO tests.
- `MigrationTestHelper` for every migration.
- Foreign key enforcement enabled in tests.
- Index usage checked with `EXPLAIN QUERY PLAN` for hot queries.

## 7. Compose tests

- `createComposeRule()` for screen tests.
- Test semantics, not pixels, for behavior.
- Roborazzi/Paparazzi for visual regression on key screens and both themes.
- Accessibility checks with `AccessibilityChecks.enable()`.

## 8. Manual device matrix

| Dimension | Values |
|---|---|
| Device | Pixel (stock), Samsung, Xiaomi, Huawei, OnePlus |
| Android | 14, 15, 16, 17 |
| Locale | en-US, en-GB, one RTL locale |
| Font scale | 100%, 200% |
| Theme | Light, dark, system |
| Network | Airplane mode (must work) |
| Timezone | UTC, UTC+5:30, UTC-8, DST zone |
| Power | Battery saver, Doze, reboot, force-stop |

### Manual scenarios

1. [MVP] Onboard, set EDD, see correct week.
2. [Post-MVP] Create a hydration reminder 10 minutes out; lock the phone; confirm it fires.
3. [Post-MVP] Reboot; confirm the reminder still fires.
4. [Post-MVP] Change timezone; confirm the reminder follows local time.
5. [Post-MVP] Deny notification permission; confirm the in-app state.
6. [Post-MVP] Log a high-risk food; confirm the gentle card.
7. [Post-MVP] Export, wipe, import; confirm all data returns.
8. [Post-MVP] Enable app lock; background; confirm lock.
9. [MVP] Enable 200% font; walk every screen.
10. [MVP] Enable TalkBack; complete onboarding and add an appointment.

## 9. CI

- `./gradlew lint detekt ktlintCheck test`
- Instrumented tests on an emulator (nightly or per-PR if fast).
- Screenshot tests on PRs that touch UI.
- Manifest check: fail if `INTERNET` or analytics SDKs appear.
- Dependency check: no known-vulnerable versions.

## 10. Definition of done (per feature)

- [ ] Unit tests for domain logic.
- [ ] DAO/migration tests if data changes.
- [ ] Compose tests for loading/empty/error/content.
- [ ] Accessibility check.
- [ ] Dark mode and 200% font verified.
- [ ] Edge cases from `edge-cases.md` addressed or explicitly deferred.
- [ ] Copy reviewed for tone and certainty.
- [ ] No new permissions without a privacy review.
- [ ] Docs updated.

## 11. Release gates

- [ ] All tests green.
- [ ] No P0/P1 bugs.
- [ ] Clinical content reviewed and signed off.
- [ ] Accessibility audit passed.
- [ ] [Post-MVP] Export/import round-trip verified on a real device.
- [ ] [Post-MVP] Notification matrix passed on at least 3 OEMs.
- [ ] Privacy policy and disclaimer in place.
- [ ] Play data safety form matches reality.
