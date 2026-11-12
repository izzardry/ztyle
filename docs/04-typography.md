# 04 · Typography

## One typeface: monospace

Every glyph in an izzardry game is monospaced: titles, buttons, body copy, numbers. Mono gives the "terminal / graph paper" calm, makes digits line up in grids and timers without effort, and makes uppercase-plus-tracking labels look engineered rather than shouty.

### Canonical stack

```css
font-family: "Fira Code", "JetBrains Mono", "SF Mono", ui-monospace, Menlo, Consolas, monospace;
font-variant-ligatures: none;      /* "=" and "->" in copy must not turn into arrows */
font-variant-numeric: tabular-nums;
```

**Self-host the brand font.** Bundle Fira Code (OFL) as `woff2` (weights 300, 400, 500, 600, 700) with the app so it works offline in the PWA and looks the same everywhere. Use `@fontsource/fira-code` or copy the files into `public/fonts`.

Zhess does it in one file imported by `main.tsx`: the `latin` and `latin-ext` subsets (enough for EN, DE, FR, ES) in all five weights, `woff2` only, precached by the service worker.

```ts
// src/fonts.ts
import "@fontsource/fira-code/latin-300.css";      // … 400, 500, 600, 700
import "@fontsource/fira-code/latin-ext-300.css";  // … 400, 500, 600, 700
```

> **Why this matters.** Zudoku's body stack is `'SF Mono', 'Fira Code', 'JetBrains Mono', monospace`, and no font is shipped. Browsers don't expose *SF Mono* to web pages, so it renders in Fira Code **only because the creator has it installed**. Everyone else gets Menlo, Consolas, Courier or Roboto Mono. On top of that, Tailwind's `font-mono` class still points to Tailwind's default stack (`ui-monospace, SFMono-Regular…`), so elements with `font-mono` (cells, the completion title, the wordmark) render in a *different* mono face from the surrounding body text. The Ztyle Tailwind preset sets **both** `fontFamily.sans` and `fontFamily.mono` to the canonical stack to fix this.

## Type scale

Sizes are Tailwind steps. Use these roles; don't invent new ones.

| Role | Classes | px | Example |
|------|---------|----|---------|
| **Wordmark** | `font-light tracking-[0.3em] text-3xl sm:text-4xl` (landscape `text-2xl`, header `text-xl`) | 30/36 | `Z U D O K U` |
| **Code display** | `text-2xl sm:text-3xl md:text-4xl font-light tracking-widest` | 24→36 | `IZZARD` |
| **Celebration title** | `text-2xl font-bold` | 24 | `Sudoku Complete!` |
| **Board digits** | `text-base sm:text-lg md:text-xl lg:text-2xl`; givens `font-bold`, user entries `font-normal text-foreground/80` | 16→24 | |
| **Selector value** | `text-lg font-medium` | 18 | `Medium` |
| **Section title** | `text-lg uppercase tracking-wider` | 18 | `ALL GAMES` |
| **List primary** | `text-lg tracking-wider` | 18 | `M00NLT` |
| **Body / button** | `text-sm` (buttons add `uppercase tracking-wider`) | 14 | `START` |
| **Overline label** | `text-sm uppercase tracking-wider text-muted-foreground` | 14 | `LEVEL 5` |
| **Meta / secondary** | `text-xs text-muted-foreground` | 12 | `10:31 PM` |
| **Eyebrow** | `text-[10px] uppercase tracking-wider text-muted-foreground` | 10 | `BOARD` |
| **Micro** | `text-[9px] sm:text-[10px] text-muted-foreground/30` | 9–10 | version stamp |
| **Pencil marks** | `text-[0.5rem] leading-none text-foreground/50` | 8 | cell notes |
| **Board coordinates** | `text-[0.5rem] leading-none text-foreground/40`, inside the edge cells | 8 | `a`–`h`, `1`–`8` (Zhess) |
| **Status line** | `text-[10px] uppercase tracking-widest text-muted-foreground` | 10 | `COMPUTER READY OFFLINE` (Zhess) |

### Weights

| Weight | Use |
|--------|-----|
| 300 light | Wordmark, code display. Big text is *light*. |
| 400 normal | Body, user-entered values, notes |
| 500 medium | Selector values, menu items, bubble titles |
| 600 semibold | Conflict digits, radial dial digits |
| 700 bold | Given/fixed board values, celebration title |

The rule of thumb is **the bigger the text, the lighter the weight**, with one exception: fixed game data (givens) is bold, so it reads as "printed" against the player's lighter "handwriting".

### Tracking

| Tracking | Use |
|----------|-----|
| `0.3em` | Wordmark only |
| `tracking-widest` (0.1em) | Codes and stat values (`IZZARD`, `0:42`, `5`) |
| `tracking-wider` (0.05em) | Every uppercase label and button |
| `tracking-wide` | Small pill buttons (`Install App`) |
| normal | Body sentences |

## Casing

| Element | Case | How |
|---------|------|-----|
| Wordmark | UPPERCASE | typed |
| Board codes | UPPERCASE | input coerced with `.toUpperCase()` |
| Buttons (primary/ghost) | UPPERCASE | `uppercase` class; strings stay Title Case in i18n (`'New Puzzle'`) |
| Labels/eyebrows | UPPERCASE | `uppercase` class |
| Headings in dialogs/bubbles | Sentence or Title Case | as written |
| Whisper text | lowercase | `board code`, `maybe another time` |

Keep translation strings in natural case and apply caps with CSS, so translators never type in caps.

## Numbers

- Times: `m:ss`, or `h:mm:ss` past an hour (`formatElapsedTime`). Never `00:42`.
- Countdown clocks: `m:ss`; under 10 s `s.t` with tenths **and** `font-bold`; at zero `0.0` in `destructive`. Low time is shown by weight and tenths, not by color alone (Zhess).
- Scores and results: en dash and half points, `2 – 1½`, `1–0`, `½–½`.
- Levels: `Level 5`, compact `Lv.5`.
- Ranges/limits: `/ 10` suffix in `muted-foreground/50`.
- Counts in strings via params: `'{count} more games'`.

## Glyphs in text

When a game has its own glyph set, use it **inside running text** as well, at the size of the letters around it. Zhess writes moves in figurine notation: the knight in `♘f3` is the board's own SVG knight at `1.05em`, drawn in the mover's ink. Screen readers get the plain letters (`Nf3`) as visually hidden text. Machine formats stay plain: PGN export and typed moves use English letters in every language.
