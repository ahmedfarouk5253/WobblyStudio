# 05: Meta and Progression

The meta layer is deliberately **light and tied to the puzzle**. Every level you clear visibly paints the world; there's no separate building game.

## 1. The Travel Journal (main meta)
- **20 chapters**, each one a destination (names and art in doc 11): Harbor Lighthouse, Cherry Blossom Garden, Desert Oasis, Alpine Village, Canal City, Savannah Sunset, Northern Lights, Rainforest Falls, Lantern Night Market, Windmill Fields, Coral Reef, Balloon Valley, Pyramid Dunes, Ice Fjord, Volcano Island, Bamboo Forest, Castle on the Hill, Moonlit Temple, Star Observatory, Sky Islands.
- Each chapter has a **postcard** (1200×800 illustration) split into **25 reveal tiles** (a 5×5 jittered Voronoi pattern, generated in code from the chapter number so it's stable).
- **Clearing a level** reveals one tile with a watercolor "paint bloom" animation, 0.9 s. The tile index follows a fixed shuffled order per chapter, ending at the center tile.
- **Clearing all 25** completes the postcard: Pip stamps it (`pip_stamp`, `stamp` sound), the chapter's **stamp badge** is awarded, +2 hints, +200 coins, and the next chapter opens with a "New destination" card.
- **Unrevealed tiles** are drawn as a soft desaturated, blurred version of the image, so players can see the picture coming. This is the **zoom-reveal** beat that Amaze GO! does well.
- The **Travel Journal screen** shows all postcards as a journal spread: locked chapters are silhouettes with a padlock, completed ones are stamped. Tap a postcard to view it full screen. In v1.1 you can also set it as wallpaper or share it.
- **Crowns:** each level can earn a crown for a perfect clear. A chapter with all 25 crowns gets a gold frame on its postcard (cosmetic, for completionists).

## 2. Coins (only soft currency, no hard currency)
| Source | Amount |
|---|---|
| Level clear | 10 / 20 / 30 / 50 (doc 02) |
| Perfect bonus | +50% |
| Chapter complete | 200 |
| Daily puzzle | Small 20, Medium 40, Big 60 |
| Daily gift (open the app once a day) | 30 (day 1) → 100 (day 7), resets after a missed day |
| Achievements | 50–300 |
| Rewarded "double coins" | ×2 level coins |

| Sink | Price |
|---|---|
| Hint | 150 |
| +1 heart (out of hearts) | 100 |
| Streak freeze | 400 (hold max 2) |
| Themes | 1,500 – 3,000 |
| Arrow styles | 1,000 – 2,500 |

**Balance target:** a free player earns about 1 hint's worth of coins every 8–10 levels and unlocks the first theme around level 60–80. Check with `test/economy_sim_test.dart`, which simulates 30 days at 20 levels a day.

## 3. Hints inventory
- Start: 3. Max 99.
- Earned from chapter completion (+2), daily gift day 3 and day 7 (+1 each), achievements, and rewarded ads (+1 each, max 10 a day).
- Purchases: Hint packs are inside the coin packs, so there's no separate SKU.

## 4. Daily streak
- The streak goes up by 1 on each UTC day you solve **any** daily size, and appears as a flame counter on Home.
- **Streak freeze:** if a day is missed and you hold a freeze, it's consumed automatically and the streak survives. A "Pip slept on it" card explains what happened. Hold max 2. Get them from coins (400), the daily gift on day 7 (1), or IAP (doc 06).
- **Monthly trophy:** solve ≥ 20 dailies in a calendar month to earn that month's trophy (12 designs drawn in code: trophy icon + month color). The trophy shelf lives on the Daily screen.
- Calendar stickers: 1 size = small dot, 2 = check, 3 = gold sticker.

## 5. Themes and arrow styles (cosmetic only)
| Theme | Unlock |
|---|---|
| Midnight (dark, default) | Free |
| Paper (light) | Free |
| Ocean | Chapter 3 complete, or 1,500 coins |
| Forest | Chapter 6 complete, or 2,000 coins |
| Dusk | 30-day streak, or 2,500 coins |
| Aurora | Supporter Pack, or 3,000 coins |

| Arrow style | Unlock |
|---|---|
| Classic (rounded) | Free |
| Bold (thick) | Free |
| Ribbon (flat ribbon with a slight twist shading) | 1,000 coins |
| Neon (glow) | 2,000 coins |
| Chalk (sketchy texture, drawn in code) | 1,500 coins |
| Paper plane (head drawn as a tiny plane) | 2,500 coins |

**The arrow *palette* is a setting, not an unlock.** Vivid, Soft, Colorblind-safe and Mono are always free (it's an accessibility feature).

## 6. Achievements (also mirrored to Play Games)
| ID | Name | Condition | Reward |
|---|---|---|---|
| first_flight | First Flight | Clear level 1 | 50 |
| perfect_10 | Steady Hand | 10 perfect clears | 100 |
| perfect_100 | Surgeon | 100 perfect clears | 300 |
| no_hint_ch | Self-Made | Finish a chapter with 0 hints | 150 |
| postcard_1 | Wish You Were Here | Complete the first postcard | 100 |
| postcard_10 | Globetrotter | 10 postcards | 300 |
| postcard_20 | World Traveler | All 20 postcards | 500 + Aurora theme |
| streak_7 | Habit Bird | 7-day streak | 100 |
| streak_30 | Early Bird | 30-day streak | 300 |
| daily_trophy | Monthly Champion | First monthly trophy | 200 |
| nightmare_10 | Nightmare Walker | 10 Nightmare clears | 300 |
| speed_15 | Quick Wing | Clear any Normal level in under 15 s | 50 |
| zen_50 | Inner Calm | 50 levels in Zen mode | 100 |
| long_arrow | Snake Charmer | Clear an arrow 40+ cells long | 100 |

## 7. Unlock timeline (what the player sees and when)
| Level | Unlock / moment |
|---|---|
| 1–3 | Tutorial; Pip says hi |
| 5 | First Hard level (signalled) |
| 8 | Daily Puzzle unlocks (Pip shows the calendar) |
| 10 | Settings hint: "Dark/light themes and Zen mode are in Settings" |
| 12 | Daily gift appears |
| 15 | First possible interstitial (doc 06) |
| 20 | Rating prompt (only if the player has ≥ 3 perfect clears and hasn't lost hearts in the last 3 levels) |
| 25 | First postcard complete + chapter 2 + Rocks |
| 26–40 | Shop visible (it's in the menu from the start, but highlighted here) |
| 100 | Beyond the Map (Endless) unlocks |

## 8. Notifications (opt-in, max 1 a day)
- A daily puzzle reminder at the player's chosen time (default: the hour they usually play, learned from the first 3 sessions; fallback 19:00 local).
- Message variety, max 40 chars: "Today's puzzle is ready 🧭", "Pip found a new puzzle", "Your 12-day streak is waiting". Never guilt ("You'll lose everything!").
- No notifications if the player hasn't opened the app in 14 days. Stop completely after 30 days.
