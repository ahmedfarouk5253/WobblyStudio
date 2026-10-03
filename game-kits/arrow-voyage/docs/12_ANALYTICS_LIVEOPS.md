# 12: Analytics, KPIs and Live Ops

## 1. Events (Firebase Analytics, names ≤ 40 chars, snake_case)
| Event | Params | When |
|---|---|---|
| `tutorial_step` | step (1–6) | Each tutorial bubble completed |
| `level_start` | level_id, mode (journey/daily/endless), tier, chapter, arrows, score, zen | Level loaded |
| `level_complete` | level_id, mode, duration_s, hearts_lost, hints_used, second_chance_used, perfect, blocked_taps, ambiguous_guards, loupe_uses, zen | Last arrow exits |
| `level_fail` | level_id, mode, duration_s, arrows_left | Hearts reach 0 |
| `level_continue` | level_id, method (ad/coins/premium/zen/restart) | Choice on the out-of-hearts sheet |
| `level_quit` | level_id, arrows_left, duration_s | Left mid-level |
| `hint_used` | level_id, source (inventory/ad/premium) | |
| `stall_helper_shown` | level_id | |
| `ad_inter_shown` | placement, levels_since_last, session_level_count | |
| `ad_inter_skipped` | reason (cooldown/cap/not_loaded/celebration/after_fail/premium) | Sampled at 10% |
| `ad_rewarded_offer` | placement | Button shown |
| `ad_rewarded_complete` | placement, fallback (bool) | Reward granted |
| `iap_view` | product_id, source | Shop/product card shown |
| `iap_purchase` | product_id, price_micros, currency | Purchase granted |
| `daily_complete` | size, duration_s, streak | |
| `streak_freeze_used` | streak | |
| `postcard_complete` | chapter | |
| `theme_changed` | theme, palette, style | |
| `setting_changed` | key, value | |
| `share` | content (postcard/replay) | v1.1 |

User properties: `install_version`, `premium` (bool), `palette`, `zen_default`, `max_level_bucket` (0-25, 26-50, …), `country_tier` (from locale).

## 2. KPI dashboard (check weekly)
| KPI | Target | Investigate if… |
|---|---|---|
| Tutorial completion (step 6) | ≥ 92% | < 85%: onboarding friction |
| Level 1 → level 15 funnel | ≥ 70% of installs reach L15 | Find the level with the biggest drop |
| Median duration, L1–15 | 10–25 s | > 30 s: too hard early |
| Fail rate per level | < 8% (Normal), < 20% (Hard), < 30% (Super Hard) | Above: lower the score / reroll |
| Blocked taps per level | watch the trend | A spike after a release suggests a touch regression |
| `ambiguous_guards` per 100 taps | < 3 | Above: improve hit-testing |
| D1 / D7 / D30 | 45 / 18 / 8% | |
| Interstitials per DAU (non-premium) | 4–8 | > 10: pacing too aggressive |
| Rewarded per DAU | ≥ 1.5 | Low: offers not appealing or not visible |
| Remove Ads conversion (of D7 users) | ≥ 2% | |
| Crash-free users | ≥ 99.5% | |
| Rating (last 30 days) | ≥ 4.7 | Read every 1–3★ review weekly |

## 3. Difficulty re-fit (monthly)
Export `level_complete` and `level_fail` to BigQuery (free Firebase link). For each level, compute the median duration and fail rate. Fit the difficulty-score weights (doc 04 §5) with a simple linear regression against `log(median_duration)` + fail rate. Then reroll outlier levels in **future** chapters only. Never change a level a player has already seen, unless it's broken.

## 4. A/B tests (Firebase Remote Config + A/B Testing)
Run one at a time, each for 14 days, with retention D7 + ad revenue as the goals.
1. `ad_inter_every_levels`: 3 vs 4
2. `ad_inter_min_level`: 15 vs 20
3. `heartsPerLevel`: 3 vs 4
4. `second_chance_enabled`: on vs off (expect ON to win on retention)
5. Remove Ads price $3.99 vs $4.99 (Play price experiment)
6. `stall_helper_seconds`: 30 vs 45
7. First Hard level at slot 5 vs slot 7

Store listing experiments (Play Console):
- icon A (arrow knot) vs B (Pip)
- feature graphic
- the first screenshot caption ("Calm. Fair. No ad spam." vs "Can you clear the board?")

## 5. Live-ops calendar (light, solo-dev friendly)
| Cadence | What |
|---|---|
| Daily | The daily puzzle (automatic) |
| Monthly | Monthly trophy design (automatic: the month color) |
| Every 4–6 weeks | **Content drop**: 2 new chapters (50 levels) + 2 postcards. Generated + validated, so it's mostly art work. |
| Seasonal (Halloween, Winter, Lunar New Year, Spring) | A 2-week **seasonal postcard event**: 15 themed levels (pumpkin/snowflake-shaped landmark boards), a limited sticker, seasonal board theme free during the event. All in-app, no server: event dates come from Remote Config `event_*` keys, levels ship in the update before. |
| Ongoing | Reply to every review ≤ 3★ within 48 h with a real answer (Vector Labz won ratings back this way; template replies don't work) |

## 6. Feedback loop
- Settings → "Help & feedback" opens email to support with the app version, device and current level filled in.
- A **"Report this level"** item in the pause menu logs `level_report {level_id, reason}`. Reasons: "Feels impossible", "Tap went to wrong arrow", "Bug", "Too easy". It's a cheap, direct signal.
