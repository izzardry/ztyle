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

### Codes for games without generated content

A game that doesn't generate its content keeps the code anyway, as the game's **name**. Zhess:

- Same alphabet, same lengths, same field: tap for a new random code (never the one shown), ✎ to type one.
- User-facing name: **"game code"**.
- The code identifies the game in history and in the URL. Typing a code that is already in history offers `RESUME` or `REVIEW` instead of `START`.
- For a two-device game the code is also the **meeting point**: both devices type or open the same code ([rooms](18-rooms.md)).

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

**`s`, `d` and `p` are shared by every izzardry game.** Other letters belong to the game and are listed in its design doc. Zudoku and Zhess both use `t`, for different things (elapsed seconds, time control), so never assume a letter beyond the three. Zhess adds:

| Param | Meaning |
|-------|---------|
| `o` | opponent: `c` computer, `l` local, `r` room |
| `w` | this device's side |
| `t` | time control, `5+0` |
| `m` | moves, two base64url characters per move (from-square, to-square) |
| `r` | result of a finished shared game, `1-0.resign` |
| `x`, `y` | one-time connection offer and answer; removed after use |

Consequences:
- **Share = copy the current URL.** On the start screen that shares the puzzle; in game it shares the puzzle *with progress*.
- **A link to a place, not a state, carries the code only.** A Zhess room link is `#/?s=K7QW2M&o=r`. Room games never write their moves to the address bar, so a screenshot or a copied address leaks nothing but the code.
- Reloading never loses anything.
- Use `navigator.share({ title, text, url })` when available. On failure or when it's missing, copy to the clipboard and toast `"Link copied!"`. **Cancelling the share sheet is not a failure** (`AbortError`): copy nothing. Zudoku copies anyway; Zhess fixed it.
- Share texts: game `"Try this Sudoku puzzle!"`, completion `"I solved this Level {difficulty} Sudoku! Can you?"`. Zhess: `"Join my chess game!"` (room), `"Look at this chess game."` (any other game), `"I beat Level {level}! Can you?"` (a win against the computer).
- Offer a plain-text export where the game has a standard format: Zhess's Share menu adds `Copy PGN`.

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

- Lowercase game name, colon, kebab-case key. Zhess used the scheme from its first commit: `zhess:games`, `zhess:rooms`, `zhess:opponent`, `zhess:color`, `zhess:clock`, `zhess:helpers`, `zhess:seating`, `zhess:lang`, and the three flags.
- Values are JSON, or `'true'`/`'false'` strings for flags.
- Wrap every read in `try/catch` and fall back to defaults (private mode, quota).
- When renaming keys in an existing game, migrate once on startup and delete the old key.

## History

- Section "Previous Games": 5 most recent cards + "N more games" → full-screen list grouped by time.
- Each card = thumbnail, code, level + label, relative time, elapsed time, solved ✓.
- Tap = resume exactly where you left off. A game that is over opens in **review** instead (Zhess), with **continue from here** to start a new game from the shown position.
- Delete = immediate + undo toast (see [feedback](09-feedback.md)).
