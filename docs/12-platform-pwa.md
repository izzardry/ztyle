# 12 · Platform & PWA

Every izzardry game is a static web app, installable as a PWA, hosted on GitHub Pages.

## index.html head (template)

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <title>Zudoku - Zero Distractions. Just Sudoku.</title>
  <meta name="description" content="Zero Distractions. Just Sudoku. A minimalist seed-based Sudoku puzzle game." />
  <meta name="author" content="izzardry" />

  <meta name="theme-color" content="#fafafa" media="(prefers-color-scheme: light)" />
  <meta name="theme-color" content="#0d0d0d" media="(prefers-color-scheme: dark)" />
  <meta name="mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="default" />
  <meta name="apple-mobile-web-app-title" content="Zudoku" />
  <link rel="apple-touch-icon" href="/pwa-192x192.png" />
  <link rel="manifest" href="/manifest.webmanifest" />

  <meta property="og:title" content="Zudoku - Zero Distractions. Just Sudoku." />
  <meta property="og:description" content="A minimalist seed-based Sudoku puzzle game. 12 difficulty levels, notes, hints, and zero ads." />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://izzardry.github.io/zudoku/" />
  <meta property="og:image" content="https://izzardry.github.io/zudoku/og-image.png" />  <!-- absolute -->
  <meta name="twitter:card" content="summary_large_image" />
  <!-- twitter:title / description / image mirror og:* -->
