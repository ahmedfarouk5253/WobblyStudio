# 11: Asset Manifest (art direction v2)

Every image and sound the game ships with. The same list is in `assets_manifest.json` (machine-readable) and on the **Asset Brief** web page. All three come from one source, so they always match.

## Why v2

The first batch came out as flat geometric shapes because the tool *drew them with code* instead of using an image model, and v1 prompts asked for "flat vector" art with exact pixel sizes and transparency, which pushed it that way. v2 fixes both. It uses two modern looks: **soft 3D** for the mascot, icons and props, and **painterly illustration** for scenes. Each image has an aspect ratio to generate at, and cutout images use a plain background for removal.

## How assets get into the game

1. Generate each image with a real image model at its **Generate as** ratio, and save it under the exact **file name** and **folder**.
2. Drop `assets/`, `store/` and `assets_src/` into the project root.
3. Tell Claude Code: *"New assets are in. Run the asset pipeline and wire them in."* That runs:
   - `python3 tool/cutout.py`: background removal with `rembg` for every `cutout` image that still has an opaque background.
   - `python3 tool/derive_assets.py`: makes the monochrome icon, notification icon and splash logo; cleans masks to pure black/white.
   - `dart run tool/optimize_assets.dart`: center-crops to the final aspect ratio, resizes to the final size, converts to WebP (icons and masks stay PNG).
   - `dart run tool/check_assets.dart`: verifies path, size, ratio and alpha against the manifest.
4. Until real assets exist, the game uses auto-generated placeholders, so development never waits for art.

## Rules

- Use a real image-generation model (see generators). Do not hand the brief to a chat agent and ask it to 'make all the files': it will draw them with code and they will look like flat clip-art.
- Paste one prompt at a time. Generate 2–4 variants and keep the best.
- Generate at the 'Generate as' aspect ratio in the highest quality available. The exact pixel size does not matter: Claude Code crops and resizes to the final size.
- Cutout images: generate on the plain light-grey background the prompt asks for. Remove the background in your generator or remove.bg, or leave it and Claude Code removes it with tool/cutout.py.
- Generate pip_master first and attach it as the reference image for every Pip prompt. Attach style_props for every UI icon.
- Save each image with the exact file name and folder shown. PNG preferred; JPG/WebP are accepted and converted.
- Never put text in images except logo_wordmark.png.
- Images marked 'Claude Code makes this' are derived automatically. Skip them.
- The puzzle board, arrows, dot grid, mechanic tiles, buttons and particles are drawn in code, so they are not in this list.

## Recommended generators

- ChatGPT (image generation) or GPT Image: best all-rounder, follows long prompts, good with a reference image.
- Google Gemini / Nano Banana Pro: excellent quality and character consistency with a reference image.
- Midjourney v7: most beautiful painterly scenes (postcards, backgrounds); use --cref with pip_master for Pip.
- FLUX (Higgsfield, Krea, fal): strong 3D renders, supports several reference images.
- Ideogram 3 or Recraft: good for the logo text and icon sets.

## The two looks

**Soft 3D** (mascot, icons, props, stamps):
> Premium stylized 3D render, the polish level of a top-grossing 2026 mobile puzzle game. Soft rounded toy-like forms, smooth matte-satin materials with gentle subsurface scattering, soft studio key light from the upper left, subtle cool rim light, soft ambient occlusion, crisp clean silhouette, rich detail, high resolution. Warm, cozy and joyful.

**Painterly** (postcards, screen backgrounds, store art):
> Beautiful painterly digital illustration for a premium cozy mobile game: hand-painted gouache and watercolor textures, soft volumetric light, atmospheric depth, gentle glow, harmonious rich colors, storybook travel-poster composition, rich detail, high resolution.

**Soft background** (board themes, screenshot backdrops):
> Soft abstract background art: smooth gradient with very subtle film grain and a few large, gentle, out-of-focus light shapes. Calm, premium, minimal, high resolution.

