# Edge Cases and Unhappy Paths

> Status: 🟡 Proposed. This is the document to read before writing a feature. If a
> scenario here has no defined behavior, the feature is not done.

## 1. Onboarding and identity

| # | Scenario | Expected behavior |
|---|---|---|
| O-01 | User closes the app mid-onboarding | Resume at the last completed step; no partial profile is treated as complete. |
| O-02 | User skips their name | Allowed. Copy uses "you"; no placeholder name is invented. |
| O-03 | User does not know LMP or EDD | Offer "I'm not sure" → pregnancy journey is incomplete; dashboard shows setup card, not a fake week. |
| O-04 | User selects the wrong stage | Settings → "Change stage" with confirmation; data is preserved, not deleted. |
| O-05 | User is pregnant and has a toddler | Two journeys; journey switcher; primary journey drives the shell. |
| O-06 | User is planning and postpartum at the same time | Allowed; both journeys active; reminders are labeled by journey. |
| O-07 | User is a non-gestational parent | Copy avoids "mother"; profile has no gender assumption. |
| O-08 | User is a surrogate or adopting | Journey outcome supports `SURROGACY` / `ADOPTION`; no assumption of a live birth. |
| O-09 | User declines the medical disclaimer | Cannot proceed to the main app; can read the disclaimer and accept later. |
| O-10 | User changes device language | App follows the system locale; dates/units update; stored data unchanged. |
| O-11 | User changes units | Display converts; stored values remain metric. |
| O-12 | User has no Google Play services | App works; no Play-dependent features in core flows. |

## 2. Pregnancy dates

| # | Scenario | Expected behavior |
|---|---|---|
| P-01 | LMP is in the future | Validation error; do not create a journey with impossible dates. |
| P-02 | EDD is more than 300 days away | Warn; allow only with confirmation (data entry error is likely). |
| P-03 | EDD is in the past | Warn; offer "the pregnancy may have ended" flow; do not auto-advance. |
| P-04 | User enters both LMP and scan EDD | Show both; ask which to use; explain the difference. |
| P-05 | User changes EDD at 30 weeks | Append to history; recalculate GA; show a one-time banner; do not rewrite logs. |
| P-06 | User is past 42 weeks | Post-term content; "talk to your provider"; no alarmist copy. |
| P-07 | User's pregnancy ends in loss | Sensitive flow: pause reminders, no congratulation, memorial option, keep data. |
| P-08 | User reports a loss by mistake | Allow undo within the session; restore reminders only on explicit confirmation. |
| P-09 | User has twins | `fetusCount = 2`; twin-specific content; date math unchanged. |
| P-10 | User had IVF | EDD from transfer date + embryo age; show the formula in a tooltip. |
| P-11 | User travels across timezones | GA is date-based and stable; appointments show the original zone + local time. |
| P-12 | User manually changes the device clock | Detect a large jump; do not fire a burst of missed reminders; recompute next triggers. |
| P-13 | DST spring-forward at 02:30 | Reminder fires at the next valid local time (03:00). |
| P-14 | DST fall-back at 01:30 | Reminder fires once, not twice. |
| P-15 | User's due date is Feb 29 | Handle leap years; store `LocalDate`, not a month/day pair. |

## 3. Cycle and fertility

| # | Scenario | Expected behavior |
|---|---|---|
| C-01 | User has no cycle data | Default 28-day estimate, clearly labeled as an estimate. |
| C-02 | User has irregular cycles | Widen the fertile window; show "predictions are less reliable"; suggest ovulation tests. |
| C-03 | User has PCOS | Disable confident predictions; show education; no "you ovulated" claims. |
| C-04 | User is on hormonal contraception | Do not predict fertility; show a note. |
| C-05 | User logs a 60-day cycle | Keep it in history; exclude from the average; flag for provider discussion. |
| C-06 | User logs two period starts on the same day | Validation error; merge or reject. |
| C-07 | User logs a period that overlaps a previous one | Warn; allow correction. |
| C-08 | User gets a positive ovulation test | Mark the day; do not override the predicted window entirely. |
| C-09 | User's period is late | Do not congratulate; offer test info; do not auto-create a pregnancy. |
| C-10 | User reports a negative test | Supportive copy; no "try again" pressure. |
| C-11 | User transitions to pregnancy | Keep cycle history; pause cycle reminders; no data loss. |
| C-12 | User returns to planning after a loss | New journey; old data readable; no reset. |

