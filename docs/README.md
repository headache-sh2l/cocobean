# Cocobean Documentation

> **Status:** Pre-implementation design. No production code should be written until the
> blocking items in [open-questions.md](open-questions.md) are resolved.
>
> **Last updated:** 2026-11-01

## Why this folder exists

`app_layout.md` and `userflow.md` describe *what the screens look like* and *the happy-path
journey*. They do not describe:

- what data exists and how it relates,
- how dates, cycles, gestational age, and risk are calculated,
- what happens when the user's life does not match the happy path,
- how reminders survive reboots, timezones, and battery savers,
- how "100% on-device" is actually enforced,
- how medical content is sourced, reviewed, and versioned.

Those are the things that cause rewrites. This folder fills those gaps.

## Source of truth

When documents disagree, the higher document wins until the lower one is updated:

1. [product-brief.md](product-brief.md) — what we are building, for whom, and what we refuse to build
2. [domain-model.md](domain-model.md) — entities, relationships, invariants, stage state machine
3. [clinical-rules.md](clinical-rules.md) — every calculation and safety rule
4. [architecture.md](architecture.md) — how the system is structured
5. [app_layout.md](app_layout.md) / [userflow.md](userflow.md) — screen-level UX
6. [open-questions.md](open-questions.md) — unresolved decisions

## Document map

| Doc | Answers | Status |
|---|---|---|
| [product-brief.md](product-brief.md) | Why does this exist? Who is it for? What is in/out of scope? | 🟡 Proposed |
| [domain-model.md](domain-model.md) | What are the nouns? How do they relate? What must always be true? | 🟡 Proposed |
| [domain-model-entity-relation.md](domain-model-entity-relation.md) | Visual ER diagrams (split by domain) for the data model | 🟡 Proposed |
| [clinical-rules.md](clinical-rules.md) | How do we calculate EDD, gestational age, fertility, risk, EPDS? | 🟡 Needs clinician review |
| [content-strategy.md](content-strategy.md) | Where do food, fruit, milestone, and vaccine data come from? | 🟡 Proposed |
| [architecture.md](architecture.md) | Modules, layers, DI, navigation, state, errors, build config | 🟡 Proposed |
| [notifications.md](notifications.md) | How reminders are scheduled and what breaks them | 🟡 Proposed |
| [data-and-privacy.md](data-and-privacy.md) | Storage, encryption, export/import, backup, deletion, threat model | 🟡 Proposed |
| [edge-cases.md](edge-cases.md) | What happens when life is messy? | 🟡 Proposed |
| [design-system.md](design-system.md) | Tokens, components, accessibility, tone, motion | 🟡 Proposed |
| [testing.md](testing.md) | What must be tested before release? | 🟡 Proposed |
| [roadmap.md](roadmap.md) | In what order do we build it? | 🟡 Proposed |
| [open-questions.md](open-questions.md) | What still needs a human decision? | 🔴 Blocking items |

**Legend:** ✅ decided · 🟡 proposed, needs confirmation · 🔴 open/blocking · ⏸ deferred

## Reading order for a new contributor

1. `product-brief.md` (10 min)
2. `domain-model.md` (20 min)
3. `clinical-rules.md` (20 min)
4. `architecture.md` (15 min)
5. The screen docs (`app_layout.md`, `userflow.md`)
6. `edge-cases.md` before writing any feature

## How decisions are made

- Every non-trivial decision gets an entry in `open-questions.md` with an ID (`Q-01`).
- A decision is not "done" until the affected doc is updated in the same change.
- Clinical rules require a named reviewer and a review date before release.
- If a rule cannot be sourced, it does not ship.

## Glossary

| Term | Meaning |
|---|---|
| **Journey** | One tracked life stage instance (a pregnancy, a postpartum period, a TTC period). A user can have more than one over time, and occasionally two at once. |
| **Primary journey** | The journey that currently drives the app shell and bottom navigation. |
| **EDD** | Estimated Due Date. |
| **LMP** | Last Menstrual Period (first day). |
| **GA** | Gestational Age, expressed as weeks+days. |
| **TTC** | Trying To Conceive. |
| **EPDS** | Edinburgh Postnatal Depression Scale. |
| **Content DB** | Bundled, versioned, read-only reference data (foods, fruit sizes, milestones, vaccines). |
| **User DB** | The Room database containing the user's private records. |
