# 02: Game Design (core loop and rules)

## 1. The board
- A rectangular grid of `width × height` cells (min 4×4, max 40×60). An optional **mask** removes cells to make a shape: heart, star, landmark silhouette.
- A **dot** is drawn at every valid cell center (the "dot grid"). It's always on and free. Players use it to trace rows and columns.
- Cells hold one of:
  - empty
  - part of exactly one **arrow**
  - a **static element**: rock, mirror, portal, one-way gate or color gate (see 03)
- Coordinates: `(x, y)` with `x` = column (0 at left) and `y` = row (0 at **top**). Directions: `up (0,-1)`, `right (1,0)`, `down (0,1)`, `left (-1,0)`.

## 2. Arrows
- An arrow is an ordered list of **2 to 60 orthogonally connected cells**: `cells[0]` = tail, `cells[last]` = head.
- The **head direction** is fixed and is NOT necessarily the direction from the previous cell to the head. Rule: the head direction must not point back into `cells[last-1]`.
- An arrow can bend any number of times but never overlaps itself or another arrow.
- Every arrow has a **color index** (0–7) from the active palette. Colors are cosmetic only and never carry a rule (that's a colorblindness rule). Key arrows are the exception: they also show a key icon (03).

## 3. Exit rule (the whole game)
When the player fires an arrow:
1. Trace the **exit ray** from the head: step one cell at a time in the head direction until the ray leaves the board's bounding rectangle. Mirrors and portals can redirect it (03).
2. If every ray cell is passable (empty, a masked-out hole, or a passable static element), the arrow **exits**: the head moves along the ray, the body follows the arrow's own path, and the arrow is removed.
3. Otherwise the tap is **blocked**: the arrow moves forward until its head touches the first blocker, bumps, then slides back to its original position. **The board does not change.** The player loses one heart, unless the free-repeat rule applies (below).
4. **Locked or frozen arrows** (03) do not move when tapped. They shake gently, show why ("2 more", ice crack), and cost **no heart**. The lock or ice is visible, so this isn't a puzzle mistake.

### Free-repeat rule (kindness rule)
Tapping the **same** blocked arrow again costs no heart until something on the board changes (any arrow exits). The arrow still bumps. This stops double-taps and frustrated repeats from draining hearts.

## 4. Hearts and Second Chance
- Each level starts with **3 hearts** (config `heartsPerLevel = 3`).
- **Second Chance:** the first heart lost in a level shows a small "↺ Refund" chip next to the hearts for 6 s. Tapping it restores the heart. Free, once per level. Premium users get it for every lost heart, at most 3 times per level.
- At **0 hearts**, the "Out of hearts" sheet appears (no ad plays automatically). It offers:
  - **Keep going (watch ad)**: +1 heart. It always pays out, and if the ad fails to load the heart is granted anyway (max 3 fallbacks a day). Premium: free, 3 times a day.
  - **Use 100 coins**: +1 heart.
  - **Restart level**: free. The board resets, hearts go back to 3, the timer resets.
  - **Turn on Zen mode**: continue with no hearts.
- Nothing carries over between levels. There is no energy system.

## 5. Zen mode
- A toggle in Settings and on the "Out of hearts" sheet. Free forever.
- No hearts are shown and blocked taps cost nothing. You still collect coins, but **crowns** (perfect clears) can't be earned. A small lotus icon in the HUD shows Zen mode is on.

## 6. Controls (Smart Touch)
| Gesture | Result |
|---|---|
| **Quick tap** (< 250 ms, moves < 10 dp) on an arrow | Fire it, unless the ambiguity guard triggers |
| **Press and hold** (≥ 250 ms) on an arrow | **Aim:** the arrow lifts and glows, and a dotted aim line shows its full exit path. It shows the path only, never whether the path is clear. If cells are smaller than 30 dp, the **Precision Loupe** appears too. Release on the same arrow to fire; slide off it and release to cancel. |
| One-finger drag | Pan (only when zoomed in or the board is bigger than the screen) |
| Pinch | Zoom (between fit-to-screen and the 56 dp cell size) |
| Double-tap on empty space | Toggle between "fit whole board" and "zoom to 40 dp cells around this point" |
| Fit button (HUD) | Animate back to fit the whole board |

### Hit testing
1. Convert the touch point to board coordinates.
2. If the cell under it belongs to an arrow, that's the candidate.
3. If the cell is empty or a static element, take the arrow whose cell center is nearest, within **0.6 cell**. Ties go to the arrow whose **head** is nearer.
4. **Ambiguity guard:** if the on-screen cell size is under 32 dp **and** the touch point is within 0.22 cell of a cell border shared with a *different* arrow, don't fire on a quick tap. Instead, highlight both arrows for 1.2 s and show "Hold to aim" once per session. The player then uses press-and-hold. No heart is lost.
5. The touch radius in dp stays constant regardless of zoom.

### Precision Loupe
- A 120 dp circle that floats 90 dp above the finger and shows the area under it at 2.2× zoom, with the candidate arrow outlined.
- The player slides a finger to change the candidate. The loupe follows, and the candidate updates on cell change, with a light haptic tick.
- Setting **"Always aim before firing"** (default off) makes every tap use press-and-release.

## 7. Camera
- At level start: fit the whole board with a 16 dp margin, between the HUD (top) and the controls (bottom).
- If the fitted cell size is under **18 dp**, start zoomed at **26 dp** cells, centered on the board's center, and show the **minimap** (top-right, 96 dp, tappable to jump).
- The camera never moves without player input, except: a hint pans to the hinted arrow, and the level-complete zoom-out.

## 8. Hints
- A hint **highlights one currently free arrow**: it pulses 3 times, its aim line glows gold, and the camera pans to it. Which free arrow: prefer the one that unlocks the most other arrows next (greedy heuristic). Tie → closest to the board center.
- Hint tokens: start with **3**. Earn them from the daily gift, chapter completion (+2), achievements, a rewarded ad (+1 each), or buy one for **150 coins**.
- **Stall helper:** if the player has made no successful exit for **30 s** and has less than 2 hints in the inventory, Pip pops up beside the hint button with "Stuck? Watch for a hint". It's a bubble, never a popup. Max once per level.
- Hints never fire an arrow by themselves.

## 9. Level flow
1. **Start:** a short board-build animation (arrows draw in from the tail, staggered, 0.6 s max total). Hearts, the level number and the difficulty badge (Normal / Hard / Super Hard / Landmark) appear in the HUD.
2. **Play:** the timer runs in the background (shown only if the setting is on). Progress bar = cleared / total arrows.
3. **Clear:** the last arrow exits → 0.3 s pause → the camera zooms out → the dots ripple outward → Pip cheers → the results card shows.
4. **Results card:**
   - Coins earned (+ crown bonus).
   - Crown if perfect.
   - Time and personal best (if timer on).
   - The postcard piece revealed (animated paint stroke into the Travel Journal thumbnail).
   - Buttons: **Next** (primary, big) · Double coins (rewarded ad, optional, small) · Home.
5. **Next:** the interstitial check (06) runs **here**, after the player taps Next and before the next level loads. It never shows on the results card itself.

## 10. Rewards per level
| Event | Coins |
|---|---|
| Normal clear | 10 |
| Hard / Super Hard clear | 20 / 30 |
| Landmark clear | 50 |
| Perfect (crown) bonus | +50% (rounded up) |
| First clear only | rewards are paid once; replays pay 0 coins but can still earn a crown |
| Double coins (rewarded ad) | ×2 on that level's coins |

## 11. Modes
| Mode | Unlock | Description |
|---|---|---|
| **Journey** | start | 20 chapters × 25 levels = 500 levels at launch. Linear; replay any cleared level. |
| **Daily Puzzle** | level 8 | One puzzle a day, the same worldwide (date-seeded), in **3 sizes**: Small (≈2 min), Medium (≈4 min), Big (≈8 min). Solving any size keeps the streak. Solving all 3 earns a calendar sticker. Solving ≥ 20 days in a month earns that month's **trophy**. Past days are replayable from the calendar for 7 days (coins only, no streak). |
| **Beyond the Map (Endless)** | after level 100 | Infinite generated levels in **Easy / Medium / Hard / Nightmare**, each tracked separately. Uses every unlocked mechanic. |
| **Zen** | always | A toggle, not a separate mode. |

## 12. Fail and quit
- **Leaving mid-level** saves the full state (remaining arrows, hearts, timer, camera), and it's restored on return. This answers "each level I have to start from the beginning".
- **Restart** is in the pause menu; no confirmation is needed if fewer than 3 arrows were cleared.
- Running out of hearts is the only "fail" state, and it always offers a free restart.

## 13. Difficulty signalling
Before a Hard / Super Hard / Landmark level starts, a 0.8 s banner slides in: "Hard level" (amber), "Super hard" (coral) or "Landmark: Harbor Lighthouse" (teal). Emhance's playtests found that signalled spikes feel fair and unsignalled ones feel cheap.

## 14. Settings (all free)
Sound, music, haptics (off / light / strong), theme, arrow palette (Vivid / Soft / Colorblind-safe / Mono-high-contrast), arrow thickness (S / M / L), show timer, always aim before firing, left-handed HUD, Zen mode, reduce motion, daily reminder time, language, restore purchases, privacy options (UMP), support email, credits.