## 4. Food safety

| # | Scenario | Expected behavior |
|---|---|---|
| F-01 | Food not in the database | `UNKNOWN`; "We don't have guidance for this yet — ask your provider." Never safe by default. |
| F-02 | User logs a high-risk food | Gentle card; do not block; do not shame; offer provider guidance. |
| F-03 | User has gestational diabetes | Condition modifier changes notes; glucose log available; no forbidden-food list. |
| F-04 | User has a food allergy | Hard warning; filter suggestions. |
| F-05 | User is vegetarian/vegan | Nutrition notes adapt; no meat-centric defaults. |
| F-06 | Content updates change a food's risk | Past entries keep their snapshot; new entries use the new risk. |
| F-07 | User searches with a typo | Fuzzy search + aliases; show "did you mean". |
| F-08 | User logs a custom food | Allowed; risk `UNKNOWN`; note explains. |
| F-09 | User logs alcohol | Non-judgmental note; no red alarm; offer support if repeated. |
| F-10 | User exceeds caffeine limit | Gentle awareness; no scolding; reset next day. |
| F-11 | User is in a different region | Food guidance is region-tagged; show the region and a disclaimer. |
| F-12 | User logs a meal offline | Works; no network dependency. |

## 5. Reminders and notifications

| # | Scenario | Expected behavior |
|---|---|---|
| N-01 | Notification permission denied | In-app banner + settings deep link; reminders still listed but marked "off". |
| N-02 | User disables a channel | Detect and explain; do not silently fail. |
| N-03 | Exact alarm permission denied | Fall back to inexact; explain. |
| N-04 | Device rebooted | Reschedule via `BOOT_COMPLETED`. |
| N-05 | App force-stopped by OEM | On next open, reschedule; show a one-time note. |
| N-06 | Battery saver kills work | Guidance card; optional exemption request. |
| N-07 | User changes timezone | Recompute wall-clock reminders. |
| N-08 | User changes quiet hours | Recompute; do not fire missed reminders. |
| N-09 | User snoozes | Reschedule one occurrence; keep the series. |
| N-10 | User marks done | Log the action; cancel the current occurrence. |
| N-11 | Two reminders at the same time | Coalesce into one notification. |
| N-12 | Journey ends | Cancel all its reminders immediately. |
| N-13 | User deletes a reminder | Cancel work and alarm; remove from list. |
| N-14 | Notification tapped after journey deleted | Deep link resolves to a safe "this journey was removed" screen. |
| N-15 | User is in DND | Respect DND; do not bypass except for user-initiated emergency calls. |
| N-16 | Lock screen shows health data | Hide sensitive content by default; user can enable previews. |

## 6. Data, export, import

| # | Scenario | Expected behavior |
|---|---|---|
| D-01 | Export while storage is full | Clear error; suggest freeing space; no partial file left behind. |
| D-02 | Export is interrupted | Write to a temp file, then atomically rename; no corrupt export. |
| D-03 | User forgets export password | Data is unrecoverable; warn clearly before export. |
| D-04 | Import a newer schema | Refuse with a clear message; suggest updating the app. |
| D-05 | Import a corrupt file | Validate checksum; refuse; no partial import. |
| D-06 | Import duplicates | Merge by ID; show conflicts; let the user choose. |
| D-07 | Import while a journey is active | Require confirmation; offer automatic backup first. |
| D-08 | Photo file missing | Placeholder; no crash; export notes the missing file. |
| D-09 | User deletes everything | Confirm with a typed phrase; offer export; return to onboarding. |
| D-10 | User restores an old export | Migrate forward; warn about missing newer data. |
| D-11 | Device storage is low | Warn before large operations; degrade gracefully. |
| D-12 | App updated with a DB migration | Run migration; if it fails, block and prompt export, never wipe silently. |

## 7. Postpartum and child data

