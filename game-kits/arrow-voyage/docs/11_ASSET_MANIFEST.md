# 11 — Asset Manifest

This is the list of every image and sound the game ships with. The same list is in `assets_manifest.json` (machine-readable) and on the **Asset Brief** web page you give to your image generator. All three are generated from one source, so they always match.

## How assets get into the game

1. Generate each image with the exact **file name**, **folder**, and **pixel size** below (bigger is fine if the aspect ratio matches).
2. Drop the folders `assets/`, `store/` and `assets_src/` into the project root (merge with existing folders).
3. Tell Claude Code: *"New assets are in. Run the asset check and wire them in."* Claude Code will run `dart run tool/check_assets.dart` (verifies every path, size, ratio and alpha against `assets_manifest.json`), then `dart run tool/optimize_assets.dart` (resize + WebP), then replace placeholders.
4. Until real assets exist, the game uses **auto-generated placeholders** (colored rounded rects with the file name), so development never blocks on art.

## Rules

- File names and folders are exact. Do not rename.
- Generate PNG at the listed pixel size (or larger with the same aspect ratio; Claude Code's tool/optimize_assets script resizes and converts to WebP).
- transparent=true means real alpha transparency (no fake checkerboard, no white box).
- Never put text in images except logo_wordmark.png.
- The puzzle board, arrows, dot grid, mechanics tiles, buttons and particles are drawn in code — they are NOT in this list on purpose.

## Global style (prepend to every prompt if your generator has no style reference)

> Soft flat vector illustration for a cozy mobile puzzle game. Rounded friendly shapes, clean edges, subtle paper grain texture, gentle soft shading (no harsh gradients), limited palette: ink navy #1B2240, warm cream #F6F0E4, coral #FF6B5B, amber #FFB547, teal #2EC4B6, sky blue #4DA8FF, violet #9B7BFF. Calm, warm, uncluttered. No text, no letters, no numbers, no logos, no watermark, no signature.

## Mascot description

> Pip, a small round chubby sparrow-like bird: deep teal-blue body (#2E6F8E), cream belly (#F6F0E4), tiny amber beak (#FFB547), big friendly dark eyes with white highlights, rosy cheeks, a small coral explorer scarf (#FF6B5B) and a tiny brown leather satchel. Its tail feathers form one clean arrowhead shape. Proportions: head and body one round shape, short stubby wings, little orange feet.

## Priority

- **P0** — needed for the first playable / closed test.
- **P1** — needed for public launch.
- **P2** — nice to have / experiments.


## 0 · Reference (generate first)

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets_src/reference/pip_character_sheet.png` | 2048×1024 | 2:1 | no | P0 | Reference only | Character sheet for the mascot. NOT shipped in the app. Use it as the reference image for every other Pip asset so Pip looks identical everywhere. |
| `assets_src/reference/style_tile.png` | 2048×1024 | 2:1 | no | P0 | Reference only | Style tile: palette, textures and a sample prop. NOT shipped. Use as a style reference for all other images. |

## 1 · App icon

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/icon/icon_master_1024.png` | 1024×1024 | 1:1 | no | P0 | flutter_launcher_icons, Play Store listing | Full-bleed square app icon. Used for the Play Store 512 px icon and legacy launcher icons (tooling crops/rounds it). |
| `assets/icon/icon_foreground_1024.png` | 1024×1024 | 1:1 | yes | P0 | flutter_launcher_icons adaptive_icon_foreground | Adaptive icon FOREGROUND layer (Android 8+). |
| `assets/icon/icon_background_1024.png` | 1024×1024 | 1:1 | no | P0 | flutter_launcher_icons adaptive_icon_background | Adaptive icon BACKGROUND layer. |
| `assets/icon/icon_monochrome_1024.png` | 1024×1024 | 1:1 | yes | P1 | flutter_launcher_icons adaptive_icon_monochrome | Android 13+ themed (monochrome) icon layer. |
| `assets/icon/icon_variant_b_1024.png` | 1024×1024 | 1:1 | no | P2 | Play Console Store listing experiments | Alternative icon for a Play Store Listing Experiment (A/B test) — features the mascot. |
| `assets/icon/notification_icon_96.png` | 96×96 | 1:1 | yes | P1 | flutter_local_notifications small icon (res/drawable/ic_stat_arrow.png) | Small status-bar icon for the daily puzzle reminder notification. |

## 2 · Branding & store

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/branding/splash_logo_1152.png` | 1152×1152 | 1:1 | yes | P1 | flutter_native_splash | Android 12+ splash screen icon (shown centered on a navy background while the app loads). |
| `assets/branding/logo_wordmark.png` | 1600×640 | 5:2 | yes | P2 | Feature graphic, website, promo | Game logo with the title 'Arrow Voyage'. OPTIONAL — the app renders the title in code with a font; this is for the feature graphic and marketing. |
| `store/feature_graphic_1024x500.png` | 1024×500 | 256:125 | no | P0 | Play Console > Main store listing > Feature graphic | Google Play feature graphic (the wide banner on your store page and in some promos). |
| `store/screenshot_bg_01.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #1 for framed store screenshots (night). A script places a real gameplay capture on it and adds a caption. |
| `store/screenshot_bg_02.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #2 for framed store screenshots (paper). A script places a real gameplay capture on it and adds a caption. |
| `store/screenshot_bg_03.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #3 for framed store screenshots (ocean). A script places a real gameplay capture on it and adds a caption. |
| `store/screenshot_bg_04.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #4 for framed store screenshots (dusk). A script places a real gameplay capture on it and adds a caption. |
| `store/screenshot_bg_05.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #5 for framed store screenshots (warm). A script places a real gameplay capture on it and adds a caption. |
| `store/screenshot_bg_06.png` | 1080×1920 | 9:16 | no | P1 | Store screenshots (composited) | Background plate #6 for framed store screenshots (aurora). A script places a real gameplay capture on it and adds a caption. |

## 3 · Mascot (Pip)

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/images/mascot/pip_idle.png` | 1024×1024 | 1:1 | yes | P0 | Home screen, empty states | Pip — standing. |
| `assets/images/mascot/pip_wave.png` | 1024×1024 | 1:1 | yes | P0 | Tutorial / first launch / welcome back | Pip — waving one wing hello with a big smile. |
| `assets/images/mascot/pip_cheer.png` | 1024×1024 | 1:1 | yes | P0 | Level complete, perfect clear, rating prompt | Pip — jumping with both wings up. |
| `assets/images/mascot/pip_think.png` | 1024×1024 | 1:1 | yes | P0 | Hint button tooltip, hint popup | Pip — one wing on chin. |
| `assets/images/mascot/pip_oops.png` | 1024×1024 | 1:1 | yes | P0 | Out of hearts popup | Pip — gently embarrassed. |
| `assets/images/mascot/pip_sleep.png` | 1024×1024 | 1:1 | yes | P1 | Streak freeze / come back tomorrow | Pip — sleeping curled up on a small folded map. |
| `assets/images/mascot/pip_travel.png` | 1024×1024 | 1:1 | yes | P1 | Travel Journal / postcard album, chapter unlock | Pip — holding an unfolded paper map with both wings. |
| `assets/images/mascot/pip_shop.png` | 1024×1024 | 1:1 | yes | P1 | Shop header, coin rewards | Pip — proudly holding a big shiny gold coin with both wings. |
| `assets/images/mascot/pip_stamp.png` | 1024×1024 | 1:1 | yes | P1 | Postcard completed celebration | Pip — stamping a postcard with a big rubber stamp. |

## 4 · UI icons

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/images/ui/icon_coin.png` | 512×512 | 1:1 | yes | P0 | HUD, rewards, shop | A shiny round gold coin (amber #FFB547) with a small embossed arrowhead symbol in the middle. |
| `assets/images/ui/icon_coin_stack.png` | 512×512 | 1:1 | yes | P1 | Shop: small coin pack | A small stack of five gold coins with arrowhead emblems. |
| `assets/images/ui/icon_coin_pile.png` | 512×512 | 1:1 | yes | P1 | Shop: medium coin pack | A big pile of gold coins with a couple of sparkles. |
| `assets/images/ui/icon_coin_chest.png` | 512×512 | 1:1 | yes | P1 | Shop: large coin pack | An open wooden treasure chest overflowing with gold coins. |
| `assets/images/ui/icon_heart_full.png` | 512×512 | 1:1 | yes | P0 | Level HUD hearts | A plump glossy coral-red heart (#FF6B5B). |
| `assets/images/ui/icon_heart_empty.png` | 512×512 | 1:1 | yes | P0 | Level HUD lost heart | The same plump heart shape but empty: pale grey-navy outline with a faint inner tint, slightly cracked line in the middle. |
| `assets/images/ui/icon_hint.png` | 512×512 | 1:1 | yes | P0 | Hint button | A glowing warm-yellow light bulb with a tiny arrowhead-shaped filament. |
| `assets/images/ui/icon_streak_flame.png` | 512×512 | 1:1 | yes | P0 | Daily streak counter | A friendly orange-amber flame with a soft inner glow. |
| `assets/images/ui/icon_streak_freeze.png` | 512×512 | 1:1 | yes | P1 | Streak freeze item | A light-blue ice crystal shield with a small snowflake in the middle. |
| `assets/images/ui/icon_gift.png` | 512×512 | 1:1 | yes | P1 | Daily reward, free gift | A teal gift box with a coral ribbon bow. |
| `assets/images/ui/icon_trophy.png` | 512×512 | 1:1 | yes | P1 | Achievements, daily monthly trophy | A gold trophy cup with a small arrowhead on the front. |
| `assets/images/ui/icon_star.png` | 512×512 | 1:1 | yes | P0 | Level select, results | A plump five-pointed gold star. |
| `assets/images/ui/icon_crown.png` | 512×512 | 1:1 | yes | P0 | Perfect clear (no hearts lost) badge | A small gold crown with three rounded points and a coral gem. |
| `assets/images/ui/icon_ad_video.png` | 512×512 | 1:1 | yes | P0 | Rewarded-ad button badge | A rounded teal rectangle play button (like a video play icon) with a tiny sparkle. |
| `assets/images/ui/icon_no_ads.png` | 512×512 | 1:1 | yes | P0 | Remove Ads product | A rounded square TV screen with a coral circle-slash over it. |
| `assets/images/ui/icon_calendar.png` | 512×512 | 1:1 | yes | P0 | Daily puzzle | A small desk calendar page with a coral top binding and a checkmark. |
| `assets/images/ui/icon_journal.png` | 512×512 | 1:1 | yes | P0 | Travel Journal (postcard album) button | A small closed leather travel journal with a compass emblem on the cover. |
| `assets/images/ui/icon_palette.png` | 512×512 | 1:1 | yes | P1 | Themes / arrow skins button | A painter's palette with dots of coral, teal, amber and violet. |
| `assets/images/ui/icon_zen.png` | 512×512 | 1:1 | yes | P1 | Zen mode (no hearts) | A calm lotus flower in teal and cream. |
| `assets/images/ui/icon_lock.png` | 512×512 | 1:1 | yes | P0 | Locked chapters / themes | A chunky rounded padlock in navy and amber. |
| `assets/images/ui/hand_pointer.png` | 512×512 | 1:1 | yes | P0 | Tutorial overlay | Cartoon hand with pointing index finger for the tutorial. |
| `assets/images/ui/supporter_pack.png` | 800×600 | 4:3 | yes | P2 | Shop: Supporter Pack card | Illustration for the Supporter Pack shop card. |

## 5 · Screen backgrounds

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/images/bg/home_bg.png` | 1080×2340 | 6:13 | no | P0 | HomeScreen | Home screen background. Buttons sit in the middle and bottom third, so keep those areas calm. |
| `assets/images/bg/journal_bg.png` | 1080×2340 | 6:13 | no | P1 | JournalScreen | Background for the Travel Journal (postcard album) screen. |
| `assets/images/bg/daily_header.png` | 1080×600 | 9:5 | no | P1 | DailyScreen header | Header art for the Daily Puzzle screen. |
| `assets/images/bg/shop_header.png` | 1080×600 | 9:5 | no | P1 | ShopScreen header | Header art for the Shop screen. |

## 6 · Board themes

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/images/themes/midnight/bg.png` | 1080×2340 | 6:13 | no | P0 | ThemeRegistry 'midnight' | Default dark theme: background behind the puzzle board. |
| `assets/images/themes/paper/bg.png` | 1080×2340 | 6:13 | no | P0 | ThemeRegistry 'paper' | Light theme: background behind the puzzle board. |
| `assets/images/themes/ocean/bg.png` | 1080×2340 | 6:13 | no | P1 | ThemeRegistry 'ocean' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/forest/bg.png` | 1080×2340 | 6:13 | no | P1 | ThemeRegistry 'forest' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/dusk/bg.png` | 1080×2340 | 6:13 | no | P1 | ThemeRegistry 'dusk' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/aurora/bg.png` | 1080×2340 | 6:13 | no | P2 | ThemeRegistry 'aurora' | Premium/Supporter theme: background behind the puzzle board. |

## 7 · Chapters (Travel Journal)

| File | Size (px) | Ratio | Alpha | Pri | Used in | What it is |
|---|---|---|---|---|---|---|
| `assets/images/chapters/ch01_postcard.png` | 1200×800 | 3:2 | no | P0 | Journal chapter 1 'Harbor Lighthouse' | Chapter 1 — Harbor Lighthouse: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch01_stamp.png` | 512×512 | 1:1 | yes | P0 | Journal chapter 1 badge, chapter-complete popup | Chapter 1 — Harbor Lighthouse: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch01_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 25 | Chapter 1 — Harbor Lighthouse: SHAPE MASK for the chapter's landmark level (the board takes the shape of a lighthouse). |
| `assets/images/chapters/ch02_postcard.png` | 1200×800 | 3:2 | no | P0 | Journal chapter 2 'Cherry Blossom Garden' | Chapter 2 — Cherry Blossom Garden: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch02_stamp.png` | 512×512 | 1:1 | yes | P0 | Journal chapter 2 badge, chapter-complete popup | Chapter 2 — Cherry Blossom Garden: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch02_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 50 | Chapter 2 — Cherry Blossom Garden: SHAPE MASK for the chapter's landmark level (the board takes the shape of a five-petal cherry blossom flower). |
| `assets/images/chapters/ch03_postcard.png` | 1200×800 | 3:2 | no | P0 | Journal chapter 3 'Desert Oasis' | Chapter 3 — Desert Oasis: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch03_stamp.png` | 512×512 | 1:1 | yes | P0 | Journal chapter 3 badge, chapter-complete popup | Chapter 3 — Desert Oasis: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch03_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 75 | Chapter 3 — Desert Oasis: SHAPE MASK for the chapter's landmark level (the board takes the shape of a palm tree). |
| `assets/images/chapters/ch04_postcard.png` | 1200×800 | 3:2 | no | P0 | Journal chapter 4 'Alpine Village' | Chapter 4 — Alpine Village: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch04_stamp.png` | 512×512 | 1:1 | yes | P0 | Journal chapter 4 badge, chapter-complete popup | Chapter 4 — Alpine Village: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch04_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 100 | Chapter 4 — Alpine Village: SHAPE MASK for the chapter's landmark level (the board takes the shape of a mountain peak with a snow cap). |
| `assets/images/chapters/ch05_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 5 'Canal City' | Chapter 5 — Canal City: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch05_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 5 badge, chapter-complete popup | Chapter 5 — Canal City: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch05_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 125 | Chapter 5 — Canal City: SHAPE MASK for the chapter's landmark level (the board takes the shape of a gondola boat). |
| `assets/images/chapters/ch06_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 6 'Savannah Sunset' | Chapter 6 — Savannah Sunset: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch06_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 6 badge, chapter-complete popup | Chapter 6 — Savannah Sunset: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch06_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 150 | Chapter 6 — Savannah Sunset: SHAPE MASK for the chapter's landmark level (the board takes the shape of a giraffe). |
| `assets/images/chapters/ch07_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 7 'Northern Lights' | Chapter 7 — Northern Lights: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch07_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 7 badge, chapter-complete popup | Chapter 7 — Northern Lights: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch07_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 175 | Chapter 7 — Northern Lights: SHAPE MASK for the chapter's landmark level (the board takes the shape of a pine tree). |
| `assets/images/chapters/ch08_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 8 'Rainforest Falls' | Chapter 8 — Rainforest Falls: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch08_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 8 badge, chapter-complete popup | Chapter 8 — Rainforest Falls: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch08_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 200 | Chapter 8 — Rainforest Falls: SHAPE MASK for the chapter's landmark level (the board takes the shape of a parrot). |
| `assets/images/chapters/ch09_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 9 'Lantern Night Market' | Chapter 9 — Lantern Night Market: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch09_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 9 badge, chapter-complete popup | Chapter 9 — Lantern Night Market: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch09_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 225 | Chapter 9 — Lantern Night Market: SHAPE MASK for the chapter's landmark level (the board takes the shape of a round paper lantern). |
| `assets/images/chapters/ch10_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 10 'Windmill Fields' | Chapter 10 — Windmill Fields: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch10_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 10 badge, chapter-complete popup | Chapter 10 — Windmill Fields: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch10_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 250 | Chapter 10 — Windmill Fields: SHAPE MASK for the chapter's landmark level (the board takes the shape of a windmill). |
| `assets/images/chapters/ch11_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 11 'Coral Reef' | Chapter 11 — Coral Reef: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch11_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 11 badge, chapter-complete popup | Chapter 11 — Coral Reef: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch11_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 275 | Chapter 11 — Coral Reef: SHAPE MASK for the chapter's landmark level (the board takes the shape of a sea turtle). |
| `assets/images/chapters/ch12_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 12 'Balloon Valley' | Chapter 12 — Balloon Valley: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch12_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 12 badge, chapter-complete popup | Chapter 12 — Balloon Valley: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch12_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 300 | Chapter 12 — Balloon Valley: SHAPE MASK for the chapter's landmark level (the board takes the shape of a hot air balloon). |
| `assets/images/chapters/ch13_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 13 'Pyramid Dunes' | Chapter 13 — Pyramid Dunes: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch13_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 13 badge, chapter-complete popup | Chapter 13 — Pyramid Dunes: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch13_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 325 | Chapter 13 — Pyramid Dunes: SHAPE MASK for the chapter's landmark level (the board takes the shape of a pyramid with a small sun above). |
| `assets/images/chapters/ch14_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 14 'Ice Fjord' | Chapter 14 — Ice Fjord: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch14_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 14 badge, chapter-complete popup | Chapter 14 — Ice Fjord: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch14_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 350 | Chapter 14 — Ice Fjord: SHAPE MASK for the chapter's landmark level (the board takes the shape of a whale tail). |
| `assets/images/chapters/ch15_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 15 'Volcano Island' | Chapter 15 — Volcano Island: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch15_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 15 badge, chapter-complete popup | Chapter 15 — Volcano Island: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch15_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 375 | Chapter 15 — Volcano Island: SHAPE MASK for the chapter's landmark level (the board takes the shape of a volcano). |
| `assets/images/chapters/ch16_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 16 'Bamboo Forest' | Chapter 16 — Bamboo Forest: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch16_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 16 badge, chapter-complete popup | Chapter 16 — Bamboo Forest: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch16_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 400 | Chapter 16 — Bamboo Forest: SHAPE MASK for the chapter's landmark level (the board takes the shape of a panda head). |
| `assets/images/chapters/ch17_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 17 'Castle on the Hill' | Chapter 17 — Castle on the Hill: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch17_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 17 badge, chapter-complete popup | Chapter 17 — Castle on the Hill: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch17_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 425 | Chapter 17 — Castle on the Hill: SHAPE MASK for the chapter's landmark level (the board takes the shape of a castle with three towers). |
| `assets/images/chapters/ch18_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 18 'Moonlit Temple' | Chapter 18 — Moonlit Temple: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch18_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 18 badge, chapter-complete popup | Chapter 18 — Moonlit Temple: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch18_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 450 | Chapter 18 — Moonlit Temple: SHAPE MASK for the chapter's landmark level (the board takes the shape of a crescent moon). |
| `assets/images/chapters/ch19_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 19 'Star Observatory' | Chapter 19 — Star Observatory: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch19_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 19 badge, chapter-complete popup | Chapter 19 — Star Observatory: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch19_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 475 | Chapter 19 — Star Observatory: SHAPE MASK for the chapter's landmark level (the board takes the shape of a rocket). |
| `assets/images/chapters/ch20_postcard.png` | 1200×800 | 3:2 | no | P1 | Journal chapter 20 'Sky Islands' | Chapter 20 — Sky Islands: postcard illustration revealed piece by piece as the 25 levels of this chapter are cleared. |
| `assets/images/chapters/ch20_stamp.png` | 512×512 | 1:1 | yes | P1 | Journal chapter 20 badge, chapter-complete popup | Chapter 20 — Sky Islands: postage-stamp badge awarded when the chapter is completed. |
| `assets/images/masks/ch20_mask.png` | 512×512 | 1:1 | no | P1 | tool/level_gen landmark level 500 | Chapter 20 — Sky Islands: SHAPE MASK for the chapter's landmark level (the board takes the shape of a small bird in flight with wings spread). |

## 8 · Audio (not images — use an AI sound/music generator or CC0 libraries)

Spec for all audio: OGG Vorbis, 44.1 kHz, mono for SFX / stereo for music, normalized around -16 LUFS (SFX peaks ≤ -1 dBFS), trimmed silence, music loops must be seamless.

| File | Pri | Used in | Sound |
|---|---|---|---|
| `assets/audio/sfx/arrow_exit_1.ogg` | P0 | Arrow leaves the board | Soft airy whoosh, short (0.35 s), pleasant, like a paper plane flying past. Variant 1. |
| `assets/audio/sfx/arrow_exit_2.ogg` | P0 | Arrow leaves the board | Soft airy whoosh, short (0.35 s), slightly higher pitch than variant 1. |
| `assets/audio/sfx/arrow_exit_3.ogg` | P0 | Arrow leaves the board | Soft airy whoosh, short (0.4 s), with a tiny wooden 'tick' at the start. |
| `assets/audio/sfx/arrow_blocked.ogg` | P0 | Blocked tap (heart lost) | Muted soft wooden bonk, 0.25 s, not harsh, slightly comedic. |
| `assets/audio/sfx/heart_lost.ogg` | P0 | Heart lost | Gentle descending two-note marimba, 0.5 s. |
| `assets/audio/sfx/level_clear.ogg` | P0 | Level complete | Bright cheerful 1.5 s marimba + glockenspiel jingle, rising, cozy. |
| `assets/audio/sfx/perfect_clear.ogg` | P0 | Perfect clear | Same mood as level_clear but with an extra sparkle chime at the end, 2 s. |
| `assets/audio/sfx/coin.ogg` | P0 | Coins awarded | Single soft coin clink, 0.3 s. |
| `assets/audio/sfx/button.ogg` | P0 | Any button | Very short soft UI pop/click, 0.08 s. |
| `assets/audio/sfx/hint.ogg` | P1 | Hint reveals an arrow | Soft magical twinkle, 0.6 s. |
| `assets/audio/sfx/unlock.ogg` | P1 | Lock opens / theme unlock | Small key turning in a lock + soft chime, 0.7 s. |
| `assets/audio/sfx/stamp.ogg` | P1 | Postcard stamped | Satisfying rubber stamp thump on paper, 0.4 s. |
| `assets/audio/sfx/ice_crack.ogg` | P1 | Frozen arrow thaws | Light ice crack/tinkle, 0.4 s. |
| `assets/audio/sfx/portal.ogg` | P2 | Arrow passes a portal | Short soft 'bloop' warp sound, 0.4 s. |
| `assets/audio/music/calm_loop_1.ogg` | P1 | Menus + gameplay | Lo-fi acoustic travel music, 80 BPM, warm guitar + soft piano, no vocals, seamless 2-minute loop. |
| `assets/audio/music/calm_loop_2.ogg` | P2 | Gameplay (alternate) | Lo-fi chill music with soft kalimba and brushed drums, 75 BPM, no vocals, seamless 2-minute loop. |

## Full prompts

Every prompt is in `assets_manifest.json` (`images[].prompt`) and on the Asset Brief page with a copy button.

