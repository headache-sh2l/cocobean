# Design System

> Status: 🟡 Proposed. The current `Theme.kt` still uses the Compose template purple and
> dynamic color. Replace it before building screens.

## 1. Brand attributes

| Attribute | Means | Does not mean |
|---|---|---|
| Calm | Soft surfaces, generous spacing, no urgency unless real | Boring, low contrast |
| Warm | Peach accents, rounded shapes, human copy | Infantilizing, cutesy |
| Competent | Clear hierarchy, precise dates, sourced content | Clinical, cold |
| Private | No account prompts, no sharing nags | Hidden, suspicious |
| Inclusive | Neutral language, diverse illustrations | Generic, erased |

## 2. Color

### 2.1. Core palette

| Token | Light | Dark | Use |
|---|---|---|---|
| `primary` | `#4E7959` | `#88B092` | Primary actions, active nav |
| `onPrimary` | `#FFFFFF` | `#0F2416` | |
| `primaryContainer` | `#D6E8DA` | `#2E4A36` | Selected cards |
| `onPrimaryContainer` | `#12281A` | `#D6E8DA` | |
| `secondary` | `#B5675F` | `#E5A49D` | Warm accents |
| `onSecondary` | `#FFFFFF` | `#3A1512` | |
| `secondaryContainer` | `#F6DDD9` | `#5A2A26` | |
| `tertiary` | `#6B7A8F` | `#AFC0D6` | Informational |
| `background` | `#F9F8F6` | `#121412` | |
| `onBackground` | `#2C3E50` | `#E6E8E6` | |
| `surface` | `#FFFFFF` | `#1A1D1A` | Cards |
| `onSurface` | `#2C3E50` | `#E6E8E6` | |
| `surfaceVariant` | `#EFEDE8` | `#2A2E2A` | Inputs, chips |
| `onSurfaceVariant` | `#5A6B7A` | `#B8C2BC` | Secondary text |
| `outline` | `#C9C4BB` | `#4A514A` | Borders |
| `error` | `#B3261E` | `#F2B8B5` | Destructive only |
| `onError` | `#FFFFFF` | `#601410` | |

### 2.2. Semantic risk colors

| Token | Light | Dark | Use |
|---|---|---|---|
| `riskSafe` | `#3F7D52` | `#8FD3A3` | Safe food |
| `riskModerate` | `#B26A00` | `#F0B45A` | Moderate |
| `riskAvoid` | `#A63D2F` | `#F09A8C` | Avoid |
| `riskUnknown` | `#6B7280` | `#A7AFB8` | Unknown |

Risk is **never** conveyed by color alone. Pair with an icon and a text label.

### 2.3. Rules

- Body text contrast ≥ 4.5:1; large text ≥ 3:1.
- Do not use pure black (`#000000`) or pure white (`#FFFFFF`) as a full-screen background
  in dark mode.
- Disable Material You dynamic color: it breaks brand and can produce low-contrast
  combinations.
- Red is reserved for genuine risk and destructive actions. Do not use it for "overdue"
  or "missed."

### 2.4. Brand vs accessible variants

`app_layout.md` lists `#D98880` (peach) and `#F39C12` (amber). Those are the **brand**
colors and are used for containers, illustrations, and large shapes. They do **not** meet
4.5:1 with white text, so text-bearing buttons and badges use the darker accessible
variants above (`#B5675F`, `#B26A00`). Do not "fix" this by shrinking the text.

## 3. Typography

Use the system font (Roboto) for MVP; a custom font is a later brand decision.

| Style | Size / line height | Weight | Use |
|---|---|---|---|
| `displayLarge` | 40 / 48 | 600 | Fruit week number |
| `headlineLarge` | 32 / 40 | 600 | Screen titles |
| `headlineMedium` | 28 / 36 | 600 | Section headers |
| `titleLarge` | 22 / 28 | 600 | Card titles |
| `titleMedium` | 18 / 24 | 500 | List headers |
| `bodyLarge` | 16 / 24 | 400 | Body |
| `bodyMedium` | 14 / 20 | 400 | Secondary |
| `labelLarge` | 14 / 20 | 500 | Buttons |
| `labelSmall` | 12 / 16 | 500 | Captions |

- Support font scaling up to 200% without clipping.
- Line length ≤ 70 characters on tablets.
- Never use all-caps for long text.

## 4. Spacing, shape, elevation

- Spacing scale: 4, 8, 12, 16, 24, 32, 48 dp.
- Screen padding: 20 dp horizontal (16 dp on small screens).
- Card padding: 16–20 dp.
- Corner radius: 12 dp (small), 16 dp (cards), 24 dp (sheets), 999 dp (pills).
- Elevation: 0 for flat cards with a 1 dp outline; 1–2 dp for raised cards. Avoid heavy
  shadows.
