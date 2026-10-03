# 08: Art and Audio Direction

## 1. Look in one sentence
**A glowing, tactile puzzle board** (code-drawn, perfectly sharp at any zoom, with depth, light and motion) **framed by soft-3D characters and painterly travel art** (AI-generated). The aim is the polish of 2025–26 top-grossing casual games, never flat clip-art.

**Art direction v2:**
- **Soft 3D** for Pip, icons, props and stamps: rounded toy-like forms, satin materials, soft studio light, rim light.
- **Painterly illustration** for postcards and screen backgrounds: gouache and watercolor textures, volumetric light, depth.
- **Code-drawn UI and board** pick up the same language: gradients, soft shadows, glow, frosted panels and spring motion (§5, §7 here and doc 07 §12).

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
Arrows should look like **soft glossy tubes** rather than flat lines. This is the single biggest thing that makes the board feel modern.
- **Body:** one continuous rounded path through the cell centers. Stroke width = `cell × thickness` (S 0.24, M 0.32, L 0.40), with round caps and joins. Corners use a quadratic curve of radius `0.38 × cell`.
- **Tube shading (Classic style):** draw the body three times:
  1. the base color
  2. a highlight stroke at 40% width in `lighten(color, 22%)`, offset 0.06 cell up-left, at 70% opacity
  3. a hairline specular stroke at 12% width in white, at 35% opacity

  The result reads as a rounded 3D tube at no extra asset cost.
- **Depth:**
  - Light themes: a soft drop shadow under every arrow (black at 18%, blur 0.10 cell, offset 0.06 cell down).
  - Dark themes: a faint outer glow instead (arrow color at 30%, blur 0.18 cell).
  - Batch all of them into the cached resting layer (§Performance).
- **Head:** a rounded triangle with slightly convex sides (a quadratic curve on each edge). Width `stroke × 2.4`, length `cell × 0.58`, with the same three-pass shading.
- **Tail:** a small round cap dot of radius `stroke × 0.62`, a shade darker than the body.
- **Touch feedback:** the pressed arrow lifts: shadow offset ×2, scale 1.04, glow +40%, 120 ms spring.
- **States:**
  - Locked: a small code-drawn padlock badge (gradient + highlight) with a number on the middle cell.
  - Frozen: a frosted glass overlay (white gradient at 30–45%, 3 crack lines, a sparkle).
  - Key: a key glyph + gate shape.
- **Styles** (doc 05): Classic is the default shaded tube; Bold is thicker; Neon has a strong glow and dark core; Ribbon is flat with a twist highlight; Chalk has a textured stroke; Paper plane has a folded-paper head. All are drawn in code.
- **Performance:** draw all resting arrows into a cached `Picture` layer, rebuilt only when an arrow exits or the theme or zoom step changes. Moving arrows go on a separate layer that repaints each frame.

## 6. Static elements (code-drawn)
- **Rock:** a rounded square (0.78 cell) in the static color with a soft inner shadow.
- **Mirror:** a diagonal bar (0.9 cell) in a light silver gradient with a 2 dp highlight.
- **Portal:** a concentric swirl (3 arcs) in the pair color + shape glyph (circle / triangle / square), rotating slowly (12 s per turn; static if reduce-motion is on).
- **One-way:** a double chevron in the static color @ 80%.
- **Gate:** a thick bar across the cell + key shape glyph. When it opens, it slides into the cell edge and fades.
- **Dot grid:** circles of radius `0.06 × cell` at every playable cell center. Masked-out cells have no dot.

## 7. Particles, light and effects (code)
- **Board card:** a rounded rect (radius 24 dp) with:
  - a vertical gradient: theme bg, lighter by 4% at the top
  - a 1 dp inner highlight border (white at 6–8%)
  - a soft outer shadow
  - a 3% noise overlay (a tiny tiled noise texture generated at startup)
- **Dot grid:** dots have a subtle radial falloff. Dots near the board center are 10% brighter, which gives a soft "lit" center.
- **Exit:**
  - The head stretches slightly along its motion (squash and stretch, up to 1.15×).
  - A short motion-blur ghost trail of 3 fading copies.
  - 6–10 sparkles in the arrow's color burst from the exit point at the board edge.
  - The cells it leaves pulse their dots once.
- **Blocked:** squash 12% against the blocker; a red ray line with a soft glow; a gentle 2 px board shake (off with reduce-motion).
- **Level clear:**
  - The dot grid ripples outward from the last exit point (0.8 s).
  - Soft light rays behind Pip on the results sheet.
  - Confetti: 40 rounded pieces with spin and gravity, 1.2 s.
- **Postcard paint bloom:** a radial reveal mask with a noisy, painterly edge (fragment shader `shaders/bloom.frag`; fallback is a radial clip).

## 8. AI image direction (for the image generator)
- Full rules, prompts and the two style blocks are in `assets_manifest.json` (`styles`, `rules`) and on the Asset Brief page.
- **Use a real image model** (ChatGPT/GPT Image, Gemini/Nano Banana Pro, Midjourney, FLUX, Ideogram). A chat agent asked to "make the files" will draw shapes with code, which is why the first batch looked flat.
- **Workflow:**
  1. Generate `pip_master` (4–8 variants, pick one) and `style_props`.
  2. Attach them as references for every Pip image and every icon.
  3. Generate P0 first.
- Generate at each asset's **Generate as** ratio. Claude Code crops and resizes to the final size, and removes the plain grey background from cutout images (`tool/cutout.py`, using `rembg`).
- Keep scenes **calm where UI sits on top**; each prompt says which area must stay quiet.
- **No text in images**, except the logo.

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
