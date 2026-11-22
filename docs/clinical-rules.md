# Clinical Rules

> **Status:** 🟡 Proposed. **Every rule in this document requires review and sign-off by a
> qualified clinician before release.** The app is a record-keeping and education tool,
> not a diagnostic device.
>
> **Implementation rule:** all thresholds live in one `ClinicalConstants` object, are
> covered by unit tests, and are never hard-coded in Composables.
>
> **MVP scope:** only date arithmetic (§1), EDD (§2), gestational age/trimester/term (§3),
> pregnancy content triggers (§5), and the kick counter (§9.2) ship in the MVP. Everything
> else is post-MVP. See [mvp.md](mvp.md).

## 0. Safety posture

1. The app may **inform**, **remind**, and **escalate**. It may not **diagnose**,
   **prescribe**, or **reassure**.
2. When data is missing or ambiguous, show uncertainty. Never fill gaps with a default
   that looks like a fact.
3. Every risk assessment ends with a next step ("talk to your provider", "call your
   local emergency number").
4. Red-flag content is always reachable in ≤ 2 taps from any screen.
5. No rule may tell a user to disregard a clinician's advice.

## 1. Date arithmetic [MVP]

All date math uses `java.time.LocalDate` and an injected `Clock`.

```kotlin
fun gestationalAge(edd: LocalDate, today: LocalDate): GestationalAge {
    val days = ChronoUnit.DAYS.between(today, edd).toInt()
    val total = 280 - days          // 280 - daysUntilEdd
    return GestationalAge(weeks = total / 7, days = total % 7, totalDays = total)
}
```

- `totalDays` may be negative before conception; clamp display at `0w0d` and show
  "not yet pregnant" only in planning, never in a pregnancy journey.
- `totalDays > 294` (42w0d) is **post-term**; show "past your due date" content and a
  "talk to your provider" prompt. Do not induce panic.
- Never compute GA from `today - lmp` when an EDD exists; the EDD is the source of truth.

## 2. Estimated Due Date (EDD) [MVP]

### 2.1. Sources and precedence

| Source | Formula | When used |
|---|---|---|
| LMP | `lmp + 280 days` (Naegele) | Default when LMP is known and no scan EDD. |
| Ultrasound | User enters the EDD printed on the scan | Preferred when available. |
| Manual | User enters a date | When the user knows their EDD but not the derivation. |
| IVF | `transferDate + 266 - embryoAgeDays` | Day-3 transfer: +263; day-5: +261; day-6: +260. |

**Precedence:** ultrasound > IVF > manual > LMP, **but the app does not silently
override**. If the user has both an LMP-derived and a scan EDD, show both and ask which
to use, with a short explanation that clinicians sometimes revise the EDD when the scan
differs.

Common clinician revision thresholds (for copy only, not auto-decision):

- < 14w0d: difference > 7 days
- 14w0d–27w6d: difference > 10 days
- ≥ 28w0d: difference > 14 days

> These thresholds vary by guideline and region. The app must not apply them
> automatically. It may say: "Your scan date differs from your LMP date. Your provider
> may adjust your due date."

### 2.2. EDD changes

- Every change appends `{oldEdd, newEdd, source, changedAt}` to `eddHistory`.
- Past logs keep their original dates; only derived values (GA, week content) change.
- Show a one-time banner: "Your due date changed. Your week-by-week guide has been
  updated."

### 2.3. Missing dates

- If neither LMP nor EDD is known, the pregnancy journey is `ACTIVE` but **incomplete**.
- Dashboard shows a setup card, not a fabricated week.
- Offer: "I know my due date", "I know my last period", "I had a scan", "I'm not sure".

## 3. Gestational age, trimester, term [MVP]

| Concept | Rule |
|---|---|
| Trimester 1 | 0w0d – 13w6d |
| Trimester 2 | 14w0d – 27w6d |
| Trimester 3 | 28w0d – birth |
| Early term | 37w0d – 38w6d |
| Full term | 39w0d – 40w6d |
| Late term | 41w0d – 41w6d |
| Post-term | ≥ 42w0d |
| Viability | Region-dependent (commonly 22–24w). Content flag, never a hardcoded claim. |

Display format: `24w 3d`, with a secondary "Week 25" label where the product uses
"week of pregnancy" language. Be consistent: **GA is completed weeks + days**, while
"you are in week 25" means the 25th week is in progress. Pick one convention per screen
and document it in the copy deck.

## 4. Cycle and fertility (Planning) [Post-MVP]

### 4.1. Cycle length

- Cycle length = days between consecutive period start dates.
- Use the last 6 cycles; discard cycles < 21 or > 60 days as outliers for the average,
  but still display them in history.
- Default 28 days when there is no data.
- Clamp the prediction input to 21–45 days.

### 4.2. Regularity

- `REGULAR` if the range of the last 3–6 cycles is ≤ 7 days.
- `IRREGULAR` if range > 7 days, or the user flags PCOS, or cycles are < 21 / > 35 days.
- `UNKNOWN` with fewer than 2 cycles.

### 4.3. Ovulation and fertile window

- `ovulationDay = cycleLength - lutealPhaseDays` (default luteal = 14).
- Fertile window = `ovulationDay - 5` through `ovulationDay + 1`.
- If `IRREGULAR` or `pcosFlag`, widen the window by ±2 days and show:
  "Your cycles vary, so this is a wider estimate. Ovulation tests are more reliable
  than calendar predictions."
- Never display a single "ovulation day" as a guarantee. Always a range.
- Do not predict while the user reports hormonal contraception.

### 4.4. Two-week wait

- From `ovulationDay + 1` to the day before the next predicted period.
- Reminders: gentle, optional, never daily pressure.
- A late period is **not** a pregnancy diagnosis. Offer "Take a test" content, not a
  congratulation.

### 4.5. Positive test transition

- User-initiated only.
- Ask for test date and, optionally, a photo (local only).
- Create a pregnancy journey; keep the planning journey as history.
- Do not auto-delete cycle data.

## 5. Pregnancy content triggers [MVP]

| Trigger | Content |
|---|---|
| Week 4–8 | Early pregnancy, booking appointment, folic acid, foods to avoid |
| Week 11–14 | First-trimester screening, NT scan window |
| Week 18–22 | Anatomy scan window |
| Week 24–28 | Glucose screening window (region-dependent), kick counting intro |
| Week 28+ | Third trimester, birth prep, hospital bag |
| Week 36+ | Weekly visits, birth plan, signs of labor |
| Week 37–42 | Term, overdue, induction conversation |
| Any week | Red flags, mental health, hydration, movement |

Content is **educational**, not a schedule of required care. Always add: "Your provider
may follow a different schedule."

### 5.1. Birth plan topics [MVP]

- Topics are educational prompts, not recommendations.
- Never imply a "correct" birth plan; every option is valid.
- Include "your provider may have different guidance" copy.
- Content requires clinician review.

## 6. Nutrition and supplements [Post-MVP]

### 6.1. Supplements (content, not prescription)

| Item | General guidance | Notes |
|---|---|---|
| Folic acid | 400–800 mcg daily, starting before conception through the first trimester | Higher doses only if prescribed. |
| Prenatal vitamin | Daily, per product/clinician | App does not recommend brands or doses. |
| Iron | If recommended by provider | Do not advise starting/stopping. |
| Calcium / vitamin D | Per provider | |
| DHA / omega-3 | Per provider | |

The app may **track** what the user takes. It must not **recommend a dose**.

### 6.2. Caffeine

- General guidance: keep total caffeine ≤ 200 mg/day during pregnancy.
- Show a running total from logged drinks, with a gentle indicator, not a red alarm.
- Include common sources: coffee, tea, cola, energy drinks, chocolate, some medications.
- If exceeded: "You're over the general 200 mg guidance today. It's okay — just something
  to be aware of. Talk to your provider if you're unsure."

### 6.3. Alcohol

- No known safe amount during pregnancy. Copy: "The safest choice is none."
- Never shame a past entry.

### 6.4. Weight

- Optional. Never required.
- If pre-pregnancy BMI is known, show the IOM/NAM gestational weight-gain **range** as
  education, not a target.
- Never show a "you gained too much" message.

## 7. Food safety [Post-MVP]

### 7.1. Risk levels

| Level | Meaning | Visual |
|---|---|---|
| `SAFE` | Generally recommended in pregnancy, with normal precautions | Green/sage |
| `MODERATE` | Limit, prepare carefully, or condition-dependent | Amber |
| `AVOID` | Generally advised against in pregnancy | Muted coral |
| `UNKNOWN` | No reviewed guidance | Neutral, with "ask your provider" |

**`UNKNOWN` must never render as `SAFE`.**

### 7.2. Core rules (US/ACOG/FDA-aligned; region review required)

| Category | Guidance |
|---|---|
| Raw/undercooked meat, poultry, seafood | Avoid |
| Unpasteurized dairy and soft cheeses (unless labeled pasteurized) | Avoid |
| Deli meats, hot dogs, cold cuts | Avoid unless heated to steaming (165°F / 74°C) |
| Refrigerated pâté, meat spreads | Avoid |
| Raw sprouts | Avoid |
| High-mercury fish (shark, swordfish, king mackerel, tilefish, marlin, bigeye tuna) | Avoid |
| Albacore/white tuna | Limit to ~6 oz/week |
| Low-mercury fish (salmon, shrimp, cod, tilapia, sardines) | 2–3 servings/week (8–12 oz) |
| Raw eggs | Avoid unless pasteurized/cooked |
| Unwashed produce | Wash |
| Alcohol | Avoid |
| Caffeine | ≤ 200 mg/day |
| Herbal teas/supplements | Check with provider; many are not studied |
| Leftovers | Refrigerate promptly, reheat thoroughly |

### 7.3. Condition modifiers

| Condition | Effect |
|---|---|
| Gestational diabetes | Carb-aware notes, glucose log, no food is "forbidden" by the app |
| Hypertension / preeclampsia history | Sodium-aware notes, provider guidance |
| Anemia | Iron-rich food education |
| Lactose intolerance | Dairy alternatives, calcium sources |
| Vegetarian/vegan | B12, iron, omega-3, protein education |
| Allergies | Hard filter + warning on logged foods |

### 7.4. Meal log

- Logging is optional and fast (≤ 3 taps for a recent food).
- Risk is snapshotted at log time.
- A high-risk entry triggers a gentle card: "This is usually avoided in pregnancy.
  If you've already eaten it, don't panic — talk to your provider if you're worried."
- Never block logging. The app is a record, not a gatekeeper.

## 8. Hydration and activity [Post-MVP]

### 8.1. Hydration

- General total water guidance: ~2.3 L/day in pregnancy, ~2.7 L/day while lactating
  (includes water from food). Present as an adjustable goal, not a prescription.
- Default reminder interval: every 2 hours within waking hours (default 08:00–20:00).
- Quiet hours default 22:00–07:00.
- If the user logs less than the goal, no shame copy. "A gentle nudge, not a grade."

### 8.2. Activity

- General guidance: ~150 minutes/week of moderate activity if the pregnancy is
  uncomplicated. Always add "if your provider hasn't restricted activity."
- Default walking reminder: every 4 hours within waking hours.
- Contraindication content: bleeding, preterm labor risk, preeclampsia, etc. — "ask your
  provider before starting."
- Postpartum: pelvic floor and gentle walking content; C-section recovery notes.

## 9. Contraction timer [Post-MVP] and kick counter [MVP]

### 9.1. Contraction timer [Post-MVP]

- Records start/end, duration, frequency, intensity.
- Displays the **5-1-1** pattern: contractions every 5 minutes, lasting 1 minute, for
  1 hour.
- If < 37w0d and regular contractions: "This could be preterm labor. Call your provider
  or emergency services now."
- If ≥ 37w0d and 5-1-1 met: "It may be time to call your provider."
- Never says "you are in labor."

### 9.2. Kick counter [MVP]

- Session: count movements until 10, or 2 hours, whichever comes first.
- If fewer than 10 in 2 hours: "Contact your provider today." Do not say "your baby is
  in distress."
- If the user reports reduced movement at any time: immediate red-flag card.
- Sessions are local only; no sharing.

## 10. Postpartum milestones [Post-MVP]

- Source: CDC "Learn the Signs. Act Early." (public domain in the US; verify before
  shipping) or an equivalent reviewed source for the user's region.
- Age windows: 2, 4, 6, 9, 12, 15, 18, 24, 30, 36 months.
- Each milestone has a window, not a deadline.
- Copy: "Most babies do this by X months. Every baby is different."
- Red flags: "Talk to your pediatrician if…" — never "your baby is delayed."
- Preterm: use adjusted age until 24 months (chronological age − weeks premature).
- Never compare children or show percentiles as a score.

## 11. Growth measurements [Post-MVP]

- Post-MVP: log weight, length, head circumference; plot a simple trend.
- Percentiles require WHO (0–2y) / CDC (2y+) LMS tables. Defer to V1.1.
- When percentiles ship: show the curve, the child's point, and "discuss with your
  pediatrician." Never label a child "underweight" or "failing to thrive."
- Unit conversion: kg ↔ lb, cm ↔ in, at display time only; store metric.

## 12. Vaccination schedule [Post-MVP]

- Region-specific. `Profile.region` selects the schedule.
- US: CDC/ACIP schedule. UK: NHS schedule. Others: defer or show "ask your provider."
- The app tracks doses and reminds; it does not recommend a vaccine.
- Copy: "This is the general schedule for your region. Your pediatrician may adjust it."
- Never mark a dose "overdue" in red; use "due" and "talk to your provider."

## 13. Mental health (EPDS) [Post-MVP]

### 13.1. Instrument

- Edinburgh Postnatal Depression Scale: 10 items, each scored 0–3, total 0–30.
- Validated for perinatal use. The app must use the official wording and scoring for the
  licensed language; verify licensing/translation before shipping.
- Screening cadence (suggested, user-initiated): 2 weeks, 6 weeks, 3 months, 6 months
  postpartum; also available anytime.

### 13.2. Scoring and escalation

| Score | Display |
|---|---|
| 0–9 | "Your answers don't suggest a concern right now. You can check in again anytime." |
| 10–12 | "Some of your answers suggest you may be struggling. Consider talking to your provider." |
| ≥ 13 | "Your answers suggest you may be experiencing depression. Please talk to your provider soon. This is not a diagnosis." |
| Item 10 > 0 | **Immediate safety card**: crisis line, emergency number, "you deserve support now." |

- Item 10 (self-harm) is checked **before** the total score.
- Never leave the user on a dead-end screen.
- Never store a crisis response in a shareable place.
- Region-specific crisis resources; if unknown, show local emergency number and
  international directories.

### 13.3. Copy rules

- "This is a screening tool, not a diagnosis."
- "Many parents feel this way. It is not your fault."
- No toxic positivity. No "just relax."
- Offer concrete next steps: talk to provider, call a helpline, tell one person.

## 14. Red flags and escalation [Post-MVP]

Show a persistent, reachable "Get help now" entry point. Trigger cards for:

| Symptom | Action |
|---|---|
| Heavy bleeding | Call provider/emergency now |
| Severe headache, vision changes, upper abdominal pain, sudden swelling | Possible preeclampsia — call now |
| Reduced fetal movement | Contact provider today |
| Regular contractions < 37w | Call provider/emergency now |
| Water breaking < 37w | Call provider now |
| Chest pain, difficulty breathing | Emergency |
| Fever ≥ 38°C / 100.4°F | Contact provider |
| Thoughts of self-harm | Crisis resources immediately |
| Seizure, fainting | Emergency |

Region-aware emergency numbers: 911 (US/CA), 999 (UK), 112 (EU), 000 (AU), 111 (NZ),
local equivalents. Always include "or your local emergency number."

## 15. Constants (single source of truth)

```kotlin
object ClinicalConstants {
    const val PREGNANCY_DAYS = 280
    const val DEFAULT_CYCLE_DAYS = 28
    const val DEFAULT_PERIOD_DAYS = 5
    const val DEFAULT_LUTEAL_DAYS = 14
    const val MIN_CYCLE_DAYS = 21
    const val MAX_CYCLE_DAYS = 45
    const val IRREGULAR_RANGE_DAYS = 7
    const val FERTILE_DAYS_BEFORE_OVULATION = 5
    const val FERTILE_DAYS_AFTER_OVULATION = 1
    const val CAFFEINE_LIMIT_MG = 200
    const val HYDRATION_GOAL_ML = 2300
    const val LACTATION_HYDRATION_GOAL_ML = 2700
    const val HYDRATION_INTERVAL_MINUTES = 120
    const val ACTIVITY_INTERVAL_MINUTES = 240
    const val QUIET_HOURS_START = 22
    const val QUIET_HOURS_END = 7
    const val KICK_TARGET = 10
    const val KICK_WINDOW_MINUTES = 120
    const val CONTRACTION_INTERVAL_MINUTES = 5
    const val CONTRACTION_DURATION_MINUTES = 1
    const val CONTRACTION_WINDOW_MINUTES = 60
    const val EPDS_MODERATE = 10
    const val EPDS_HIGH = 13
    const val PRETERM_WEEKS = 37
    const val POST_TERM_DAYS = 294
}
```

## 16. Sources to cite in-app

- ACOG (American College of Obstetricians and Gynecologists) patient FAQs
- CDC (US Centers for Disease Control and Prevention), including "Learn the Signs. Act Early."
- FDA/EPA fish advice
- NHS (UK) pregnancy and baby guidance
- WHO growth standards
- IOM/NAM gestational weight gain guidance
- EPDS original validation literature and licensing

> Do not copy source text verbatim. Write original summaries, cite the source, and have a
> clinician review. Record `sourceRef` and `reviewedOn` on every content row.

## 17. Review checklist before release

- [ ] Every constant has a source and a reviewer.
- [ ] Every red-flag path was tested on a real device.
- [ ] EPDS wording and scoring match the licensed instrument.
- [ ] Crisis numbers are correct for every supported region.
- [ ] No screen uses diagnostic language.
- [ ] No screen shames the user.
- [ ] Uncertainty is visible wherever data is incomplete.
- [ ] A clinician has signed off on the content version.
