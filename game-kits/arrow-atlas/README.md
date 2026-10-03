# Arrow Atlas: Build Kit

**Arrow Atlas** is the working title for Wobbly Studio's arrow-escape puzzle game for Android, built with Flutter.

This folder has everything Claude Code needs to build the full game, from an empty folder to a Play Store release. Every rule, number, screen, asset and service is written down, so Claude Code doesn't have to guess.

> **Rename the game if you like.** Before you publish, search Google Play for "Arrow Atlas" to check the name is free. To rename, change `GAME_NAME` and `PACKAGE_ID` in `CLAUDE.md`, then ask Claude Code to update everything else.

---

## What's inside

| File | What it's for |
|---|---|
| `CLAUDE.md` | **Project memory.** Claude Code reads it automatically every session: stack, rules, commands, invariants. |
| `assets_manifest.json` | The machine-readable list of every image and sound: path, size, alpha, prompt. |
| `asset-brief.html` | Offline copy of the Asset Brief web page. Open it in any browser. Online version: https://claude.ai/artifact/51mjGuKF2M9XJ2pqraefJL |
| `docs/00_MARKET_RESEARCH.md` | Research on the genre: who wins and loses, what players love and hate, sources. |
| `docs/01_PRODUCT_VISION.md` | Pillars, positioning, target players, the 12 differentiators, the name. |
| `docs/02_GAME_DESIGN.md` | Core rules, controls, hearts, hints, results, modes. |
| `docs/03_MECHANICS.md` | Every board element, with exact rules and the Monotonic Rule. |
| `docs/04_LEVEL_SYSTEM.md` | Level JSON format, generator algorithm, solver, difficulty score, curve, daily puzzle. |
| `docs/05_META_PROGRESSION.md` | Atlas postcards, coins, themes, streaks, achievements. |
| `docs/06_MONETIZATION.md` | Fair-ads policy, ad rules, IAP catalog, remote config defaults. |
| `docs/07_UX_UI_SPEC.md` | Every screen, flow, layout, animation and tutorial step. |
| `docs/08_ART_AUDIO_DIRECTION.md` | Palettes, how arrows are drawn, motion, sound and haptics. |
| `docs/09_TECH_ARCHITECTURE.md` | Flutter project structure, packages, state, renderer, input, persistence. |
| `docs/10_SERVICES_INTEGRATION.md` | AdMob + UMP consent, Billing, Play Games, Firebase, notifications. |
| `docs/11_ASSET_MANIFEST.md` | Human-readable asset list and how assets are wired in. |
| `docs/12_ANALYTICS_LIVEOPS.md` | Events, KPIs and targets, A/B tests, live-ops calendar. |
| `docs/13_ACCESSIBILITY_LOCALIZATION.md` | Colorblind palettes, touch sizes, languages. |
| `docs/14_QA_TESTING.md` | Test plan, device matrix, acceptance checklist. |
| `docs/15_RELEASE_PLAYSTORE.md` | Play Console setup, closed testing, data safety, store listing copy, ASO. |
| `docs/16_BUILD_PLAN.md` | **The step-by-step build plan, with copy-paste prompts for Claude Code.** |
| `docs/17_ROADMAP.md` | Updates after launch. |

---

## How to use this with Claude Code

1. Create an empty folder, for example `arrow-atlas/`. Turn it into a git repo and push it to GitHub.
2. Copy **everything in this zip** into the root of that folder. `CLAUDE.md` must be at the root.
3. Open Claude Code in that folder.
4. Open `docs/16_BUILD_PLAN.md`. Paste the prompts in order, one milestone at a time (M0 → M12). After each one, check the acceptance list before you start the next.
5. Generate the art in parallel. Open the **Asset Brief** web page (the link you were given) or `docs/11_ASSET_MANIFEST.md`. Generate each image with the exact file name and size, then drop the folders into the project root. Tell Claude Code: *"New assets are in, run the asset check and wire them in."*

The game runs on **auto-generated placeholder art** from M1 onward, so you never have to wait for art before you can play.

## The one-paragraph pitch

> A calm, fair arrow-escape puzzle. Tap a bendy arrow and it slides off the board if its path is clear. Clear every arrow to finish the level. Arrow Atlas keeps the simple loop that took the genre to #1 in the world, and fixes what players hate about the current leaders: mis-taps on big boards, ads after every level, Remove Ads that doesn't remove ads, punishing hearts, and repetitive "just more arrows" difficulty. Each cleared chapter reveals a hand-painted travel postcard in your Atlas, and new board elements arrive every 25 levels, so level 400 doesn't play like level 4.
