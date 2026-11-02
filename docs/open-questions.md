# Open Questions and Decision Log

> Status: 🔴 Several items block implementation. Each has a recommendation; if you agree
> with all recommendations, reply "go with the recommendations" and I will mark them
> decided and update the affected docs.

## How to use this

- **Blocking** means code written before a decision will likely be rewritten.
- **Non-blocking** can be decided during the phase that needs it.
- When a decision is made, change `Status` to ✅, add the date, and update the affected
  doc in the same change.

## Summary of recommendations

| ID | Question | Recommendation | Blocking |
|---|---|---|---|
| Q-01 | MVP scope | Pregnancy-first (Phases 0–5) | 🔴 |
| Q-02 | Launch region | US first, region-aware architecture | 🔴 |
| Q-03 | Clinical reviewer | Required before release; start sourcing now | 🔴 |
| Q-04 | Content licensing | Original summaries + citations; no copied text | 🔴 |
| Q-05 | Multiple journeys | Yes, from day one | 🔴 |
| Q-06 | Loss support | Yes, in MVP | 🔴 |
| Q-07 | Package / app ID | `app.cocobean` (or your domain) | 🔴 |
| Q-08 | DB encryption | FBE + app lock + encrypted export; defer SQLCipher | 🟡 |
| Q-09 | Monetization | Free, no ads, no IAP in MVP | 🟡 |
| Q-10 | minSdk | Keep 34 | 🟡 |
| Q-11 | Localization | English only at launch; externalize strings | 🟡 |
| Q-12 | Calendar integration | One-way insert only | 🟡 |
| Q-13 | App lock default | Off, opt-in | 🟡 |
| Q-14 | Exact alarms | Appointments only, with fallback | 🟡 |
| Q-15 | Export encryption | Encrypted by default | 🟡 |
| Q-16 | Partner mode | Defer | ⏸ |
| Q-17 | Widgets / Wear | Defer | ⏸ |
| Q-18 | Analytics | None, ever | ✅ |
| Q-19 | AI features | None in MVP | ⏸ |
| Q-20 | Regulatory posture | Wellness tool, not a medical device | 🔴 |

---

## Q-01 — What is the MVP scope?

**Options**

- A. All three stages at launch.
- B. Pregnancy-first (recommended).
- C. Planning-first.
- D. Postpartum-first.

**Recommendation: B.** Pregnancy exercises dates, content, reminders, and safety; it is
the most differentiated; it keeps the content review burden manageable. Planning and
postpartum follow in V1.1/V1.2, but the **data model supports all three from day one**.

**Impact:** roadmap, content volume, release date.

---

## Q-02 — Which region(s) do we support at launch?

**Options**

- A. US only (CDC/ACOG/FDA content, 911, oz/lb).
- B. UK only (NHS, 999, metric).
- C. US + UK.
- D. Global with generic content.

**Recommendation: A**, with `Profile.region` and region-tagged content so B/C are additive.
Vaccine schedules, crisis numbers, food guidance, and emergency numbers are region-specific;
shipping the wrong region's guidance is a safety issue.

**Impact:** content, units, crisis resources, vaccine schedule, store listing.

---

## Q-03 — Do we have a clinical reviewer?

**Options**

- A. Named clinician (OB/GYN, midwife, or pediatrician) on the team.
- B. Paid per-review consultant.
- C. No reviewer; ship general public guidance with citations.

**Recommendation: A or B.** C is not acceptable for a health app that gives food-safety
and mental-health guidance. Budget for an initial review plus quarterly updates.

**Impact:** release gate, content workflow, legal risk.

---

## Q-04 — How do we source content?

**Options**

- A. Original summaries from public sources, cited (recommended).
- B. License a commercial food-safety database.
- C. Copy public guidance verbatim.

**Recommendation: A.** C has licensing and accuracy problems. B is expensive and may not
fit the offline model. A keeps control and attribution.

**Impact:** content effort, legal review, differentiation.

---

## Q-05 — Do we support multiple journeys?

**Options**

- A. Single stage at a time (current docs).
- B. Multiple journeys with a primary (recommended).

**Recommendation: B.** A parent can be postpartum and pregnant again; a user can plan
after a loss. A single-stage model forces data deletion or a rewrite.

**Impact:** domain model, navigation, notifications, settings.

---

## Q-06 — Do we support pregnancy loss at launch?

**Options**

- A. Yes, with a sensitive flow (recommended).
- B. Defer; only live-birth paths.

**Recommendation: A.** Loss is common and the app will be used by people it happens to.
Shipping without it means cheerful notifications after a loss — a serious harm.

**Impact:** state machine, copy, notifications, content, testing.

---

## Q-07 — What is the package / application ID?

**Options**

- A. `app.cocobean`
- B. `com.cocobean.app`
- C. Your own domain, reversed.

**Recommendation: C if you own a domain; otherwise A.** Do this **before** the first
release. Changing it later breaks installs, exports, and store listings.

**Impact:** build config, deep links, export metadata.

---

## Q-08 — How much encryption?

**Options**

