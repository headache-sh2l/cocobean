# Product Brief

> Status: 🟡 Proposed — confirm the MVP cut and target region in [open-questions.md](open-questions.md).

## 1. One-liner

**Cocobean is a private, offline pregnancy and postpartum companion that turns an
overwhelming amount of health information into a calm, week-by-week plan — with every
record staying on the user's device.**

## 2. The problem

The process of becoming a parent can be overwhelming. There are multiple sources of information, appointments strewn everywhere, and most website, blogs, tiktoks or apps tell you what to do. There's no one non-subscription place for most parents to create their own to-do list, add their birthing plan, track their own nutrition and mental health while also visualizing how their baby is growing, and what to look out for and track milestones.



## 3. Who it is for

Primary audience:

- People trying to conceive, including those with irregular cycles.
- Newly pregnant people from first trimester through postpartum.
- Postpartum parents, especially during night feeds and the first year.

Explicitly included:

- Same-sex couples and non-gestational parents.
- Adoptive parents, surrogates, and intended parents.
- People who have experienced pregnancy loss.
- People with limited or no internet access.

We do **not** assume:

- that the user is a mother, is partnered, or is in a heterosexual relationship,
- that the pregnancy is wanted or will end in a live birth,
- that the user has a regular cycle, a known LMP, or a smartphone with Google Play services.

## 4. Product principles

1. **Local by default, not as a setting.** No account, no sync, no analytics, no ads.
   The app should not request `INTERNET` permission in the MVP.
2. **Calm over clinical.** Warm language, soft color, no red unless there is genuine risk.
3. **Never shame.** No streaks that punish, no "you missed a day" guilt, no weight
   judgments, no "good/bad mother" framing.
4. **Uncertainty is a feature.** If the app does not know, it says so. Predictions are
   ranges, not promises.
5. **Safety escalates, it does not diagnose.** The app can say "contact your provider
   now." It cannot say "you have preeclampsia."
6. **The user owns the data.** Export must be complete, readable, and restorable without
   our servers.


## 5. What Cocobean is / is not

| Is | Is not |
|---|---|
| A personal tracking and education tool | A medical device or diagnostic tool |
| A local record the user can export | A cloud backup service |
| A gentle reminder system | A replacement for a clinician |
| A stage-aware companion | A social network or forum |
| A source of reviewed general guidance | Personalized medical advice |

## 6. Scope

### 6.1. MVP (recommended cut)

**Pregnancy-first vertical slice** — the smallest shippable app. It exercises the hardest
problems (dates and content) and is the most differentiated.

**In the MVP (5 features):**

1. **Onboarding** — local profile, stage selection (pregnancy only; planning/postpartum
   shown as "coming soon"), due-date setup, disclaimer.
2. **Pregnancy journey** — EDD from LMP/ultrasound/manual/IVF, gestational age, trimester,
   week-by-week dashboard, fruit comparison, countdown.
3. **Kick counter** — session count with red-flag copy.
4. **Appointments** — local list + optional system calendar insert.
5. **Hospital bag checklist.**

Everything else is **post-MVP**. See [mvp.md](mvp.md) for the full in/out list.

Entity mapping:

