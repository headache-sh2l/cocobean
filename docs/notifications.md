# Notifications and Reminders

> Status: 🟡 Proposed. Reminders are a core promise. A reminder that silently stops is
> worse than no reminder, because the user trusts it.

## 1. Requirements

- Reminders must survive: reboot, app update, timezone change, DST, clock change,
  Doze, and normal battery optimization.
- Reminders must respect quiet hours and the user's notification permission.
- Reminders must be **gentle**: no streak shaming, no "you failed" copy.
- Every reminder must deep-link to the relevant screen.
- The user must be able to see **why** a reminder did not fire and how to fix it.

## 2. Reminder types

| Type | Default schedule | Channel | Notes |
|---|---|---|---|
| Hydration | Every 2h, 08:00–20:00 | Hydration | Pauses during quiet hours. |
| Activity / walking | Every 4h, 08:00–20:00 | Activity | "Gentle movement" copy. |
| Supplement | Daily at user time | Supplements | One per supplement. |
| Appointment | 1 day + 2h before | Appointments | Exact time matters. |
| Fertile window | 1 day before window | Wellbeing | Planning only. |
| Test reminder | User-set date | Wellbeing | Two-week wait. |
| Milestone | Weekly / age window | Milestones | Postpartum. |
| Vaccine | 1 week + 1 day before due | Milestones | Region-aware. |
| Wellbeing check-in | Weekly | Wellbeing | EPDS / mood. |
| Custom | User-defined | Custom | |

## 3. Permission model

