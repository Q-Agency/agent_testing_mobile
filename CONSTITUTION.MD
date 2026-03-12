# Project Constitution — Mobile

## Metadata

| Field | Value |
|-------|-------|
| Project Name | AutoMarket — Car Selling Platform |
| Project Type | GREENFIELD |
| Status | VALID |
| Author | AI Agency OS / BA Agent |
| Date | 2026-03-12 |
| Stack | Mobile |
| Label | Mobile |
---

## Overview

AutoMarket mobile app provides the native iOS and Android experience for the car selling platform. Built with Flutter for cross-platform delivery from a single codebase. Core features include vehicle browsing and search with map integration, camera-based listing creation, push notifications for offers and messages, and offline-capable saved searches. Communicates with the same FastAPI backend as the web frontend.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Dart 3.5+ |
| Framework | Flutter |
| State Management | flutter_bloc (Bloc / Cubit) |
| Navigation | go_router |
| HTTP Client | dio |
| Local Storage | drift (SQLite) |
| Secure Storage | flutter_secure_storage |
| Auth | Supabase Flutter SDK (supabase_flutter) |
| Image Handling | cached_network_image + image_picker |
| Maps | google_maps_flutter |
| Push Notifications | Firebase Cloud Messaging |
| Testing | flutter_test (unit) + bloc_test + integration_test (e2e) |
| Linting | flutter_lints + custom analysis_options.yaml |

---

## Environments

| Environment | Purpose | Config Source |
|-------------|---------|--------------|
| development | Local dev, mock or local backend | `--dart-define-from-file=config/dev.json` |
| staging | Pre-production, real backend staging | `--dart-define-from-file=config/staging.json` |
| production | Live app store builds | `--dart-define-from-file=config/prod.json` |

Env switching via `--dart-define-from-file` at build time. No secrets in config files — secrets injected by CI. Flavor-based build variants for app icons and bundle IDs per environment.

---

## Coding Standards

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files | snake_case | `vehicle_detail_page.dart` |
| Classes | PascalCase | `VehicleDetailPage` |
| Functions / methods | camelCase | `fetchVehicleById()` |
| Variables | camelCase | `listingCount` |
| Constants | camelCase with `k` prefix | `kDefaultPadding` |
| Private members | `_` prefix | `_isLoading` |
| Blocs | PascalCase + suffix | `VehicleSearchBloc`, `VehicleSearchState`, `VehicleSearchEvent` |
| Repositories | PascalCase + `Repository` | `VehicleRepository` |
| Models / Entities | PascalCase | `Vehicle`, `Offer` |
| Enums | PascalCase, camelCase values | `ListingStatus.active` |
| Extensions | PascalCase + `X` suffix | `StringX`, `BuildContextX` |

### File Structure

```
lib/
├── main.dart              # Entry point, DI init
├── app.dart               # MaterialApp.router config
├── core/
│   ├── config/            # App config, theme, colors, typography
│   ├── constants/         # API constants, domain constants
│   ├── di/                # get_it service locator setup
│   ├── error/             # Custom exceptions, Failure sealed classes
│   ├── network/           # dio client, auth interceptor, connectivity
│   └── utils/             # Formatters, validators, extensions
├── features/
│   └── [feature]/
│       ├── data/          # Datasources (remote + local), models, repo implementations
│       ├── domain/        # Entities, repository contracts, use cases
│       └── presentation/  # Bloc/Cubit, pages, widgets
├── shared/
│   └── widgets/           # Cross-feature widgets (loading, error, empty, pagination)
test/
├── unit/
├── widget/
├── integration/
├── fixtures/              # JSON fixtures for API mocking
├── mocks/                 # Mocktail mock classes
```

### Patterns to Use

