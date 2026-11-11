# 11 · Persistence & sharing

## Board codes

Every izzardry game generates its content **deterministically from a code**.

| Property | Standard |
|----------|----------|
| Alphabet | `0-9A-Z`, uppercase only |
| Generated length | 6 (`IZZARD`, `K7QW2M`, `M00NLT`) |
| Accepted length | 1–8 (user-typed codes are welcome: names, words) |
| Normalization | `toUpperCase()`, strip `[^0-9A-Z]`, cut to 8 |
| PRNG | Seeded (Zudoku: Mulberry32 from a string hash). Never `Math.random()` inside generation. |
| User-facing name | **"board code"** (or "\<thing\> code"). Never "seed". |
| Identity | `code + level` identifies a game. The same code at another level is a different game. |

A code's *capabilities* can depend on its content (Zudoku: some codes can't reach level 12). Show the limit inline (`LEVEL 5 / 10`) and clamp the selection rather than erroring. Offer a long-press **"search for a code with …"** to find one that can.

## The URL is the save

All game state lives in the query string, written with `replace: true` (no history spam). The app uses a `HashRouter`, so it works on static hosting.

| Param | Meaning |
|-------|---------|
| `s` | code |
| `d` | difficulty/level |
| `p` | `1` = playing (vs. start screen) |
| `b` | encoded current board (compact string; `~`-separated notes section) |
| `i` | encoded initial board |
| `t` | elapsed seconds |

Consequences:
- **Share = copy the current URL.** On the start screen that shares the puzzle; in game it shares the puzzle *with progress*.
- Reloading never loses anything.
- Use `navigator.share({ title, text, url })` when available. On failure or when it's missing, copy to the clipboard and toast `"Link copied!"`.
- Share texts: game `"Try this Sudoku puzzle!"`, completion `"I solved this Level {difficulty} Sudoku! Can you?"`.

## Autosave

- Save on **every** move (value, note, undo, redo, hint) to `localStorage`.
- Keep the **last 100** games, newest first.
- A resumed game only bumps its "last played" timestamp if the player changes something **or** stays ≥ 15 s. Peeking at an old game shouldn't reorder history.
- Undo history is session-only. After resume, history starts at the current board.
- Remember the last chosen level (default **5** of 12, the middle).

## Storage keys

Zudoku mixes `seedoku_games`, `zudoku_safety_mode`, `zudoku-language`, `pwa-install-dismissed`. The izzardry standard is:

```
<game>:<key>        e.g.  zudoku:games  zudoku:level  zudoku:helpers  zudoku:lang
                          zudoku:tutorial-seen  zudoku:donation-shown  zudoku:install-dismissed
```

- Lowercase game name, colon, kebab-case key.
- Values are JSON, or `'true'`/`'false'` strings for flags.
- Wrap every read in `try/catch` and fall back to defaults (private mode, quota).
- When renaming keys in an existing game, migrate once on startup and delete the old key.

## History

- Section "Previous Games": 5 most recent cards + "N more games" → full-screen list grouped by time.
- Each card = thumbnail, code, level + label, relative time, elapsed time, solved ✓.
- Tap = resume exactly where you left off.
- Delete = immediate + undo toast (see [feedback](09-feedback.md)).
