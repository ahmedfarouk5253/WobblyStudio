# 09: Technical Architecture

## 1. Why Flutter, without an engine
- The whole game is 2D vector drawing on a grid. `CustomPainter` is ideal: crisp at any zoom, with no sprite atlases.
- Everything is plain Dart text files, which Claude Code edits reliably. There are no binary scene files like in Unity or Godot.
- First-party plugins exist for AdMob, Billing, Firebase and Play Games.
- Small APK, fast startup, 60 fps on low-end devices if the painting is cached properly.

## 2. Packages
Add with `flutter pub add <name>` (this picks the **latest stable** at project start), then commit `pubspec.lock`. Don't upgrade major versions mid-project without a reason.

| Package | Purpose |
|---|---|
| flutter_riverpod | State management |
| go_router | Navigation |
| path_provider, shared_preferences | Save file + settings |
| google_mobile_ads | AdMob + UMP consent (`ConsentInformation`, `ConsentForm`) |
| in_app_purchase | Google Play Billing |
| games_services | Play Games sign-in, achievements, saved games (cloud save) |
| firebase_core, firebase_analytics, firebase_crashlytics, firebase_remote_config | Analytics, crashes, tuning |
| flutter_local_notifications, timezone | Daily reminder |
| audioplayers | SFX (`AudioPool` for whooshes) + music |
| in_app_review | Native rating prompt |
| package_info_plus, device_info_plus | Version + device info for support emails |
| url_launcher | Privacy policy, email |
| collection, meta | Utilities |
| **dev:** flutter_lints (or very_good_analysis), flutter_launcher_icons, flutter_native_splash, test, mocktail, image (for the tools) | |

## 3. Project structure
```
lib/
  main.dart                    # bootstrap: Firebase, crashlytics zone, ProviderScope
  app.dart                     # MaterialApp.router, theme, l10n
  core/
    config/game_config.dart    # all tunables (defaults); RemoteConfig overrides
    config/ids.dart            # ad unit IDs, product IDs (from --dart-define)
    theme/                     # ThemeData, BoardTheme registry, palettes
    assets.g.dart              # GENERATED asset path constants
    widgets/                   # Buttons, sheets, PlaceholderImage, CoinCounter…
    utils/
  l10n/app_en.arb …            # strings
  engine/                      # PURE DART: no flutter imports
    model/ (cell.dart, direction.dart, arrow.dart, statics.dart, level.dart, board.dart)
    ray.dart                   # traceRay
    engine.dart                # GameState + tryFire + freeArrows
    solver.dart
    difficulty.dart
    rng.dart                   # SplitMix64, Xoroshiro128**
    level_codec.dart           # JSON <-> Level
    gen/ (generator.dart, daily_generator_v1.dart, endless_generator.dart, shapes.dart, mask_raster.dart, colors.dart)
  game/
    board_view.dart            # widget: camera + gesture handling + painters
    camera.dart                # fit/zoom/pan math, minimap
    input/hit_test.dart        # smart touch, ambiguity guard
    input/loupe.dart
    painters/ (board_painter.dart, arrow_painter.dart, statics_painter.dart, fx_painter.dart)
    anim/ (exit_anim.dart, bump_anim.dart, path_follower.dart)
    level_controller.dart      # Riverpod notifier: engine + hearts + hints + timers + events
  features/
    home/ level/ results/ daily/ atlas/ endless/ shop/ settings/ onboarding/
  services/
    save/ (save_service.dart, save_model.dart, cloud_save.dart, migrations.dart)
    ads/ (ads_service.dart, ad_policy.dart)        # ad_policy = pure, unit-tested rules from doc 06
    iap/ (iap_service.dart, entitlements.dart)
    analytics/ (analytics_service.dart, events.dart)
    remote_config_service.dart
    audio_service.dart, haptics_service.dart
    notifications_service.dart
    play_games_service.dart
tool/
  level_gen.dart, validate_levels.dart, level_preview.dart, curve_report.dart
  check_assets.dart, optimize_assets.dart, gen_asset_constants.dart, compose_screenshots.dart
test/
  engine/ (ray_test, engine_test, solver_test, generator_test, difficulty_test, codec_test, mechanics/*)
  services/ (ad_policy_test, economy_test, save_migration_test)
  game/ (hit_test_test, camera_test)
  widget/ (level_screen_test, results_test)
assets/
  levels/  images/  audio/  fonts/  icon/  branding/
store/      # store listing images (not bundled in the app)
assets_src/ # reference images (not bundled)
```

## 4. Rendering
- `BoardView` is a `StatefulWidget` with a `TickerProviderStateMixin`. It holds:
  - the `Camera` (scale and offset in board units)
  - the `RepaintBoundary` layers:
    1. background image
    2. board + dots + statics (cached `Picture`, rebuilt on theme or zoom-step change)
    3. resting arrows (cached `Picture`, rebuilt on exit)
    4. moving arrows + effects (repainted every frame while animating)
