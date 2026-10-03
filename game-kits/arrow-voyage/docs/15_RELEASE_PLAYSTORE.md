# 15: Release on Google Play, ASO and Launch

## 1. Play Console setup checklist
- [ ] Create the app "Arrow Voyage: Escape Puzzle". Game → Puzzle. Free. Contains ads: **Yes**.
- [ ] **Play App Signing:** on. Upload the AAB signed with your upload key (keep the `.jks` + passwords in a password manager **and** an offline backup).
- [ ] **App content:**
  - privacy policy URL (Wobbly Studio site)
  - ads declaration = yes
  - app access = no restrictions
  - content rating questionnaire (IARC) → expected "Everyone / PEGI 3"
  - target audience = **18+, or 13+** (picking ages under 13 triggers Families policy requirements and ad restrictions; this game is made for adults)
  - news = no
  - data safety (doc 10 §8)
  - government apps = no
  - financial features = none
  - health = none
- [ ] **Target API level:** must meet Play's current requirement. Check *Policy → App content → Target API level* before the first upload.
- [ ] **Store settings:** category Puzzle; tags (Logic, Brain teaser, Casual, Relaxing, Offline); contact email; website.
- [ ] **In-app products** created and activated (doc 06).
- [ ] **Play Games Services** configured and **published** (achievements, leaderboards, saved games).

## 2. Testing tracks
1. **Internal testing** (you + up to 100 testers, instant): every build from M8 onward.
2. **Closed testing:** *Personal developer accounts created after Nov 2023 must run a closed test with at least 12 testers opted in for 14 consecutive days* before applying for production access. Check Play Console's current requirement; if your account already passed this for an earlier game, it may not apply again.
   - Recruit testers: friends and family, r/AndroidClosedTesting, r/TestersCommunity, and testers of your other games.
   - Ask them to actually play and send feedback. Google looks at engagement, and the feedback is gold.
3. **Production:** staged rollout 10% → 50% → 100% over about 5 days, watching Android vitals (ANR < 0.47%, crash < 1.09% user-perceived thresholds).

## 3. Website updates (WobblyStudio repo)
- `games/arrow-voyage/index.html` + `privacy-policy.html` (copy the structure of the other games).
- Root `app-ads.txt`: make sure it contains `google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0` with your real AdMob publisher ID.
- Set the **Developer website** in Play Console to the site root, so the crawler finds `app-ads.txt`.

## 4. Store listing (English source; translate to the 12 languages)
**Title (≤ 30):** `Arrow Voyage: Escape Puzzle`

**Short description (≤ 80):** `Calm arrow puzzle. Tap arrows to clear the board and travel the world.`

**Full description:**
```
Tap an arrow. If its path is clear, it flies away. Clear the board to win.

Arrow Voyage is the calm, fair arrow escape puzzle. Easy to learn, surprisingly deep, and made to be relaxing.

WHY PLAYERS LOVE IT
• Smart Touch – taps go to the arrow you meant. On huge boards, a magnifier lets you aim before you fire.
• Second Chance – the first slip in every level can be undone for free.
• Zen Mode – no hearts, no pressure. Free for everyone.
• No timers. Play offline, anywhere.
• Dark & light themes, colorblind-safe colors, thicker arrows – all free, forever.

ALWAYS SOMETHING NEW
• 500 levels across 20 hand-painted destinations
• New board elements every chapter: rocks, locks, ice, mirrors, keys & gates, portals, one-way gates
• Landmark levels shaped like the places you visit
• Daily Puzzle in 3 sizes, streaks and monthly trophies
• Beyond the Map: endless puzzles from Easy to Nightmare

FILL YOUR TRAVEL JOURNAL
Every level you clear paints part of a travel postcard. Collect all 20 stamps with Pip, the little explorer bird.

OUR AD PROMISE
• No ads in your first 15 levels
• Never an ad in the middle of a puzzle, no banners over the board
• Never an ad right after you fail
• Videos are always optional – and always give the reward
• Remove Ads removes every forced ad, forever

Made with care by Wobbly Studio, a one-person indie studio.
```

**Screenshots (8, 1080×1920):** gameplay is captured from the dev flavor with `integration_test` + `tool/compose_screenshots.dart`, composited onto `store/screenshot_bg_0N.png` with captions:
1. "Tap. Slide. Clear the board." (a satisfying mid-level board, one arrow flying)
2. "Paint the world, one level at a time" (Travel Journal postcard reveal)
3. "Never mis-tap again" (Precision Loupe on a big board)
4. "New tricks every chapter" (mirrors + portals)
5. "Daily puzzles in 3 sizes" (calendar + streak)
6. "Zen mode: no hearts, no pressure"
7. "Dark, light & colorblind-friendly"
8. "Fair ads. Our promise." (the pledge card)

**Feature graphic:** `store/feature_graphic_1024x500.png`, with the logo composited by the script.

**Promo video (optional, high impact):** 20–30 s screen recording: a fast satisfying clear → a big board with the loupe → a postcard reveal → the "Fair ads" end card. No music with lyrics. Upload to YouTube (unlisted is fine) and link it in the listing.

## 5. ASO keywords (put them naturally in the title, short and full description; no keyword stuffing)
Primary: arrow puzzle, arrows, arrow escape, tap away, puzzle escape, logic puzzle, brain teaser.
Secondary: relaxing puzzle, offline puzzle, brain game, IQ puzzle, maze, daily puzzle, zen puzzle.
Don't use competitors' brand names ("Lessmore", "Amaze GO", "Tap Away" as a brand) in the listing; that violates the metadata policy.

## 6. Launch and growth plan (solo budget)
1. **Weeks −2 to 0 (closed test):** fix everything testers report; get 20+ early reviews ready.
2. **Launch week:** staged rollout; reply to every review; post a 15-s clip on TikTok, Shorts, Reels and r/AndroidGaming (rule-compliant self-promo threads); add the game to the Wobbly Studio site; cross-promote from your other 5 games via a "More games" button (house ads are free).
3. **Short-video content engine** (this format sells itself on video), 3–5 clips a week:
   - "Only 2% can clear this board" plus the reveal
   - an ASMR clearing compilation (the whoosh sounds)
   - a fail clip (a wrong tap bounces) with "where would you tap?"
   - a Nightmare board time-lapse
   - a postcard reveal
4. **Small paid UA test** (after D7 ≥ 15% and rating ≥ 4.6): Google App Campaigns (tCPI) at $10–20/day for 2 weeks, US + Tier-2 (BR, MX, TR, ID) split. Measure the D7 ROAS from AdMob revenue linked in Firebase. Scale only if the payback curve looks reachable within 60–90 days.
5. **Featuring:** apply for Google Play's indie programs and fill in the Play Console "Featuring nomination" form 6–8 weeks before big updates.