| Entity | Domain model § | MVP use |
|---|---|---|
| Profile | [3.1](domain-model.md#31-profile-singleton-id--1) | Local profile, disclaimer |
| Journey | [3.2](domain-model.md#32-journey) | Pregnancy journey container |
| PregnancyDetails | [3.3](domain-model.md#33-pregnancydetails-11-with-a-pregnancy-journey) | EDD, GA inputs |
| Appointment | [3.13](domain-model.md#313-appointment) | Local appointment list + calendar insert |
| HospitalBagItem | [3.22](domain-model.md#322-emergencycontact--hospitalbagitem) | Hospital bag checklist |
| FruitSize / WeekContent | [3.24](domain-model.md#324-fruitsize--weekcontent-content) | Fruit comparison + week-by-week content |
| KickSession | [3.26](domain-model.md#326-kicksession--contractionlog) | Kick counter |

**Moved to post-MVP:** daily log + mood check-in, food entry, reminders, export/import,
settings, safety/emergency contacts, loss/termination flow, contraction timer, planning,
postpartum, food safety, and mental health.

### 6.2. Roadmap 

**Post-MVP (moved out of the MVP):** daily log + mood check-in, food entry, reminders,
export/import, settings, safety/emergency contacts, loss/termination flow, contraction
timer.

#### V1.1
- Food Safety : searchable bundled database, risk levels, condition modifiers, meal log.

 #### V1.2
- List of questions to ask during appointment

#### V1.3
- To-do list for self to help keep track of things

#### V1.4
Postpartum: milestones, feeding/diaper/sleep logs, growth measurements,
  vaccination schedule, EPDS check-ins, recovery content.
- Transition pregnancy → postpartum on birth.
- Mental health check-ins for new mothers

#### V1.5
- Transition planning → pregnancy on positive test.
#### V1.6
 Planning to conceive: cycle log, predictions, fertile window, two-week wait.



### 6.5
.** Later / explicitly deferred**

- Growth percentiles with WHO/CDC LMS tables (log first, percentiles later).
- Partner/caregiver mode.
- Home-screen widgets and Wear OS.
- Multi-profile on one device.
- Localization beyond English.
- Any AI assistant or chat.
- Cloud sync, accounts, web app.
- Social/community features.
- Monetization experiments.

### 6.6. Non-goals

- Diagnosis, treatment, or dosage recommendations.
- Replacing prenatal care.
- Storing data on a server.
- Selling or sharing data.
- Tracking a pregnancy without the user's explicit setup.

## 7. Success metrics (privacy-preserving)

We cannot use in-app analytics without breaking principle 1. Use:

- **Play Console vitals** (crash-free rate, ANR rate) — OS-level, no app telemetry.
- **Moderated usability tests** with 8–12 target users per stage.
- **Task completion** in tests: onboard, set up pregnancy, view the dashboard, use the
  kick counter, add an appointment, and check the hospital bag.

Targets for MVP:

- Crash-free sessions ≥ 99.5%.
- [Post-MVP] Export → wipe → import round-trip preserves 100% of records.
- Zero network calls (verifiable by absence of `INTERNET` permission).

## 8. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Medical content is wrong or outdated | Harm, trust loss, legal | Named clinician review, dated sources, versioned content, review cadence |
| App is perceived as medical advice | Regulatory/legal | Persistent disclaimer, "talk to your provider" copy, no diagnostic language |
| Reminders silently fail | Core promise broken | WorkManager + boot/timezone rescheduling, in-app status, device matrix testing |
| Scope is too large for one release | Never ships | Pregnancy-first MVP, explicit cut line |
| "Offline" claim undermined by backup | Privacy breach | `allowBackup=false`, encrypted export, documented threat model |
| Loss/infertility handled insensitively | Real user harm | Sensitive-mode flows, reviewed copy, no confetti on negative tests |
| OEM battery killers | Missed reminders | Guidance screen, optional exemption request, test on Xiaomi/Samsung/Huawei |
| Accessibility debt | Excludes users | A11y acceptance criteria per screen, not a later pass |
| Play policy (health, exact alarms) | Rejection | Avoid `USE_EXACT_ALARM`; justify any exact alarm; health disclaimer |

## 9. Personas and jobs

### P1 — "Trying, and tired of guessing" (TTC, irregular cycles)
- Job: "Help me understand my cycle without pretending you know exactly when I ovulate."
- Needs: log periods, see ranges, flag irregularity, not be sold certainty.
- Failure: app shows a single "ovulation day" as fact.

### P2 — "First pregnancy, information overload"
- Job: "Tell me what matters this week and what I can ignore."
- Needs: GA, fruit size, appointments, food safety, reminders.
- Failure: 40 articles and a red badge.

### P3 — "Second-time parent, pregnant with a toddler"
- Job: "Let me track this pregnancy without losing my first child's records."
- Needs: multiple journeys, journey switcher, per-child data.
- Failure: single-stage model overwrites the postpartum record.

### P4 — "Postpartum at 3 a.m."
- Job: "Let me log a feed in 5 seconds and tell me if this is normal."
- Needs: one-handed logging, dark mode, large targets, no judgment.
- Failure: 6-tap flow, bright white screen.

### P5 — "I lost the pregnancy"
- Job: "Let me stop the reminders without deleting the memory."
- Needs: pause, memorial, sensitive copy, no "congratulations."
- Failure: auto-advance to postpartum, cheerful notifications.

### P6 — "I need help now"
- Job: "Tell me whether to call someone, and how."
- Needs: red-flag guidance, emergency contacts, crisis resources, ensure "call 911 for an emergency" and "talk to your provider, this is not medical advice".
- Failure: buried in a settings screen.

## 10. Constraints

- Android only, `minSdk 34`, `targetSdk 37` (current template).
- Kotlin + Jetpack Compose + Material 3.
- No `INTERNET` permission in MVP.
- All content bundled; updates ship with app releases.
- Single developer/small team; architecture must stay simple.
- Medical content must be reviewed by a qualified clinician before release.

## 11. Medical and legal posture

- Cocobean is a **general wellness and record-keeping tool**, not a medical device.
- Every clinical screen carries a short, non-dismissible-forever disclaimer, and the
  onboarding requires explicit acknowledgement.
- The app never states a diagnosis, never recommends a dose, and never tells a user to
  ignore a clinician.
- Emergency guidance (post-MVP) is region-aware and always includes "call your local
  emergency number."
- A privacy policy and terms of use are required before store release, even with no
  server, because the app handles health data.
