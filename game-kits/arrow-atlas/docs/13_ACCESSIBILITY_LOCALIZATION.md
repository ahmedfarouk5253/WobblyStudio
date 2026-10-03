# 13: Accessibility and Localization

## 1. Accessibility (these are real selling points; mention them in the store listing)
- **Touch:**
  - Smart Touch (doc 02 §6). Every button ≥ 48 dp.
  - Arrow hit radius ≥ 0.6 cell, with a fixed minimum of 14 dp.
  - The Precision Loupe kicks in whenever cells are under 30 dp.
  - Left-handed HUD.
- **Vision:**
  - Arrow thickness S/M/L.
  - 4 arrow palettes, including **Colorblind-safe (Okabe-Ito)** and **Mono high-contrast**.
  - Dark and light themes.
  - Dot grid always on.
  - Gates, keys and portals carry **shapes** as well as colors.
  - UI text scales with system font size up to 1.3×; the layout must not break (test at 1.3×).
- **Motion:** "Reduce motion" setting (doc 07 §11). Also respect the Android "Remove animations" setting (`MediaQuery.disableAnimations`).
- **Pressure:** Zen mode, no timers by default.
- **Screen readers:** full TalkBack support for menus, settings and the shop (semantic labels on every icon button). The **board** itself isn't screen-reader playable in v1. Give the board a semantic summary instead: "Level 37. 42 arrows left. 2 hearts."
- **Contrast:** text ≥ 4.5:1 against its background; UI icons ≥ 3:1. Checked by `test/palette_contrast_test.dart`.

## 2. Localization
- Flutter `gen-l10n` with ARB files in `lib/l10n/`. English (`app_en.arb`) is the source.
- **Launch languages (12):**
  - English
  - Spanish (LatAm)
  - Portuguese (Brazil)
  - French
  - German
  - Italian
  - Turkish
  - Indonesian
  - Russian
  - Japanese
  - Korean
  - Hindi
- Translation: Claude Code produces the first pass from `app_en.arb` (every string has a `@description` with context). For Japanese, Korean and German, ask a native speaker or a community volunteer to review the store listing, since it drives installs.
- Keep strings short; Pip's lines are ≤ 30 chars in English. German runs about 30% longer, so check that buttons can grow.
- Plurals and numbers use ICU syntax: `{count, plural, one{1 arrow left} other{{count} arrows left}}`. Use `NumberFormat` for coins, and store-provided price strings for prices.
- Chapter names and place facts are in ARB too (`chapter_01_name`, `chapter_01_fact`).
- Fonts: Baloo 2 + Nunito cover Latin, Cyrillic (Nunito), Vietnamese and Devanagari (Baloo 2). For **Japanese and Korean**, fall back to `Noto Sans JP/KR` subsets bundled for UI only (≈ 2–4 MB each). Alternatively, rely on system fonts for CJK to keep the app small. That's the recommended v1 choice.
- RTL (Arabic/Hebrew) isn't in v1, but don't hard-code left/right in layouts (use `EdgeInsetsDirectional`), so it can be added later.
