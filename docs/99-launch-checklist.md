# ✓ Launch checklist

Copy into the new game's first issue/PR. Every box must be ticked before the game is announced.

## Brand
- [ ] Name follows the Z-pattern (or a deliberate exception is noted)
- [ ] Wordmark: UPPERCASE, `font-light`, `tracking-[0.3em]`
- [ ] Tagline `Zero Distractions. Just <Game>.` on the start screen, `<title>`, OG, manifest
- [ ] `author` meta = `izzardry`
- [ ] Icon set: 192, 512, 512 maskable, favicon (all grayscale, all real PNG)
- [ ] OG image 1200 × 630 in the izzardry style

## Look
- [ ] Ztyle tokens imported (`ztyle.css` + Tailwind preset); no ad-hoc colors
- [ ] Brand mono font bundled (works offline, same on all OSes)
- [ ] `halftone-bg` on the page root
- [ ] Only red/green used as color, and only semantically
- [ ] Light **and** dark checked; follows the system theme
- [ ] Buttons: one primary per screen, pills everywhere, icon buttons labeled

## Layout
- [ ] Board centered and stationary across start ⇄ game
- [ ] Start screen = blurred preview + Start pill
- [ ] All chrome floating, safe-area aware, hidden by tapping the background
- [ ] Portrait and mobile-landscape layouts both designed (rails in landscape)
- [ ] Board fits without scrolling from 320 × 568 up to 4K

## Interaction
- [ ] Touch, mouse and keyboard can each complete a game
- [ ] Long-press durations follow the ladder (300 / 500 / 1000 / 3000 ms) and suppress the trailing click
- [ ] Undo/redo for every move
- [ ] Destructive actions are immediate + undo toast (no confirm dialogs)
- [ ] `active:scale-95` on pressables; `select-none` on game surfaces

## Feedback
- [ ] Helpers toggle (on by default) bundles all spoiler-ish assistance
- [ ] Hints: tap = one step, hold = peek
- [ ] End state: page tint (success/destructive) + celebration overlay (🎉, title, summary, Share, New)
- [ ] Long work in workers; spinner in place; tooltip progress; tap to cancel

## Data
- [ ] Deterministic generation from an uppercase board code (6 generated, ≤ 8 typed)
- [ ] Full state in the URL (`s`, `d`, `p`, …); share = URL
- [ ] Autosave every move; last 100 games; history with thumbnails, swipe/hover delete
- [ ] Storage keys `<game>:<key>`, all reads in try/catch

## Platform
- [ ] PWA manifest + service worker; network-first HTML; offline playable
- [ ] Install: once-only bottom sheet + persistent small pill
- [ ] Update toast with Reload; hourly check; version stamp (tap = copy + check)
- [ ] GitHub Pages deploy workflow; `base: "./"`; HashRouter

## Words
- [ ] EN, DE, FR, ES complete, including aria-labels; `<html lang>` follows
- [ ] Difficulty ladder (12 levels) reused if applicable; default 5
- [ ] Copy reviewed against the voice table (short, calm, `!` only on success)

## Respect
- [ ] No ads, no tracking, no accounts
- [ ] Donation prompt once, after the first win, 1.5 s delay, "maybe another time"
- [ ] Support button at the bottom of the scroll area
- [ ] Tutorial overlay on first game; dismiss by tapping anywhere; reachable via `?`
- [ ] One small easter egg

## Accessibility
- [ ] Brand focus ring (no blue default)
- [ ] `prefers-reduced-motion` honored
- [ ] Never color-only state
- [ ] Touch targets ≥ 44 px on coarse pointers
