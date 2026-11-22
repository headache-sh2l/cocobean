# MVP Scope

> Single source of truth for what ships in the MVP. Everything not listed here is
> post-MVP. The canonical cut lives in
> [product-brief.md §6.1](product-brief.md#61-mvp-recommended-cut).

## In the MVP (7 features)

1. **Onboarding** — local profile, stage selection (pregnancy only; planning/postpartum
   shown as "coming soon"), due-date setup, disclaimer.
2. **Pregnancy journey** — EDD from LMP/ultrasound/manual/IVF, gestational age, trimester,
   week-by-week dashboard, fruit comparison, countdown.
3. **Kick counter** — session count with red-flag copy.
4. **Appointments** — local list + optional system calendar insert.
5. **Hospital bag checklist.**
6. **Birth plan** — common topics with a "considered" toggle and the user's chosen options.
7. **Pause Journey / Sensitive Mode** — minimal toggle to instantly halt active reminders and switch app tone to gentle, neutral copy (mitigating insensitive notifications after a loss).

## MVP entities

| Entity | Domain model § |
|---|---|
| Profile | [3.1](domain-model.md#31-profile-singleton-id--1) |
| Journey | [3.2](domain-model.md#32-journey) |
| PregnancyDetails | [3.3](domain-model.md#33-pregnancydetails-11-with-a-pregnancy-journey) |
| Appointment | [3.13](domain-model.md#313-appointment) |
| HospitalBagItem | [3.22](domain-model.md#322-emergencycontact--hospitalbagitem) |
| FruitSize / WeekContent | [3.24](domain-model.md#324-fruitsize--weekcontent-content) |
| KickSession | [3.26](domain-model.md#326-kicksession--contractionlog) |
| BirthPlanTopic / BirthPlanItem | [3.27](domain-model.md#327-birthplantopic-content--birthplanitem) |

## Explicitly post-MVP

- Daily log + mood check-in (`DailyLog`, `MoodCheckIn`, `SymptomDefinition`)
- Food entry and food safety (`FoodEntry`, `FoodItem`, `Condition`)
- Reminders (`Reminder`, `Supplement`, `HydrationLog`, `ActivityLog`)
- Export/import (`ExportRecord`)
- Settings (units, theme, app lock, data deletion, about)
- Safety: emergency contacts, red-flag guidance (`EmergencyContact`)
- Full Loss/termination workflow (minimal pause/sensitive mode is included in MVP)
- Contraction timer (`ContractionLog`)
- Planning to conceive (`CycleProfile`, `CycleEntry`, `OvulationTest`)
- Postpartum (`PostpartumDetails`, `Child`, milestones, growth, feeding/diaper/sleep,
  vaccines, EPDS)
- Mental health (`EpdsAssessment`)

## How the MVP is marked in the docs

- **`[MVP]`** — ships in the MVP.
- **`[Post-MVP]`** — deferred.
- Unmarked items are post-MVP unless the surrounding section is marked `[MVP]`.

## Risks of this cut

- **Full loss/termination workflow** is post-MVP (though a minimal pause/sensitive mode is in MVP).
- **Emergency contacts and red-flag guidance** are post-MVP. The product brief promises
  "call your provider" guidance; confirm this is acceptable.
- **Export/import and data deletion** are post-MVP. This weakens the "user owns the data"
  promise and may affect store/privacy expectations.
- **Birth plan content** must be clinician-reviewed. It is educational and must never imply
  a "correct" plan.
