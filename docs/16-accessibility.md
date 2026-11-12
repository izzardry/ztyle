# 16 · Accessibility

Minimal UI makes accessibility easy to forget. These are requirements, not nice-to-haves.

## Labels

- Every icon-only button has a translated `aria-label`.
- The label describes the **current action**, including hidden gestures: `"Get hint (hold for full reveal)"`.
- Toggles describe what pressing will do: helpers on → `"Disable safety helpers"`. Name i18n keys after the text they hold. Zudoku has `enableSafety: 'Disable safety helpers'`, which is confusing to maintain.
- Decorative glyphs (🎉, ✓, ×) get `aria-hidden` or a text alternative. Glyphs inside text (figurines) are `aria-hidden` next to a visually hidden plain form (`Nf3`).

## Focus

- Interactive grid cells are focusable (`tabIndex={0}`) and keyboard-operable.
- **Style focus with the brand ring**, never the browser default (Zudoku currently shows the default blue outline on focused cells):

```css
:focus-visible { outline: 2px solid hsl(var(--ring)); outline-offset: 2px; }
.cell:focus-visible { outline: none; box-shadow: inset 0 0 0 2px hsl(var(--ring) / .5); }
```

- Overlays trap focus while open (Radix dialogs do this) and return focus on close.
- Add arrow-key navigation between cells in grid games (Zudoku lacks it). Zhess uses a **roving `tabindex`**: one cell is in the tab order, the arrow keys move it.
- A pointer press on the board must not move DOM focus (`preventDefault` on `mousedown`), so no stray ring appears after a click. Keyboard users tab in.
- When the game points at a cell (a hint), move focus there with a visible ring (`focus({ focusVisible: true })`), so the mark survives reduced motion.
- Take the blurred start preview out of the tab order and the accessibility tree (`inert`). It is decoration until Start.

## Board semantics

A board is a `role="grid"` of `row`s and `gridcell`s. Label each cell with its content and its name: Zhess says `white knight, g1`, and adds the state (`possible move`, `in check`, `last move`). A visually hidden description explains the keys.

In a turn-based game, **announce each move in words** in a polite live region: `Black: knight to f6, check`. Announce only what changed; don't re-announce on unrelated renders. Clocks are `role="timer"`.

## Contrast

| Pair | Ratio (light) | Note |
|------|---------------|------|
| foreground on background | ~19:1 | ✓ |
| muted-foreground `#737373` on `#fafafa` | ~4.6:1 | ✓ AA body text |
| muted-foreground/70 on background | ~3:1 | Only for ≥ 12 px non-essential captions |
| muted-foreground/30 (version stamp) | ~1.5:1 | Decorative only. Never use it for information the player needs. |
| destructive `#dc2828` on `destructive/15` | ~4:1 | Bold digits keep it readable. Color is never the *only* signal: conflicts also get a ring and a pulse. |

Never convey state by color alone: solved = ✓ **and** green; conflict = red **and** ring **and** pulse.

## Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 1ms !important; animation-iteration-count: 1 !important; transition-duration: 150ms !important; }
  .board-blur { filter: none !important; }
}
```

Included in `tokens/ztyle.css`, together with a still fallback for the hint pulse. Every one-shot mark needs one, or reduced motion removes the information along with the motion ([motion](07-motion.md#reduced-motion)). Drop piece slides too; infinite pulses get `motion-reduce:animate-none` and keep their static state.

## Touch targets

- Toolbar icon buttons are 36 px. Aim for **44 px** on touch (`p-2.5` with a 20 px icon) in new games. Zhess uses 44 px for every toolbar button on every device and shrinks only the spacing.
- The radial dial lets small cells (≥ 28 px) still be operated precisely.
- Board cells below 44 px are accepted when the whole board must fit. Say so in the design doc: Zhess's squares are ≥ 34 px.
- Every long press has a key ([interaction](08-interaction.md#keyboard-twin-for-a-long-press)).

## Zoom

Zudoku disables pinch-zoom for game feel. If you do that, make sure all text is legible at default size and the board scales to the viewport (it does: `--cell-size` clamp).