| # | Scenario | Expected behavior |
|---|---|---|
| B-01 | Baby is premature | Adjusted age until 24 months; milestone windows shift; explain. |
| B-02 | Baby is in the NICU | Content acknowledges NICU; no "you should be home" copy. |
| B-03 | User is formula-feeding | No judgment; feeding logs support all methods. |
| B-04 | User is exclusively pumping | Support pumping logs. |
| B-05 | User has twins | Two child records; separate logs; no combined percentile. |
| B-06 | User has a second child | New child + postpartum journey; first child's data untouched. |
| B-07 | Baby's sex is unknown | Growth charts work without sex; prompt only when needed. |
| B-08 | User misses a milestone window | No red badge; "every baby is different"; offer red-flag info. |
| B-09 | User reports a concern | Surface "talk to your pediatrician" and red flags; do not diagnose. |
| B-10 | User stops logging for weeks | No streak shame; welcome back gently. |
| B-11 | User's child dies | Sensitive mode: pause reminders, memorial, no cheerful content. This must be designed, not improvised. |
| B-12 | User changes a child's birth date | Recalculate age and milestone windows; keep logs. |

## 8. Mental health

| # | Scenario | Expected behavior |
|---|---|---|
| M-01 | EPDS item 10 > 0 | Immediate crisis card; local emergency number; no dead end. |
| M-02 | EPDS score ≥ 13 | Encourage provider contact; not a diagnosis; offer resources. |
| M-03 | User dismisses the crisis card | Still reachable in ≤ 2 taps; do not hide it permanently. |
| M-04 | User is in a region with no configured crisis line | Show local emergency number + international directory. |
| M-05 | User takes EPDS repeatedly | Keep history; show trend gently; no "you're getting worse" framing. |
| M-06 | User writes a crisis note | Local only; surface resources; do not share. |

## 9. Accessibility and device

| # | Scenario | Expected behavior |
|---|---|---|
| A-01 | 200% font scaling | No clipped text; layouts scroll; buttons remain reachable. |
| A-02 | TalkBack | Every icon has a label; charts have summaries; focus order is logical. |
| A-03 | Color blindness | Risk levels use text + icon, not color alone. |
| A-04 | Reduced motion | Disable non-essential animations. |
| A-05 | Dark mode | All screens pass contrast; no pure black; images remain legible. |
| A-06 | Small screen / split screen | Layouts adapt; no fixed heights that clip. |
| A-07 | RTL locale | Mirrored layouts; icons that imply direction are mirrored. |
| A-08 | One-handed use at 3 a.m. | Primary actions in the bottom third; large targets. |
| A-09 | Screen reader on a chart | Provide a textual summary and a data table alternative. |
| A-10 | Haptic feedback | Optional; never the only signal. |

## 10. Security and privacy

| # | Scenario | Expected behavior |
|---|---|---|
| S-01 | App backgrounded | Lock after timeout if enabled. |
| S-02 | Biometric fails | Device credential fallback; no permanent lockout. |
| S-03 | Screenshot attempted | Blocked if enabled; otherwise allowed. |
| S-04 | Recents preview | Hide sensitive content if app lock is on. |
| S-05 | User shares an export | Warn that the file contains health data. |
| S-06 | User loses the device | No server to wipe; document that FBE + lock are the protection. |
| S-07 | Malicious app tries to read data | App-private storage blocks it; root is out of scope. |
| S-08 | Notification on lock screen | Hide content by default. |

## 11. Content and correctness

| # | Scenario | Expected behavior |
|---|---|---|
| X-01 | Content is outdated | Show "reviewed on" date; ship updates; never silently stale. |
| X-02 | Region-specific content is missing | Fall back to a general note + "ask your provider"; do not show another region's schedule as fact. |
| X-03 | Translation is missing | Fall back to English; mark the screen as partially translated. |
| X-04 | User reports an error | In-app "Report a content issue" (local draft; user chooses to share). |
| X-05 | Conflicting sources | Prefer the region's authority; document the choice; do not average. |

## 12. Recovery principles

When in doubt:

1. **Preserve data.** Never delete to resolve a conflict.
2. **Ask, don't assume.** Especially about loss, stage, and dates.
3. **Show the state.** "We couldn't schedule this reminder" beats silence.
4. **Offer a next step.** Every error has an action.
5. **Be gentle.** The user may be exhausted, grieving, or scared.
