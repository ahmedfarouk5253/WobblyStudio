# 04: Level System (format, generator, solver, difficulty)

Journey levels are **generated offline** by `tool/level_gen.dart`, **validated** by `tool/validate_levels.dart`, and shipped as JSON packs in `assets/levels/`. Shipped levels are frozen data: changing the generator never changes a shipped level. Daily and Endless levels are generated **on device** from a seed with a **version-locked** generator.

## 1. Level JSON format (v1)
One file per chapter: `assets/levels/journey_ch01.json` … `journey_ch20.json`.

```json
{
  "format": 1,
  "chapter": 1,
  "levels": [
    {
      "id": "J001",
      "w": 6, "h": 8,
      "mask": null,
      "tier": "normal",
      "mechanics": ["arrow"],
      "arrows": [
        { "t": "2,5", "m": "UUR", "d": "R", "c": 3 },
        { "t": "0,0", "m": "R",   "d": "U", "c": 1, "lock": 2 },
        { "t": "4,1", "m": "DD",  "d": "D", "c": 5, "ice": 1 },
        { "t": "5,6", "m": "L",   "d": "L", "c": 0, "key": "A" }
      ],
      "statics": [
        { "k": "rock",   "p": "3,3" },
        { "k": "mirror", "p": "1,4", "v": "/" },
        { "k": "portal", "p": "0,7", "v": 1 },
        { "k": "portal", "p": "5,0", "v": 1 },
        { "k": "oneway", "p": "2,2", "v": "R" },
        { "k": "gate",   "p": "4,4", "v": "A" }
      ],
      "score": 23.4,
      "par": 25,
      "seed": "0x9E3779B97F4A7C15",
      "gen": 1
    }
  ]
}
```
| Field | Meaning |
|---|---|
| `t` | Tail cell `"x,y"` |
| `m` | Moves from tail toward head, one letter per step (`U R D L`). Cell count = `len(m) + 1`. The last cell is the head. |
| `d` | Head direction (`U R D L`). It must not point at the previous cell. |
| `c` | Color index 0–7 |
| `lock`, `ice` | Optional, default 0 |
| `key` | Optional, gate color `A`–`D` |
| `mask` | `null`, or a base64 bitset (row-major, 1 = playable cell) |
| `tier` | `normal`, `hard`, `superhard` or `landmark` |
| `score` | Difficulty 0–100 (section 5) |
| `par` | Estimated seconds for an average player |
| `seed`, `gen` | Reproducibility info (generator version) |

## 2. Engine model (pure Dart, `lib/engine/`)
- `Board`: dimensions, mask, static map, and the cell → arrowId occupancy grid (`Int32List`, -1 = empty).
- `Arrow`: id, cells (`List<Cell>`), dir, color, lock, ice, key.
- `GameState`: board, remaining arrows, hearts, gates opened, lock/ice counters, free-repeat memory, elapsed ms, plus an undo log for state save/restore.
- `Engine.tryFire(arrowId) → FireResult`:
  - `Exited(path)`
  - `Blocked(blockerCell, reason, heartLost)`
  - `Locked`
  - `Frozen`
- `Engine.freeArrows() → List<int>`: used by the hint, the solver and the scorer.
- Everything is deterministic and allocation-light; target **< 0.2 ms** per `tryFire` on a 40×60 board.
- Incremental optimisation: keep, for each cell, the set of arrows whose ray passes through it ("ray watchers"). When cells free up, re-check only those arrows. Implement the simple full scan first, add this only if profiling needs it.

## 3. Solver (exact, thanks to the Monotonic Rule)
```
solve(level):
  state = initial
  order = []
  loop:
    free = all arrows that are not locked, not frozen, and whose ray is Clear
    if free is empty: break
    a = pick(free)                // any choice works; use the lowest id for determinism
    state.exit(a)                 // applies lock decrements, ice thaw, gate opening
    order.add(a)
  return state.remaining.isEmpty ? Solved(order) : Unsolvable(state.remaining)
```
The validator runs this on every shipped level, and unit tests prove it on hand-made edge cases: mirror loops, portal into own body, key behind its own gate, lock count too high.

