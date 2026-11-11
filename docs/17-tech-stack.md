# 17 · Tech stack

The reference stack is what Zudoku uses. A new game may deviate (e.g. a canvas engine) but should keep the *outputs*: tokens, PWA, URL state, i18n table.

## Stack

| Concern | Choice |
|---------|--------|
| UI | React 18 + TypeScript |
| Build | Vite 5 + `@vitejs/plugin-react-swc` |
| Styling | Tailwind CSS 3 + `tailwindcss-animate` + **Ztyle preset** |
| Primitives | shadcn/ui (Radix): Dialog, Popover, Tooltip, DropdownMenu, Toast. Only add what you use. |
| Icons | `lucide-react` |
| Toasts | `sonner` (system) + custom undo toasts |
| Routing | `react-router-dom` `HashRouter`, single `/` route + `*` 404 |
| PWA | `vite-plugin-pwa` (Workbox) |
| Workers | Native `new Worker(new URL("./x.worker.ts", import.meta.url), { type: "module" })` |
| Tests | Vitest + Testing Library + jsdom |
| Hosting | GitHub Pages via Actions |

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

## Conventions

- **Engine is pure.** `lib/<game>.ts` has no React and no DOM. It is deterministic from `(code, level)`. It is unit-tested.
- **One hook per persisted preference** (`useSafetyMode`, `useTutorial`, …), each owning its storage key.
- **Class composition via `cn()`** (`clsx` + `tailwind-merge`). Conditional state classes go inline: `isSelected && "…"`.
- **Shared visual recipes live in `@layer components`** (`.btn-primary`, `.icon-button`, …), from `ztyle.css`. Don't redefine them per component.
- **Portals** for anything that must escape transforms/filters (dial, tutorial).
- **Long-press** via a small hook returning `{ onMouseDown, onMouseUp, onMouseLeave, onTouchStart, onTouchEnd }`.
- Constants for gesture thresholds at the top of the file (`DIGIT_NEAR_THRESHOLD = 40`).
- Delete scaffold leftovers (`App.css`, unused shadcn components, unused deps like `recharts`, `react-hook-form`, `zod`, `next-themes` if unused).
- `npm run lint`, `npm test` and `npm run build` must pass before push. Deploys happen on push to `main`.
