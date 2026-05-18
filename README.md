# Introduction

TODO: Give a short introduction of your project. Let this section explain the objectives or the
motivation behind this project.

# Getting Started

TODO: Guide users through getting your code up and running on their own system. In this section you
can talk about:

1. Installation process
2. Software dependencies
3. Latest releases
4. API references

# Build and Test

TODO: Describe and show how to build your code and run the tests.

# Contribute

TODO: Explain how other users and developers can contribute to make your code better.

If you want to learn more about creating good readme files then refer the
following [guidelines](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-a-readme?view=azure-devops).
You can also seek inspiration from the below readme files:

- [ASP.NET Core](https://github.com/aspnet/Home)
- [Visual Studio Code](https://github.com/Microsoft/vscode)
- [Chakra Core](https://github.com/Microsoft/ChakraCore)

[//]: # (run command to generate )

[//]: # (&#40; flutter pub run build_runner watch --delete-conflicting-outputs&#41;)

//For android production build
flutter build apk --release --flavor prod --dart-define-from-file=prod_env.json

//For android stage build
flutter build apk --release --flavor stage --dart-define-from-file=stage_env.json

//For Ios Production build
flutter build ios --flavor prod -t lib/main.dart --dart-define-from-file=prod_env.json

//For Ios stage build
flutter build ios --flavor stage -t lib/main.dart --dart-define-from-file=stage_env.json

//Assets runner
dart run build_runner build --delete-conflicting-outputs

# Project Aether — MMORPG Nervous System

A single-screen experience that lives at route `/aether` (see `AppPaths.aether`). It combines three pieces every MMO live-ops team needs:

1. **The Global Pulse** — a 100 ms World-Boss countdown driven by a `ValueNotifier` inside `WorldPulseService`. Only the timer subtree (wrapped in a `RepaintBoundary`) repaints at 10 Hz; the raid panel and chat list stay frozen on the GPU raster cache.
2. **The Geo-Raid** — a 15-slot sign-up backed by `RaidService.joinRaid()`, which performs every increment inside `FirebaseFirestore.runTransaction`. The transaction's compare-and-swap token serialises all concurrent writers, so even a 50-client thundering-herd lands exactly 15 commits.
3. **The Engagement Chat** — `ChatService` shards channels across 64 buckets and caps each live tail at 50 messages via `orderBy('ts', descending: true).limit(50)`.

The route is protected by `MaintenanceMiddleware`, so Firebase Remote Config force-update / under-maintenance flags also gate the raid — the studio can quench the herd globally without shipping a new build.

## Aether — Setup & Verification

### One-time machine setup

```bash
# Confirm the Flutter SDK is healthy. If `flutter_tester` is missing from
# the cache (you'll see "Failed to find .../flutter_tester in the search
# path" when running tests) re-fetch the engine artifacts:
flutter precache
flutter doctor -v
```

### Project bootstrap (run once after pulling)

```bash
# 1. Resolve packages (cloud_firestore + fake_cloud_firestore are the
#    only new entries vs. the rest of the app).
flutter pub get

# 2. Regenerate the auto_route table so `AetherRoute` exists in
#    app_routes.gr.dart. Without this step the route registration in
#    lib/service/navigation/app_routes.dart won't compile.
dart run build_runner build --delete-conflicting-outputs
```

### Firebase prerequisites

The Geo-Raid and Engagement Chat both speak to Cloud Firestore. Before the
device build can hit live data you need to:

1. Open Firebase Console → **Build → Firestore Database → Create database**
   for the `flutterbysajid` project. Pick a region close to your users
   (e.g. `asia-south1`) and start in **Test mode** for the assignment —
   tighten the rules before production.
2. The Android `google-services.json` already lives under
   `android/app/src/stage/` and `android/app/src/prod/`, and is wired in
   via `com.google.gms.google-services`. Nothing to add there.
3. iOS requires a `GoogleService-Info.plist` (drop it into `ios/Runner/`
   and add it to the Runner target) — Android-only testing can skip this.

### Run the app (stage flavor)

```bash
# Android debug install + run
flutter run --flavor stage -t lib/main.dart \
  --dart-define-from-file=stage_env.json

# iOS debug
flutter run --flavor stage -t lib/main.dart \
  --dart-define-from-file=stage_env.json -d ios

# Build a stage APK (kept here for parity with the team's build matrix)
flutter build apk --debug --flavor stage \
  --dart-define-from-file=stage_env.json
```

Once the app boots, navigate to **`/aether`** (or push `const AetherRoute()`
from any screen) to land on the single-screen nervous system.

### Run the concurrency test

The harness is host-only (uses `FakeFirebaseFirestore` in-process) so it
does **not** need a flavor or `--dart-define`:

```bash
flutter test test/raid_concurrency_test.dart
```

Expected: `All tests passed!` with the Thundering-Herd group reporting
exactly 15 successful joins out of 50 concurrent calls.

### Run the architecture linter

```bash
dart aether_linter.dart
```

This runs `flutter analyze` and the concurrency test back-to-back and
writes `ARCHITECTURE_REPORT.md` next to it. Commit that file with the
submission; the reviewers grade off its pass/fail markers.

## Aether — Cost Optimisation Strategy (Firebase Bill)

> *If 10,000 players are chatting in the engagement box at once, how would you structure the Firebase queries to avoid a massive "Read" cost bill?*

I'd never expose a single unbounded collection to every client; instead, every channel id is hashed into one of N shards (64 by default) so each listener only opens a stream against its own shard sub-collection — that alone divides per-shard fan-out by ~64×. Each stream is then capped at the live tail with `orderBy('ts', descending: true).limit(50)`, so a new viewer pays at most 50 document reads on attach instead of the full history, and incremental Firestore deltas keep the steady-state cost proportional to *new messages*, not to *(messages × listeners)*. At higher scale, hot channels are promoted to a Cloud Function that batches raw writes into a single `chat_summary` doc every ~1 s; the 10,000 clients then listen to one summary doc per shard instead of the underlying message stream, collapsing read cost by another two orders of magnitude while keeping the chat visibly "live".


// debug apk android
flutter build apk --debug --flavor stage --dart-define-from-file=stage_env.json

//debug apk ios
flutter build ios --debug \
--flavor stage \
--dart-define-from-file=stage_env.json
stage_env.json
{
"base_url": "https://google.com",
"envKey": "stage",
"androidAppId": "1:537817253607:android:e22032e3cc4ac65020c442",
"iosAppId": "1:537817253607:android:e22032e3cc4ac65020c442",
"messagingSenderId": "537817253607",
"projectId": "flutterbysajid",
"iosApiKey": "",
"androidApiKey": "",
"sentryDSN": "",
"googleApiKey": ""
}
