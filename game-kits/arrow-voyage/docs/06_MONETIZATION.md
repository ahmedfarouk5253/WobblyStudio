# 06: Monetization

Model: **ad-first hybrid.** Most revenue comes from interstitial and rewarded ads from players who never pay. The main purchase is **Remove Ads**. Everything is tuned to keep the rating ≥ 4.7, because rating drives organic installs, and organic installs are the business for a solo studio.

## 1. The Fair Ads Pledge (shown in-game: Settings → "Our ad promise", and on the store listing)
1. No ads in your first 15 levels.
2. Never an ad in the middle of a puzzle, and never a banner over the board.
3. Never an ad right after you run out of hearts.
4. Rewarded videos are always your choice, and they always give the reward.
5. Remove Ads removes every forced ad, forever, on all your devices with the same Google account.
6. We'll never make free features paid, or cut rewards in an update.

## 2. Interstitials (AdMob)
They show **only** at the "Next" transition between two levels, after the results card is dismissed, and only when all of these are true:

| Rule | Default | Remote Config key |
|---|---|---|
| Player has cleared at least N Journey levels in total | 15 | `ad_inter_min_level` |
| At least N levels completed since the last interstitial | 3 | `ad_inter_every_levels` |
| At least S seconds since the last interstitial **or rewarded** ad | 180 | `ad_inter_min_gap_s` |
| At least S seconds since app start / resume | 60 | `ad_inter_session_grace_s` |
| Daily cap | 12 | `ad_inter_daily_cap` |
| The level just cleared wasn't a Landmark / chapter-complete moment | true | `ad_inter_skip_on_celebration` |
| The player didn't just buy something / watch a rewarded ad in this transition | true | n/a |
| The player didn't run out of hearts during this level | true | `ad_inter_skip_after_fail` |
| Not owned: Remove Ads | n/a | n/a |

- Also allowed: when leaving Daily or Endless results to go Home, using the same counters.
- Preload the next interstitial right after one shows. If it isn't loaded, **skip it**: never wait or show a spinner.
- Before the ad, a 0.6 s "Ad" chip fades in so it never feels like a crash. After the ad, the next level loads immediately.

## 3. Rewarded ads (all optional)
| Placement | Reward | Limit/day |
|---|---|---|
| Out of hearts → Keep going | +1 heart | unlimited |
| Results → Double coins | ×2 level coins | 10 |
| Hint button with 0 hints, or stall helper | +1 hint | 10 |
| Daily gift → Double it | ×2 gift | 1 |
| Missed streak (no freeze) → Repair streak | restores the streak | 1 per break, within 48 h |
| Theme try-on | Use a locked theme for 3 levels | 3 |

- **Always pays:** if the ad fails to load within 4 s or errors, grant the reward anyway, show "Thanks for your patience!", and count it as `rewarded_fallback` (max 3 a day; after that the button shows "No video available right now, try later").
- Rewarded ads count as "an ad" for the interstitial gap timer (rule 3 above).
- Premium (Remove Ads) players never need to watch them for "Keep going" and "Hint": they get **3 free uses a day** of each. Other rewarded placements stay available and optional.

## 4. Banners
None in v1. A `ad_banner_home_enabled` flag exists (default **false**) to A/B test a small banner on the **Home screen only**, never in a level, if revenue requires it later.

## 5. In-app products (Google Play Billing via `in_app_purchase`)
| Product ID | Type | Price (USD; others auto) | Contents |
|---|---|---|---|
| `remove_ads` | Non-consumable | **$3.99** | No interstitials forever; premium perks (3 free heart refills + 3 free hints a day); "Thank you" badge on profile |
| `supporter_pack` | Non-consumable | $9.99 | Everything in remove_ads + Aurora theme + Paper-plane arrow style + 3,000 coins + 10 hints. If remove_ads is already owned, show a $6.99 upgrade (`supporter_upgrade`) |
| `coins_small` | Consumable | $0.99 | 600 coins |
| `coins_medium` | Consumable | $2.99 | 2,000 coins + 3 hints |
| `coins_large` | Consumable | $6.99 | 5,500 coins + 8 hints |
| `coins_huge` | Consumable | $14.99 | 13,000 coins + 20 hints |
| `streak_freeze_3` | Consumable | $1.49 | 3 streak freezes (hold cap raised to 5 while owned) |

- No packs above $14.99. No time-limited offers in v1, and no countdown timers ever.
- **Price test:** Remove Ads at $3.99 vs $4.99 (Play Console price experiments, after day 30).
- **Starter offer** (one time, after level 30, when the player has watched ≥ 5 rewarded ads): "Starter Bundle: Remove Ads + 1,000 coins, $3.99". This is the same price as Remove Ads alone, so it feels generous. ID `starter_bundle`, non-consumable, grants `remove_ads`. It shows once as a card on the results screen after a **win** (never after a failure), and then lives in the shop for 72 h.
- **Restore:** `restorePurchases()` on first launch and from Settings. Non-consumables are tied to the Google account.
- **Server verification:** none in v1 (client-side, accepted risk for a small game). Acknowledge every purchase within 3 days (Billing requirement); the plugin does this when `completePurchase` is called.
- After buying Remove Ads, the Remove Ads button and the shop card disappear and an "Ads removed ✓" row appears in Settings.

## 6. Shop screen layout (top to bottom)
1. Header art (`shop_header`) with the coin balance.
2. **Remove Ads** card. Big, honest copy: "No more ads between levels. Forever. $3.99."
3. Supporter Pack card.
4. Coin packs: 4 cards in a row on tablets, a 2×2 grid on phones.
5. Streak freezes.
6. Free section: "Watch a video: +1 hint" (if under the daily limit), "Daily gift" status.
7. Restore purchases (text button) and the Fair Ads Pledge link.

## 7. Revenue expectations (planning only)
Rough blended ARPDAU for ad-first puzzle games is $0.03–0.06. This varies hugely by country mix: US/EU players earn 5–10× more per ad than IN/BR/ID. Example: 5,000 DAU × $0.04 ≈ $200/day ≈ $6K/month before taxes and Google's fees. Treat this as a target to test against, not a promise. Organic installs alone rarely reach 5K DAU; plan a small UA test (doc 15).

## 8. Ad mediation and setup
- v1: AdMob only, plus **AdMob bidding** (Meta Audience Network, AppLovin, Unity, Liftoff, all via AdMob mediation adapters) added after 1K DAU.
- Ad units (create in AdMob, put the IDs in `--dart-define`): `ARROW_INTER`, `ARROW_REWARDED`. Use test IDs in debug.
- Content rating for ads: **max ad content rating G or PG**. Block categories: gambling, dating, politics, "real money games".
- `app-ads.txt`: the Wobbly Studio website root already hosts one. Add the AdMob line with the real `pub-` ID there (doc 15).
