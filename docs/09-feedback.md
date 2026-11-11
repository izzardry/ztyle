# 09 · Feedback & states

## Board cell states

Board cells are the most state-dense element. The recipe below generalizes to any grid game (tiles, cards, slots). States combine in this priority, lowest to highest:

| State | Visual | z |
|-------|--------|---|
| Base | `bg-card border border-border/50` | |
| Group complete | `bg-success/5` | |
| Given / fixed | `font-bold` | |
| Player value | `font-normal`, digit at `foreground/80` (dim the ink, not the cell) | 30 |
| Related (same row/col/box as selection) | `foreground/5` overlay | |
| Selected | `foreground/10` overlay + `ring-2 ring-foreground/50` | 40 |
| Conflict | `text-destructive font-semibold bg-destructive/15 ring-1 ring-destructive/40` + synced error pulse | 20 |
| Just hinted | `animate-hint-pulse` (green, 800 ms, once) | |

**Implementation note.** State fills are *overlays on the cell's card color*, not replacements. In plain CSS that's `background-image: linear-gradient(<tint>, <tint>)`; in Tailwind, make sure the state class wins over the base `bg-card` (put both in the same layer, or apply `bg-card` only when no state is active). In Zudoku the state classes sit in `@layer components` while `bg-card` is a utility, so the related, selected and conflict fills never render (see the [audit](90-zudoku-audit.md)).

Pencil marks (notes): `absolute text-[0.5rem] leading-none text-foreground/50`. They fill the **4 corners first, then the 4 edge midpoints** (max 8). A note that conflicts with a placed value is hidden automatically.

## Helpers ("safety mode")

A single toggle (`ShieldCheck`/`ShieldOff`, persisted, **on by default**) controls all assistive feedback:

- conflict highlighting,
- disabling impossible values in the dial,
- auto-hiding conflicting notes,
- strict group-complete detection.

Off = purist mode: the icon dims to `text-muted-foreground/40`. Each new game should bundle its "helpful but spoiler-ish" features under this one toggle, not under several settings.

## Hints

- **Tap:** reveal one correct cell (prefer the selected cell), select it, play the green hint pulse. It goes through undo history like any move.
- **Hold 1 s:** peek at the full solution while held.
- Hints are unlimited and not shamed (no counter, no score penalty).

## End of game

<img src="../assets/screenshots/07-completion.png" width="220" alt="Completion overlay" />

1. When the board is full, the **page background tints over 1000 ms**: `success/12` if it's correct, `destructive/12` if it's full but wrong. A wrong board gets no modal; the tint *is* the message.
2. If correct: the timer stops and a **completion overlay** appears (`bg-background/80 backdrop-blur-sm animate-fade-in`, content `animate-scale-in`):
   - `🎉` at `text-6xl mb-6`
   - Title `text-2xl font-mono font-bold`: "Sudoku Complete!"
   - Summary `text-muted-foreground`: `Board: IZZARD • Level 5 • 0:09` (bullet `•` separators)
   - Actions: ghost **Share** + primary **New Puzzle** (stacked on mobile, row ≥ 640 px)
   - Tapping the scrim dismisses (to admire the board).
3. Shown once per game. 1.5 s later, the one-time donation prompt may follow (see [monetization](13-monetization.md)).

## Toasts

Two kinds:

**System toasts** (sonner): one line, no icon, e.g. `toast.success("Link copied!")`. Styled `bg-background text-foreground border-border shadow-lg`. Use them for confirmations of invisible actions (copy, update ready).

**Undo toasts** (custom, for destructive actions):

```
┌──────────────────────────────────────┐
│ M00NLT removed           [UNDO]  ×   │   rounded-xl bg-card border shadow-lg px-4 py-3
└──────────────────────────────────────┘
          [ UNDO ALL ]                     appears when ≥ 2 are stacked
```

- Bottom-center, `w-[min(90vw,360px)]`, stacked with `gap-2`.
- Subject in mono + tracking (`M00NLT`), verb in `muted-foreground` (`removed`).
- `Undo` is a small inverted button (`bg-foreground text-background rounded-md px-3 py-1 text-xs uppercase tracking-wider`).
- **5 s** lifetime; every new deletion restarts the timer for the whole stack.
- The deletion happens immediately; undo restores. Never ask "Are you sure?".

## Update available

A toast with title "A new version is available" and a **Reload** action, kept for 15 s.

## Loading & long work

- Replace the triggering icon with `Loader2 animate-spin` in the same spot. Don't add a separate spinner.
- Dim the dependent content (`opacity-50 cursor-wait`) instead of hiding it.
- A tooltip on the spinner shows real progress (`font-mono text-xs whitespace-pre-line`): phase, `current / total (percent%)`, or for searches: tried, current, best so far, worker count, and "Click to cancel".
- Clicking the spinner cancels and **restores the previous state**.
- The preview board shows an empty grid while generating. No skeleton shimmer.

## Disabled

`opacity-30 cursor-not-allowed` on icon buttons (undo at history start, stepper ends). Keep disabled controls in place, so the layout doesn't shift.

## Empty states

If there's nothing to show, show nothing. The history section simply doesn't render, and the scroll chevron disappears with it. No "No games yet" illustrations.