Palette line added to 3D and painterly prompts:
> Color mood: deep twilight navy, warm cream, coral red, honey amber, turquoise teal, sky blue and soft violet, used as a harmony with natural shading, not as flat fills.

Ending added to every prompt:
> Avoid: any text, letters, numbers, logos, watermark or signature; flat clip-art; simple geometric shapes; vector or SVG look; thick black outlines; muddy or harsh colors; clutter; blur; deformed anatomy.

## Mascot description

> Pip, an adorable round, chubby little bird mascot (a fluffy sparrow-like chick): soft deep-teal-blue plumage with a slightly fluffy feather texture, a round cream-colored belly, a tiny honey-amber beak, huge glossy dark eyes with bright catchlights, soft pink cheeks, a small coral knitted explorer scarf with two tassels, and a tiny brown leather satchel with a brass buckle worn across the body. The tail feathers come to one neat arrowhead shape. Short stubby wings, small orange feet. Head and body merged into one big round shape, about 1.3 heads tall.

## Priority

- **P0**: needed for the first playable / closed test.
- **P1**: needed for public launch.
- **P2**: nice to have, or made automatically by Claude Code.


## 0 · Reference (generate first)

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets_src/reference/pip_master.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Reference only | The master image of the mascot. NOT shipped. Every other Pip image uses it as the reference, so Pip looks identical everywhere. |
| `assets_src/reference/pip_turnaround.png` | 2048×1024 | 16:9 | 3d | no | P1 | Reference only | Turnaround sheet (front / side / back) for consistency. NOT shipped. |
| `assets_src/reference/style_props.png` | 2048×1024 | 16:9 | 3d | no | P0 | Reference only | Style reference: the 3D prop look. NOT shipped. Upload as a style reference for icons. |