- **Clean Architecture (feature-first)** — each feature has `data/`, `domain/`, `presentation/`. Domain depends on nothing external. Data implements domain contracts.
- **BLoC for state management** — `Bloc` for complex event-driven flows (search, offer negotiation), `Cubit` for simpler state (detail loading, form state).
- **Repository pattern** — abstract interfaces in `domain/`, implementations in `data/`. All data access through repositories.
- **Use cases** — single-purpose classes in `domain/usecases/` encapsulating one business action. Injectable and testable.
- **Dependency injection via get_it** — all repositories, data sources, blocs registered in `injection.dart`.
- **Sealed classes for states and failures** — Dart 3 sealed classes with exhaustive pattern matching in UI.
- **Freezed for data models** — `freezed` + `json_serializable` in `data/models/`. Domain entities are plain Dart classes.
- **go_router for navigation** — declarative routing with deep link support from day one.
- **Offline-first for key flows** — search results cached in drift. Show cached data immediately, refresh in background.

### Patterns to Avoid

- **`setState()` for anything beyond trivial UI** — use Cubit for form state, Bloc for complex flows. `setState` only for animations or ephemeral toggles.
- **Business logic in widgets** — widgets render state and dispatch events. Logic lives in Blocs, Cubits, or use cases.
- **God Blocs** — one Bloc per feature concern. Don't bundle search, detail, and creation in one Bloc.
- **Direct dio calls in Blocs** — all network calls through repositories → data sources.
- **Hardcoded user-facing strings** — use `l10n` (intl package) for all text, even if launching single-language.
- **`print()` statements** — use `logger` package. No `print()` in committed code.
- **Magic numbers in layouts** — extract spacing, sizes, breakpoints to theme constants.
- **Ignoring lifecycle** — handle app lifecycle (background, resume) for token refresh, WebSocket reconnection.

---

## Git Conventions

| Convention | Value |
|------------|-------|
| Branch Naming | `feature/AM-{ticket}-short-desc`, `fix/AM-{ticket}-short-desc`, `chore/`, `hotfix/` |
| Commit Format | Conventional Commits: `feat(vehicles): add gallery swipe`, `fix(offers): handle expired offer state` |
| PR Size Guideline | ≤400 lines changed. Larger PRs must be split or justified. |
| Default Branch | `main` |
| Merge Strategy | Squash merge to `main` |

---

## Error Handling

- dio interceptor catches HTTP errors and maps to typed `ApiException(statusCode, errorCode, message)`.
- Repositories catch `ApiException` and return `Either<Failure, T>` (or sealed Result type) to use cases.
- Blocs emit error states with typed `Failure` — UI pattern-matches to show appropriate message.
- Network offline detected via connectivity listener. Offline state shown globally, degraded gracefully to cached data.
- 401 triggers silent token refresh via auth interceptor. If refresh fails, emit auth expired state → redirect to login.
- 429 shows user-friendly "try again shortly" with countdown.
- Unhandled errors caught by Flutter's error zone and reported to crash reporting.
- Never show raw backend error messages to users. Map error codes to localized strings.

---

## Logging & Observability

| Convention | Value |
|------------|-------|
| Log Format | `logger` package with structured output |
| Log Levels | `debug` (local only, stripped in release), `info` (key user actions), `warning` (retries, degraded state), `error` (crashes, API failures) |
| What Gets Logged | API failures with endpoint/status, critical user actions, state transitions in key Blocs, app lifecycle events |
| What Never Gets Logged | Auth tokens, passwords, PII, full response bodies |
| Crash Reporting | Firebase Crashlytics (or Sentry) — all unhandled exceptions and fatal errors |

Request IDs from backend responses propagated into logs for cross-layer correlation.

---

## Testing Standards

| Requirement | Value |
|-------------|-------|
| Minimum Coverage | 75% (domain + data layers), 50% (presentation) |
| Unit Tests Required | YES |
| Widget Tests Required | YES (reusable components) |
| Integration Tests Required | YES (critical user journeys) |

### Test File Conventions

