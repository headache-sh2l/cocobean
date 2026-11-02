# cocobean

A private, offline-first pregnancy and postpartum companion for Android.

## Status

**Pre-implementation.** The design is being completed before feature code is written.
Start with [docs/README.md](docs/README.md).

## Documentation

| Doc | Purpose |
|---|---|
| [docs/README.md](docs/README.md) | Index, source-of-truth order, glossary |
| [docs/product-brief.md](docs/product-brief.md) | Vision, audience, scope, MVP, risks |
| [docs/domain-model.md](docs/domain-model.md) | Entities, relationships, invariants, stage state machine |
| [docs/clinical-rules.md](docs/clinical-rules.md) | EDD, gestational age, fertility, food safety, EPDS, red flags |
| [docs/content-strategy.md](docs/content-strategy.md) | Sourcing, licensing, schemas, review workflow |
| [docs/architecture.md](docs/architecture.md) | Modules, layers, DI, navigation, build config |
| [docs/notifications.md](docs/notifications.md) | Scheduling, permissions, failure modes |
| [docs/data-and-privacy.md](docs/data-and-privacy.md) | Storage, encryption, export/import, deletion |
| [docs/edge-cases.md](docs/edge-cases.md) | Unhappy paths and sensitive scenarios |
| [docs/design-system.md](docs/design-system.md) | Tokens, components, accessibility, tone |
| [docs/testing.md](docs/testing.md) | Test strategy, device matrix, release gates |
| [docs/roadmap.md](docs/roadmap.md) | Phased delivery plan |
| [docs/open-questions.md](docs/open-questions.md) | Decisions needed before coding |
| [docs/app_layout.md](docs/app_layout.md) | Screen-level layout blueprint |
| [docs/userflow.md](docs/userflow.md) | User journey flows |

## Tech stack

- Kotlin, Jetpack Compose, Material 3
- Room, DataStore, WorkManager, Hilt
- `minSdk 34`, `targetSdk 37`
- No `INTERNET` permission; all data stays on device

## Build

```bash
./gradlew assembleDebug
./gradlew test
```