## 4. Generator (reverse placement, `lib/engine/gen/`)
Core idea: build the board in **reverse removal order**. Each new arrow's exit ray only has to be clear of the arrows already placed, because those leave *after* it. That guarantees solvability by construction, and the validator then confirms it.

```
generate(params, seed):
  rng = Xoroshiro128(seed)
  board = empty(params.w, params.h, params.mask)
  placeStatics(board, params.statics, rng)          // rocks, mirrors, portals, one-ways, gates (gates start "open")
  placed = []                                       // in reverse removal order
  failures = 0
  while coverage(board) < params.targetCoverage and failures < params.maxFailures:
    head = random empty cell (bias: 60% near already placed arrows for density)
    dir  = random direction
    ray  = traceRay(head, dir, board, gatesClosedFor = keysAlreadyPlaced)
    if ray is Blocked or ray crosses head: failures++; continue
    len  = sampleLength(params.lengthDist, rng)
    body = growBackward(head, len, bendChance = params.bendChance,
                        avoid = occupied ∪ statics ∪ ray.cells, rng)
    if body.length < 2: failures++; continue
    arrow = Arrow(cells: body (tail..head), dir)
    placeOn(board, arrow); placed.add(arrow)
    maybeMakeKey(arrow, ...)                        // see mechanics rules below
  removalOrder = placed.reversed
  assignLocks(removalOrder, params.lockRate)        // lock n ≤ index in removalOrder, n ≤ 9
  assignIce(removalOrder, params.iceRate)           // only if ≥ ice adjacent arrows come earlier in removalOrder
  assignColors(placed)                              // greedy: adjacent arrows get different colors, 8-color palette
  return level
```
Mechanic specifics during placement:
- **Gates:** while a gate's key arrow is *not yet placed*, the gate is passable. That's correct, because arrows placed earlier leave after the key, when the gate is open. The key arrow is chosen at about 40–60% of the placement count, and from then on the gate is **closed** for later placements (including the key's own ray). Reject the key if no earlier-placed arrow's ray passes the gate, since a gate nobody needs is decoration.
- **Mirrors / portals / one-way:** handled inside `traceRay`. Reject a static if fewer than 2 arrows' rays end up passing through it, since unused elements confuse players.
- **Long arrows (Bamboo chapter):** `growBackward` with a self-avoiding random walk plus a "prefer straight" bias of 0.55 to avoid noodle soup.

### Candidate selection
For each level slot, generate **N = 40 candidates** (seeds `baseSeed + i`). Compute each one's score, and keep the candidate closest to the slot's target score that also passes the readability limits (03). Store the winning seed.

## 5. Difficulty score (0–100)
Computed by simulating "a reasonable player", in `lib/engine/difficulty.dart`:
```
N      = arrow count
steps  = greedy removal with free-set recorded each step
scar   = mean over steps of (1 - |free_t| / remaining_t)          // 0 = everything free, 1 = one free
bottle = count of steps where |free_t| == 1, divided by N
depth  = longest chain in "A blocks B" DAG, divided by N
area   = visible cells / 400 (cap 3)
bend   = mean bends per arrow (cap 4) / 4
mech   = sum of weights of mechanics present (lock .08, ice .08, mirror .12, gate .10, portal .12, oneway .08), cap .35
score  = 100 * clamp( .22*log2(N)/6 + .26*scar + .14*bottle + .12*depth + .10*min(area,1) + .06*bend + mech*0.29 , 0, 1)
par_s  = 1.2*N*(0.6 + scar) + 4*bottle*N + 8                    // rough seconds estimate, tuned with analytics later
```
Store both the score and `par`. After launch, **re-fit the weights** with real data: the `level_complete` event carries `duration_s`, `hearts_lost` and `hints_used` (doc 12).

## 6. Journey curve
- **Within each chapter** (25 levels), slot tiers:
  - Slots 1–4: ramp up
  - **5: Hard**
  - 6–7: breathers (−8 score)
  - 8–9: ramp
  - **10: Hard**
  - 11–14: ramp
  - **15: Hard**
  - 16–17: breathers
  - 18–19: ramp
  - **20: Hard**
  - 21: breather
  - **22: Super Hard**
  - 23–24: medium
  - **25: Landmark** (shaped by the chapter mask, medium-hard, big and beautiful)
- **Across chapters:**

| Chapter | Normal target score | Board size range (w×h) | Arrow length | Notes |
|---|---|---|---|---|
| 1 | 8 → 30 | 4×4 → 10×14 | 2–4 | L1–3 tutorial; first 15 levels ≈ 10–25 s each |
| 2 | 25 → 38 | 9×12 → 12×16 | 2–6 | Rocks |
| 3 | 30 → 42 | 10×14 → 13×18 | 2–7 | Locks |
| 4 | 33 → 45 | 11×15 → 14×20 | 2–8 | Ice |
| 5–8 | 36 → 55 | 12×16 → 18×26 | 2–10 | One new mechanic each |
| 9–14 | 45 → 65 | 14×20 → 22×32 | 3–12 | Remixes |
| 15–17 | 55 → 72 | 18×26 → 26×38 | 3–60 | Long arrows chapter 16 |
| 18–20 | 60 → 80 | 20×30 → 30×44 | 3–20 | Finale |

- Hard = normal target + 12; Super Hard = +22; Landmark = +8, with a bigger board.
- **Breathers are important:** after every spike, give 1–2 easy levels.
- **Levels 1–15 must average ≤ 20 s.** The fast early wins are the leaders' secret weapon.

## 7. Landmark masks
- Source: `assets/images/masks/chNN_mask.png` (black = playable).
- **Rasterize:** choose a grid so the mask fits in about 24×32 cells while keeping its aspect ratio. A cell is playable if ≥ 50% of its pixels are black. Then remove islands smaller than 4 cells and fill 1-cell holes.
- The rasterized mask is baked into the level JSON at generation time, so the PNG isn't needed at runtime for gameplay. It's still shipped, to show the silhouette on the Travel Journal page.
- **Fallback:** if a mask file is missing, use built-in procedural shapes (heart, star, circle, diamond, house) from `lib/engine/gen/shapes.dart`.

## 8. Daily puzzle (on-device)
- `seed = fnv1a64("arrowvoyage-daily-v1|" + yyyy-mm-dd in UTC)`
- **Sizes:**
  - Small: score target 35, ≈ 12×16
  - Medium: 50, ≈ 18×26
  - Big: 65, ≈ 26×38
- Mechanics: any 1–2 from those the player has unlocked, chosen by the seed. Players who haven't unlocked any mechanics get arrows only. The mechanic set depends on unlock state, so the daily is identical for everyone *with the same unlocks*. That's fine, because there's no global leaderboard in v1.
- **Generator version lock:** `DailyGeneratorV1` is **frozen code**. Any improvement goes into `DailyGeneratorV2`, switched on at a month boundary via Remote Config `daily_gen_version`.
- Generation must take **< 400 ms** on a low-end device (Snapdragon 450 class). Run it in an isolate (`compute`) and show the calendar while it generates.

## 9. Endless ("Beyond the Map")
- `seed = hash(difficulty, playerEndlessIndex)`. Difficulty presets:
  - Easy 30
  - Medium 50
  - Hard 65
  - Nightmare 82 (big boards, long arrows, all mechanics, `bottle` weighted ×1.5)
- Each difficulty tracks its own counter and best streak of perfect clears.

## 10. Tools
| Command | Does |
|---|---|
| `dart run tool/level_gen.dart --chapter 3` | Regenerates chapter 3 → `assets/levels/journey_ch03.json` |
| `dart run tool/level_gen.dart --all` | All 20 chapters (≈ 500 levels × 40 candidates; should finish in < 2 min) |
| `dart run tool/validate_levels.dart` | Solves and scores all levels, checks bands and readability limits, prints a table; non-zero exit on any failure |
| `dart run tool/level_preview.dart J137 --png out.png` | Renders a level to PNG for review |
| `dart run tool/curve_report.dart` | Prints a score-per-level ASCII chart to eyeball the roller-coaster |

## 11. Hand-tuning
Generated levels are a starting point. Before launch, the owner plays chapters 1–4 and flags levels in `tool/level_overrides.json`:
```json
{ "J005": { "reroll": 3 }, "J012": { "targetScoreDelta": -5 } }
```
The generator honors overrides deterministically.
