# 08: Art and Audio Direction

## 1. Look in one sentence
**A calm, crisp puzzle board** (code-drawn, perfectly sharp at any zoom) **framed by cozy hand-painted travel art** (AI-generated images). The board is minimal; everything around it is warm.

## 2. Brand palette
| Token | Hex | Use |
|---|---|---|
| ink | #1B2240 | dark surfaces, text on light |
| cream | #F6F0E4 | light surfaces, text on dark |
| coral | #FF6B5B | primary buttons, hearts, alerts |
| amber | #FFB547 | coins, stars, Hard badge |
| teal | #2EC4B6 | success, Zen, Landmark badge |
| sky | #4DA8FF | links, info |
| violet | #9B7BFF | premium, Super Hard accents |

## 3. Board themes (theme = colors in code + a background image)
| Theme | Board bg | Dot color | Static elements | Text | Image |
|---|---|---|---|---|---|
| midnight (default) | #141A2E | #F6F0E4 @ 18% | #2A3352 | #F6F0E4 | themes/midnight/bg.png |
| paper | #F6F0E4 | #1B2240 @ 16% | #D9CFBD | #1B2240 | themes/paper/bg.png |
| ocean | #0F3B4C | #E6FFFB @ 18% | #1D5366 | #E6FFFB | themes/ocean/bg.png |
| forest | #1E3A2F | #F0F7EC @ 18% | #2C5143 | #F0F7EC | themes/forest/bg.png |
| dusk | #2B2347 | #FFF1E8 @ 18% | #3D3363 | #FFF1E8 | themes/dusk/bg.png |
| aurora | #121A33 | #E8F0FF @ 20% | #24304F | #E8F0FF | themes/aurora/bg.png |

The board itself is a rounded rect (radius 20 dp) in the board bg color at 85% opacity over the background image, so the image only peeks around the edges.

## 4. Arrow palettes (8 colors each, cosmetic only)
| Palette | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| Vivid (dark themes) | #FF6B5B | #FFB547 | #8BD46A | #2EC4B6 | #4DA8FF | #9B7BFF | #FF7AC6 | #F6F0E4 |
| Vivid (light themes) | #E5483A | #E39420 | #4FA83A | #149C90 | #2A7FD6 | #7457E0 | #D94C9E | #2A3150 |
| Soft | #F4A39A | #F5CF8E | #B8DFA3 | #93DDD4 | #A6CCF5 | #C6B6F7 | #F5B3D8 | #E9E2D6 |
| Colorblind-safe (Okabe-Ito) | #E69F00 | #56B4E9 | #009E73 | #F0E442 | #0072B2 | #D55E00 | #CC79A7 | #FFFFFF / #000000 by theme |
| Mono high-contrast | all #FFFFFF on dark / #111111 on light, with a 1 dp outline |

Every pair of colors that can sit next to each other must have enough contrast against the board bg (≥ 3:1). `test/palette_contrast_test.dart` checks this.

## 5. How arrows are drawn (`ArrowPainter`)
- **Body:** one continuous rounded polyline through the cell centers. Stroke width = `cell × thickness` (S 0.22, M 0.30, L 0.38), with round caps and round joins. Corners use a quadratic curve with radius `0.35 × cell` for a soft bend.
- **Head:** a filled rounded triangle at the head cell, pointing in `d`, width `stroke × 2.3`, length `cell × 0.55`, its tip at `0.42 × cell` beyond the head center.
- **Tail:** a small round dot of radius `stroke × 0.62`, which helps players see where each arrow starts.
- **Shading:** a 1.5 dp lighter highlight offset up-left on the body (Classic style); none in Mono.
- **States:**
  - locked: padlock badge (code-drawn) at the body's middle cell, plus the number
  - frozen: frosted overlay (white @ 35%, crack lines)
  - key: key glyph + gate shape on the middle cell
- **Performance:** draw all static arrows into a cached `Picture` layer. Redraw it only when an arrow exits. Moving arrows go on a separate layer that repaints each frame.

## 6. Static elements (code-drawn)
- **Rock:** a rounded square (0.78 cell) in the static color with a soft inner shadow.
- **Mirror:** a diagonal bar (0.9 cell) in a light silver gradient with a 2 dp highlight.
- **Portal:** a concentric swirl (3 arcs) in the pair color + shape glyph (circle / triangle / square), rotating slowly (12 s per turn; static if reduce-motion is on).
- **One-way:** a double chevron in the static color @ 80%.
- **Gate:** a thick bar across the cell + key shape glyph. When it opens, it slides into the cell edge and fades.
- **Dot grid:** circles of radius `0.06 × cell` at every playable cell center. Masked-out cells have no dot.

## 7. Particles and effects (code)
- Exit trail: 6–10 small dots along the path, fading over 0.4 s.
- Level clear: dot-grid ripple from the last arrow's exit point (radius grows over 0.8 s), plus confetti (40 small rounded rects in the palette colors, 1.2 s).
- Postcard paint bloom: a radial reveal mask with a noisy edge (fragment shader `shaders/bloom.frag`; fallback is a simple radial clip).

## 8. AI image direction (for the image generator)
- The global style prompt and the mascot description are in `assets_manifest.json` → `global_style`, `mascot`.
- **Workflow:**
  1. Generate `pip_character_sheet.png` and `style_tile.png` first.
  2. Use them as reference images for everything else.
  3. Generate the P0 assets first.
- Keep illustrations **calm and low-contrast where UI sits on top**. Each prompt says which area must stay empty.
- **No text in images**, except the optional wordmark.

## 9. Audio direction
- **Mood:** cozy, acoustic, wooden. Marimba, kalimba, soft whooshes. Nothing harsh or casino-like.
- **SFX list:** see the manifest (`audio`). Exit whooshes rotate through 3 variants, with ±5% random pitch so they don't feel repetitive.
- **Music:** 2 lo-fi loops, default volume 40%, ducked to 25% during the level-clear jingle. Music is OFF by default on first launch (many puzzle players listen to their own audio), and offered in Settings and once on Home after level 10.
- **Mixing:** SFX bus and music bus are separate. Respect the device's silent mode for music.
- **Sources:** ElevenLabs Sound Effects / Stable Audio for SFX; Suno or Udio (a paid plan with commercial rights) for music; or CC0 sound effects from freesound.org / Kenney. Keep a `CREDITS.md` with every source and license.

## 10. Haptics
| Event | Android call |
|---|---|
| Touch-down / loupe tick | `HapticFeedback.selectionClick()` |
| Exit | `HapticFeedback.lightImpact()` |
| Blocked | `HapticFeedback.mediumImpact()` ×2, 70 ms apart |
| Level clear | light, light, medium (90 ms apart) |
| Strong mode | all one step stronger |
