# 08 · Interaction

## Input model

| Platform | Primary | Secondary |
|----------|---------|-----------|
| Touch | Tap to select, **long-press + drag** for the radial dial | Swipe, tap-background |
| Mouse | Click | Hover reveals (delete ×) |
| Keyboard | Digits `1`–`9` place, `Shift`+digit toggles a note, `Backspace`/`Delete`/`0` clears | `Enter` confirms / `Escape` cancels edits |

Read keys by **physical code** (`e.code === "Digit3"`), not `e.key`, so `Shift+1` still means "1" on every keyboard layout.

## The long-press ladder

Long-press is the izzardry "secondary action". Durations are tiered by how deliberate the action should be:

| Hold | Action | Release behavior |
|------|--------|------------------|
| **300 ms** | Open the radial dial on a cell | Release commits the selection |
| **500 ms** | Open the options popover on the "new code" button (search for a hard seed) | Short press = normal action |
| **1000 ms** | **Peek**: reveal the whole solution | Reveal lasts only while held; release hides it |
| **3000 ms** | Easter egg (donation prompt) on the help button | |

Rules:
- A long press must **suppress the click** that follows release (`isLongPress` ref, reset 50 ms later).
- Cancel on `mouseleave` / `touchcancel`.
- Every long-press feature is announced in the button's `aria-label`: `"Get hint (hold for full reveal)"`, `"Generate new seed (hold for options)"`.

## Radial dial (signature)

<p>
<img src="../assets/screenshots/04-radial-note.png" width="220" alt="Radial dial, aiming (note)" />
<img src="../assets/screenshots/05-radial-commit.png" width="220" alt="Radial dial, on top (commit)" />
</p>

A thumb-sized pie menu for choosing from ≤ 9 values without covering the board with a keypad.

| Spec | Value |
|------|-------|
| Trigger | 300 ms touch hold on an editable cell |
| Position | At the finger, clamped so the dial (60 px radius + 20 px item) stays 80 px inside the viewport |
| Items | 40 px circles (`w-10 h-10 rounded-full bg-card border-2 border-foreground/20 shadow-lg font-mono font-semibold text-sm`), on a 60 px radius, item 1 at 12 o'clock going clockwise |
| Center | `×` = clear. Selected when the finger is < 25 px from the start point. |
| Aim zone | 40–55 px from start: the item under the angle is **aimed** → `bg-accent scale-[1.6] ring-2 ring-foreground/30`. Releasing here = **secondary action** (toggle note). |
| Commit zone | ≥ 55 px: the item pops out to a 105 px radius and inverts (`bg-foreground text-background`, `scale-[1.6]`). Releasing here = **primary action** (place digit). |
| Disabled | Impossible values are `text-muted-foreground bg-muted` and **skipped**: aiming at them selects nothing. |
| Current state | Values already noted get `ring-2 ring-foreground/40`. |
| Rubber band | An SVG line from the dial center to the finger, `stroke currentColor`, 2 px, 30 % opacity, round caps. |
| Render | Portaled to `body` (escapes the board's filter stacking context). |

The two-distance idea (aim = soft action, push through = hard action) is reusable for any game with a primary and a secondary action per target.

## Tap background to hide UI

In game, tapping empty stage space (`e.target === e.currentTarget`) toggles all chrome. Tapping again brings it back. The tutorial teaches it ("Tap background to hide UI").

## Swipe to delete (touch) / hover × (mouse)

- **Direction lock:** after 8 px of movement, the larger axis wins. A vertical lock lets the page scroll; a horizontal lock calls `preventDefault`.
- **Left only.** A rightward drag resets.
- **Threshold:** 40 % of card width.
- **Desktop:** an `X` in a 24 px round button (`absolute top-2 right-2 bg-card border shadow-sm`) appears on card hover (`opacity/scale-90 → 100`, 150 ms) and turns red on its own hover.
- Both paths end in the same collapse animation and an **undo toast** (see [feedback](09-feedback.md)).

## Selectors

- **Stepper:** `‹ value ›` with chevron icon-buttons. Ends disable at `opacity-30`. Use it for ordered values (level 1–12).
- **Code field:** big tappable code. Tap = new random code, ✎ = edit inline (an underlined input with max length and allowed-character filter), ✓ / `Enter` / blur = confirm, `Escape` = revert.
- **Dropdown:** only for unordered sets (language).

## Feedback to touch

- Every pressable element has `active:scale-95`.
- Text selection is disabled on game surfaces (`select-none`). Long-press must never select text or open the iOS callout.
- Pinch-zoom is disabled (`maximum-scale=1, user-scalable=no`). The layout must therefore be legible without zoom.

## Scrolling

- The start screen scrolls (history below the fold). The game screen never does.
- A bouncing `ChevronDown` (`animate-bounce text-muted-foreground/50`) under the start footer hints that there's more. Clicking it smooth-scrolls one viewport.
- After 50 px of scroll, the start header is swapped for a sticky wordmark header, and a back-to-top FAB appears.
