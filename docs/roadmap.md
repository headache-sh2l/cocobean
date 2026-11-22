# Roadmap

> Status: 🟡 Proposed. The order is chosen to de-risk the hardest problems first and to
> keep a shippable app at every phase.

## Phase 0 — Decisions and foundations (1–2 weeks) [MVP]

**Goal:** no feature code until the contract is stable.

- [ ] Resolve blocking items in `open-questions.md`.
- [ ] Rename package/application ID.
- [ ] Set Java 17, add KSP, Hilt, Room, DataStore, WorkManager, Navigation, serialization.
- [ ] Replace template theme with `design-system.md` tokens; disable dynamic color.
- [ ] Set `allowBackup=false`; add manifest CI check for `INTERNET`.
- [ ] Create `core/common` (`Clock`, `Result`, `AppError`, `IdGenerator`).
- [ ] Set up CI: lint, detekt, unit tests.
- [ ] Draft the content schema and load a 20-item sample food DB.

**Exit:** app builds, theme is correct, CI is green, no `INTERNET` permission.

## Phase 1 — Onboarding and journeys (1–2 weeks)

- [ ] [MVP] Profile + DataStore.
- [ ] [MVP] Onboarding: welcome, name (optional), stage (pregnancy only; planning/postpartum
  shown as "coming soon"), disclaimer.
- [ ] [MVP] Journey model + repository + primary journey.
- [ ] [MVP] App shell with stage-derived bottom nav.
- [ ] [Post-MVP] Settings: units, theme, change stage, delete data.
- [ ] [MVP] Empty states for every tab.

**Exit:** a user can onboard into pregnancy.

## Phase 2 — Pregnancy core (2–3 weeks)

- [ ] [MVP] EDD setup: LMP, ultrasound, manual, IVF, "not sure".
- [ ] [MVP] GA, trimester, term calculations with tests.
- [ ] [MVP] Pregnancy dashboard: week, fruit, countdown.
- [ ] [MVP] Fruit content (40 weeks) + illustrations.
- [ ] [MVP] Week-by-week content (first pass).
- [ ] [MVP] EDD change flow with audit + banner.
- [ ] [MVP] Kick counter with red-flag copy.
- [ ] [MVP] Appointments: local list + optional system calendar insert.
- [ ] [MVP] Hospital bag checklist.
- [ ] [Post-MVP] Daily log: symptoms, weight, notes; daily mood check-in.
- [ ] [Post-MVP] Symptom catalog content.
- [ ] [Post-MVP] Contraction timer with red-flag copy.
- [ ] [Post-MVP] Emergency contacts and red-flag guidance.
- [ ] [Post-MVP] Loss/termination flow: pause reminders, sensitive copy, no auto-advance.

**Exit:** a pregnant user can set up, see the correct week every day, use the kick counter,
add an appointment, and check the hospital bag.

## Phase 3 — Reminders and notifications (2 weeks) — post-MVP

- [ ] Reminder model + CRUD.
- [ ] Scheduler abstraction: WorkManager + AlarmManager.
- [ ] Hydration, activity, supplement, appointment reminders.
- [ ] Quiet hours, snooze, mark done.
- [ ] Permission flows and in-app status.
- [ ] Boot/timezone/DST rescheduling.
- [ ] Notification channels and deep links.
- [ ] Device matrix testing.

**Exit:** reminders fire reliably on stock and at least two OEM devices, survive reboot,
and degrade visibly when permission is denied.

## Phase 4 — Food safety (2 weeks) — post-MVP (V1.1)

- [ ] Food content schema + 300 reviewed items.
- [ ] Search with aliases and FTS.
- [ ] Risk detail with rationale and sources.
- [ ] Condition modifiers.
- [ ] Meal log with risk snapshot.
- [ ] Caffeine tracker.
- [ ] Gentle high-risk card.

**Exit:** a user can search any common food, understand the risk, and log a meal in ≤ 3
taps.

## Phase 5 — Export, import, privacy (1–2 weeks) — post-MVP

- [ ] JSON export with schema version and checksum.
- [ ] Encrypted export (password).
- [ ] Import with preview, merge/replace, rollback.
- [ ] Photo export sidecar.
- [ ] App lock (biometric + credential).
- [ ] Delete everything flow.
- [ ] Privacy policy + About/Sources screen.

**Exit:** export → wipe → import restores everything on a real device.

## Phase 6 — Planning to conceive (2 weeks) — post-MVP

- [ ] Cycle profile + entries.
- [ ] Predictions, fertile window, two-week wait.
- [ ] Irregular/PCOS handling.
- [ ] Planning dashboard + calendar.
- [ ] Transition to pregnancy.

**Exit:** a user with irregular cycles sees honest ranges, not false certainty.

## Phase 7 — Postpartum (3 weeks) — post-MVP

- [ ] Birth recording + child record.
- [ ] Milestones (CDC-based) with windows and red flags.
- [ ] Feeding, diaper, sleep logs.
- [ ] Growth measurements (log + trend; percentiles deferred).
- [ ] Vaccination schedule by region.
- [ ] EPDS with crisis escalation.
- [ ] Recovery content.
- [ ] Transition from pregnancy.

**Exit:** a postpartum user can log a feed in ≤ 5 seconds and complete an EPDS safely.

## Phase 8 — Polish and release (2 weeks) [MVP]

- [ ] Accessibility audit and fixes.
- [ ] Dark mode and 200% font pass.
- [ ] Performance: startup, list scrolling, search.
- [ ] Baseline profile.
- [ ] Clinical content sign-off.
- [ ] Beta with 10–20 users (pregnancy).
- [ ] Store listing, screenshots, data safety form.
- [ ] Release.

**Exit:** release gates in `testing.md` pass.

## MVP cut line

**MVP = Onboarding + Pregnancy journey + Kick counter + Appointments + Hospital bag
checklist.** Everything else is post-MVP. See [mvp.md](mvp.md).

Why: this is the smallest slice that still exercises the hardest problems (dates and
content) and is the most differentiated. Shipping more at once delays feedback and
multiplies the content review burden.

## Dependencies

| Phase | Depends on |
|---|---|
| 1 | 0 (theme, DI, DB) |
| 2 | 1 (journey model) |
| 3 | 1 (journey scoping) |
| 4 | 0 (content schema), 2 (pregnancy context) |
| 5 | 1 (data model) |
| 6 | 1, 3 (reminders) |
| 7 | 1, 3, 5 (export) |
| 8 | all |

## Risks by phase

| Phase | Risk | Mitigation |
|---|---|---|
| 0 | Package rename after release | Do it now. |
| 2 | Date math bugs | Property-based tests, fixed clock. |
| 3 | OEM notification failures | Device matrix, WorkManager fallback. |
| 4 | Content review bottleneck | Start sourcing in Phase 0; 300 items, not 1,000. |
| 5 | Export corruption | Atomic writes, checksums, round-trip tests. |
| 7 | Sensitive content (loss, PPD) | Clinician + user review; sensitive-mode design. |
| 8 | Accessibility debt | Per-screen acceptance criteria, not a final pass. |
