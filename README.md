# Wobbly Studio

The website for **Wobbly Studio**, a solo indie Android game studio. Plain
HTML/CSS/JS, no build step — deploy as-is (e.g. GitHub Pages).

## Structure

```
index.html                          Home page (about + game list)
app-ads.txt                         Root ads.txt (this is the one that counts, see below)
assets/
  css/style.css                     Shared styles
  js/main.js                        Shared script (footer year, etc.)
  img/logo.svg                      Studio logo/favicon
games/
  zoozle/
    index.html                      Game page
    privacy-policy.html             Game's privacy policy
    app-ads.txt                     Per-game copy (informational, see below)
  pawfect-sort/            ...same layout
  neon-divide/              ...same layout
  equal-grid/                ...same layout
  forgotten-and-found/        ...same layout
```

## Things to customize before you publish

- **Play Store links** — every game page has a "Get it on Google Play" button
  pointing at `#`. Swap in the real Play Store listing URL once each app is
  published.
- **`app-ads.txt` publisher ID** — every `app-ads.txt` file has a placeholder
  `pub-PUB-ACCOUNT-ID`. Replace it with your real AdMob/Ad Manager publisher
  ID (found in AdMob under **Apps → App Settings → App ads.txt**).
  - **Important:** Google Play only crawls `app-ads.txt` at the **root** of
    the domain set as each app's "Developer website" in Play Console. If all
    five games list this same site as their developer website, only the
    **root** `/app-ads.txt` actually matters — the copies inside
    `games/<slug>/app-ads.txt` are included for convenience/reference but
    won't be read unless a game's developer website is set to that specific
    subfolder.
- **Privacy policies** — each `privacy-policy.html` is a generic template
  covering AdMob/Firebase/Play Games style data collection. Read through each
  one and adjust it to match what that specific game actually collects
  (skip sections for services you don't use, add any you do).
- **About section / bio** — `index.html`'s About section uses a generic
  placeholder bio and a single-letter avatar. Personalize it with your name
  (or preferred handle), photo, and any links you want (Twitter/X, itch.io,
  etc.).
- **Contact email** — currently set to `ahmedfarouk5253@gmail.com` across the
  site (home page + every privacy policy's footer/contact section).

## Deploying (GitHub Pages)

1. Push this repo to GitHub (branch `main` or similar).
2. In the repo's **Settings → Pages**, set the source to that branch, root
   folder.
3. Once live, set each app's "Developer website" in Play Console to your
   Pages URL, and each app's "Privacy policy" URL to e.g.
   `https://<your-domain>/games/zoozle/privacy-policy.html`.
