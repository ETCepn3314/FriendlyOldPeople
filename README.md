# FriendlyOldPeople

Spot for our family / friends betting — **The Real World Cup Cesspool** 🏆⚽

A single-page live tracker for our 2026 FIFA World Cup pool. Every player drafted
4 teams; last team standing takes the pot. The page recomputes everyone's status
(Still In / Eliminated / Champion) from a public results feed each time it loads.

## Files
- `index.html` — the whole app (no build step, no server needed)
- `manifest.webmanifest` + `icon.svg` — let phones "Add to Home Screen" with an icon

## Hosting it (GitHub Pages — free)
1. Push this branch and merge it into `main` (or just enable Pages on this branch).
2. On GitHub: **Settings → Pages**.
3. Under **Build and deployment → Source**, pick **Deploy from a branch**.
4. Choose the branch (`main`) and folder **`/ (root)`**, then **Save**.
5. Wait ~1 minute. Your site goes live at:
   `https://etcepn3314.github.io/FriendlyOldPeople/`

Share that link with friends — it works in any phone browser, and they can
tap the browser's Share menu → **Add to Home Screen** to get an app-like icon.

The page auto-refreshes every couple minutes and pulls results from the
public-domain [openfootball](https://github.com/openfootball/worldcup.json) feed.
