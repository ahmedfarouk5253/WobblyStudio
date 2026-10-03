# 07: UX / UI Spec

Portrait only. Design at **390×844 dp**, and test from 320×640 up to tablets (on tablets, center the content at 600 dp max width; the board can use the full width). All touch targets are ≥ 48 dp. Use safe-area insets everywhere.

## 1. Screen map
```
Splash → (first launch) Consent (UMP) → Level 1 directly (no menu first!)
                                         ↓ after level 3
Home ─┬─ Play (continue Journey) → Level → Results → Next…
      ├─ Daily → Calendar → Daily level → Daily results
      ├─ Travel Journal → Postcard viewer
      ├─ Beyond the Map (after L100) → difficulty picker → Level
      ├─ Shop
      └─ Settings → Themes & Arrows / Our ad promise / Privacy / Credits
```
**First-time flow:** the app opens straight into Level 1 with no menu and no login. That's the leaders' fastest-onboarding trick. Home appears for the first time after Level 3 ("Welcome to your Travel Journal!", with Pip waving).

## 2. Home
- Background `home_bg`, with Pip (`pip_idle`) sitting on the map, breathing animation (scale 1.00↔1.02, 3 s).
- Top bar: coins (tap → Shop), hints count, settings gear.
- Center: the current chapter's postcard card (partially revealed), with the progress "Level 37 · Canal City 12/25".
- **Big primary button:** "Play · Level 37", full width minus 32 dp, 64 dp tall, coral, with a soft pulse every 4 s.
- Row of 3 secondary tiles (icon + label): Daily (streak flame + count, "New!" dot if unsolved), Travel Journal, Shop. Beyond the Map is a 4th tile once unlocked.
- Daily gift chest bubble: top-left, when available.

## 3. Level screen (the most important screen)
```
┌───────────────────────────────────────┐
│ ⏸  Lv 37  [Hard]        ♥ ♥ ♡  ↺     │  ← HUD top (56 dp). ↺ = Second Chance chip (temporary)
│ ▓▓▓▓▓▓▓▓▓░░░░░░░░  18/42             │  ← progress bar (4 dp) + count
│                                       │
│                                       │
│           B O A R D                   │  ← fills all remaining space, 16 dp margins
│                                       │
│                               [mini]  │  ← minimap (only when zoomed in)
│                                       │
│  [💡 3]                 [⤢ Fit]       │  ← bottom bar (72 dp): Hint (left), Fit (right)
└───────────────────────────────────────┘
```
- In **left-handed mode** the bottom bar is mirrored.
- **Pause sheet:** Resume, Restart, Settings (sound/haptics/theme quick toggles + Zen toggle), Home.
- **Timer** (if enabled): small, top-right under the hearts, monospaced, grey.
- **Zen mode:** a lotus icon replaces the hearts.
- No other buttons. No shop, no ads, no banners, no offers on this screen. Ever.

### Board interaction feedback
| Event | Visual | Sound | Haptic |
|---|---|---|---|
| Touch-down on arrow | Arrow brightens 15%, scale 1.03 | n/a | selection tick (light) |
| Hold (aim) | Dotted aim line along the exit path, animated dash flow; Precision Loupe if small | n/a | light |
| Exit | Head accelerates (ease-in, 0.25–0.7 s), body follows, fading trail of dots for 0.4 s, original cells flash with tiny dot ripples | `arrow_exit_[1-3]` (random, pitch ±5%) | light impact |
| Blocked | Arrow slides to the blocker, squash 10%, slides back (0.35 s). The blocking arrow wiggles. A red dotted line shows the ray up to the blocker for 0.8 s | `arrow_blocked` + `heart_lost` | medium impact ×2 |
| Free-repeat blocked | Same bump, no heart change, no red line | `arrow_blocked` (quieter) | light |
| Locked / frozen tap | Padlock or ice shakes, "2 more" chip | soft tick | light |
| Last arrow | Slow-motion 0.6× on the final exit, then the level-clear sequence | `level_clear` / `perfect_clear` | success pattern |

## 4. Results card (a bottom sheet that rises to 70% height)
- Title: "Level clear!" or "Perfect!" (with the crown icon bouncing in).
- Pip `pip_cheer`, coins counter rolling up (`coin` ticks), time + PB (if timer on).
- Postcard thumbnail with the newly painted tile glowing.
- **Next** (primary, full width). Under it, smaller: "▶ Double coins" (rewarded) and "Home".
- Auto-advance: none. The player always taps Next.