## 1 · App icon

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/icon/icon_master_1024.png` | 1024×1024 | 1:1 | 3d | no | P0 | flutter_launcher_icons, Play Store listing | Full-bleed square app icon: the Play Store 512 px icon and legacy launcher icon. |
| `assets/icon/icon_foreground_1024.png` | 1024×1024 | 1:1 | 3d | yes | P0 | flutter_launcher_icons adaptive_icon_foreground | Adaptive icon FOREGROUND layer (Android 8+). |
| `assets/icon/icon_background_1024.png` | 1024×1024 | 1:1 | soft | no | P0 | flutter_launcher_icons adaptive_icon_background | Adaptive icon BACKGROUND layer. |
| `assets/icon/icon_monochrome_1024.png` | 1024×1024 | Claude Code makes this | flat | no | P2 | flutter_launcher_icons adaptive_icon_monochrome | Android 13+ themed (monochrome) icon layer. |
| `assets/icon/icon_variant_b_1024.png` | 1024×1024 | 1:1 | 3d | no | P2 | Play Console store listing experiments | Alternative icon for a Play Store listing experiment, featuring the mascot. |
| `assets/icon/notification_icon_96.png` | 96×96 | Claude Code makes this | flat | no | P2 | flutter_local_notifications small icon | Status-bar icon for the daily reminder. |

## 2 · Branding & store

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/branding/splash_logo_1152.png` | 1152×1152 | Claude Code makes this | flat | no | P2 | flutter_native_splash | Android 12+ splash icon. |
| `assets/branding/logo_wordmark.png` | 1600×640 | 21:9 | 3d | yes | P1 | Feature graphic, website, promo | Game logo reading 'Arrow Voyage' for the store graphic, website and promo. |
| `store/feature_graphic_1024x500.png` | 1024×500 | 16:9 | paint | no | P0 | Play Console > Main store listing > Feature graphic | Google Play feature graphic (the wide banner on the store page). |
| `store/screenshot_bg_01.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #1 for framed store screenshots (night). Claude Code places a real gameplay capture and a caption on it. |
| `store/screenshot_bg_02.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #2 for framed store screenshots (paper). Claude Code places a real gameplay capture and a caption on it. |
| `store/screenshot_bg_03.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #3 for framed store screenshots (ocean). Claude Code places a real gameplay capture and a caption on it. |
| `store/screenshot_bg_04.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #4 for framed store screenshots (dusk). Claude Code places a real gameplay capture and a caption on it. |
| `store/screenshot_bg_05.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #5 for framed store screenshots (warm). Claude Code places a real gameplay capture and a caption on it. |
| `store/screenshot_bg_06.png` | 1080×1920 | 9:16 | soft | no | P1 | Store screenshots (composited) | Backdrop #6 for framed store screenshots (aurora). Claude Code places a real gameplay capture and a caption on it. |

## 3 · Mascot (Pip)

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/images/mascot/pip_idle.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Home screen, empty states | Pip: standing. |
| `assets/images/mascot/pip_wave.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Tutorial, first launch, welcome back | Pip: waving one wing hello with a big open smile. |
| `assets/images/mascot/pip_cheer.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Level complete, perfect clear | Pip: jumping with both wings raised. |
| `assets/images/mascot/pip_think.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Hint button tooltip | Pip: one wing on chin. |
| `assets/images/mascot/pip_oops.png` | 1024×1024 | 1:1 | 3d | yes | P0 | Out-of-hearts sheet | Pip: gently embarrassed. |
| `assets/images/mascot/pip_sleep.png` | 1024×1024 | 1:1 | 3d | yes | P1 | Streak freeze, come back tomorrow | Pip: sleeping curled up on a small folded paper map. |
| `assets/images/mascot/pip_travel.png` | 1024×1024 | 1:1 | 3d | yes | P1 | Travel Journal, chapter unlock | Pip: holding an unfolded paper map with both wings. |
| `assets/images/mascot/pip_shop.png` | 1024×1024 | 1:1 | 3d | yes | P1 | Shop header, coin rewards | Pip: proudly hugging a big shiny gold coin. |
| `assets/images/mascot/pip_stamp.png` | 1024×1024 | 1:1 | 3d | yes | P1 | Postcard complete | Pip: pressing a big wooden rubber stamp onto a postcard. |

