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
| `"..."` | example value (rendered as the comment column) |
| **content** | bundled, read-only, versioned reference data |

---

## 1. Identity & journeys

```mermaid
erDiagram
    PROFILE ||--o{ JOURNEY : owns
    JOURNEY ||--o| CHILD : results_in

    PROFILE {
        int id PK "1"
        string primaryJourneyId FK "01J8Z3K4Q7"
        string units "METRIC"
        string region "US"
        string themeMode "SYSTEM"
        boolean appLockEnabled "false"
        boolean screenshotProtection "false"
    }
    JOURNEY {
        string id PK "01J8Z3K4Q7"
        string type "PREGNANCY"
        string status "ACTIVE"
        string outcome "ONGOING"
        boolean isPrimary "true"
        date startedOn "2026-03-01"
        date endedOn "null"
        instant deletedAt "null"
    }
    CHILD {
        string id PK "01J9A1B2C3"
        date birthOn "2026-11-20"
        string sex "FEMALE"
        boolean isPreterm "false"
        date adjustedAgeUntilOn "null"
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
        string journeyId PK "01J8Z3K4Q7"
        date lmpOn "2026-02-01"
        date eddOn "2026-11-08"
        string eddSource "LMP"
        int fetusCount "1"
        string riskFlags "GDM"
        json eddHistory "2 entries"
    }
    POSTPARTUM_DETAILS {
        string journeyId PK "01J9B4D5E6"
        date birthOn "2026-11-20"
        string birthType "VAGINAL"
        int gestationalAgeAtBirthWeeks "39"
        string feedingMethod "BREAST"
        string childId FK "01J9A1B2C3"
    }
    CYCLE_PROFILE {
        string journeyId PK "01J7C8D9E0"
        int averageCycleLengthDays "28"
        int averagePeriodLengthDays "5"
        int lutealPhaseDays "14"
        string regularity "IRREGULAR"
        boolean pcosFlag "false"
    }
    CYCLE_ENTRY {
        string id PK "01J7D1E2F3"
        string journeyId FK "01J7C8D9E0"
        date startOn "2026-01-05"
        date endOn "2026-01-09"
        string flow "MEDIUM"
        boolean isPredicted "false"
    }
    OVULATION_TEST {
        string id PK "01J7E4F5G6"
        string journeyId FK "01J7C8D9E0"
        date testedOn "2026-01-18"
        string result "POSITIVE"
        instant testedAt "2026-01-18T07:30:00Z"
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
    DAILY_LOG |o--o| MOOD_CHECKIN : "mood for the day"
    DAILY_LOG }o--o{ SYMPTOM_DEFINITION : references

    DAILY_LOG {
        string id PK "01J8F7G8H9"
        string journeyId FK "01J8Z3K4Q7"
        date loggedOn "2026-06-01"
        string moodCheckInId FK "01J8J7K8L9"
        int energy "3"
        double weightKg "68.4"
        double sleepHours "7.5"
        string symptoms "nausea, fatigue"
    }
    HYDRATION_LOG {
        string id PK "01J8G1H2J3"
        string journeyId FK "01J8Z3K4Q7"
        date loggedOn "2026-06-01"
        int amountMl "250"
    }
    ACTIVITY_LOG {
        string id PK "01J8H4J5K6"
        string journeyId FK "01J8Z3K4Q7"
        date loggedOn "2026-06-01"
        string activityType "WALK"
        int minutes "20"
    }
    MOOD_CHECKIN {
        string id PK "01J8J7K8L9"
        string journeyId FK "01J8Z3K4Q7"
        date loggedOn "2026-06-01"
        int score "4"
        string tags "calm"
        string note "Felt anxious in the morning"
    }
    EPDS_ASSESSMENT {
        string id PK "01J8K1L2M3"
        string journeyId FK "01J9B4D5E6"
        date takenOn "2026-12-01"
        int totalScore "7"
        int item10Score "0"
    }
    SYMPTOM_DEFINITION {
        string key PK "symptom.nausea"
        string label "Nausea"
        string category "digestive"
        boolean redFlag "false"
    }
```

## 4. Food & conditions

```mermaid
erDiagram
    JOURNEY ||--o{ FOOD_ENTRY : logs
    JOURNEY ||--o{ CONDITION : has
    FOOD_ITEM ||--o{ FOOD_ENTRY : referenced_by

    FOOD_ITEM {
        string id PK "food.salmon.farmed"
        string name "Farmed salmon"
        string category "fish"
        string risk "SAFE"
        string mercuryLevel "LOW"
        string contentVersion "1.0.0"
    }
    FOOD_ENTRY {
        string id PK "01J8M4N5P6"
        string journeyId FK "01J8Z3K4Q7"
        string foodItemId FK "food.salmon.farmed"
        date loggedOn "2026-06-01"
        string meal "DINNER"
        string riskSnapshot "SAFE"
    }
    CONDITION {
        string id PK "01J8N7P8Q9"
        string journeyId FK "01J8Z3K4Q7"
        string type "GESTATIONAL_DIABETES"
        date diagnosedOn "2026-06-10"
        boolean isActive "true"
    }
```

## 5. Kick counter & contractions

