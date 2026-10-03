# 03: Mechanics (board elements)

Each mechanic is introduced at the start of a chapter, with a 3-level mini-tutorial: one forced move, one free move, one small puzzle. It then gets mixed into later chapters. All of them are drawn in code (doc 08), with no image assets.

## The Monotonic Rule (read first)
> **No event in the game may ever make any arrow more blocked than it was before.**

Allowed state changes: an arrow leaves (frees cells), a lock counter goes down, ice thaws, a gate opens. Static elements never change.

Two consequences, both relied on in code:
1. **Solvability is order-independent.** If an arrow can leave now, it can still leave later. So a greedy solver ("remove any free arrow, repeat") is **exact**: a level is solvable if and only if greedy clears it. No search or backtracking is needed.
2. **A level can never become unsolvable during play.** There are no dead ends, so we never need a "stuck, restart?" detector. Running out of hearts is the only fail.

A new mechanic must come with a proof sketch that it keeps this rule. Otherwise it's rejected.

## Exit-ray tracing (shared by all mechanics)
```
traceRay(arrow):
  pos = arrow.head; dir = arrow.headDir; path = []; visited = {}
  loop:
    pos = pos + dir
    if pos outside bounding rect: return Clear(path)
    if (pos, dir) in visited: return Blocked(at: pos, reason: loop)      // mirror/portal loop
    visited.add((pos, dir))
    cell = board[pos]
    switch cell:
      masked-out hole / empty           -> path.add(pos)
      arrow body (any arrow, incl. self) -> return Blocked(at: pos, reason: arrow)
      rock                              -> return Blocked(at: pos, reason: rock)
      colorGate(closed)                 -> return Blocked(at: pos, reason: gate)
      colorGate(open)                   -> path.add(pos)
      oneWay(d)  where d == dir         -> path.add(pos)
      oneWay(d)  where d != dir         -> return Blocked(at: pos, reason: oneWay)
      mirror('/' or '\\')               -> path.add(pos); dir = reflect(dir, type)
      portal(id)                        -> path.add(pos); pos = partnerOf(pos)  // continue from partner, same dir
                                           path.add(pos)
```
`reflect('/')`: right→up, up→right, left→down, down→left. `reflect('\\')`: right→down, down→right, left→up, up→left.

The exit animation path = the arrow's own cells (tail→head) + ray `path` + 3 extra cells beyond the edge in the final direction (so the arrow visibly flies off-screen). Portals split the path into two segments with a teleport fade.

---

## M1: Arrows (levels 1–25, chapter 1)
- **Rules:** see doc 02.
- **Tutorial:**
  - L1: 4×4 board, 3 straight arrows, only one free. A hand pointer shows where to tap.
  - L2: 5×5, 5 arrows, one bend.
  - L3: 5×6, 7 arrows, first blocked-tap lesson ("Blocked! The path must be clear." It costs no heart on L3).
  - From L4: normal rules.

## M2: Rocks (chapter 2, levels 26–50)
- **What:** a static, unmovable stone in a cell. It blocks any ray forever.
- **Rule:** the generator never places an arrow whose ray hits a rock. Rocks also shape the board and create long "corridors".
- **Visual:** a rounded square in the board's muted color, with a soft inner shadow.
- **Monotonic:** static.

## M3: Locks (chapter 3, levels 51–75)
- **What:** an arrow carries a padlock badge with a number `n` (1–9).
- **Rule:**
  - Every time **any** other arrow exits, all lock counters go down by 1.
  - At 0 the padlock pops open (sound `unlock`) and the arrow becomes normal.
  - A locked arrow can't be fired; tapping it shakes the padlock and costs no heart.
  - A locked arrow's body still blocks other rays.
- **Generator note:** a lock's `n` must be ≤ the number of arrows that greedy removal frees before this arrow. The validator checks this by simulation.
- **Monotonic:** counters only go down.

## M4: Ice (chapter 4, levels 76–100)
- **What:** an arrow encased in ice (frosted overlay, with a crack pattern that grows).
- **Rule:**
  - A frozen arrow can't be fired; tapping it costs no heart.
  - It **thaws** when any arrow that was **orthogonally adjacent** to one of its cells exits (sound `ice_crack`). The thaw finishes when that exit animation ends.
  - Thick ice (`ice: 2`) needs two such neighbor exits.
- **Monotonic:** ice only goes down.

## M5: Mirrors (chapter 5, levels 101–125)
- **What:** a cell with a diagonal mirror, `/` or `\`. It's never occupied by an arrow.
- **Rule:** a ray entering the cell turns 90°. An arrow whose ray passes through a mirror bends around it as it flies out.
- **Aim line:** the aim line shows the bend, which teaches the mirror visually.
- **Monotonic:** static.

## M6: Keys and Gates (chapter 6, levels 126–150)
- **What:**
  - A **gate** is a colored bar across a cell. Colors are A–D, using the special key palette.
  - A **key arrow** has a key icon of the same color on its body.
- **Rule:**
  - A closed gate blocks rays.
  - When the key arrow exits, every gate of its color opens: it slides away (sound `unlock`) and becomes passable for good.
  - Gates are never occupied by arrows.
- **Colorblind:** the gate and key also carry a shape (circle, triangle, square or diamond), so color is never the only signal.
- **Monotonic:** gates only open.

## M7: Portals (chapter 7, levels 151–175)
- **What:** paired swirl cells (pairs 1–3, each with its own color + shape).
- **Rule:** a ray entering portal A continues from portal B in the **same direction**. Portal cells are never occupied.
- **Animation:** the arrow shrinks into A and grows out of B (sound `portal`).
- **Monotonic:** static.

## M8: One-way gates (chapter 8, levels 176–200)
- **What:** a cell with a chevron (›) pointing in one direction.
- **Rule:** a ray can pass only if it travels in the chevron's direction; any other direction is blocked. The cell is never occupied.
- **Monotonic:** static.

## Chapters 9–20 (levels 201–500): remix
Every chapter has 1–2 **featured** mechanics and 1 **guest**:

| Ch | Place | Featured | Guest |
|---|---|---|---|
| 9 | Lantern Night Market | Locks + Ice | Rocks |
| 10 | Windmill Fields | Mirrors + Rocks | Locks |
| 11 | Coral Reef | Portals | Ice |
| 12 | Balloon Valley | Keys & Gates + Locks | Mirrors |
| 13 | Pyramid Dunes | One-way + Rocks | Portals |
| 14 | Ice Fjord | Ice (thick) | Keys & Gates |
| 15 | Volcano Island | Mirrors + Portals | One-way |
| 16 | Bamboo Forest | Long winding arrows (30–60 cells) | Locks |
| 17 | Castle on the Hill | Keys & Gates + One-way | Ice |
| 18 | Moonlit Temple | All, low density | n/a |
| 19 | Star Observatory | All, big boards | n/a |
| 20 | Sky Islands | All, mixed, finale | n/a |

## Mechanic density limits (readability)
- At most **3 mechanic types** on one board (chapters 18–20 may use 4).
- Static elements ≤ 12% of cells. Locked + frozen arrows ≤ 25% of arrows.
- At most 3 portal pairs and 4 gate colors.
