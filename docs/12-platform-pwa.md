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
  <meta property="og:image" content="/og-image.png" />
  <meta name="twitter:card" content="summary_large_image" />
  <!-- twitter:title / description / image mirror og:* -->
</head>
```

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
    icons: [
      { src: "/pwa-192x192.png", sizes: "192x192", type: "image/png" },
      { src: "/pwa-512x512.png", sizes: "512x512", type: "image/png" },
      { src: "/pwa-512x512-maskable.png", sizes: "512x512", type: "image/png", purpose: "maskable" },
    ],
  },
  workbox: {
    globPatterns: ["**/*.{js,css,html,ico,png,svg,woff,woff2}"],
    runtimeCaching: [{ urlPattern: ({ request }) => request.mode === "navigate",
                       handler: "NetworkFirst", options: { cacheName: "html-cache", networkTimeoutSeconds: 3 } }],
  },
})
```

- HTML is **network-first** with a 3 s timeout, so online users always get the newest shell, while offline still works.
- Fonts are bundled (`woff2` in the precache), not fetched from Google.

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
  - reloads once on `controllerchange`.
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