- Test files mirror source: `lib/features/vehicles/domain/usecases/search_vehicles.dart` → `test/unit/features/vehicles/domain/usecases/search_vehicles_test.dart`.
- `mocktail` for mocking (not mockito).
- `bloc_test` for Bloc/Cubit tests.
- Widget tests use `pumpWidget()` with all required providers.
- Integration tests via `integration_test` package on real device/emulator.
- JSON fixtures in `test/fixtures/` for consistent API mocking.
- Naming: `group("VehicleSearchBloc")` → `test("emits [Loading, Loaded] when search succeeds")`.

---

## API Standards

| Convention | Value |
|------------|-------|
| HTTP Client | dio instance with base URL from build config |
| Auth | JWT in `Authorization` header via auth interceptor, auto-refresh |
| Error Handling | `ApiException` caught in repositories, mapped to `Failure` sealed class |
| Response Models | `freezed` + `json_serializable` in `data/models/`, mapped to domain entities |
| Offline Cache | drift for key data, TTL-based in-memory cache via dio interceptor |

Data source → repository → use case → Bloc call chain. Specific endpoints and response shapes defined by SA Agent in `design.md`.

---

## Security Standards

- JWT access tokens in `flutter_secure_storage` (Keychain iOS, EncryptedSharedPreferences Android). Never plain SharedPreferences.
- Refresh handled automatically by auth interceptor on 401.
- No secrets in Dart code or committed config files. Use `--dart-define` for build-time values.
- Certificate pinning for API domain in production builds.
- ProGuard (Android) enabled for release builds.
- No logging of tokens, passwords, or PII, even in debug.
- Image uploads validated client-side (type, size, EXIF stripped) before backend validation.
- Deep links validated against expected patterns before navigation.
- Dependency audit via `dart pub outdated` in CI.

---

## Accessibility Standards

- Minimum touch target size: 48×48 dp for all interactive elements.
- All interactive widgets have `Semantics` labels for screen readers.
- All images have `semanticLabel` (or excluded via `excludeFromSemantics: true` for decorative images).
- Sufficient color contrast (WCAG 2.1 AA: 4.5:1 normal text, 3:1 large text).
- Focus traversal order is logical. Custom focus management for modals and bottom sheets.
- TalkBack (Android) and VoiceOver (iOS) tested for critical flows.
- Dynamic text scaling supported — no fixed font sizes that break at large accessibility sizes.
- No information conveyed by color alone (use icons or text alongside color indicators).

---

## Documentation Standards

| What | Requirement |
|------|-------------|
| Public classes / methods | DartDoc comment with description, params, return value. |
| Blocs | Doc comment explaining what the Bloc manages, key events, and state transitions. |
| Use cases | Doc comment describing the business action and when it's called. |
| Widgets | Doc comment for reusable widgets explaining purpose and required parameters. |
| Architecture decisions | ADR in `docs/adr/` for non-trivial choices. |

---

## Quality Gates

Before merge, code must pass:

- [ ] `dart analyze` — zero issues (errors, warnings, infos)
- [ ] `dart format --set-exit-if-changed` — code formatted
- [ ] All unit tests pass
- [ ] All widget tests pass
- [ ] Coverage ≥ 75% on domain and data layers
- [ ] No `// ignore:` suppressions without justification
- [ ] No `dynamic` types without justification
- [ ] No `print()` statements
- [ ] Android release build compiles (`flutter build appbundle`)
- [ ] iOS release build compiles (`flutter build ipa`)
- [ ] All user-facing strings use l10n
- [ ] PR description includes context, screenshots/recordings, and testing notes

---

## Open Questions

- [ ] Minimum supported OS versions (iOS 16+? Android API 26+?) — affects available APIs and test matrix
- [ ] Offline sync strategy depth (cache-only vs. full optimistic writes with conflict resolution)
- [ ] Background processing approach for large image uploads (WorkManager Android, BGTaskScheduler iOS)
