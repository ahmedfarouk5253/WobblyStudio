# CLAUDE.md: Arrow Voyage

You are building **Arrow Voyage**, an arrow-escape puzzle game for Android by Wobbly Studio, a solo indie studio. The design docs in `docs/` are the spec. When code and docs disagree, the docs win. If a doc is wrong, update the doc in the same commit and say so.

## Identity
- GAME_NAME: `Arrow Voyage`
- PACKAGE_ID: `com.wobblystudio.arrowvoyage`
- Studio site (privacy policy and app-ads.txt are hosted here): `https://<your-github-username>.github.io/WobblyStudio/`. Ask the owner for the final domain before you set it.
- Support email: ahmedfarouk5253@gmail.com

## Stack (fixed, don't swap without asking)
- **Flutter (stable channel, Dart 3)**. Android first. Keep iOS compiling but don't ship it.
- No game engine. The board is drawn with `CustomPainter` and animated with a `Ticker`. Do not add Flame.
- State: `flutter_riverpod`. Navigation: `go_router`.
- Persistence: a JSON save file (`path_provider`) plus `shared_preferences` for settings. Cloud save goes through Play Games Saved Games.
- Services: `google_mobile_ads` (with UMP consent), `in_app_purchase`, `games_services`, `firebase_core`, `firebase_analytics`, `firebase_crashlytics`, `firebase_remote_config`, `flutter_local_notifications`, `audioplayers`.
- Full package list and versions are in `docs/09_TECH_ARCHITECTURE.md`. Pin versions in `pubspec.yaml`.

## Commands
```bash
flutter pub get
flutter analyze                       # must be clean (0 issues) before every commit
dart format --set-exit-if-changed .   # format
flutter test                          # all unit + widget tests must pass
dart run tool/level_gen.dart --all    # regenerate level packs into assets/levels/
dart run tool/validate_levels.dart    # solve + score every shipped level; fails on any unsolvable level
dart run tool/check_assets.dart       # compare assets/ and store/ to assets_manifest.json
dart run tool/optimize_assets.dart    # resize/convert generated PNGs to WebP
flutter build appbundle --release     # AAB for Play Console
```

## Non-negotiable game invariants
1. **The Monotonic Rule.** Nothing on the board can ever become *more* blocked as the game goes on. An arrow leaving, a lock opening or ice thawing only ever frees things. That's why a greedy solver is exact and a level can never become unsolvable mid-play. Any new mechanic that breaks this rule is rejected. See `docs/03_MECHANICS.md`.
2. **A blocked tap changes nothing on the board.** The arrow bumps and slides back, and only the heart counter changes.
3. **Every shipped level is validated** by `tool/validate_levels.dart` (solvable, matches the difficulty band for its slot). CI fails otherwise.
4. **The engine is pure Dart** (`lib/engine/`): no Flutter imports, deterministic, fully unit-tested. Rendering and input live in `lib/game/`.
5. **Seeded randomness only.** Use `SplitMix64`/`Xoroshiro` from `lib/engine/rng.dart`, never `dart:math Random()` without a seed, so daily puzzles are identical on every device.

## Non-negotiable player-respect rules (these are our marketing)
- Never show an ad during a level, in the middle of a tutorial, or after a failed level.
- Never put a banner over the board. v1 has **no banners at all**.
- Rewarded ads are always optional and always pay out. If the ad fails to load or show, grant the reward anyway, at most 3 times a day.
- Remove Ads removes **all** interstitials, permanently, restores on reinstall, and hides its own button once owned.
- Never move a free feature behind a paywall in an update (grid dots, aim line, dark mode, colors, Zen mode).
- No energy or lives system across levels. Hearts exist only inside a single level.
- No fake "beats 87% of players" stats, no fake timers on offers, no "last chance" fail-offer popups.

## Code conventions
- Folders: `lib/engine` (pure logic), `lib/game` (board widget, painter, input, animations), `lib/features/<feature>` (screens + controllers), `lib/services` (ads, iap, analytics, save, audio, haptics), `lib/core` (theme, l10n, utils), `tool/` (CLI scripts).
- Every user-visible string goes in `lib/l10n/app_en.arb`. No hard-coded UI strings.
- Every tunable number lives in `lib/core/config/game_config.dart`, mirrored by Remote Config keys (`docs/06_MONETIZATION.md`, `docs/12_ANALYTICS_LIVEOPS.md`).
- Every analytics event name is a constant in `lib/services/analytics/events.dart`, matching `docs/12`.
- Asset paths are constants in `lib/core/assets.g.dart`, **generated** from `assets_manifest.json` by `tool/gen_asset_constants.dart`. Never type an asset path by hand.
- If an asset file is missing at runtime, show the generated placeholder (`PlaceholderImage`). Never crash.
- Prefer small files (< 300 lines). Write a unit test for every engine function and every economy rule.
- Commit after each milestone step with a clear message. Never commit keys or keystores. `android/key.properties` and `*.jks` stay in `.gitignore`.

## Working style for this repo
- Follow `docs/16_BUILD_PLAN.md` milestone by milestone. At the end of each milestone, run analyze + tests and give a short report: what works, what's stubbed, how to try it.
- When something in the docs is ambiguous, pick the option that is kinder to the player and simpler to build, and note it in `docs/DECISIONS.md` (create it if missing).
- Test IDs: use Google's official **test** ad unit IDs and the IAP test flow until the owner provides real ones. Real IDs go in `lib/core/config/ids.dart`, read from `--dart-define` at build time.
