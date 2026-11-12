# 07 · Motion

Motion in izzardry games is **short, ease-out, and spatial**: things fade while sliding a few pixels from the direction they live in. Nothing bounces except the scroll hint, and nothing loops except "attention needed" states.

## Durations

| Token | ms | Use |
|-------|----|-----|
| `instant` | 100 | Dial items following the finger |
| `fast` | 150 | Cell state changes, icon-button hover/press, hover-delete reveal |
| `base` | 200 | Buttons, scale-in, accordion, dial item pop, list collapse, swipe snap |
| `chrome` | 300 | Show/hide floating chrome, fade-in, slide-up, header, FAB |
| `screen` | 500 | Start ⇄ game transition, start-screen UI enter/exit, tutorial bubble enter |
| `pulse` | 800 | Hint pulse (once) |
| `ambient` | 1000 | Page tint on solve/fail |
| `alarm` | 1500–2000 | Error pulse cycle (infinite) |

**Easing:** `ease-out` for everything that enters or responds. `ease-in-out` only for looping pulses. No springs or overshoot curves.

## Keyframes

```js
"fade-in":     { from: { opacity: 0 }, to: { opacity: 1 } },                                            // 300ms
"scale-in":    { from: { opacity: 0, transform: "scale(0.95)" }, to: { opacity: 1, transform: "scale(1)" } }, // 200ms
"slide-up":    { from: { opacity: 0, transform: "translateY(10px)" }, to: { opacity: 1, transform: "translateY(0)" } }, // 300ms
"error-pulse": { "0%,100%": { boxShadow: "inset 0 0 0 0 hsl(var(--destructive) / .15)" },
                 "50%":     { boxShadow: "inset 0 0 6px 1px hsl(var(--destructive) / .3)" } },          // 2s infinite
"hint-pulse":  { 0%:   { boxShadow: "0 0 0 0 hsl(var(--success)/.7)", background: "hsl(var(--success)/.3)" },
                 50%:  { boxShadow: "0 0 0 8px hsl(var(--success)/0)", background: "hsl(var(--success)/.15)" },
                 100%: { boxShadow: "0 0 0 0 hsl(var(--success)/0)", background: "transparent" } },   // 800ms once
```

Entry animations use `animation-fill-mode: both` so they never flash their final state first.

## Show/hide pattern for chrome

All floating chrome uses the same toggle. It is never unmounted mid-animation, only hidden:

```tsx
className={cn(
  "transition-all duration-300 ease-out",
  visible ? "opacity-100 translate-y-0" : "opacity-0 translate-y-4 pointer-events-none"
)}
```

- The offset is **toward the edge the element lives on**: bottom toolbar `translate-y-4`, top items `-translate-y-4/-8`, left rail `-translate-x-8`, right rail `translate-x-4`.
- Round floating buttons scale instead: `opacity-0 scale-90` ⇄ `opacity-100 scale-100`.
- Always add `pointer-events-none` to the hidden state.

## The start ⇄ game transition (signature)

1. Press **Start** → `transitionState = 'starting'`.
2. Over 500 ms: start UI fades/slides out (header up `-translate-y-8`, footer down `translate-y-8`, Start pill `scale-95` fade), **board un-blurs** `blur(8px)→0` and `opacity .6→1`, game chrome fades in.
3. After 500 ms the game state is committed.
4. Exit mirrors it exactly.

The board's `transition` is only active *during* the transition (`duration-500`, otherwise `duration-0`), so in-game re-renders never animate the blur.

If the start preview was drawn smaller to clear the start chrome ([layout](05-layout.md#board-sizing)), its `scale` and `translate` ride on the same 500 ms transition, so the board sharpens and grows in one movement.

## Stagger

Groups that appear together enter with **100 ms stagger** in reading order (tutorial bubbles at 100, 200, 300, 350, 400, 500 ms: `opacity-0 translate-y-2 → visible`, 500 ms each). Exit is all at once (400 ms fade).

## Synchronized loops

When many elements pulse (all conflict cells), they must pulse **in phase**. Zudoku does this with a large negative delay so every instance shares the same timeline no matter when it mounted:

```css
.sudoku-cell-error { animation: error-pulse 2s ease-in-out infinite; animation-delay: -10000s; }
```

The preset's `animate-error-pulse` carries the same delay; Zhess's pulsing king in check uses it unchanged.

Other loops that are allowed, because they mean "waiting for someone": Zhess's presence dot (hollow and pulsing while the opponent is away) and the button of a pending request (`animate-pulse`). Both stop under reduced motion and keep their static state.

## Direct manipulation

When a finger drives the motion (swipe, dial), the element tracks the finger with **no transition**. On release it animates to its rest state over 200 ms ease-out:

- Swipe-to-delete: `translateX(dx)` and `opacity 1 → .5` while dragging. Past 40 % of the width it slides to `-100%` and fades, *then* the wrapper collapses its height to 0 over 200 ms. Below 40 % it snaps back.
- Desktop delete runs the same height collapse (200 ms) without the slide.

## Moves on the board

A piece or token that moves by itself slides from its old cell to the new one: 200 ms, ease-out, `transform` only. Skip the slide when the player dragged it there (the finger already did the moving) and under reduced motion.

One-shot marks on a single cell are short and quiet:

| Mark | Recipe |
|------|--------|
| Hint | `hint-pulse`, 800 ms, once, drawn **above** the neighbouring cells so the 8 px ring is not cut off ([surfaces](06-surfaces.md#corners-and-clipping)) |
| Rejected move (Zhess blunder guard) | The piece itself pulses: `scale(1.08)` plus a `destructive` drop-shadow glow, 800 ms, once |
| Typed input that didn't parse (Zhess) | The input chip shakes ±4 px, 200 ms, twice, struck through in `destructive` |

## Toasts

`animate-in slide-in-from-bottom-3 fade-in zoom-in-95 duration-250 ease-out` (tailwindcss-animate).

## Reduced motion

Respect `prefers-reduced-motion: reduce`: drop translate/scale/blur transitions to opacity-only at 150 ms, and stop infinite pulses (keep the static red fill). The Ztyle CSS includes this rule. See [accessibility](16-accessibility.md).

**A one-shot pulse needs a still fallback.** The global rule shortens every animation to 1 ms, so a hint pulse would be over before it is seen, and the player loses the information. Show the mark statically for twice the pulse's duration, then clear it. The global rule shortens durations, not delays, so a 1 ms clearing animation delayed by that time does the job:

```css
@media (prefers-reduced-motion: reduce) {
  .cell--hint, .animate-hint-pulse {
    box-shadow: inset 0 0 0 999px hsl(var(--success) / .3);
    animation: z-hint-hold 1ms linear calc(var(--dur-pulse) * 2) 1 forwards !important;
  }
}
```

`ztyle.css` ships this for the hint. Do the same for every game-specific one-shot mark: Zhess holds its red blunder glow the same way, and also moves keyboard focus to the hinted square with a visible ring.
