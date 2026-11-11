# 06 · Surfaces

## The halftone page

The page background is never flat. It carries a barely-visible dot grid, like graph or dot-matrix paper:

```css
.halftone-bg    { background-image: radial-gradient(circle, hsl(var(--foreground) / 0.03) 1px, transparent 1px); background-size: 8px 8px; }
.halftone-dense { background-image: radial-gradient(circle, hsl(var(--foreground) / 0.06) 1px, transparent 1px); background-size: 4px 4px; }
```

`halftone-bg` goes on the root element of every izzardry game. Because it uses `--foreground`, it inverts automatically in dark mode. `halftone-dense` is for small textured areas (OG images, empty states).

## Elevation

There are only three depths. Elevation comes from **shadow plus translucency**, not from gray steps.

| Level | Recipe | Used for |
|-------|--------|----------|
| **Flat** | `bg-card border border-border` | List cards, cells, thumbnails |
| **Floating** | `bg-card/80 backdrop-blur-sm border border-border shadow-lg` | All in-game chrome: toolbar, stats pill, round buttons, language switcher |
| **Overlay** | `bg-card border border-border shadow-xl` (bubbles `bg-card/95 backdrop-blur-sm`) | Install prompt, tutorial bubbles, toasts (`shadow-lg`) |

Scrims are **blurred background**, not black: completion `bg-background/80 backdrop-blur-sm`, tutorial `bg-background/60 backdrop-blur-[2px]`. Avoid `bg-black/80` (the shadcn dialog default).

The board itself sits on `shadow-lg`. On the start screen it is additionally blurred (`filter: blur(8px)`, `opacity: .6`), which is the most recognizable izzardry image.

## Radius

| Shape | Class | Used for |
|-------|-------|----------|
| **Full / pill** | `rounded-full` | Every button (primary, ghost, icon, FAB, language, install), toolbars, toast "Undo All", dial items |
| **2xl** (16 px) | `rounded-2xl` | Floating info panels (stats pill/rail), install prompt card |
| **xl** (12 px) | `rounded-xl` | Toasts, tutorial bubbles, wordmark highlight |
| **lg** (`--radius`, 8 px) | `rounded-lg` | Board frame, list cards, "more games" button |
| **md** (6 px) | `rounded-md` | Menu items, small in-toast buttons |
| **sm** (4 px) | `rounded-sm` | Thumbnails |
| **none** | | Grid cells (the grid *is* the shape) |

The rule is: **actions are round, containers are rounded rectangles, and the bigger the container, the bigger the radius.**

## Borders and lines

- Default hairline: `border border-border` (1 px, `#d9d9d9` / `#333`).
- Grid cell lines: `border-border/50`. Group/block dividers: `border-2 border-foreground/30`. **Thick lines are foreground-tinted, thin lines are border-tinted.**
- Dividers inside pills: `w-px h-6 bg-border` (vertical) or `h-px w-6/w-8 bg-border` (horizontal), short and centered, never full-length.
- Inputs have no box: just a bottom line `border-b-2 border-foreground/30`, becoming `focus:border-foreground`.

## Iconography

- **Library:** [lucide](https://lucide.dev) (`lucide-react`), default 2 px stroke, never filled.
- **Sizes:** `w-5 h-5` (20 px) in chrome and toolbars; `w-6 h-6` in the large chevrons of selectors and the dismiss chevron; `w-4 h-4` inline with button text; `w-3 h-3`/`w-3.5` in toasts, tutorial and small pills.
- **Color:** inherits text color. Resting icon buttons are `text-foreground/70`, hover → `text-foreground`. Tutorial icons use `text-primary`.
- **Vocabulary** (use the same icon for the same concept in every game):

| Concept | Icon |
|---------|------|
| Back / exit to start | `ArrowLeft` |
| Undo / Redo | `Undo2` / `Redo2` |
| Hint | `Lightbulb` |
| Helpers on / off | `ShieldCheck` / `ShieldOff` |
| Share | `Share2` |
| Help / tutorial | `HelpCircle` |
| New random code | `RefreshCw` (→ `Loader2 animate-spin` while working) |
| Edit / confirm | `Pencil` / `Check` |
| Prev / next value | `ChevronLeft` / `ChevronRight` |
| Scroll hint / back to top | `ChevronDown` (rotated 180° for top) |
| Close / delete | `X` |
| Install | `Download` |
| Notes | `StickyNote` |
| Hide UI | `EyeOff` |
| Donate | `Coffee` |

- Clear-cell in the dial uses the typographic `×`, not an icon. The solved mark uses `✓`. The celebration uses the emoji 🎉. Language uses flag emoji. Text glyphs are fine where an icon would be heavy.
