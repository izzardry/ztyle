# Ztyle — the izzardry style guide

**Ztyle** is the house style for every game published under **izzardry**.
It was extracted from **[Zudoku](https://github.com/holzschneider/zudoku)**, the first izzardry title, and proven by **[Zhess](https://github.com/izzardry/zhess)**, the second, which was built on Ztyle from its first commit.

Both are reference implementations. When this guide is silent, do what they do: Zudoku for a solo puzzle, Zhess for a game with an opponent, a two-tone board or a second device. Where either departs from this guide, the guide wins. The departures are listed in [the Zudoku audit](docs/90-zudoku-audit.md) and [the Zhess audit](docs/91-zhess-audit.md); the second also records what Zhess taught the guide.

<p align="center">
  <img src="assets/screenshots/01-start-mobile.png" width="200" alt="Zudoku start screen" />
  <img src="assets/screenshots/02-game-mobile.png" width="200" alt="Zudoku game screen" />
  <img src="assets/screenshots/05-radial-commit.png" width="200" alt="Zudoku radial number dial" />
  <br /><sub>Zudoku</sub>
</p>

<p align="center">
  <img src="assets/screenshots/zhess/01-start-mobile.png" width="200" alt="Zhess start screen" />
  <img src="assets/screenshots/zhess/02-game-mobile-dark.png" width="200" alt="Zhess game screen in the dark theme" />
  <img src="assets/screenshots/zhess/04-face-to-face.png" width="200" alt="Zhess face-to-face seating" />
  <br /><sub>Zhess · <a href="https://izzardry.github.io/zhess/">play</a></sub>
</p>

**→ Live: [izzardry.github.io/ztyle](https://izzardry.github.io/ztyle/)** (source: [`index.html`](index.html)): a single static page that teaches the whole style step by step, with live demos (stage transition, radial dial, long-press ladder, synced pulses, undo toasts) built from the real tokens. The markdown chapters below are the detailed reference.

## The idea in one line

> **Zero Distractions. Just _the game_.**

An izzardry game is a monochrome, monospaced, paper-quiet surface with the game board in the middle. Everything else floats on top of it, shows up only when needed and gets out of the way. No ads, no accounts, no noise.

## The ten rules

1. **The board is the hero.** It is always centered and never moves between screens. Menus and chrome fade in around it ([layout](docs/05-layout.md)).
2. **Grayscale first.** All UI is neutral gray (hue 0, saturation 0). Color is reserved for *meaning*: red = conflict, green = success. Shades that are identity, like white and black pieces, stay put in both themes ([color](docs/03-color.md)).
3. **Everything is monospace.** One mono typeface for every word and number. Uppercase plus wide tracking for labels, `font-light` for display text ([typography](docs/04-typography.md)).
4. **Pills and circles.** Buttons are fully rounded. Panels are `rounded-2xl` glass ([surfaces](docs/06-surfaces.md)).
5. **Chrome floats.** Toolbars are translucent cards (`bg-card/80 backdrop-blur-sm`) pinned to the screen edges and safe areas. Tapping the background hides them all.
6. **Motion explains, never decorates.** Fade plus a short slide, ease-out, 150–500 ms. Screen changes blur the board in and out ([motion](docs/07-motion.md)).
7. **Touch is first-class.** Use gestures where they beat buttons (long-press, drag-to-dial, swipe-to-delete), and always offer a mouse or keyboard path too ([interaction](docs/08-interaction.md)).
8. **Every game has a code.** A short, shareable, uppercase code generates the content deterministically, or, where nothing is generated, names the game and the room two devices meet in. The URL *is* the save game ([persistence & sharing](docs/11-persistence-and-sharing.md), [rooms](docs/18-rooms.md)).
9. **Forgiving by default.** Undo everything. Autosave everything. Deletions get an undo toast. Helpers can be switched off, but are on by default ([feedback](docs/09-feedback.md)).
10. **Ship as a PWA, on the web, for free.** Installable, offline-capable, localized, and supported by an optional, once-only donation ask ([platform](docs/12-platform-pwa.md), [monetization](docs/13-monetization.md)).

## Contents

| # | Chapter | What's in it |
|---|---------|--------------|
| 01 | [Brand](docs/01-brand.md) | izzardry, game naming, tagline formula, wordmark, icon, OG image |
| 02 | [Principles](docs/02-principles.md) | Design principles, with the Zudoku and Zhess decisions behind each |
| 03 | [Color](docs/03-color.md) | Token table (HSL + hex, light + dark), alpha ladder, semantic colors, theme-stable inks, two-tone boards |
| 04 | [Typography](docs/04-typography.md) | Typeface, type scale, tracking, casing rules |
| 05 | [Layout](docs/05-layout.md) | Board-centric layout, portrait/landscape, board sizing, board-edge strips, seating, safe areas, z-index scale |
| 06 | [Surfaces](docs/06-surfaces.md) | Radius, borders, shadows, glass, halftone texture, iconography |
| 07 | [Motion](docs/07-motion.md) | Durations, easings, keyframes, screen transitions, stagger |
| 08 | [Interaction](docs/08-interaction.md) | Long-press ladder, radial dial, swipe, keyboard, hide-UI tap |
| 09 | [Feedback & states](docs/09-feedback.md) | Selection, conflict, hint, completion, toasts, undo, loading |
| 10 | [Components](docs/10-components.md) | Specs for every reusable UI piece |
| 11 | [Persistence & sharing](docs/11-persistence-and-sharing.md) | Board codes, URL state, autosave, storage keys, history |
| 12 | [Platform & PWA](docs/12-platform-pwa.md) | Manifest, icons, meta tags, updates, version stamp, deploy |
| 13 | [Monetization](docs/13-monetization.md) | No ads, the donation prompt, the easter egg |
| 14 | [Copy & i18n](docs/14-copy-and-i18n.md) | Voice, casing, string patterns, translation setup |
| 15 | [Onboarding](docs/15-onboarding.md) | The coach-mark tutorial overlay |
| 16 | [Accessibility](docs/16-accessibility.md) | Labels, focus, contrast, reduced motion |
| 17 | [Tech stack](docs/17-tech-stack.md) | Reference stack, project layout, conventions, testing |
| 18 | [Rooms](docs/18-rooms.md) | Two devices, no server: codes as rooms, seat key, sync, presence, link/QR fallback |
| 90 | [Zudoku audit](docs/90-zudoku-audit.md) | Where Zudoku departs from this guide, and the fixes |
| 91 | [Zhess audit](docs/91-zhess-audit.md) | How Zhess conforms, where it deviates, and what it taught the guide |
| ✓ | [Launch checklist](docs/99-launch-checklist.md) | Go/no-go list for a new izzardry game |

## Drop-in tokens

| File | Use it when |
|------|-------------|
| [`tokens/ztyle.css`](tokens/ztyle.css) | Any web project. Plain CSS: variables, base, component classes. No build step. |
| [`tokens/tailwind.preset.ts`](tokens/tailwind.preset.ts) | Tailwind 3 projects (the reference stack). Use together with `ztyle.css`. |
| [`tokens/ztyle.tokens.json`](tokens/ztyle.tokens.json) | Non-web targets (native, game engines, design tools). |
| [`index.html`](index.html) | The guide as one page: open it locally, or host it with GitHub Pages (root of `main`). Uses only `ztyle.css`. |

Keep a Ztyle checkout at `./ztyle/` as below, or do what Zhess does: copy `ztyle.css` and `tailwind.preset.ts` into `src/ztyle/` **verbatim**, with a one-line header naming the Ztyle revision. Either way, never edit the copy: change Ztyle first, then copy again. Game-only tokens go in the game's own CSS with a `--<game>-` prefix ([tech stack](docs/17-tech-stack.md#conventions)).

```ts
// tailwind.config.ts
import ztyle from "./ztyle/tokens/tailwind.preset";
export default { presets: [ztyle], content: ["./index.html", "./src/**/*.{ts,tsx}"] };
```

```css
/* src/index.css */
@import "./ztyle/tokens/ztyle.css";
@tailwind base; @tailwind components; @tailwind utilities;
```

## Open decisions

These came up during extraction and need a call from the owner. Until then, the guide uses the default marked ★. Zhess has since shipped every ★ default, so each is now proven in a live game.

- **Typeface.** ★ Self-host **Fira Code** (what Zudoku looks like on the creator's machine), or pick JetBrains Mono. Zudoku ships no font; Zhess bundles Fira Code through `@fontsource`. See [typography](docs/04-typography.md).
- **App icon language.** The Zudoku icon (cream tile, slate frame, sprout glyph) and OG image (navy gradient, Courier) don't match the grayscale UI. ★ Bring both into the grayscale system. Zhess's icon and OG image are the first made that way, without the sprout. Is the sprout an izzardry mark to keep? See [brand](docs/01-brand.md).
- **Dark mode.** The dark tokens exist but are never turned on in Zudoku. ★ Follow the system setting, as Zhess does. See [color](docs/03-color.md).
- **Z-naming.** *Zudoku*, *Zhess* and *Ztyle* follow a "swap the first letter for Z" convention. ★ Treat it as the default naming pattern. See [brand](docs/01-brand.md).
- **Times in history.** Zudoku shows the time of day or a date; Zhess shows a relative time (`5 min. ago`). ★ The guide keeps time of day / `MMM d` until one is chosen for the catalogue. See [the Zhess audit](docs/91-zhess-audit.md).
- **Support link.** Both games link Buy Me a Coffee to a personal handle. ★ Keep it personal; the label stays in metadata. See [monetization](docs/13-monetization.md).
