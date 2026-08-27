# ASCEND

**A habit tracker where you are the stock.**

Every habit you keep pushes your share price up. Every one you miss pushes it
down. Your discipline is the ticker — `ISWR · Iswara Capital`, opening at
$100.00.

It runs as a dark, phone-shaped PWA you can add to your home screen. No
account, no server, no sync: everything lives in your browser.

---

## The idea

Most habit trackers give you a checkmark and a streak counter. ASCEND gives you
a **market position**.

Completing a habit pays out cash *and* moves your price. Skipping one drops it.
A long streak compounds — the multiplier grows 12% per consecutive day up to a
cap of 20 days, so a maxed streak is worth **3.4×** a cold start. Harder habits
swing the price further in both directions.

The cash you earn is spendable. You can buy shares in yourself, or spend it on
your avatar and the room it stands in.

## Features

| Tab | What it does |
| --- | --- |
| **Today** | Your habit list, streaks, daily payout, and a free-text log |
| **Assets** | Track arbitrary numbers over time — savings, weight, anything |
| **Market** | Buy and sell shares in your own ticker with earned cash |
| **Porto** | Your avatar, outfits, and an isometric room you furnish |
| **History** | Streaks, habits tended, and a 28-day completion grid |

Plus:

- **106 habit icons** across 11 categories (Health, Fitness, Mind, Learn, Work,
  Home, Social, Money, Craft, Limit, Extra)
- **Three difficulty tiers** — light (0.7×), standard (1×), hard (1.4×)
- **Five accent themes** — Voltage, Frost, Ember, Aurum, Jade
- **Installable** as a standalone iOS/Android app

## How the economy works

Reward and price movement come from four values in `files/index.html`:

```js
const BASE_PRICE = 100;                                  // opening share price
const multiplier = (s) => 1 + Math.min(s, 20) * 0.12;    // streak bonus, capped at 20 days
const DIFF_MULT  = { light: 0.7, standard: 1, hard: 1.4 };
const priceBumpPct = (h) => 0.006 * (h.base / 15) * multiplier(h.streak) * DIFF_MULT[h.difficulty];
```

A standard habit worth 15 points on a cold streak moves the price about 0.6%.
The same habit on a 20-day streak moves it roughly 2%. Missed habits apply
`missDropPct`, which deliberately ignores the streak multiplier — you lose the
day, not the compounding.

Tune these numbers to make the game feel harsher or gentler.

## Running it locally

There is **no build step and no dependencies to install**. The whole app is one
HTML file that compiles JSX in the browser via Babel standalone.

```bash
git clone https://github.com/JohanIswaraa/ascend.git
cd ascend/files
python3 -m http.server 8000
```

Open <http://localhost:8000>.

> Serve it over HTTP rather than opening `index.html` directly — `file://`
> blocks the manifest and breaks PWA behaviour.

Any static server works (`npx serve`, `php -S localhost:8000`, etc.).

## Deploying

The site is a static folder. Drag **`files/`** onto
[Netlify Drop](https://app.netlify.com/drop) and it is live.

If you switch to Git-based deploys instead, set the publish directory to
`files` — `index.html` is not at the repo root, and pointing Netlify at the root
will 404. That is what `netlify.toml` exists for:

```toml
[build]
  publish = "files"
```

## Installing on your phone

1. Open the deployed URL in Safari (iOS) or Chrome (Android)
2. **Share → Add to Home Screen**
3. Launch from the icon — it runs fullscreen with no browser chrome

On iOS, `apple-mobile-web-app-status-bar-style` is read **once, at the moment
you add it**. If you change that meta tag later, delete the icon and re-add it
or the old value sticks.

## Project layout

```
files/
  index.html            the entire app — UI, state, economy, artwork
  manifest.webmanifest  PWA metadata
  icon-180/192/512.png  home screen icons
netlify.toml            publish config for Git-based deploys
```

`index.html` is deliberately a single file. React 18, Babel, and Tailwind load
from CDNs at runtime; there is nothing to bundle.

## Your data

Everything is stored in `localStorage` under the key `terra-v6`. It never
leaves your device — there is no backend and no analytics.

That also means **clearing site data wipes your history**, and your progress
does not follow you to another browser or device.

## Notes on the iOS viewport

Two fixes keep the bottom navigation flush on iPhone, both worth knowing before
you touch the layout:

- The shell height comes from a JS-measured `--app-h` (`window.innerHeight`)
  rather than `100dvh`, which does not reliably track Safari's collapsing URL
  bar. `innerHeight` is used instead of `visualViewport.height` so the keyboard
  does not drag the nav upward.
- The status bar style is `black`, not `black-translucent`. The translucent
  variant shifts the webview under the status bar *without* returning the
  height, leaving a status-bar-sized dead band along the bottom.

iOS also reports `env(safe-area-inset-*)` as `0` inside a standalone web app, so
`--safe-bottom` is derived in JS to keep the nav clear of the home indicator.

## License

No license specified — all rights reserved.
