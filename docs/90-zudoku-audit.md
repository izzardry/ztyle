# 90 · Zudoku audit

Where Zudoku (as of `717c4d3`, 2026-02-15) departs from Ztyle. These are the fixes to bring the reference game in line, ordered by impact.

## Brand & visual

| # | Issue | Where | Fix |
|---|-------|-------|-----|
| 1 | **Font isn't shipped.** It renders in Fira Code only on machines that have it installed; `SF Mono` isn't web-accessible. | `src/index.css` | Bundle Fira Code `woff2`; use the canonical stack ([typography](04-typography.md)). |
| 2 | **Two mono stacks.** `font-mono` (Tailwind default) differs from the body stack, so the wordmark, cells and completion title render in a different face. | `tailwind.config.ts` | Use the Ztyle preset (sets `sans` and `mono`). |
| 3 | **Dark mode never activates.** Dark tokens exist but nothing adds `.dark`. | `main.tsx` | Follow `prefers-color-scheme` ([color](03-color.md#dark-mode)). |
| 4 | **theme-color mismatch**: meta `#fafafa`, manifest `#0a0a0a`. Neither matches the `#0d0d0d` token. | `index.html`, `vite.config.ts` | Per-scheme metas; manifest `#0d0d0d`. |
| 5 | **OG image** is a navy gradient with a Courier serif, which is off-palette. | `public/og-image.png` | Regenerate per [brand](01-brand.md#social--og-image). |
| 6 | **App icon** is cream/slate, off-palette. The PWA "PNGs" are actually JPEGs, and `pwa-192x192.png` is 512 px. There is no dedicated maskable icon. | `public/` | Re-export real PNGs at real sizes, plus a maskable variant. |
| 7 | **Browser-default blue focus outline** on focused cells. | `.sudoku-cell` | Brand focus ring ([accessibility](16-accessibility.md#focus)). |
| 8 | **Donation dialog** uses the shadcn defaults: black 80 % scrim, square full-width sheet on mobile. | `DonationPrompt.tsx`, `ui/dialog.tsx` | Blurred `background/80` scrim, rounded card with margin on mobile. |
| 9 | **Install prompt buttons** are `rounded-xl` sans-weight buttons. | `PWAInstallPrompt.tsx` | Pill buttons (`btn-ghost` / `btn-primary`). |
| 10 | **Cell state fills never render.** `.sudoku-cell-highlight`, `-selected` and `-error` set backgrounds in `@layer components`, but each cell also has the utility `bg-card`, which wins. Related cells get no tint; selected and conflict cells only show their rings. Separately, `.sudoku-cell-input { opacity-80 }` dims the *whole* editable cell (background included), which is why empty and player cells look grayer than givens. That's a nice "printed vs. writable" effect, but an accidental one. | `index.css`, `SudokuCell.tsx` | Apply `bg-card` only in the base state (or move the state classes into utilities). Make the writable-cell tint explicit (`bg-muted/40`) and dim only the digit. |
| 11 | **404 page** is a stock template (`bg-muted`, underlined link, "Oops!"). | `pages/NotFound.tsx` | Halftone page, wordmark, `text-muted-foreground` line, `btn-primary` "Back". |

## Behavior & code

| # | Issue | Where | Fix |
|---|-------|-------|-----|
| 12 | Undo toast **× does nothing**. | `DeleteUndoToasts.tsx` | Dismiss that toast (commit its deletion). |
| 13 | Storage keys mix three schemes (`seedoku_games`, `zudoku_safety_mode`, `zudoku-language`, `pwa-install-dismissed`). | hooks, `i18n.tsx` | `zudoku:<key>` + one-time migration ([persistence](11-persistence-and-sharing.md#storage-keys)). |
| 14 | Safety i18n keys are inverted (`enableSafety: 'Disable safety helpers'`). | `i18n.tsx`, `StatusBar.tsx` | Rename keys to match their text. |
| 15 | Hard-coded English: `"Delete game"`, `"Back"`, `"Change language"`, progress tooltip, `"Unknown"`. | `GameHistory.tsx`, `LanguageSwitcher.tsx`, `SeedSelector.tsx`, `i18n.tsx` | Move to translation table. |
| 16 | `<html lang>` stays `en` regardless of the chosen language. | `I18nProvider` | Set `document.documentElement.lang`. |
| 17 | No `prefers-reduced-motion` handling. | `index.css` | Include Ztyle's reduced-motion block. |
| 18 | No arrow-key navigation between cells. | `SudokuCell.tsx` | Arrow keys move selection. |
| 19 | Toolbar touch targets are 36 px. | `.icon-button` | 44 px on coarse pointers. |
| 20 | Scaffold leftovers: `App.css` (Vite logo CSS, unused), unused shadcn components, unused deps. `package.json` name is `vite_react_shadcn_ts`. | repo | Remove; rename the package to `zudoku`. |
| 21 | README clone URL points at `holzschneider/zudoku`; the author meta is `holzschneider`. | `README.md`, `index.html` | Point at `izzardry/…` once moved; `author = izzardry`. |
| 22 | README claims "full light and dark theme support". | `README.md` | True once #3 lands. |

## What Zudoku gets exactly right (keep it)

- Blurred-preview start screen with the board never moving.
- Halftone page, grayscale tokens, the alpha ladder.
- Radial dial with aim/commit zones, conflict skipping and rubber band.
- Synchronized error pulse (`animation-delay: -10000s`).
- Page-wide success/failure tint.
- Tap-background-to-hide chrome; edge-directional show/hide.
- URL-as-save-state; code + level identity; 15 s rule for resume timestamps.
- Swipe-to-delete with direction lock, height collapse and stacked undo toasts.
- Once-only, gently worded donation ask after the first win.
- Version stamp that copies and checks for updates.
