# Domain Model — Entity Relationship Diagrams

> Companion to [domain-model.md](domain-model.md). The authoritative field definitions,
> rules, and invariants live there; this file is a visual index. Field lists here are
> abbreviated (keys + the most important columns).

## Why several small diagrams instead of one big one?

Zed's markdown preview renders each mermaid block as a **static SVG** — there is no
pan/zoom or horizontal scroll, so a wide diagram overflows and is hard to read. Each
diagram below is deliberately small (≤ ~6 entities). For the full graph, see
[domain-model.md §2](domain-model.md#2-entity-relationship-overview).

**Viewing tips**

1. Use the split diagrams below — they are sized to fit.
2. For the full graph, copy a block into <https://mermaid.live> (pan, zoom, export SVG/PNG).
3. Widen the editor pane, or open the preview in its own window/tab.
4. Optional: export to an image with the Mermaid CLI
   (`npx @mermaid-js/mermaid-cli -i diagram.mmd -o diagram.svg`; requires Node).

**Legend**

| Notation | Meaning |
|---|---|
| `||--o|` | one to zero-or-one |
| `||--o{` | one to many |
| `}o--o{` | many to many |
| `PK` / `FK` | primary key / foreign key |
| **content** | bundled, read-only, versioned reference data |

---

## 1. Identity & journeys

```mermaid
erDiagram
    PROFILE ||--o{ JOURNEY : owns
    JOURNEY ||--o| CHILD : results_in

    PROFILE {
        int id PK
        string primaryJourneyId FK
        string units
        string region
        string themeMode
        boolean appLockEnabled
        boolean screenshotProtection
    }
    JOURNEY {
        string id PK
        string type
        string status
        string outcome
        boolean isPrimary
        date startedOn
        date endedOn
        instant deletedAt
    }
    CHILD {
        string id PK
        date birthOn
        string sex
        boolean isPreterm
        date adjustedAgeUntilOn
    }
```

## 2. Stage details

```mermaid
erDiagram
    JOURNEY ||--o| PREGNANCY_DETAILS : has
    JOURNEY ||--o| POSTPARTUM_DETAILS : has
    JOURNEY ||--o| CYCLE_PROFILE : has
    JOURNEY ||--o{ CYCLE_ENTRY : logs
    JOURNEY ||--o{ OVULATION_TEST : logs

    PREGNANCY_DETAILS {
        string journeyId PK
        date lmpOn
        date eddOn
        string eddSource
        int fetusCount
        string riskFlags
        json eddHistory
    }
    POSTPARTUM_DETAILS {
        string journeyId PK
        date birthOn
        string birthType
        int gestationalAgeAtBirthWeeks
        string feedingMethod
        string childId FK
    }
    CYCLE_PROFILE {
        string journeyId PK
        int averageCycleLengthDays
        int averagePeriodLengthDays
        int lutealPhaseDays
        string regularity
        boolean pcosFlag
    }
    CYCLE_ENTRY {
        string id PK
        string journeyId FK
        date startOn
        date endOn
        string flow
        boolean isPredicted
    }
    OVULATION_TEST {
        string id PK
        string journeyId FK
        date testedOn
        string result
        instant testedAt
    }
```

## 3. Daily logging

```mermaid
erDiagram
    JOURNEY ||--o{ DAILY_LOG : logs
    JOURNEY ||--o{ HYDRATION_LOG : logs
    JOURNEY ||--o{ ACTIVITY_LOG : logs
    JOURNEY ||--o{ MOOD_CHECKIN : logs
    JOURNEY ||--o{ EPDS_ASSESSMENT : completes
    DAILY_LOG }o--o{ SYMPTOM_DEFINITION : references

    DAILY_LOG {
        string id PK
        string journeyId FK
        date loggedOn
        int mood
        int energy
        double weightKg
        double sleepHours
        string symptoms
    }
    HYDRATION_LOG {
        string id PK
        string journeyId FK
        date loggedOn
        int amountMl
    }
    ACTIVITY_LOG {
        string id PK
        string journeyId FK
        date loggedOn
        string activityType
        int minutes
    }
    MOOD_CHECKIN {
        string id PK
        string journeyId FK
        date loggedOn
        int score
        string tags
    }
    EPDS_ASSESSMENT {
        string id PK
        string journeyId FK
        date takenOn
        int totalScore
        int item10Score
    }
    SYMPTOM_DEFINITION {
        string key PK
        string label
        string category
        boolean redFlag
    }
```

## 4. Food & conditions

```mermaid
erDiagram
    JOURNEY ||--o{ FOOD_ENTRY : logs
    JOURNEY ||--o{ CONDITION : has
    FOOD_ITEM ||--o{ FOOD_ENTRY : referenced_by

    FOOD_ITEM {
        string id PK
        string name
        string category
        string risk
        string mercuryLevel
        string contentVersion
    }
    FOOD_ENTRY {
        string id PK
        string journeyId FK
        string foodItemId FK
        date loggedOn
        string meal
        string riskSnapshot
    }
    CONDITION {
        string id PK
        string journeyId FK
        string type
        date diagnosedOn
        boolean isActive
    }
```

## 5. Kick counter & contractions

```mermaid
erDiagram
    JOURNEY ||--o{ KICK_SESSION : tracks
    JOURNEY ||--o{ CONTRACTION_LOG : tracks

    KICK_SESSION {
        string id PK
        string journeyId FK
        instant startedAt
        instant endedAt
        int kickCount
    }
    CONTRACTION_LOG {
        string id PK
        string journeyId FK
        instant startedAt
        instant endedAt
        int durationSeconds
        string intensity
    }
```

## 6. Care & reminders

```mermaid
erDiagram
    PROFILE ||--o{ EMERGENCY_CONTACT : has
    JOURNEY ||--o{ APPOINTMENT : schedules
    JOURNEY ||--o{ REMINDER : configures
    JOURNEY ||--o{ SUPPLEMENT : takes
    JOURNEY ||--o{ HOSPITAL_BAG_ITEM : prepares

    EMERGENCY_CONTACT {
        string id PK
        int profileId FK
        string name
        string relationship
        string phone
        boolean isPrimary
    }
    APPOINTMENT {
        string id PK
        string journeyId FK
        string title
        string type
        instant startsAt
        string timeZone
        string status
    }
    REMINDER {
        string id PK
        string journeyId FK
        string type
        string scheduleType
        int intervalMinutes
        boolean enabled
        instant nextTriggerAt
    }
    SUPPLEMENT {
        string id PK
        string journeyId FK
        string name
        string schedule
        boolean isActive
    }
    HOSPITAL_BAG_ITEM {
        string id PK
        string journeyId FK
        string label
        string category
        boolean isPacked
    }
```

## 7. Postpartum — milestones & growth

```mermaid
erDiagram
    CHILD ||--o{ MILESTONE_ACHIEVEMENT : achieves
    CHILD ||--o{ GROWTH_MEASUREMENT : measures
    MILESTONE_DEFINITION ||--o{ MILESTONE_ACHIEVEMENT : defines

    MILESTONE_DEFINITION {
        string key PK
        string title
        string category
        int windowStartMonths
        int windowEndMonths
    }
    MILESTONE_ACHIEVEMENT {
        string id PK
        string childId FK
        string milestoneKey FK
        date achievedOn
        boolean concernFlag
    }
    GROWTH_MEASUREMENT {
        string id PK
        string childId FK
        date measuredOn
        double weightKg
        double lengthCm
        double headCircumferenceCm
    }
```

## 8. Postpartum — baby logs & vaccines

```mermaid
erDiagram
    CHILD ||--o{ FEEDING_LOG : logs
    CHILD ||--o{ DIAPER_LOG : logs
    CHILD ||--o{ SLEEP_LOG : logs
    CHILD ||--o{ VACCINATION_RECORD : receives
    VACCINE_DEFINITION ||--o{ VACCINATION_RECORD : defines

    FEEDING_LOG {
        string id PK
        string childId FK
        instant startedAt
        instant endedAt
        string type
        int amountMl
    }
    DIAPER_LOG {
        string id PK
        string childId FK
        instant occurredAt
        string type
        string consistency
    }
    SLEEP_LOG {
        string id PK
        string childId FK
        instant startedAt
        instant endedAt
        string location
    }
    VACCINE_DEFINITION {
        string key PK
        string name
        int doseNumber
        int recommendedAgeMonths
        string region
    }
    VACCINATION_RECORD {
        string id PK
        string childId FK
        string vaccineKey FK
        date dueOn
        date administeredOn
        string status
    }
```

## 9. Content tables (cross-cutting)

```mermaid
erDiagram
    FOOD_ITEM {
        string id PK
        string name
        string category
        string risk
        string contentVersion
    }
    FRUIT_SIZE {
        int week PK
        string name
        double lengthCm
        double weightG
        string contentVersion
    }
    WEEK_CONTENT {
        int week PK
        string title
        string body
        string contentVersion
    }
    SYMPTOM_DEFINITION {
        string key PK
        string label
        string category
        boolean redFlag
    }
    MILESTONE_DEFINITION {
        string key PK
        string title
        string category
    }
    VACCINE_DEFINITION {
        string key PK
        string name
        int doseNumber
        string region
    }
```

## 10. Export

```mermaid
erDiagram
    EXPORT_RECORD {
        string id PK
        instant createdAt
        string fileName
        string schemaVersion
        string appVersion
        boolean encrypted
        string checksum
    }
```

---

## Relationship matrix

A compact, text-only view of the same relationships (useful when a diagram is too wide).

| Entity | Kind | Parent / FK | Related entities |
|---|---|---|---|
| Profile | user | — | Journey, EmergencyContact |
| Journey | user | Profile.primaryJourneyId | PregnancyDetails, PostpartumDetails, CycleProfile, CycleEntry, OvulationTest, DailyLog, FoodEntry, Condition, Appointment, Reminder, Supplement, HydrationLog, ActivityLog, MoodCheckIn, EpdsAssessment, HospitalBagItem, KickSession, ContractionLog, Child |
| PregnancyDetails | user | Journey (1:1) | — |
| PostpartumDetails | user | Journey (1:1), Child | — |
| Child | user | Journey (1:1) | MilestoneAchievement, GrowthMeasurement, FeedingLog, DiaperLog, SleepLog, VaccinationRecord |
| CycleProfile | user | Journey (1:1) | — |
| CycleEntry | user | Journey | — |
| OvulationTest | user | Journey | — |
| DailyLog | user | Journey | SymptomDefinition (M:N) |
| FoodItem | content | — | FoodEntry |
| FoodEntry | user | Journey, FoodItem | — |
| Condition | user | Journey | — |
| Appointment | user | Journey | — |
| Reminder | user | Journey (nullable) | — |
| Supplement | user | Journey | — |
| HydrationLog | user | Journey | — |
| ActivityLog | user | Journey | — |
| MilestoneDefinition | content | — | MilestoneAchievement |
| MilestoneAchievement | user | Child, MilestoneDefinition | — |
| GrowthMeasurement | user | Child | — |
| FeedingLog | user | Child | — |
| DiaperLog | user | Child | — |
| SleepLog | user | Child | — |
| VaccineDefinition | content | — | VaccinationRecord |
| VaccinationRecord | user | Child, VaccineDefinition | — |
| MoodCheckIn | user | Journey | — |
| EpdsAssessment | user | Journey | — |
| EmergencyContact | user | Profile | — |
| HospitalBagItem | user | Journey | — |
| ExportRecord | user | — | — |
| FruitSize | content | — | — |
| WeekContent | content | — | — |
| SymptomDefinition | content | — | DailyLog (M:N) |
| KickSession | user | Journey | — |
| ContractionLog | user | Journey | — |

## Key invariants (see [domain-model.md §5](domain-model.md#5-invariants))

- Exactly one `Profile` row.
- Exactly one `Journey.isPrimary = true` (or zero during onboarding).
- A `PREGNANCY` journey has at most one `PregnancyDetails`.
- A `POSTPARTUM` journey has exactly one `Child`.
- `eddOn` changes are append-only in `eddHistory`.
- `FoodEntry.riskSnapshot` is immutable after insert.
- Predicted cycle entries are never treated as facts.
- Deleting a `Child` requires deleting or reassigning its logs in one transaction.
- No user data is written outside the app's private storage.
- Every content row has a `sourceRef` and `reviewedOn`.