- Board units: 1 cell = 1.0. The painter applies `canvas.scale(camera.scale)`. Stroke widths are set in **cell units**, so arrows scale with zoom.
- **Path follower:** an exit path is a polyline of cell centers. Precompute cumulative lengths. At time t, the head is at distance `s(t)` along the path and each body vertex is at `s(t) - offset_i`. Draw the sub-polyline from `s - L` to `s`.
- Large boards (40×60, 300 arrows): the cached picture keeps the frame cost low. Target < 4 ms raster on mid-range devices.

## 5. Input pipeline
`Listener` (raw pointers) → `GestureArena`-free custom state machine in `board_view.dart`:
- 1 pointer: `down` → start timers; `move` > 10 dp → becomes a pan (if pannable); `up` < 250 ms → quick tap → `HitTest.resolve` → `controller.fire(id)` or the ambiguity guard. Held ≥ 250 ms → aim mode (+ loupe if cell < 30 dp), and `up` → fire or cancel.
- 2 pointers: pinch zoom + pan; any tap in progress is cancelled.
- All thresholds are in `game_config.dart`.

## 6. State and flow
- `LevelController` (Riverpod `Notifier<LevelUiState>`) owns:
  - the `GameState` from the engine
  - hearts and Second Chance
  - hints
  - the timer
  - the animation queue (input is blocked only for the *same* arrow while it animates; other arrows can be fired during an animation, as the leaders allow)
  - emitting analytics events
  - calling `SaveService` on every exit (debounced 500 ms)
- **Animation vs. engine:** the engine applies the exit immediately (the state is authoritative), and the animation is purely visual. During the animation, the exiting arrow's cells are already free in the engine. That's fine, because rays are checked against engine state.

## 7. Persistence
`save.json` in the app documents directory:
```json
{
  "schema": 1,
  "journey": { "unlocked": 37, "cleared": {"J001": {"crown": true, "bestMs": 12034}} },
  "inProgress": { "levelId": "J037", "state": "<base64 engine snapshot>", "hearts": 2, "elapsedMs": 41000, "secondChanceUsed": true },
  "daily": { "streak": 12, "lastSolvedUtc": "2026-10-02", "freezes": 1, "history": {"2026-10-02": [1,1,0]}, "trophies": ["2026-09"] },
  "endless": { "easy": 4, "medium": 0, "hard": 0, "nightmare": 0 },
  "wallet": { "coins": 840, "hints": 5 },
  "unlocks": { "themes": ["midnight","paper"], "arrowStyles": ["classic","bold"] },
  "entitlements": { "removeAds": false, "supporter": false },
  "adCounters": { "lastInterAt": 0, "levelsSinceInter": 0, "interToday": 0, "day": "2026-10-03", "rewardFallbacksToday": 0 },
  "stats": { "levelsCleared": 36, "perfects": 20, "hintsUsed": 4, "zenLevels": 0, "playMs": 0 },
  "achievements": ["first_flight"],
  "settings": {}
}
```
- Write atomically (write `save.json.tmp`, then rename). Keep `save.bak` (the previous good save). On a parse error, load the backup and report it to Crashlytics.
- `migrations.dart` upgrades older schemas step by step, and has tests.
- **Cloud save** (Play Games Saved Games): upload a snapshot on level clear (throttled to 1 per 2 min) and on app pause. On a new install or device, if a cloud snapshot is newer (more `journey.unlocked`), offer to restore it. **Merge rule:** take the max of progress, the union of unlocks and achievements, the max of the coins/hints wallet (simple and generous), and OR the entitlements.

## 8. Remote Config
- Defaults are compiled in (`game_config.dart`). Fetch on startup with `minimumFetchInterval = 12 h` and apply on the **next** app start (never mid-session).
- Keys are listed in doc 06 (ads) and doc 12 (experiments).

## 9. Build config
- Android: `minSdk 23`, `targetSdk` = the latest required by Play (check Play Console's target API requirement at release), `compileSdk` = latest.
- R8 minify + resource shrinking on release. Split ABIs are handled by the AAB.
- Signing: upload key in `android/key.properties` (gitignored), with **Play App Signing** enabled.
- Flavors: `dev` (test ads, debug overlays, a level-select cheat menu) and `prod`.
- `--dart-define` values: `ADMOB_APP_ID`, `ADMOB_INTER_ID`, `ADMOB_REWARDED_ID`, `ENV=dev|prod`.
- The AdMob **App ID** also goes into `AndroidManifest.xml` meta-data (it's required at startup), wired from Gradle `manifestPlaceholders`.

## 10. Debug tools (dev flavor only)
- A long-press on the level number opens the dev menu:
  - jump to any level
  - show free arrows (green outline)
  - show difficulty score
  - instant clear
  - +1000 coins
  - reset ad counters
  - simulate a date for the daily
  - toggle placeholder assets
- An FPS overlay toggle.
