# 14: QA and Testing

## 1. Automated tests (must pass before every commit: `flutter test`)
| Area | Must cover |
|---|---|
| Ray tracing | Each direction; edge exit; blocked by arrow / rock / closed gate / wrong one-way; mirror both types × 4 directions; portal pairs; mirror loop → blocked; ray into own body → blocked; masked holes passable |
| Engine | Exit removes cells; blocked changes nothing but hearts; free-repeat rule; locks decrement on every exit; ice thaws only from orthogonal neighbours; thick ice = 2; gates open when the key exits; hearts can't go below 0; Zen = no heart loss; snapshot → restore round-trip is identical |
| Solver | Solvable fixtures; unsolvable fixtures (2-arrow head-on cycle, key behind its own gate, lock too high); order-independence property test: 200 random generated levels × 20 random removal orders, each must clear |
| Generator | Determinism (same seed → byte-identical JSON); every generated level validates; candidate selection hits the target score ±4; readability limits respected; generation time per level < 50 ms (journey sizes) |
| Daily | Same date → same 3 levels on every run; `DailyGeneratorV1` golden test: the hash of the generated JSON for 10 fixed dates is hard-coded and **must never change** |
| Difficulty | Monotone-ish sanity: more arrows + same structure → higher score; fixture scores stay within ±1 of expected |
| Hit testing | Taps on cell centers, borders, gaps; ambiguity guard triggers only below 32 dp; constant dp radius across zoom |
| Camera | Fit math for 4×4 … 40×60 on 320×640 and 1280×800; min/max zoom clamps |
| Ad policy | Every rule in doc 06 §2, with clock injection; premium never shows; after-fail skip; daily cap resets at local midnight |
| Economy | Rewards per tier; perfect bonus rounding; first-clear only; purchase grants once per purchaseId; 30-day economy simulation stays within targets (doc 05 §2) |
| Save | Atomic write; corrupt file → backup; migrations; cloud merge rule |
| Widgets | Level screen renders a 10×14 board; the results sheet shows the correct buttons for premium vs free; the shop hides Remove Ads when owned |

CI: GitHub Actions `flutter analyze`, `flutter test` and `dart run tool/validate_levels.dart` on every push.

## 2. Manual test checklist (before every release)
- [ ] Fresh install → opens straight into Level 1 → tutorial completes → Home after Level 3.
- [ ] Play 20 levels in a row: no interstitial before level 15; after that, ≤ 1 every 3 levels and ≥ 3 min apart.
- [ ] Run out of hearts: no interstitial on that transition. Keep going with an ad works; with airplane mode on, the reward is still granted (fallback).
- [ ] Buy Remove Ads (license tester) → no interstitials, the button is gone, premium refills work. Uninstall/reinstall → restore works.
- [ ] Big board (Nightmare): zoom, pan, minimap, loupe, Fit button; no mis-fires when panning.
- [ ] Kill the app mid-level → reopen → same board, hearts and timer.
- [ ] Daily: solve Small → streak +1; change the device date (dev menu) → streak logic + freeze consumption.
- [ ] All settings persist after restart. Theme/palette changes apply live on the board.
- [ ] TalkBack navigation through Home, Settings and Shop.
- [ ] Font scale 1.3×, display size largest: no overflow (check debug overflow stripes).
- [ ] Reduce-motion on: no zooms or trails.
- [ ] Offline from first launch: the game fully works (ads/IAP gracefully unavailable).
- [ ] Rotate / split-screen: portrait locked; split-screen doesn't crash.
- [ ] UMP consent shown in an EEA test (`ConsentDebugSettings` with geography EEA); privacy options button works.
- [ ] Notification permission is asked only after the first daily; the reminder fires.
- [ ] Back button (Android): from a level → pause sheet; from Home → exit confirmation toast ("Press back again to exit").

## 3. Device matrix (minimum)
| Class | Example | Why |
|---|---|---|
| Low-end, Android 8–9, 2 GB RAM | Samsung Galaxy A10 / Redmi 7A | Emerging-market volume |
| Mid, Android 12–13 | Pixel 4a / Galaxy A52 | Typical |
| High, Android 14–16, 120 Hz | Pixel 8 / Galaxy S23 | High refresh |
| Small screen | 5" 720p | Layout |
| Tablet | Galaxy Tab A 10" | Layout, board size |
| Foldable (optional) | Galaxy Z Fold | Resize behavior |
Use Firebase Test Lab's free tier (Robo test) for crash smoke tests on 10+ devices.

## 4. Performance budgets
- Cold start to first interactive frame ≤ 2.0 s on low-end.
- Board 40×60 / 300 arrows: steady 60 fps on mid-range while an arrow animates; ≤ 16 ms worst frame on low-end, with effects reduced.
- Memory ≤ 250 MB. Install size ≤ 40 MB (WebP assets, 2 music loops at 96 kbps OGG).
- Level JSON packs: ≤ 1.5 MB total.

## 5. Acceptance criteria for "launch ready"
All automated tests green · manual checklist passed on 3+ devices · crash-free ≥ 99.5% in closed testing · no P0/P1 bugs open · all P0+P1 assets in place (check_assets clean) · store listing complete in 12 languages · Data safety + content rating + target audience + ads declaration done · privacy policy live.
