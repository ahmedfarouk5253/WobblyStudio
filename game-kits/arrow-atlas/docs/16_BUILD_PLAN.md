# 16: Build Plan (copy-paste prompts for Claude Code)

How to use it:
- Paste **one milestone prompt at a time** into Claude Code, opened at the project root.
- Wait until Claude Code reports done, then check the acceptance list yourself (run the app on your phone with `flutter run`).
- Commit and push before moving on.
- If something is off, describe what you see in plain words ("arrows look blurry when zoomed", "the tap went to the wrong arrow"). Claude Code will fix it.

Expected effort: about 3–6 weeks of evenings for M0–M12, depending on how much you polish.

---

## M0: Project bootstrap
```
Read CLAUDE.md and every file in docs/ (skim 00 and 15, read 02, 03, 04, 09 carefully).
Then bootstrap the project:
- flutter create --org com.wobblystudio --project-name arrow_atlas --platforms android,ios .
  (keep CLAUDE.md, docs/, assets_manifest.json in place)
- Add the packages from docs/09 §2 with flutter pub add; set up analysis_options with flutter_lints + strict casts.
- Create the folder structure from docs/09 §3 with placeholder files.
- Write tool/gen_asset_constants.dart (reads assets_manifest.json → lib/core/assets.g.dart) and run it.
- Write lib/core/widgets/placeholder_image.dart: renders a rounded rect in a color hashed from the path, with the file name and size, used whenever an asset file is missing.
- Write tool/check_assets.dart (checks existence, pixel size/aspect ratio, alpha per the manifest; prints a table; exit code 1 if P0 missing when --strict).
- Register asset folders in pubspec.yaml. Bundle fonts Baloo 2 + Nunito (download the OFL TTFs from Google Fonts into assets/fonts and add OFL.txt).
- Add GitHub Actions CI: analyze + test.
- Create docs/DECISIONS.md.
Run analyze + test, commit "M0 bootstrap".
```
**Accept:** `flutter run` shows a blank app with the correct name; CI is green; `dart run tool/check_assets.dart` lists all assets as missing (expected).

## M1: Pure engine + solver + tests
```
Implement lib/engine exactly as docs/02 §1–3, docs/03 (all mechanics) and docs/04 §1–3:
models, level_codec (JSON format v1), ray tracing (with mirrors, portals, one-way, gates, masks, loop detection),
GameState + tryFire (Exited/Blocked/Locked/Frozen, free-repeat rule, hearts, Zen), snapshot/restore, freeArrows,
the greedy solver, and rng.dart (SplitMix64 + Xoroshiro128**).
No Flutter imports in lib/engine. Write the tests listed in docs/14 §1 for Ray, Engine and Solver, including the
order-independence property test. Add 10 hand-written fixture levels in test/fixtures/ covering every mechanic.
```
**Accept:** `flutter test` is green with 60+ engine tests; the property test runs 200 levels × 20 orders.

## M2: Board rendering + Smart Touch (the feel milestone)
```
Build lib/game per docs/02 §6–7, docs/07 §3, docs/08 §5–7 and docs/09 §4–5:
BoardView with camera (fit, pinch zoom, pan, double-tap zoom, Fit button, minimap), cached picture layers,
ArrowPainter (rounded bends, head, tail dot, states), statics painter, dot grid, exit animation via path follower,
blocked bump + red ray line, Smart Touch hit testing with ambiguity guard, press-and-hold aim line, Precision Loupe,
haptics hooks. Add a temporary debug screen that loads test/fixtures levels and a "random level" button using a
simple generator stub. Write hit_test and camera tests from docs/14.
```
**Accept:**
- On your phone the arrows look crisp at every zoom, and the exit animation is smooth (no stutter).
- Tapping is accurate, the loupe appears on small cells, and panning never fires an arrow.
- **Spend real time here.** Play 30 random boards. This milestone *is* the game feel.

## M3: Level generator + validator + Journey packs
```
Implement docs/04 §4–7 and §10: generator (reverse placement, statics, keys/gates, locks, ice, colors),
difficulty score, candidate selection, landmark mask rasterizer (with procedural shape fallback when a mask PNG
is missing), the journey curve table, and the tools level_gen, validate_levels, level_preview, curve_report.
Generate all 20 chapters into assets/levels/. Add generator + difficulty tests from docs/14.
Show me the curve_report output for chapters 1–4.
```
**Accept:** `validate_levels` passes 500/500; the curve report shows the roller-coaster; levels 1–15 are tiny and quick.

## M4: Game loop UI: Level screen, results, hearts, hints
```
Implement LevelController and the Level screen per docs/02 §4–5, §8–13 and docs/07 §3–5 and §10:
HUD, hearts + Second Chance chip, out-of-hearts sheet (ad/coins buttons call stub services for now), Zen mode,
hints (+ stall helper bubble), progress bar, timer option, pause sheet, results sheet (coins, crown, PB),
difficulty banners, tutorial for L1–3 and the mechanic intro cards, mid-level save/restore (SaveService with
atomic writes + backup per docs/09 §7). Wire Journey progression level 1 → 500.
```
**Accept:** you can play from L1 through chapter 2 with no menus missing; killing the app mid-level restores the board exactly.