</head>
```

Vite rewrites root paths in `index.html` (`/favicon.ico`, `/pwa-192x192.png`) to the `base`, so they work under `base: "./"`. It does **not** touch `og:image`, and crawlers don't resolve relative URLs, so social images take the absolute address of the deployed file. Zhess learned this after its first deploy.

## Manifest (vite-plugin-pwa)

```ts
VitePWA({
  registerType: "autoUpdate",
  includeAssets: ["favicon.ico", "robots.txt"],
  manifest: {
    name: "Zudoku - Zero Distractions. Just Sudoku.",
    short_name: "Zudoku",
    description: "Zero Distractions. Just Sudoku.",
    theme_color: "#0d0d0d",          // must match the theme-color meta for the default scheme
    background_color: "#0d0d0d",
    display: "standalone",
    orientation: "any",              // portrait AND landscape are designed for
    start_url: "./",
    icons: [                        // relative: no leading slash (see below)
      { src: "pwa-192x192.png", sizes: "192x192", type: "image/png" },
      { src: "pwa-512x512.png", sizes: "512x512", type: "image/png" },
      { src: "pwa-512x512-maskable.png", sizes: "512x512", type: "image/png", purpose: "maskable" },
    ],
  },
  workbox: {
    globPatterns: ["**/*.{js,css,html,ico,png,svg,woff,woff2}"],
    // Let navigations reach the NetworkFirst route below. Otherwise the precache route answers
    // "./" from its directory index (index.html), cache-first, and the route never runs.
    directoryIndex: null,
    navigateFallback: undefined,
    runtimeCaching: [{ urlPattern: ({ request }) => request.mode === "navigate",
                       handler: "NetworkFirst",
                       options: { cacheName: "html-cache", networkTimeoutSeconds: 3, expiration: { maxEntries: 10 },
                                  precacheFallback: { fallbackURL: "index.html" } } }],
  },
})
```

- HTML is **network-first** with a 3 s timeout, so online users always get the newest shell, while offline still works. The precached `index.html` is the last fallback, so the app also starts offline right after the first load.
- **Turn the precache's directory index off.** `precacheAndRoute` is registered before the runtime routes and, by default, answers `./` from the precached `index.html`. Without the two lines above (which earlier versions of this guide lacked), HTML is effectively cache-first and the NetworkFirst route never runs. Zhess found this and fixed it with `directoryIndex: null` plus `precacheFallback`.
- **Manifest icon paths are relative.** Vite doesn't rewrite the generated manifest, so `/pwa-192x192.png` points at the domain root and breaks on a GitHub Pages project site (`/zhess/`).
- Fonts are bundled (`woff2` in the precache), not fetched from Google.

## Large assets

An optional asset that is large compared with the app (Zhess's chess engine: 1.8 MB of WASM against a small shell) stays **out of the precache** and is still available offline:

1. **Exclude it from the precache** (`globIgnores: ["engine/**"]`), so installing or updating the app never waits for it.
2. **Serve it from a runtime cache, cache-first** (`engine-cache`, `CacheFirst`, statuses `[0, 200]`).
3. **Download it from the page, not from the service worker**, after the first paint (Zhess waits 1 s). A page can't see the progress of a service-worker install; it can see its own `fetch` stream. Write the files into the same runtime cache with `cache.put`, and nothing partial on failure.
4. **Show it in a status line on the start screen** ([components](10-components.md#status-line)): `Computer 43%` with a thin bar, then `Computer ready offline`. The start button shows its own spinner only if the player starts before the download is done.
5. **Check again on every launch** and download again if the copy is gone: iOS evicts caches. Offline, wait for the `online` event. With Save-Data on, wait for a tap. After a failure, offer a retry.
6. Test it: Zhess's e2e suite starts a computer game offline on a browser that never played one online.

Weigh the cost: every first visit downloads the asset, also for visitors who never use it. Zhess accepted that so its computer works offline from the first launch.

## Assets checklist

| File | Spec |
|------|------|
| `favicon.ico` | 256 px + 32 + 16, grayscale icon |
| `pwa-192x192.png` | real 192 × 192 PNG |
| `pwa-512x512.png` | real 512 × 512 PNG |
| `pwa-512x512-maskable.png` | 512 PNG, motif inside the central 80 % |
| `og-image.png` | 1200 × 630, see [brand](01-brand.md#social--og-image) |
| `robots.txt` | allow all |

## Updates

- `registerType: "autoUpdate"` plus a `useSWUpdate` hook:
  - checks hourly (`registration.update()`),
  - when a new worker is waiting, shows a toast "A new version is available" with a **Reload** action (15 s),
  - reloads once on `controllerchange`, **but never in the middle of a game**: then it shows the Reload toast instead (Zhess). It also doesn't reload when the very first worker takes control.
- Tapping the version stamp runs a manual check.

## Version stamp

Inject at build time:

```ts
define: {
  __GIT_HASH__: JSON.stringify(execSync("git rev-parse --short HEAD").toString().trim()),
  __GIT_DATE__: JSON.stringify(execSync("git log -1 --format=%ci").toString().trim().slice(0, 19)),
}
```

It is displayed as `2026-02-15 16:15:09 (717c4d3)`. See [components](10-components.md#version-stamp).

## Install

- Capture `beforeinstallprompt`. Show the bottom-sheet prompt **once** on the start screen (never in game). After "Not now", show only the small "Install App" pill under the level selector.
- Hide everything once installed (`display-mode: standalone`).

## Hosting & deploy

- `base: "./"` in Vite, plus `HashRouter`, so the build works at any path (GitHub Pages project sites).
- GitHub Actions: on push to `main` → `npm ci && npm run build` → `actions/upload-pages-artifact` (`./dist`) → `actions/deploy-pages`. Node 20, `concurrency: pages`, no cancel-in-progress.

## Heavy computation

Generation/search runs in **Web Workers** (`src/workers/*.worker.ts`). A pool of workers (`navigator.hardwareConcurrency`) is used for searches. Workers post progress messages that drive the progress tooltip. The main thread never blocks for more than a frame.

- GitHub Pages can't send the COOP/COEP headers that `SharedArrayBuffer` needs, so **WASM runs single-threaded**. Zhess ships Stockfish's `lite-single` build for that reason.
- A prebuilt worker can be the worker file itself: Zhess loads `engine/stockfish-19-lite-single.js` with `new Worker(new URL(path, document.baseURI))` and talks to it in its own text protocol (UCI). No wrapper worker is needed.

## Third-party assets and licences

Ship a third-party binary **unmodified, as a separate static asset**, with its licence file next to it (`public/engine/Copying.txt`), and credit it in one line at the bottom of the start screen's scroll area ([components](10-components.md#credits-line)). Zhess is MIT; the GPLv3 engine stays a separate program the app talks to, not code linked into the bundle.
