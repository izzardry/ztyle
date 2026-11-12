# 03 · Color

## System

- Tokens are **HSL triplets without `hsl()`** in CSS custom properties (`--background: 0 0% 98%`), so alpha can be applied anywhere: `hsl(var(--foreground) / 0.1)`, or in Tailwind `bg-foreground/10`.
- All neutrals have **hue 0, saturation 0**: true gray, no blue or warm tint.
- Light and dark are **mirror images**. `primary` equals `foreground` (black in light, white in dark). There is no brand color.
- Dark mode is applied with the `.dark` class on `<html>`. The default is to **follow the system** (`prefers-color-scheme`). See [Dark mode](#dark-mode).

## Tokens

| Token | Role | Light HSL | Light hex | Dark HSL | Dark hex |
|-------|------|-----------|-----------|----------|----------|
| `background` | Page | `0 0% 98%` | `#fafafa` | `0 0% 5%` | `#0d0d0d` |
| `foreground` | Text, ink, strokes | `0 0% 5%` | `#0d0d0d` | `0 0% 95%` | `#f2f2f2` |
| `card` | Raised surfaces, cells | `0 0% 100%` | `#ffffff` | `0 0% 8%` | `#141414` |
| `card-foreground` | | `0 0% 5%` | `#0d0d0d` | `0 0% 95%` | `#f2f2f2` |
| `popover` | Menus, popovers | `0 0% 100%` | `#ffffff` | `0 0% 8%` | `#141414` |
| `primary` | Primary button fill | `0 0% 5%` | `#0d0d0d` | `0 0% 95%` | `#f2f2f2` |
| `primary-foreground` | | `0 0% 98%` | `#fafafa` | `0 0% 5%` | `#0d0d0d` |
| `secondary` | | `0 0% 92%` | `#ebebeb` | `0 0% 15%` | `#262626` |
| `secondary-foreground` | | `0 0% 15%` | `#262626` | `0 0% 90%` | `#e6e6e6` |
| `muted` | Quiet fills, disabled | `0 0% 96%` | `#f5f5f5` | `0 0% 12%` | `#1f1f1f` |
| `muted-foreground` | Secondary text, labels | `0 0% 45%` | `#737373` | `0 0% 55%` | `#8c8c8c` |
| `accent` | Hover fill, "note" state | `0 0% 90%` | `#e6e6e6` | `0 0% 18%` | `#2e2e2e` |
| `accent-foreground` | | `0 0% 10%` | `#1a1a1a` | `0 0% 95%` | `#f2f2f2` |
| `border` / `input` | Hairlines | `0 0% 85%` | `#d9d9d9` | `0 0% 20%` | `#333333` |
| `ring` | Focus ring | `0 0% 5%` | `#0d0d0d` | `0 0% 80%` | `#cccccc` |
| `destructive` | Conflict, error, delete-hover | `0 72% 51%` | `#dc2828` | `0 62% 50%` | `#cf3030` |
| `destructive-foreground` | | `0 0% 98%` | `#fafafa` | `0 0% 98%` | `#fafafa` |
| `success` | Solved, complete, hint | `142 76% 36%` | `#16a249` | `142 70% 45%` | `#22c35d` |
| `success-foreground` | | `0 0% 98%` | `#fafafa` | `0 0% 98%` | `#fafafa` |

Base radius `--radius: 0.5rem` (see [surfaces](06-surfaces.md)).

## The alpha ladder

Most of the visual nuance comes from **foreground at low alpha**, not from extra gray tokens. Use these steps and nothing in between:

| Alpha | Use |
|-------|-----|
| `foreground/3` | Halftone dot texture (page) |
| `foreground/5` | Highlight of related cells (same row/col/box), ghost-button hover |
| `foreground/6` | Dense halftone |
| `foreground/10` | Board frame fill, selected cell fill, icon-button hover |
| `foreground/20` | Ghost button border, radial item border |
| `foreground/25` | Move-target markers: legal-move dots, capture rings (Zhess) |
| `foreground/30` | Block dividers (thick grid lines), card hover border, radial note ring, input underline |
| `foreground/40–50` | Note digits, selection ring, noted ring in dial, board coordinates (`/40`) |
| `foreground/60` | User-entered digits in thumbnails |
| `foreground/70` | Icon-button resting icon color |

`muted-foreground` alpha steps for text: **/70** (sub-label), **/60** (dismiss links, tap hint), **/50** (chevron hint, "/ 10" suffix), **/40** (disabled toggle icon), **/30** (version stamp).

Translucent surfaces: `card/80` (floating chrome), `card/90` (back-to-top), `card/95` (tutorial bubbles), `background/95` (sticky header), `background/80` (completion scrim), `background/60` (tutorial scrim).

## Semantic color usage

| State | Recipe |
|-------|--------|
| Conflict cell | text `destructive` semibold, fill `destructive/15`, `ring-1 ring-destructive/40`, inset glow pulse (`destructive/15 → /30`) |
| Completed group (box/row/region) | fill `success/5` (barely there) |
| Board solved | page background tints to `success/12` over 1000 ms |
| Board full but wrong | page background tints to `destructive/12` over 1000 ms |
| Hint placed | `success` pulse: fill `success/30 → 0`, ring `0 → 8px`, 800 ms, once |
| King in check (Zhess) | The conflict recipe on the king's square: `destructive/15` fill, `ring-1 destructive/40`, synced pulse |
| Threat peek (Zhess) | `destructive/10` fill on every attacked square, only while the hint is held |
| Mistake bounced back (Zhess) | The piece pulses `destructive` once (drop-shadow glow, 800 ms) |
| Solved marker in lists | `✓` in `text-success font-mono text-xs` |
| Delete affordance hover | `hover:text-destructive hover:border-destructive/50` |

Tinting the **whole page** for the end state is a signature move. Use it for any game's win/lose result instead of a modal color.

In a game against an opponent, tint from the point of view of the player who holds the device: `success/12` for a win, `destructive/12` for a loss, **no tint** for a draw. When one device belongs to both players (pass-and-play), there is no single point of view, so there is no tint either; the overlay names the winner.

## Theme-stable colours

Some shades are not decoration but identity: white and black pieces, stones, the two players' sides, a QR code that must scan. **They keep their shade in both themes.** Only the outline follows the surface.

```css
:root, .light { --ink-light: var(--background); --ink-dark: var(--foreground); }
.dark         { --ink-light: var(--foreground); --ink-dark: var(--background); }
```

`--ink-light` is always the lighter neutral and `--ink-dark` the darker one. Both are in `ztyle.css`; the Tailwind preset exposes them as `ink-light` / `ink-dark` (`fill-ink-light`, `bg-ink-dark/50`). Declare any game-specific variant on the same selectors as the tokens, so a `.light` or `.dark` subtree resolves its own values.

| Piece | Fill | Stroke | Light theme | Dark theme |
|-------|------|--------|-------------|------------|
| White | `ink-light` | `foreground` | reads as an outline | reads as a solid shape |
| Black | `ink-dark` | `foreground` | reads as a solid shape | reads as an outline |

<p>
<img src="../assets/screenshots/zhess/03-game-desktop.png" width="400" alt="Zhess in the light theme" />
<img src="../assets/screenshots/zhess/02-game-mobile-dark.png" width="185" alt="Zhess in the dark theme" />
</p>

Zhess learned this after release: with pieces filled from `background` and `foreground`, the dark theme turned white pieces black. Its QR codes follow the same rule: dark modules on a white plate in both themes.

## Two-tone patterns

A checkerboard is two tones of the cell colour, one step apart: `card`, and `card` plus a `foreground/10` overlay. **Put the overlay where it keeps the light cells lighter in both themes**: on the dark cells in the light theme, on the light cells in the dark theme (`foreground` is light there, so the overlay brightens).

```css
:root, .light { --checker-light: transparent;                --checker-dark: hsl(var(--foreground) / .1); }
.dark         { --checker-light: hsl(var(--foreground) / .1); --checker-dark: transparent; }
```

These two tokens are complete colours. Use them as the lowest overlay: `background-image: linear-gradient(var(--checker-dark), var(--checker-dark))`. Without the swap, a dark theme turns a1 light and the board reads as a mirror image.

### State fills at half the step

Grey state fills on a two-tone board use **half the checker step: `foreground/5`**. A highlighted square then sits between the two tones and never matches the other colour, so the pattern holds. With `foreground/10`, a highlighted light square becomes indistinguishable from a dark one. Rings, dots and red fills are unaffected; they don't compete with the gray steps.

Stack the overlays in one `background-image` list, top first: state fills, then the checker tone, over `bg-card`.

## Off-system colors

- **Amber** (`amber-100`/`amber-600`, dark `amber-900/30`/`amber-400`) is used only for the coffee icon disc in the donation dialog.
- The Buy Me a Coffee button keeps its own yellow. It is a third-party brand, so leave it as is, also when you draw the button yourself ([monetization](13-monetization.md)).
- Never use the default browser focus color (blue). See [accessibility](16-accessibility.md).

## Dark mode

<img src="../assets/screenshots/13-game-desktop-dark-preview.png" width="640" alt="Dark mode preview" />

The dark palette is complete and looks right (above: Zudoku with `.dark` forced on). Zhess ships it exactly as below. Ship it by following the OS setting:

```ts
// main.tsx, before render
const mq = window.matchMedia("(prefers-color-scheme: dark)");
const apply = () => document.documentElement.classList.toggle("dark", mq.matches);
apply(); mq.addEventListener("change", apply);
```

Also update `<meta name="theme-color">` to match: `#fafafa` in light, `#0d0d0d` in dark (use two meta tags with `media="(prefers-color-scheme: …)"`).

A manual toggle is optional. If added, it is an icon-button (`Sun`/`Moon`) on the start screen next to the language switcher, never in game.

**Check every game surface in both themes.** Inverting tokens is right for chrome and wrong for identity: see [theme-stable colours](#theme-stable-colours) and [two-tone patterns](#two-tone-patterns).