## 5. Out-of-hearts sheet
- Pip `pip_oops`, "Out of hearts!"
- Options as described in doc 02 §4. Button order: **Keep going (▶ ad / free for premium)**, Use 100 coins, Turn on Zen mode, Restart. No close (X) delay and no timer.

## 6. Daily screen
- `daily_header` art, then a streak flame + count + freezes held.
- 3 size cards: Small / Medium / Big, each with an estimated time, a status (Play / ✓ / time), and a reward.
- Month calendar (7×5 grid) with stickers; tap a past day (last 7) to replay.
- Trophy shelf (horizontal scroll).

## 7. Travel Journal screen
- `journal_bg`, a vertical list of 20 postcards in a two-column "journal" grid.
- Each card: the postcard with reveal state, the chapter name, "12/25", crowns count, stamp (if complete), padlock (if locked).
- Postcard viewer: full screen, pinch to zoom, chapter name, place facts (1 friendly sentence each, in `l10n`), Close.

## 8. Shop
See doc 06 §6. Prices always come from the store (`ProductDetails.price`), never hard-coded.

## 9. Settings
Grouped list. "Gameplay": Zen, Show timer, Always aim, Left-handed, Reduce motion. "Look": Theme, Arrow palette, Arrow style, Arrow thickness. "Sound": Sound, Music, Haptics. Then "Daily reminder", "Language", "Purchases" (restore, status), "Our ad promise", "Privacy options" (UMP form), "Help & feedback" (email + app version + device info copied), "Credits".

## 10. Tutorial script (levels 1–3 + mechanic intros)
- **L1:** the board fades in. The hand pointer taps the only free arrow. Text bubble (Pip): "Tap an arrow to send it flying." After it exits: "Clear them all!" The other 2 arrows are free in sequence.
- **L2:** no hand. If the player is idle for 5 s, the hand points at a free arrow.
- **L3:** The first blocked tap triggers a freeze-frame: "Blocked! An arrow needs a clear path." The red ray line shows the blocker. **No heart lost on L3.** Then "Each blocked tap costs a ♥. You have 3."
- **Mechanic intro (first level of each mechanic chapter):** a full-width card before the level: an animated mini-board demo drawn by the same painter (looping 3 s) plus one sentence. Example: "Locked arrows open after other arrows leave." Then the 3 intro levels follow (doc 03).
- At most one tutorial bubble on screen at a time, and every bubble can be dismissed by tapping anywhere.

## 11. Motion and transitions
- Screen transitions: shared-axis fade/slide, 250 ms. A level-to-level transition (on Next) cross-fades the board, 300 ms.
- Reduce-motion setting: no camera zooms (use fades), no particle trails, no slow-mo, and the exit animation is shortened to 0.2 s.
- Keep 60 fps on mid-range devices. On low-end devices (auto-detected by average frame time > 20 ms over 120 frames), disable trails and the dot ripples.

## 12. Typography and UI kit (drawn in code)
- **Display:** "Baloo 2" (Google Fonts, OFL) at 600/700, for titles, level numbers and buttons.
- **Body/UI:** "Nunito" (OFL) at 400/600/700.
- **Numbers:** Nunito with tabular figures (`FontFeature.tabularFigures()`).
- Bundle the fonts in `assets/fonts/` (don't fetch at runtime; offline-first).
- **Buttons ("candy" style):**
  - 18 dp radius, a vertical gradient (top 8% lighter), a 1 dp inner top highlight (white at 35%), a 4 dp darker bottom lip, and a soft colored shadow.
  - Press: the lip collapses to 1 dp and the button moves down 3 dp with a spring.
  - Primary is coral, secondary is turquoise, quiet buttons are frosted.
- **Panels and sheets:** frosted glass over the painted backgrounds (`BackdropFilter` blur 18, surface color at 72%, 1 dp white border at 10%, 28 dp radius, layered soft shadows). Drag handle on sheets.
- **HUD chips** (coins, hearts, hints): pill-shaped frosted chips with the 3D icon overlapping the left edge slightly (it pops out of the chip).
- **Motion:** spring physics everywhere (`SpringDescription(mass: 1, stiffness: 400, damping: 22)`), staggered entrances (40 ms apart), and number roll-ups for coins and timers. Idle elements breathe gently (scale 1.00↔1.02).
- **Hero numbers** (level number, coins on results): Baloo 2 800 with a soft 2 dp drop shadow and a subtle top-to-bottom gradient fill.
- **Reference bar:** screenshots of the game should hold up next to the polish of the top casual puzzle games. If a screen looks flat, add depth (shadow, gradient, glow) before adding more things.