## 4 · UI icons

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/images/ui/icon_coin.png` | 512×512 | 1:1 | 3d | yes | P0 | HUD, rewards, shop | A shiny gold coin with a raised arrow symbol pointing right in the middle. |
| `assets/images/ui/icon_coin_stack.png` | 512×512 | 1:1 | 3d | yes | P1 | Shop: small coin pack | A neat stack of five shiny gold coins with raised arrow symbols. |
| `assets/images/ui/icon_coin_pile.png` | 512×512 | 1:1 | 3d | yes | P1 | Shop: medium coin pack | A generous pile of shiny gold coins with a couple of sparkles. |
| `assets/images/ui/icon_coin_chest.png` | 512×512 | 1:1 | 3d | yes | P1 | Shop: large coin pack | An open wooden treasure chest with brass trim overflowing with gold coins. |
| `assets/images/ui/icon_heart_full.png` | 512×512 | 1:1 | 3d | yes | P0 | Level HUD hearts | A plump glossy coral-red heart. |
| `assets/images/ui/icon_heart_empty.png` | 512×512 | 1:1 | 3d | yes | P0 | Level HUD lost heart | The same plump heart, but made of frosted translucent pale-blue glass and empty inside. |
| `assets/images/ui/icon_hint.png` | 512×512 | 1:1 | 3d | yes | P0 | Hint button | A glowing warm-yellow light bulb with a small arrow-shaped filament. |
| `assets/images/ui/icon_streak_flame.png` | 512×512 | 1:1 | 3d | yes | P0 | Daily streak counter | A friendly orange-and-amber flame with a soft inner glow. |
| `assets/images/ui/icon_streak_freeze.png` | 512×512 | 1:1 | 3d | yes | P1 | Streak freeze item | A light-blue ice crystal shield with a snowflake emblem. |
| `assets/images/ui/icon_gift.png` | 512×512 | 1:1 | 3d | yes | P1 | Daily gift | A turquoise gift box tied with a coral ribbon bow. |
| `assets/images/ui/icon_trophy.png` | 512×512 | 1:1 | 3d | yes | P1 | Achievements, monthly trophy | A gold trophy cup with a small arrow emblem on the front. |
| `assets/images/ui/icon_star.png` | 512×512 | 1:1 | 3d | yes | P0 | Level select, results | A plump glossy five-pointed gold star. |
| `assets/images/ui/icon_crown.png` | 512×512 | 1:1 | 3d | yes | P0 | Perfect clear badge | A small gold crown with three rounded points and a coral gem. |
| `assets/images/ui/icon_ad_video.png` | 512×512 | 1:1 | 3d | yes | P0 | Rewarded-ad button badge | A rounded turquoise video play button with a tiny sparkle. |
| `assets/images/ui/icon_no_ads.png` | 512×512 | 1:1 | 3d | yes | P0 | Remove Ads product | A small retro TV with a coral prohibition circle-slash in front of its screen. |
| `assets/images/ui/icon_calendar.png` | 512×512 | 1:1 | 3d | yes | P0 | Daily puzzle | A small desk calendar with a coral top binding and a turquoise check mark on the page. |
| `assets/images/ui/icon_journal.png` | 512×512 | 1:1 | 3d | yes | P0 | Travel Journal button | A closed brown leather travel journal with a brass compass emblem and a ribbon bookmark. |
| `assets/images/ui/icon_palette.png` | 512×512 | 1:1 | 3d | yes | P1 | Themes and arrow styles | A wooden painter's palette with blobs of coral, turquoise, amber and violet paint. |
| `assets/images/ui/icon_zen.png` | 512×512 | 1:1 | 3d | yes | P1 | Zen mode | A calm lotus flower with turquoise and cream petals. |
| `assets/images/ui/icon_lock.png` | 512×512 | 1:1 | 3d | yes | P0 | Locked chapters and themes | A chunky rounded padlock in navy and brass. |
| `assets/images/ui/hand_pointer.png` | 512×512 | 1:1 | 3d | yes | P0 | Tutorial overlay | Cartoon hand pointing, for the tutorial. |
| `assets/images/ui/supporter_pack.png` | 800×600 | 4:3 | 3d | yes | P2 | Shop: Supporter Pack card | Illustration for the Supporter Pack card in the shop. |

## 5 · Screen backgrounds

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/images/bg/home_bg.png` | 1080×2340 | 9:16 | paint | no | P0 | HomeScreen | Home screen background. Buttons sit in the middle and bottom, so those areas must stay calm. |
| `assets/images/bg/journal_bg.png` | 1080×2340 | 9:16 | paint | no | P1 | JournalScreen | Travel Journal screen background. |
| `assets/images/bg/daily_header.png` | 1080×600 | 16:9 | paint | no | P1 | DailyScreen header | Header art for the Daily Puzzle screen. |
| `assets/images/bg/shop_header.png` | 1080×600 | 16:9 | paint | no | P1 | ShopScreen header | Header art for the Shop screen. |

