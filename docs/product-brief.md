# Product Brief

> Status: 🟡 Proposed — confirm the MVP cut and target region in [open-questions.md](open-questions.md).

## 1. One-liner

**Cocobean is a private, offline pregnancy and postpartum companion that turns an
overwhelming amount of health information into a calm, week-by-week plan — with every
record staying on the user's device.**

## 2. The problem

The process of becoming a parent can be overwhelming. There are multiple sources of information, appointments strewn everywhere, and most website, blogs, tiktoks or apps tell you what to do. There's no one non-subscription place for most parents to create their own to-do list, add their birthing plan, track their own nutrition and mental health while also visualizing how their baby is growing, and what to look out for.

Expectant and new parents are asked to track a lot at once: cycle dates, appointments,
supplements, hydration, food safety, fetal movement, newborn feeding, milestones,
vaccinations, and their own mental health. The information is fragmented across apps,
websites, and paper handouts. Most apps are cloud-first, ad-supported, or designed around
a clinical workflow rather than a tired human at 3 a.m.

Cocobean's bet: **calm, local, and complete beats feature-bloated and connected.**

## 3. Who it is for

Primary audience:

- People trying to conceive, including those with irregular cycles.
- Newly pregnant people from first trimester through postpartum.
- Postpartum parents, especially during night feeds and the first year.

Explicitly included:

- Same-sex couples and non-gestational parents.
- Adoptive parents, surrogates, and intended parents.
- People who have experienced pregnancy loss.
- People with disabilities, low vision, or limited literacy.
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
7. **Accessible by construction.** 48dp targets, 200% font scaling, TalkBack, reduced
   motion, color-independent status.
8. **Content is reviewed, sourced, and versioned.** No unreviewed medical claims.

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

**Pregnancy-first vertical slice**, because it exercises the hardest problems (dates,
content, reminders, safety) and is the most differentiated.

- Onboarding: local profile, stage selection, due-date setup, disclaimer.
- Pregnancy journey: EDD from LMP/ultrasound/manual/IVF, gestational age, trimester,
  week-by-week dashboard, fruit comparison, countdown.
- Daily log: symptoms, mood, weight, notes.
- Reminders: hydration, activity, supplements, appointments; quiet hours; permission
  handling; reboot/timezone resilience.
- Food safety: searchable bundled database, risk levels, condition modifiers, meal log.
- Appointments: local list + optional system calendar insert.
- Export/import: complete JSON, optional password encryption.
- Settings: units, theme, app lock, data deletion, about/disclaimer.

### 6.2. V1.1

- Planning to conceive: cycle log, predictions, fertile window, two-week wait.
- Transition planning → pregnancy on positive test.

### 6.3. V1.2

- Postpartum: milestones, feeding/diaper/sleep logs, growth measurements,
  vaccination schedule, EPDS check-ins, recovery content.
- Transition pregnancy → postpartum on birth.

### 6.4. Later / explicitly deferred

- Growth percentiles with WHO/CDC LMS tables (log first, percentiles later).
- Partner/caregiver mode.
- Home-screen widgets and Wear OS.
- Multi-profile on one device.
- Localization beyond English.
- Any AI assistant or chat.
- Cloud sync, accounts, web app.
- Social/community features.
- Monetization experiments.

### 6.5. Non-goals

- Diagnosis, treatment, or dosage recommendations.
- Replacing prenatal care.
- Storing data on a server.
- Selling or sharing data.
- Tracking a pregnancy without the user's explicit setup.

## 7. Success metrics (privacy-preserving)

We cannot use in-app analytics without breaking principle 1. Use:

- **Play Console vitals** (crash-free rate, ANR rate) — OS-level, no app telemetry.
- **Moderated usability tests** with 8–12 target users per stage.
- **Task completion** in tests: set up pregnancy, log a meal, find a food, schedule a
  reminder, export and restore data.
- **Accessibility audit** pass (TalkBack, 200% font, contrast).
- **Clinical content review** sign-off with dated sources.
- **Notification reliability** measured in a controlled device matrix, not in production.

Targets for MVP:

- Crash-free sessions ≥ 99.5%.
- Export → wipe → import round-trip preserves 100% of records.
- All critical flows usable with TalkBack and 200% font.
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
- Needs: red-flag guidance, emergency contacts, crisis resources.
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
- Emergency guidance is region-aware and always includes "call your local emergency
  number."
- A privacy policy and terms of use are required before store release, even with no
  server, because the app handles health data.
