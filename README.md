# Scorecard PWA

Self-contained golf scorecard. Works offline once installed.

## Deploy to GitHub Pages (fastest)

1. Create a new public repo on GitHub, e.g. `scorecard`
2. Drop these 5 files into the repo root:
   - `index.html`
   - `manifest.json`
   - `sw.js`
   - `icon-180.png`
   - `icon-512.png`
3. Settings → Pages → Source: `Deploy from a branch`, Branch: `main` / root → Save
4. Wait ~30s. URL will be `https://<username>.github.io/scorecard/`

## Install on iPhone

1. Open the URL in **Safari** (must be Safari, not Chrome — only Safari can install PWAs on iOS)
2. Tap Share → **Add to Home Screen**
3. Tap the new icon. Launches full-screen, no browser chrome.
4. Works offline after first load. localStorage persists round state.

## Local test before deploying

```bash
cd golf-scorecard
python3 -m http.server 8000
# open http://localhost:8000
```

Service workers require HTTPS or localhost — they won't register from `file://`.

## Notes

- Pars default to a standard 72 layout (4-4-3-5-4-4-3-4-5 x2). Edit per-round via the "Set par per hole" disclosure.
- Reset round clears scores but keeps par config.
- Vibrates on tap if the device supports it.
- Dark mode follows system setting.