- Minimum touch target: 48 × 48 dp, even if the visual is smaller.

## 5. Components

| Component | Notes |
|---|---|
| `CocobeanScaffold` | Edge-to-edge, insets, bottom nav, top bar. |
| `PrimaryButton` | Filled, 48 dp min height, loading state. |
| `SecondaryButton` | Outlined. |
| `TextButton` | Low emphasis. |
| `CocobeanCard` | Surface, 16 dp radius, optional outline. |
| `StatCard` | Big number + label + optional trend. |
| `RiskBadge` | Icon + label + color; accessible. |
| `ProgressRing` | Hydration, goals; text alternative. |
| `WeekTimeline` | Horizontal week selector. |
| `CalendarMonth` | Period, fertile window, appointments; legend. |
| `LogRow` | Time + icon + value + edit. |
| `EmptyState` | Illustration + one sentence + one action. |
| `ErrorState` | What happened + retry. |
| `LoadingState` | Skeleton, not a spinner for content. |
| `InsightCard` | One tip, dismissible, sourced. |
| `ConfirmDialog` | Destructive actions; typed confirmation for wipe. |
| `BottomSheet` | Quick log, filters. |
| `SegmentedControl` | Units, tabs. |
| `Chip` | Filters, symptoms. |
| `Stepper` | Onboarding. |
| `DisclaimerBanner` | Persistent, dismissible per session. |

## 6. Motion

- Duration: 150–250 ms for micro-interactions; 300 ms for screen transitions.
- Easing: `FastOutSlowIn` / `easeOut`.
- Respect `Settings.Global.ANIMATOR_DURATION_SCALE` and Compose's reduced-motion signal.
- No confetti, no bouncing, no celebratory animation for medical events.
- A gentle fade/scale for success; no flashing.

## 7. Accessibility

- Every interactive element has a content description or visible label.
- Icons that are decorative are `null` in semantics.
- Charts expose a summary and a table alternative.
- Focus order follows visual order.
- Errors are announced and linked to fields.
- Color is never the only signal.
- Support TalkBack, Switch Access, and 200% font.
- Test with Accessibility Scanner on every screen.
- Haptic feedback is optional and never the only confirmation.

## 8. Copy and tone

### 8.1. Principles

- **Warm, plain, specific.** "A glass of water when you can" > "Hydration goal incomplete."
- **No shame.** Never "you missed," "you failed," "you should have."
- **No false certainty.** "Most people" / "generally" / "your provider may."
- **No toxic positivity.** "It's okay to not be okay" > "Stay positive!"
- **Short.** One idea per sentence.

### 8.2. Examples

| Instead of | Write |
|---|---|
| "You missed your hydration goal." | "A little water when you can." |
| "Your baby is behind." | "Every baby develops at their own pace. Here's what to watch for." |
| "You are at risk of preeclampsia." | "These symptoms can be serious. Call your provider now." |
| "Congratulations on your pregnancy!" (after loss) | Never shown in a loss journey. |
| "Failed to load." | "We couldn't open this. Try again." |
| "Invalid date." | "That date is in the future. Check the year?" |

### 8.3. Sensitive modes

- Loss: muted palette, no fruit celebration, no "your baby" if the user prefers, memorial
  option, pause reminders.
- Infertility: no "just relax," no "it will happen," no baby imagery in planning.
- NICU: no "you should be home by now."
- Postpartum: no weight-loss framing, no "bounce back."

## 9. Illustration and iconography

- Soft, rounded, diverse skin tones and family structures.
- Fruit illustrations: consistent style, one per week, alt text with size.
- Icons: 24 dp, rounded, 2 dp stroke, consistent set (Material Symbols or custom).
- No gendered baby colors as the only differentiator.
- No realistic medical gore.

## 10. Dark mode

- True dark surfaces (`#121412`), not pure black.
- Reduce saturation of accents; keep contrast.
- Images: provide dark variants or a subtle scrim.
- Test at night: no white flash on launch (set the window background).

## 11. Edge-to-edge and insets

- `enableEdgeToEdge()` in `MainActivity`.
- Use `WindowInsets.safeDrawing` for content; `safeContent` for scrollable lists.
- Bottom nav respects the navigation bar inset.
- IME insets for text fields.
- No content under the status bar without a scrim.

## 12. Design QA checklist

- [ ] Light and dark mode pass contrast.
- [ ] 200% font scaling: no clipping.
- [ ] TalkBack: every action reachable and labeled.
- [ ] Color-blind simulation: risk levels still distinguishable.
- [ ] 48 dp targets.
- [ ] Empty, loading, error, and offline states designed.
- [ ] RTL layout checked.
- [ ] No red for non-risk states.
- [ ] Copy reviewed for shame and certainty.
- [ ] Sensitive-mode variants exist.