## M5: Home, Atlas, meta and economy
```
Implement docs/05 and docs/07 §1–2, §7, §9: first-time flow (straight into L1, Home after L3), Home screen,
Atlas (postcard reveal tiles with paint-bloom effect, stamps, viewer), coins + rewards table, achievements
(local), themes + arrow styles + palettes + thickness (Settings), daily gift, unlock timeline. Write the
economy tests including the 30-day simulation.
```
**Accept:** clearing levels paints the postcard; chapter completion stamps it; switching theme/palette updates the board live.

## M6: Daily puzzle + Endless
```
Implement docs/04 §8–9 and docs/05 §4: DailyGeneratorV1 (frozen, golden-hash test), calendar UI, 3 sizes,
streaks, freezes, monthly trophies, 7-day replay; Beyond the Map with 4 difficulties. Generation runs in an isolate.
```
**Accept:** the same daily appears on two devices/emulators; the dev-menu date change exercises streaks and freezes.

## M7: Audio + haptics + polish pass
```
Implement docs/08 §9–10: AudioService (AudioPool for whooshes, random variant + pitch), music with ducking,
haptics service with strength setting, reduce-motion behavior, low-end auto-detect. Add placeholder audio
(generate simple synthesized tones in tool/gen_placeholder_audio.dart if real files are missing).
Do a polish pass on transitions (docs/07 §11) and the level-clear sequence.
```
**Accept:** the game feels juicy with sound on and calm with sound off; it's still 60 fps on your phone.

## M8: Firebase + Remote Config + analytics
```
Set up Firebase per docs/10 §5 (I'll run flutterfire configure when you tell me — give me exact steps).
Implement AnalyticsService with every event from docs/12 §1 as constants, RemoteConfigService with defaults
from game_config.dart, Crashlytics with custom keys. Add a dev-menu screen listing the last 50 analytics events.
```
**Accept:** events show in Firebase DebugView; a forced test crash appears in Crashlytics.

## M9: Ads (AdMob + UMP), fair policy
```
Implement docs/06 §1–4 and docs/10 §2: UMP consent flow, privacy options, AdsService with test IDs, AdPolicy
(pure, fully unit-tested), interstitial at the Next transition only, all rewarded placements with the
always-pays fallback, "Our ad promise" screen in Settings. Give me the exact AdMob console steps to create the
app and ad units, and the AndroidManifest/Gradle wiring for the App ID via --dart-define.
```
**Accept:** no ad before L15; the rules hold in a 40-level session; airplane-mode rewarded still grants (fallback); ad_policy tests are green.

## M10: In-app purchases + Play Games + cloud save + notifications + review
```
Implement docs/06 §5–6, docs/10 §3–4, §6–7: IapService + Entitlements + Shop screen (prices from the store),
restore purchases, starter bundle rules, premium perks; Play Games sign-in, achievements, daily leaderboards,
Saved Games cloud save with the merge rule; daily reminder notifications with permission asked after the first
daily; in-app review trigger. Give me the Play Console steps for products, license testers and Play Games setup.
```
**Accept:** a license-tester purchase of Remove Ads removes interstitials; reinstall + restore works; cloud save restores on a second device.

## M11: Real assets, localization, accessibility
```
New assets are in. Run tool/check_assets.dart, then tool/optimize_assets.dart (resize to the manifest sizes,
convert to WebP quality 88 except icons/masks which stay PNG), regenerate assets.g.dart, remove placeholders
where real files exist, wire launcher icons (flutter_launcher_icons with adaptive + monochrome), native splash,
notification icon. Then: localize into the 12 languages (docs/13), check font scale 1.3×, TalkBack labels,
and the contrast test.
```
**Accept:** `check_assets --strict` is clean; the app looks finished; switching the language works.

## M12: Release prep
```
Prepare release per docs/15 and docs/14 §2–5: release signing config (tell me how to create the upload
keystore), R8, obfuscation + split-debug-info with Crashlytics symbol upload, versioning, the privacy policy
page for the WobblyStudio website (match the other games' pages), store listing text in all 12 languages
(as files under store/listing/<lang>/), screenshot capture via integration_test + compose_screenshots.dart,
and a RELEASE_CHECKLIST.md I can tick through. Build the release AAB.
```
**Accept:** the AAB is uploaded to internal testing and installs from Play; the full manual checklist passes.

---

## Prompts for common situations
- **"It feels laggy on my phone."** → *"Profile the level screen on a 30×45 board in profile mode, find the frame-time hotspots, and fix them per the budgets in docs/14 §4."*
- **"A level feels impossible / unfair."** → *"Level J142 feels unfair: <what happened>. Check it with validate_levels and level_preview, and reroll it via level_overrides if the score is off-band."*
- **"Add a new chapter."** → *"Add chapters 21–22 per docs/17, using mechanics X and Y; generate, validate, and add the assets to assets_manifest.json (then regenerate the docs table and the asset brief data)."*
- **"Players say ads are too many."** → *"Show me the current ad policy values and the analytics for interstitials per DAU, and propose new Remote Config values."*