```mermaid
erDiagram
    JOURNEY ||--o{ KICK_SESSION : tracks
    JOURNEY ||--o{ CONTRACTION_LOG : tracks

    KICK_SESSION {
        string id PK "01J8P1Q2R3"
        string journeyId FK "01J8Z3K4Q7"
        instant startedAt "2026-08-01T20:00:00Z"
        instant endedAt "2026-08-01T20:30:00Z"
        int kickCount "10"
    }
    CONTRACTION_LOG {
        string id PK "01J8Q4R5S6"
        string journeyId FK "01J8Z3K4Q7"
        instant startedAt "2026-11-20T02:00:00Z"
        instant endedAt "2026-11-20T02:01:00Z"
        int durationSeconds "60"
        string intensity "MODERATE"
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
        string id PK "01J8R7S8T9"
        int profileId FK "1"
        string name "Alex Doe"
        string relationship "Partner"
        string phone "+1-555-0100"
        boolean isPrimary "true"
    }
    APPOINTMENT {
        string id PK "01J8S1T2U3"
        string journeyId FK "01J8Z3K4Q7"
        string title "20-week scan"
        string type "ULTRASOUND"
        instant startsAt "2026-06-15T14:00:00Z"
        string timeZone "America/New_York"
        string status "SCHEDULED"
    }
    REMINDER {
        string id PK "01J8T4U5V6"
        string journeyId FK "01J8Z3K4Q7"
        string type "HYDRATION"
        string scheduleType "INTERVAL"
        int intervalMinutes "120"
        boolean enabled "true"
        instant nextTriggerAt "2026-06-01T10:00:00Z"
    }
    SUPPLEMENT {
        string id PK "01J8U7V8W9"
        string journeyId FK "01J8Z3K4Q7"
        string name "Folic acid"
        string schedule "DAILY"
        boolean isActive "true"
    }
    HOSPITAL_BAG_ITEM {
        string id PK "01J8V1W2X3"
        string journeyId FK "01J8Z3K4Q7"
        string label "Phone charger"
        string category "essentials"
        boolean isPacked "false"
    }
```

## 7. Postpartum — milestones & growth

```mermaid
erDiagram
    CHILD ||--o{ MILESTONE_ACHIEVEMENT : achieves
    CHILD ||--o{ GROWTH_MEASUREMENT : measures
    MILESTONE_DEFINITION ||--o{ MILESTONE_ACHIEVEMENT : defines

    MILESTONE_DEFINITION {
        string key PK "social.first_smile"
        string title "First social smile"
        string category "SOCIAL"
        int windowStartMonths "1"
        int windowEndMonths "3"
    }
    MILESTONE_ACHIEVEMENT {
        string id PK "01J9C4D5E6"
        string childId FK "01J9A1B2C3"
        string milestoneKey FK "social.first_smile"
        date achievedOn "2027-01-05"
        boolean concernFlag "false"
    }
    GROWTH_MEASUREMENT {
        string id PK "01J9D7E8F9"
        string childId FK "01J9A1B2C3"
        date measuredOn "2027-01-20"
        double weightKg "5.2"
        double lengthCm "58.0"
        double headCircumferenceCm "39.5"
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
        string id PK "01J9E1F2G3"
        string childId FK "01J9A1B2C3"
        instant startedAt "2027-01-01T03:00:00Z"
        instant endedAt "2027-01-01T03:20:00Z"
        string type "BREAST_LEFT"
        int amountMl "null"
    }
    DIAPER_LOG {
        string id PK "01J9F4G5H6"
        string childId FK "01J9A1B2C3"
        instant occurredAt "2027-01-01T04:00:00Z"
        string type "WET"
        string consistency "null"
    }
    SLEEP_LOG {
        string id PK "01J9G7H8J9"
        string childId FK "01J9A1B2C3"
        instant startedAt "2027-01-01T22:00:00Z"
        instant endedAt "2027-01-02T02:00:00Z"
        string location "crib"
    }
    VACCINE_DEFINITION {
        string key PK "us.dtap.1"
        string name "DTaP"
        int doseNumber "1"
        int recommendedAgeMonths "2"
        string region "US"
    }
    VACCINATION_RECORD {
        string id PK "01J9H1J2K3"
        string childId FK "01J9A1B2C3"
        string vaccineKey FK "us.dtap.1"
        date dueOn "2027-01-20"
        date administeredOn "2027-01-22"
        string status "DONE"
    }
```

## 9. Content tables (cross-cutting)

```mermaid
erDiagram
    FOOD_ITEM {
        string id PK "food.salmon.farmed"
        string name "Farmed salmon"
        string category "fish"
        string risk "SAFE"
        string contentVersion "1.0.0"
    }
    FRUIT_SIZE {
        int week PK "8"
        string name "Raspberry"
        double lengthCm "1.6"
        double weightG "1.0"
        string contentVersion "1.0.0"
    }
    WEEK_CONTENT {
        int week PK "8"
        string title "Week 8"
        string body "Tiny but growing fast."
        string contentVersion "1.0.0"
    }
    SYMPTOM_DEFINITION {
        string key PK "symptom.nausea"
        string label "Nausea"
        string category "digestive"
        boolean redFlag "false"
    }
    MILESTONE_DEFINITION {
        string key PK "social.first_smile"
        string title "First social smile"
        string category "SOCIAL"
    }
    VACCINE_DEFINITION {
        string key PK "us.dtap.1"
        string name "DTaP"
        int doseNumber "1"
        string region "US"
    }
```

## 10. Export

```mermaid
erDiagram
    EXPORT_RECORD {
        string id PK "01J9J4K5L6"
        instant createdAt "2026-11-01T12:00:00Z"
        string fileName "cocobean-2026-11-01.json"
        string schemaVersion "1"
        string appVersion "1.0.0"
        boolean encrypted "true"
        string checksum "sha256:ab12cd34"
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
| DailyLog | user | Journey, MoodCheckIn | SymptomDefinition (M:N) |
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
| MoodCheckIn | user | Journey | DailyLog (1:1) |
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
- Mood is stored only in `MoodCheckIn`; a journey has at most one `MoodCheckIn` per day, and
  `DailyLog.moodCheckInId` (if set) references it.