- A. Android FBE + app lock + encrypted export (recommended).
- B. Add SQLCipher for the Room DB.
- C. FBE only.

**Recommendation: A.** B adds complexity and key-management risk for a threat (rooted
device) that is largely out of scope. Revisit if the threat model changes.

**Impact:** dependencies, performance, export, support.

---

## Q-09 — Monetization?

**Options**

- A. Free, no ads, no IAP (recommended for MVP).
- B. One-time purchase.
- C. Subscription.
- D. Ads.

**Recommendation: A** for MVP. D contradicts the privacy promise. B is compatible with
the brand later.

**Impact:** store setup, feature gating, privacy policy.

---

## Q-10 — Keep `minSdk 34`?

**Options**

- A. Keep 34 (recommended).
- B. Lower to 26–30 for reach.

**Recommendation: A** for MVP. It simplifies `java.time`, notification permission
behavior, and testing. Revisit after launch if analytics (store-level) show demand.

**Impact:** reach, test matrix, API usage.

---

## Q-11 — Localization at launch?

**Options**

- A. English only, strings externalized (recommended).
- B. English + one language.
- C. Many languages.

**Recommendation: A.** Medical translation requires review; shipping unreviewed
translations is risky. Externalize everything now so B is cheap later.

**Impact:** content, layout, testing.

---

## Q-12 — How deep is calendar integration?

**Options**

- A. One-way insert into the system calendar (recommended).
- B. Two-way sync.
- C. In-app only.

**Recommendation: A.** Two-way sync is a sync engine with conflict resolution — a large
project. One-way insert covers the real need.

**Impact:** permissions, testing, support.

---

## Q-13 — Is app lock on by default?

**Options**

- A. Off, opt-in (recommended).
- B. On by default.

**Recommendation: A.** On-by-default adds friction for a user checking a symptom at 3 a.m.
Offer it prominently in onboarding/settings.

**Impact:** UX, support, security posture.

---

## Q-14 — Do we use exact alarms?

**Options**

- A. Appointments only, with inexact fallback (recommended).
- B. All reminders exact.
- C. No exact alarms.

**Recommendation: A.** B risks Play policy and battery complaints. C makes appointment
reminders unreliable.

**Impact:** permissions, scheduler, store review.

---

## Q-15 — Is export encrypted by default?

**Options**

- A. Yes, password-protected (recommended).
- B. Plain JSON by default, encryption optional.

**Recommendation: A.** The export contains health data. Plain JSON is a footgun. Offer a
plain option with a clear warning for users who need interoperability.

**Impact:** export UX, support, data portability.

---

## Q-16 — Partner / caregiver mode?

**Options**

- A. Defer (recommended).
- B. Read-only partner view.
- C. Full multi-user.

**Recommendation: A.** It multiplies the data model and privacy surface. Revisit after
launch.

**Impact:** domain model, auth, privacy.

---

## Q-17 — Widgets or Wear OS?

**Options**

- A. Defer (recommended).
- B. Home-screen widget for today's log.
- C. Wear app for quick logging.

**Recommendation: A.** Nice, but not core. A widget is a good V1.1 candidate.

**Impact:** scope, testing.

---

## Q-18 — Any analytics?

**Status: ✅ Decided — none.**

No in-app analytics, no crash SDK, no ads, no `INTERNET` permission. Use Play Console
vitals and moderated usability tests.

---

## Q-19 — Any AI features?

**Options**

- A. None in MVP (recommended).
- B. On-device summarization of logs.
- C. Cloud chat assistant.

**Recommendation: A.** C breaks the privacy promise. B is a later experiment with clear
guardrails and no medical advice.

**Impact:** scope, privacy, safety.

---

## Q-20 — Regulatory posture?

**Options**

- A. General wellness / record-keeping tool, not a medical device (recommended).
- B. Pursue a regulated medical-device classification.

**Recommendation: A.** Keep the app educational and record-keeping. Avoid diagnostic
claims, dosing, and treatment recommendations. Consult a lawyer for your jurisdiction.

**Impact:** copy, features, store listing, liability.

---

## Additional questions worth answering before Phase 2

| ID | Question | Recommendation |
|---|---|---|
| Q-21 | Do we show weight gain ranges? | Optional, BMI-based, education only; never a target. |
| Q-22 | Do we include a kick counter and contraction timer in MVP? | Yes, they are small and high-value; include red-flag copy. |
| Q-23 | Do we include a hospital bag checklist? | Yes, low cost, high perceived value. |
| Q-24 | Do we include emergency contacts? | Yes, profile-level, no permissions needed. |
| Q-25 | Do we support twins at launch? | Data model yes; content minimal. |
| Q-26 | Do we support adoption/surrogacy at launch? | Data model yes; content later. |
| Q-27 | Do we show a "content reviewed on" date? | Yes, in About and on detail screens. |
| Q-28 | Do we allow custom reminders? | Yes, one-off and daily. |
| Q-29 | Do we support dark mode at launch? | Yes, it is a core night-feed need. |
| Q-30 | Do we support tablets/foldables? | Responsive layouts, no separate app. |