## 6 · Board themes

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/images/themes/midnight/bg.png` | 1080×2340 | 9:16 | soft | no | P0 | ThemeRegistry 'midnight' | Default dark theme: background behind the puzzle board. |
| `assets/images/themes/paper/bg.png` | 1080×2340 | 9:16 | soft | no | P0 | ThemeRegistry 'paper' | Light theme: background behind the puzzle board. |
| `assets/images/themes/ocean/bg.png` | 1080×2340 | 9:16 | soft | no | P1 | ThemeRegistry 'ocean' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/forest/bg.png` | 1080×2340 | 9:16 | soft | no | P1 | ThemeRegistry 'forest' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/dusk/bg.png` | 1080×2340 | 9:16 | soft | no | P1 | ThemeRegistry 'dusk' | Unlockable theme: background behind the puzzle board. |
| `assets/images/themes/aurora/bg.png` | 1080×2340 | 9:16 | soft | no | P2 | ThemeRegistry 'aurora' | Supporter theme: background behind the puzzle board. |

## 7 · Chapters (Travel Journal)

| File | Final size | Generate as | Look | Cutout | Pri | Used in | What it is |
|---|---|---|---|---|---|---|---|
| `assets/images/chapters/ch01_postcard.png` | 1200×800 | 3:2 | paint | no | P0 | Journal chapter 1 | Chapter 1, Harbor Lighthouse: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch01_stamp.png` | 512×512 | 1:1 | 3d | yes | P0 | Journal chapter 1 badge | Chapter 1, Harbor Lighthouse: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch01_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 25 | Chapter 1, Harbor Lighthouse: shape mask for the landmark level (the board takes the shape of a lighthouse). |
| `assets/images/chapters/ch02_postcard.png` | 1200×800 | 3:2 | paint | no | P0 | Journal chapter 2 | Chapter 2, Cherry Blossom Garden: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch02_stamp.png` | 512×512 | 1:1 | 3d | yes | P0 | Journal chapter 2 badge | Chapter 2, Cherry Blossom Garden: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch02_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 50 | Chapter 2, Cherry Blossom Garden: shape mask for the landmark level (the board takes the shape of a five-petal cherry blossom flower). |
| `assets/images/chapters/ch03_postcard.png` | 1200×800 | 3:2 | paint | no | P0 | Journal chapter 3 | Chapter 3, Desert Oasis: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch03_stamp.png` | 512×512 | 1:1 | 3d | yes | P0 | Journal chapter 3 badge | Chapter 3, Desert Oasis: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch03_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 75 | Chapter 3, Desert Oasis: shape mask for the landmark level (the board takes the shape of a palm tree). |
| `assets/images/chapters/ch04_postcard.png` | 1200×800 | 3:2 | paint | no | P0 | Journal chapter 4 | Chapter 4, Alpine Village: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch04_stamp.png` | 512×512 | 1:1 | 3d | yes | P0 | Journal chapter 4 badge | Chapter 4, Alpine Village: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch04_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 100 | Chapter 4, Alpine Village: shape mask for the landmark level (the board takes the shape of a mountain peak with a snow cap). |
| `assets/images/chapters/ch05_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 5 | Chapter 5, Canal City: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch05_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 5 badge | Chapter 5, Canal City: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch05_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 125 | Chapter 5, Canal City: shape mask for the landmark level (the board takes the shape of a gondola boat). |
| `assets/images/chapters/ch06_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 6 | Chapter 6, Savannah Sunset: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch06_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 6 badge | Chapter 6, Savannah Sunset: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch06_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 150 | Chapter 6, Savannah Sunset: shape mask for the landmark level (the board takes the shape of a giraffe). |
| `assets/images/chapters/ch07_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 7 | Chapter 7, Northern Lights: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch07_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 7 badge | Chapter 7, Northern Lights: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch07_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 175 | Chapter 7, Northern Lights: shape mask for the landmark level (the board takes the shape of a pine tree). |
| `assets/images/chapters/ch08_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 8 | Chapter 8, Rainforest Falls: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch08_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 8 badge | Chapter 8, Rainforest Falls: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch08_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 200 | Chapter 8, Rainforest Falls: shape mask for the landmark level (the board takes the shape of a parrot). |
| `assets/images/chapters/ch09_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 9 | Chapter 9, Lantern Night Market: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch09_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 9 badge | Chapter 9, Lantern Night Market: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch09_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 225 | Chapter 9, Lantern Night Market: shape mask for the landmark level (the board takes the shape of a round paper lantern). |
| `assets/images/chapters/ch10_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 10 | Chapter 10, Windmill Fields: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch10_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 10 badge | Chapter 10, Windmill Fields: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch10_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 250 | Chapter 10, Windmill Fields: shape mask for the landmark level (the board takes the shape of a windmill). |
| `assets/images/chapters/ch11_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 11 | Chapter 11, Coral Reef: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch11_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 11 badge | Chapter 11, Coral Reef: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch11_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 275 | Chapter 11, Coral Reef: shape mask for the landmark level (the board takes the shape of a sea turtle). |
| `assets/images/chapters/ch12_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 12 | Chapter 12, Balloon Valley: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch12_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 12 badge | Chapter 12, Balloon Valley: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch12_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 300 | Chapter 12, Balloon Valley: shape mask for the landmark level (the board takes the shape of a hot air balloon). |
| `assets/images/chapters/ch13_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 13 | Chapter 13, Pyramid Dunes: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch13_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 13 badge | Chapter 13, Pyramid Dunes: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch13_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 325 | Chapter 13, Pyramid Dunes: shape mask for the landmark level (the board takes the shape of a pyramid with a small sun above). |
| `assets/images/chapters/ch14_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 14 | Chapter 14, Ice Fjord: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch14_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 14 badge | Chapter 14, Ice Fjord: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch14_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 350 | Chapter 14, Ice Fjord: shape mask for the landmark level (the board takes the shape of a whale tail). |
| `assets/images/chapters/ch15_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 15 | Chapter 15, Volcano Island: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch15_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 15 badge | Chapter 15, Volcano Island: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch15_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 375 | Chapter 15, Volcano Island: shape mask for the landmark level (the board takes the shape of a volcano). |
| `assets/images/chapters/ch16_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 16 | Chapter 16, Bamboo Forest: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch16_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 16 badge | Chapter 16, Bamboo Forest: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch16_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 400 | Chapter 16, Bamboo Forest: shape mask for the landmark level (the board takes the shape of a panda head). |
| `assets/images/chapters/ch17_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 17 | Chapter 17, Castle on the Hill: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch17_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 17 badge | Chapter 17, Castle on the Hill: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch17_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 425 | Chapter 17, Castle on the Hill: shape mask for the landmark level (the board takes the shape of a castle with three towers). |
| `assets/images/chapters/ch18_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 18 | Chapter 18, Moonlit Temple: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch18_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 18 badge | Chapter 18, Moonlit Temple: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch18_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 450 | Chapter 18, Moonlit Temple: shape mask for the landmark level (the board takes the shape of a crescent moon). |
| `assets/images/chapters/ch19_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 19 | Chapter 19, Star Observatory: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch19_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 19 badge | Chapter 19, Star Observatory: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch19_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 475 | Chapter 19, Star Observatory: shape mask for the landmark level (the board takes the shape of a rocket). |
| `assets/images/chapters/ch20_postcard.png` | 1200×800 | 3:2 | paint | no | P1 | Journal chapter 20 | Chapter 20, Sky Islands: postcard painting revealed piece by piece over the chapter's 25 levels. |
| `assets/images/chapters/ch20_stamp.png` | 512×512 | 1:1 | 3d | yes | P1 | Journal chapter 20 badge | Chapter 20, Sky Islands: postage-stamp badge for completing the chapter. |
| `assets/images/masks/ch20_mask.png` | 512×512 | 1:1 | flat | no | P1 | tool/level_gen landmark level 500 | Chapter 20, Sky Islands: shape mask for the landmark level (the board takes the shape of a small bird in flight with wings spread). |

## 8 · Audio (not images; use an AI sound/music generator or CC0 libraries)

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

Every full prompt is in `assets_manifest.json` (`images[].prompt`) and on the Asset Brief page with a copy button.