| Permission | When | If denied |
|---|---|---|
| `POST_NOTIFICATIONS` (API 33+) | After the user creates their first reminder, with a pre-permission explainer | In-app banner: "Reminders are off. Turn them on in Settings." Deep link to app notification settings. Never nag more than once per session. |
| `SCHEDULE_EXACT_ALARM` | Only if the user enables "exact time" for an appointment | Fall back to inexact; explain the tradeoff. |
| `USE_EXACT_ALARM` | **Do not use** unless the app qualifies for the Play policy exception | — |
| `READ_CALENDAR` / `WRITE_CALENDAR` | Only when the user taps "Add to calendar" | Manual entry still works. |
| `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | Optional, after a reminder fails or via Settings | Explain; never block the app. |
| `POST_NOTIFICATIONS` re-request | Only after a user action | — |

**Pre-permission explainer** (shown before the system dialog):

> "Cocobean can send gentle reminders for hydration, supplements, and appointments.
> They stay on your device. You can turn them off anytime."

## 4. Scheduling engine

### 4.1. Two mechanisms, one interface

```kotlin
interface ReminderScheduler {
    suspend fun schedule(reminder: Reminder)
    suspend fun cancel(reminderId: String)
    suspend fun rescheduleAll()
    suspend fun nextTrigger(reminderId: String): Instant?
}
```

- **WorkManager** for interval, daily, weekly, and deferrable reminders.
  - `PeriodicWorkRequest` with a unique name `reminder:{id}`.
  - `ExistingPeriodicWorkPolicy.UPDATE`.
  - Minimum interval 15 minutes (WorkManager limit).
- **AlarmManager** for exact, user-visible, time-critical reminders (appointments).
  - `setExactAndAllowWhileIdle` only if `canScheduleExactAlarms()`.
  - Otherwise `setAndAllowWhileIdle` (inexact) and tell the user.
  - `AlarmManager` does not survive force-stop on all OEMs; WorkManager is the fallback.

### 4.2. Why not only AlarmManager

- Android 12+ restricts exact alarms.
- Android 13+ requires notification permission.
- Doze defers inexact alarms.
- OEM battery savers kill background work.
- WorkManager persists across reboot and is the recommended API for deferrable work.

### 4.3. Why not only WorkManager

- Periodic work has a 15-minute minimum and is inexact by design.
- Appointment reminders need a specific time.
- Use both, behind one interface.

## 5. Scheduling rules

1. **Anchor to local wall clock.** A daily 08:00 reminder is 08:00 in the user's current
   timezone, not a fixed UTC instant.
2. **Quiet hours.** Default 22:00–07:00. Reminders that would fire inside quiet hours are
   moved to the next allowed time, not dropped.
3. **Coalescing.** If multiple reminders would fire within 15 minutes, group them into one
   notification with a summary.
4. **Idempotency.** Unique work names; scheduling twice must not duplicate.
5. **Cancellation.** Disabling a reminder cancels its work and any pending alarm.
6. **Journey scoping.** Ending a pregnancy journey cancels all its reminders in the same
   transaction.
7. **Snooze.** 15 minutes, 1 hour, or "tomorrow morning" (next allowed time).
8. **Completion.** "Mark done" records the log and cancels the current occurrence, not the
   series.
9. **Max notifications.** No more than 3 per day per journey by default; the user can
   change this.
10. **No back-to-back.** Minimum 30 minutes between notifications of the same type.

## 6. Rescheduling triggers

| Trigger | Action |
|---|---|
| `BOOT_COMPLETED` | Reschedule all enabled reminders. |
| `ACTION_TIME_CHANGED` | Recompute next triggers. |
| `ACTION_TIMEZONE_CHANGED` | Recompute wall-clock triggers. |
| App update | Reschedule after migration. |
| Permission granted/revoked | Re-evaluate; show status. |
| Reminder created/edited/deleted | Schedule/cancel that reminder. |
| Journey started/ended | Schedule/cancel journey reminders. |
| User changes quiet hours | Recompute. |
| Device date moved backward | Do not fire missed reminders in a burst; skip to next future occurrence. |

## 7. Notification design

### 7.1. Channels

| Channel | Importance | Sound | Use |
|---|---|---|---|
| Hydration | Default | Optional | Gentle hydration |
| Activity | Default | Optional | Movement |
| Supplements | Default | Yes | Daily supplement |
| Appointments | High | Yes | Time-critical |
| Wellbeing | Default | No | Check-ins, EPDS |
| Milestones | Default | No | Postpartum |
| Custom | Default | User choice | |

- Channel names and descriptions are user-facing and localized.
- Once created, channel importance cannot be changed by the app; link to system settings.

### 7.2. Content

- Title: short, specific ("Time for your prenatal vitamin").
- Body: one sentence, no guilt ("A glass of water when you can.").
- Tap: deep link to the relevant screen.
- Actions: "Done", "Snooze", "Open".
- No emoji unless the user opts in.
- No medical claims in notifications.

### 7.3. Grouping

- Group by journey and type.
- Summary notification: "3 reminders for today".
- Do not stack more than 5 individual notifications.

## 8. Failure modes and UX

| Failure | Detection | UX |
|---|---|---|
| Notification permission denied | `NotificationManagerCompat.areNotificationsEnabled()` | Banner in Reminders screen + Settings row. |
| Channel disabled | `getNotificationChannel().importance == NONE` | Row: "This channel is off in system settings." |
| Exact alarm denied | `canScheduleExactAlarms()` | Use inexact; explain. |
| Battery optimization | `isIgnoringBatteryOptimizations()` | Optional guidance card. |
| OEM force-stop | Cannot detect reliably | "If reminders stop, open Cocobean once after restarting your phone." |
| Reboot | `BOOT_COMPLETED` | Reschedule silently. |
| Timezone change | Broadcast | Reschedule silently. |
| DST gap | Compute next valid local time | Fire at the next valid time. |
| DST overlap | Compute once | Do not double-fire. |
| Storage full | Write failure | In-app error; do not crash. |

## 9. Testing

- Unit: next-trigger calculation across DST, timezones, quiet hours, month boundaries.
- WorkManager: `WorkManagerTestInitHelper` + `TestDriver`.
- AlarmManager: Robolectric shadow, or instrumented test.
- Manual matrix:
  - Pixel (stock Android), Samsung, Xiaomi, Huawei, OnePlus.
  - Android 14, 15, 16, 17.
  - Reboot, force-stop, battery saver, timezone change, manual clock change.
  - Notification permission denied, channel disabled, DND on.
- Acceptance: a reminder set for 10 minutes from now fires within ±2 minutes on a stock
  device, and within ±15 minutes on a battery-optimized OEM device.

## 10. Open questions

- Do we need exact alarms at all in MVP? (Recommend: only for appointments, with
  inexact fallback.)
- Do we request battery optimization exemption? (Recommend: explain, don't force.)
- Do we support "critical" reminders that bypass DND? (Recommend: no, except a
  user-initiated emergency call shortcut.)
