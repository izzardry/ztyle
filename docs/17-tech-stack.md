# 17 · Tech stack

The reference stack is what Zudoku uses, and Zhess was built on it unchanged, which makes it the proven default. A new game may deviate (e.g. a canvas engine) but should keep the *outputs*: tokens, PWA, URL state, i18n table.

## Stack

| Concern | Choice |
|---------|--------|
| UI | React 18 + TypeScript, `strict: true` (Zudoku: off; Zhess: on, for null safety in game and sync code) |
| Build | Vite 5 + `@vitejs/plugin-react-swc` |
| Styling | Tailwind CSS 3 + `tailwindcss-animate` + **Ztyle preset** |
| Font | `@fontsource/fira-code`, `latin` + `latin-ext`, 300–700 ([typography](04-typography.md)) |
| Primitives | shadcn/ui (Radix): Dialog, Popover, Tooltip, DropdownMenu, Toast. Only add what you use. |
| Icons | `lucide-react` |
| Toasts | `sonner` (system) + custom undo toasts |
| Routing | `react-router-dom` `HashRouter`, single `/` route + `*` 404 |
| PWA | `vite-plugin-pwa` (Workbox) |
| Workers | Native `new Worker(new URL("./x.worker.ts", import.meta.url), { type: "module" })` |
| Tests | Vitest + Testing Library + jsdom; **Playwright** end-to-end against the production build |
| Hosting | GitHub Pages via Actions |

Game-specific dependencies are fine when they are small and do one thing. Zhess adds exactly four: `chess.js` (rules), `peerjs` (signalling for [rooms](18-rooms.md)), `qrcode-generator` (manual connection fallback) and the Stockfish WASM build as a static asset, not an npm import. `peerjs` is loaded with a dynamic `import()`, so solo players never download it.

## Project layout

```
src/
  pages/         Index.tsx (the stage), NotFound.tsx
  components/    Game UI: Board, Cell, StatusBar, SeedSelector, DifficultySelector,
                 GameHistory, BoardThumbnail, CompletionOverlay, TutorialOverlay,
                 DeleteUndoToasts, DonationPrompt, PWAInstallPrompt, InstallButton,
                 LanguageSwitcher
    ui/          shadcn primitives (generated, lightly themed)
  hooks/         useGameState, useSavedGames, useSafetyMode, useTutorial, useDonationPrompt,
                 usePWAInstall, useSWUpdate, use-mobile, use-orientation, use<Game>Worker(Pool)
  lib/           <game>.ts (pure engine: generate, validate, encode/decode, hint),
                 i18n.tsx, timeUtils.ts, utils.ts (cn)
  workers/       <game>.worker.ts
public/          favicon.ico, pwa-*.png, og-image.png, robots.txt, fonts/
```

The component names above are the **shared vocabulary**. Reuse them (with the game's noun) so every izzardry codebase reads the same.

When the components outgrow one folder, group them by **area of the stage**, as Zhess does:

```
src/
  ztyle/         ztyle.css, tailwind.preset.ts: verbatim copies, with a header naming the Ztyle rev
  index.css      @import "./ztyle/ztyle.css"; then game-only tokens and keyframes
  components/
    board/       the board, cells, glyph set, radial dial, board.css
    game/        toolbar (StatusBar), info pill, player strips, move list, overlay, share
    start/       code field, opponent stepper, option pills, Start pill, status line, preview
    room/        sheets, request and presence toasts, banner (multiplayer only)
    platform/    history, tutorial, install, language, donation, credits
    ui/          shadcn primitives
  lib/
    i18n/        one table per area + define.ts + tables.ts (the registry)
    transport/   network transports behind one interface (multiplayer only)
e2e/             Playwright specs
scripts/         screenshots, brand images, engine self-play
```

## Conventions

- **Engine is pure.** `lib/<game>.ts` has no React and no DOM. It is deterministic from `(code, level)`. It is unit-tested.
- **One hook per persisted preference** (`useSafetyMode`, `useTutorial`, …), each owning its storage key.
- **Class composition via `cn()`** (`clsx` + `tailwind-merge`). Conditional state classes go inline: `isSelected && "…"`.
- **Shared visual recipes live in `@layer components`** (`.btn-primary`, `.icon-button`, …), from `ztyle.css`. Don't redefine them per component.
- **Copy Ztyle, don't edit the copy.** Put `ztyle.css` and `tailwind.preset.ts` in `src/ztyle/` byte for byte, with a one-line header naming the Ztyle revision. A change goes into Ztyle first and is copied again.
- **Game tokens get the game's prefix** and live in `src/index.css`: Zhess's `--zhess-ink-light`. Declare them on the same selectors as the Ztyle tokens (`:root, .light` and `.dark`) so scoped subtrees resolve them. When a game token turns out to be general, move it into Ztyle (as `--ink-light` was).
- **Layout math is a pure function with a test.** Zhess keeps the toolbar geometry (`toolbarLayout`), the start-preview fit (`previewFit`) and the toast placement (`toasterPlacement`) out of the components, so a layout bug is caught by a unit test, not by a screenshot.
- **Portals** for anything that must escape transforms/filters (dial, tutorial).
- **Long-press** via a small hook returning `{ onMouseDown, onMouseUp, onMouseLeave, onTouchStart, onTouchEnd }`.
- Constants for gesture thresholds at the top of the file (`DIGIT_NEAR_THRESHOLD = 40`).
- Delete scaffold leftovers (`App.css`, unused shadcn components, unused deps like `recharts`, `react-hook-form`, `zod`, `next-themes` if unused).
- `npm run lint`, `npm run typecheck`, `npm test` and `npm run build` must pass before push. Deploys happen on push to `main`.

## Testing

| Layer | Tool | What Zhess covers |
|-------|------|-------------------|
| Pure logic | Vitest | Rules wrapper, move codec, clock math, sync rules, level picker with a seeded PRNG, layout functions |
| Components | Vitest + Testing Library | Every input path on the board (tap, drag, keyboard, typed), the dial, toasts, sheets |
| End to end | Playwright | Smoke, one spec per milestone; offline start of the computer; two browser contexts playing a room |
| Visual | `scripts/screenshots.mjs` | 10 screens × 5 sizes × light/dark against a running preview |

- **Run e2e against the production build** (`vite build && vite preview`), not the dev server, so the service worker, `base: "./"` and static asset paths are tested as shipped. Use `reuseExistingServer: false`, so a run never tests a stale preview.
- Two Playwright browser contexts on one machine are enough to test WebRTC between two devices; mDNS host candidates connect without extra flags.
- Look at the screenshots in both themes before every release, and judge them against intent, not only against the spec. Zhess's dark screenshots showed white pieces drawn black; that matched its spec as written, passed review, and was caught by the owner after release.
