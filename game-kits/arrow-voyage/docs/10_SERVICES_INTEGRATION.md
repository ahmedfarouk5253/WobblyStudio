# 10: Services Integration

Wobbly Studio's other games already use AdMob, Play Games Services (cloud save) and Firebase (Analytics + Crashlytics). Reuse the same accounts.

## 1. Startup order (`main.dart`)
1. `WidgetsFlutterBinding.ensureInitialized()`
2. `Firebase.initializeApp()`. Crashlytics: `FlutterError.onError` + `PlatformDispatcher.onError` + `runZonedGuarded`.
3. Load `save.json` (the fallback chain from doc 09 §7).
4. Remote Config: activate the previously fetched values, start a new fetch in the background.
5. `runApp`. The first frame must appear in < 1.5 s on a mid-range device.
6. **After** the first frame:
   1. UMP consent flow
   2. then `MobileAds.instance.initialize()`
   3. then preload the interstitial + rewarded ads (only if `ConsentInformation.canRequestAds()`)
7. IAP: listen to `purchaseStream` from app start (required, so pending purchases aren't missed), then call `restorePurchases()` on the first launch.
8. Play Games: silent sign-in (it never blocks gameplay; if it fails, try again later from Settings → "Connect Play Games").

## 2. AdMob + UMP consent
- **UMP** (in `google_mobile_ads`):
  1. `ConsentInformation.instance.requestConsentInfoUpdate(params, …)`
  2. if a form is available and required, `ConsentForm.loadAndShowConsentFormIfRequired`
- A "Privacy options" button in Settings calls `ConsentForm.showPrivacyOptionsForm` when `privacyOptionsRequirementStatus == required`.
- Ads are only requested when `canRequestAds()` is true.
- In AdMob, create the GDPR message (EEA/UK/CH) and the US state regulations message.
- **Test devices:** add your device's hashed ID to `RequestConfiguration.testDeviceIds` in the dev flavor. Use Google's sample test ad unit IDs during development.
- **`AdsService` API:**
  ```dart
  Future<void> preload();
  bool get interstitialReady;
  Future<void> maybeShowInterstitial(AdContext ctx); // calls AdPolicy.shouldShow(ctx, counters, entitlements, rc)
  Future<RewardResult> showRewarded(RewardPlacement p); // returns granted / fallbackGranted / unavailable
  ```
- `AdPolicy` is **pure Dart** and unit-tested against every rule in doc 06 §2.
- Pause the music while a full-screen ad is open, and resume afterwards.

## 3. In-app purchases
- Product IDs are in doc 06 §5. Create them in Play Console → Monetize → Products → In-app products. Fill in the store listing for each in the main languages.
- **`IapService`:**
  - `queryProductDetails()` on start, caching prices
  - `buy(productId)`
  - listen to `purchaseStream`:
    - on `purchased` / `restored`: grant via `Entitlements.apply`, persist, then `completePurchase`
    - on `pending`: show "Purchase pending"
    - on `error`: a friendly message, with no crash
- Consumables: `buyConsumable(autoConsume: true)`. Grant coins **only once per purchaseID**: keep `grantedPurchaseIds` in the save file.
- **Testing:** add License testers in Play Console (your Gmail). Test purchases need an internal-testing build installed from Play.

## 4. Play Games Services
- In Play Console, create Play Games Services for the app, link the OAuth client (SHA-1 of the **app signing key** from Play Console *and* the upload/debug keys), and enable **Saved Games**.
- **Achievements:** create the 14 from doc 05 §6 in Play Console. Map IDs in `lib/core/config/ids.dart`. Unlock locally first, sync when signed in.
- **Leaderboards (v1):** one per daily size ("Daily Small time", "Daily Medium time", "Daily Big time"). Daily boards reset daily in Play Games' time-span views. Submit the time in ms only for no-hint, ≤ 3-hearts-lost solves.
- **Cloud save:** see doc 09 §7.

## 5. Firebase
- Create the Firebase project "arrow-voyage", add the Android app (package `com.wobblystudio.arrowvoyage`), download `google-services.json`. It's not a secret, so it's fine to commit; restrict its API key in Google Cloud to the Android app's package name and SHA-1.
- Use FlutterFire CLI: `flutterfire configure`.
- **Analytics:**
  - Disable advertising-ID collection unless consent is given (`setConsent`).
  - Event list in doc 12.
  - Link AdMob to Firebase for ad revenue events (`ad_impression` is automatic with linking). Also log `paid_event` revenue via `onPaidEvent` callbacks for ROAS later.
- **Crashlytics:**
  - Upload symbols with `--split-debug-info` + `--obfuscate`.
  - Custom keys: `level_id`, `board_size`, `mechanics`.
- **Remote Config:** default values match `game_config.dart`. Use conditions for A/B tests (doc 12).

## 6. Notifications
- `flutter_local_notifications` with an exact-alarm-free schedule (`zonedSchedule` + `AndroidScheduleMode.inexactAllowWhileIdle`), so no special permission is needed.
- **Android 13+:** ask for `POST_NOTIFICATIONS` **only** after the player solves their first daily ("Want a reminder for tomorrow's puzzle?"). Never ask on first launch.
- Small icon: `notification_icon_96.png` → copied to `android/app/src/main/res/drawable/ic_stat_arrow.png`.

## 7. In-app review
After level 20 (conditions in doc 05 §7): call `InAppReview.requestReview()`. Never more than once every 120 days, and never after a fail or an ad. Before calling it, there is **no** "Do you like the game?" pre-prompt (Google discourages gating).

## 8. Privacy and data safety (what the app collects)
| Data | Why | Shared with |
|---|---|---|
| Advertising ID, approximate location (IP), device info, ad interactions | Ads (AdMob), when consented | Google |
| App interactions, crash logs, diagnostics | Analytics, Crashlytics | Google (Firebase) |
| Purchase history | Billing | Google |
| Play Games player ID + saved game | Cloud save, achievements | Google |

No account system, no email collection, no user-generated content in v1. The privacy policy page goes on the Wobbly Studio website, next to the other games (`games/arrow-voyage/privacy-policy.html`), in the same style. Claude Code can generate it from this table.
