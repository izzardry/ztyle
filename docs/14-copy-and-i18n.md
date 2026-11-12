# 14 · Copy & i18n

## Voice

**Calm, brief, friendly, a little personal.** Like a well-made tool with a human behind it.

| Do | Don't |
|----|-------|
| `Progress saves automatically` | `Don't worry! Your progress is safely stored in your browser's local storage.` |
| `Same code = same puzzle` | `Puzzles are generated deterministically from the board code.` |
| `maybe another time` | `No thanks, I don't want to support indie devs` |
| `Sudoku Complete!` | `AMAZING!!! You're a genius! 🔥🔥🔥` |
| `M00NLT removed` + Undo | `Are you sure you want to delete this game? This cannot be undone.` |
| `Computer ready offline` | `The chess engine has been downloaded and cached for offline use!` |
| `Could not reach your opponent directly. Try another network.` | `Error: ICE negotiation failed. Please check your NAT configuration.` |
| `Your opponent offers a draw` + Decline / Accept | `Draw offer received! Do you want to accept the draw? Yes / No` |

- **Fewest words that work.** Labels are 1–2 words. Hints are one line. Tutorial bodies fit in 200 px at 10 px.
- **Exclamation marks only for success** (`Sudoku Complete!`, `Link copied!`, `Checkmate!`). Share texts speak for the player and may carry one (`Join my chess game!`). A loss gets none: `Checkmate`, `Out of time`.
- **Say what failed and what to do, in the player's words.** Name the other person (`your opponent`), not the protocol. Offer the one action that can help (`Connect manually`, `Retry`).
- **"I/me" only in the donation ask.** Everywhere else the UI doesn't refer to itself.
- **Address the player directly but sparingly** ("Did you enjoy your first round?").
- **Technical honesty in tooltips** (`Tried: 312 seeds`). Plain words in the UI ("board code").
- Short symbols beat words: `=`, `→`, `•`, `/`, `×`, `✓`.

## Patterns

| Pattern | Format | Example |
|---------|--------|---------|
| Summary line | `Label: value • Label value • time` | `Board: IZZARD • Level 5 • 0:09` |
| Compact level | `Lv.N Label` | `Lv.10 Expert` |
| Limit | `value / max` | `Level 5 / 10` |
| Count | `{count} more <things>` | `7 more games` |
| Removal | `<subject> removed` | `M00NLT removed` |
| Hold hint in aria | `<action> (hold for <more>)` | `Get hint (hold for full reveal)` |
| Tutorial pair | **Title** + one-line explanation | **Exit Game** / Progress saves automatically |
| Short variants | `…Short` keys for landscape/tight layouts | `Exit` / `Progress saved` |
| Status line | `<thing> <state>` | `Computer 43%` · `Computer ready offline` |
| Request | `Your opponent <asks>` + Decline / Accept | `Your opponent wants a rematch` |
| Presence | `Opponent <state>` | `Opponent connected` · `Reconnected` |
| Hold hint for a surprise | `<action> (hold for a surprise)` | `Show help (hold for a surprise)` |

## Difficulty ladder

Twelve levels with single-word names. Reuse this ladder (and its translations) in every izzardry game that has difficulty, mapping your own parameters onto it:

| # | EN | DE | FR | ES |
|---|----|----|----|----|
| 1 | Beginner | Anfänger | Débutant | Principiante |
| 2 | Novice | Neuling | Novice | Novato |
| 3 | Easy | Leicht | Facile | Fácil |
| 4 | Casual | Entspannt | Tranquille | Casual |
| 5 | Medium *(default)* | Mittel | Moyen | Medio |
| 6 | Moderate | Mäßig | Modéré | Moderado |
| 7 | Tricky | Knifflig | Rusé | Engañoso |
| 8 | Hard | Schwer | Difficile | Difícil |
| 9 | Tough | Anspruchsvoll | Corsé | Duro |
| 10 | Expert | Experte | Expert | Experto |
| 11 | Master | Meister | Maître | Maestro |
| 12 | Insane | Wahnsinn | Dément | Locura |


## Languages

| Code | Label | Flag | Address form |
|------|-------|------|--------------|
| `en` | English | 🇬🇧 | you |
| `de` | Deutsch | 🇩🇪 | **du** (informal) |
| `fr` | Français | 🇫🇷 | **vous** (formal, as in Zudoku) |
| `es` | Español | 🇪🇸 | **tú** (informal) |

These four are the baseline for every game. Language labels are always in their own language.

Zhess ships all four, aria-labels included, and settled the multiplayer words. Reuse them:

| EN | DE | FR | ES |
|----|----|----|----|
| Room | Raum | Salon | Sala |
| Opponent | Gegner | Adversaire | Rival |
| Two players | Zwei Spieler | Deux joueurs | Dos jugadores |
| One device / Two devices | Ein Gerät / Zwei Geräte | Un appareil / Deux appareils | Un dispositivo / Dos dispositivos |
| Computer ready offline | Computer offline bereit | Ordinateur prêt hors ligne | Ordenador listo sin conexión |

## i18n implementation

A tiny in-house context, no library:

```tsx
const translations = { en: { start: "Start", moreGames: "{count} more games", … }, de: {…}, fr: {…}, es: {…} };
type TranslationKey = keyof typeof translations.en;     // English is the source of truth & type
t(key, params?) → translations[lang][key] ?? translations.en[key] ?? key, then replace {param}
```

- Initial language: stored choice → `navigator.language` (first 2 letters) if supported → `en`.
- Persist under `<game>:lang`.
- **Translate aria-labels too.** They live in the same table.
- Keys: camelCase, grouped by screen with comments. Namespaced dot keys for enumerations (`difficulty.expert`).
- Keep casing natural in strings and uppercase via CSS (see [typography](04-typography.md#casing)).
- Brand strings are **not** translated: wordmark, tagline, `shareTitle`.
- Set `<html lang>` to the active language.
- **Machine formats stay English.** Zhess shows moves with figurines in the UI, but PGN export and typed moves use English letters in every language.
- **Format dates and relative times with `Intl`** in the active language (`Intl.RelativeTimeFormat`), not with hand-written strings.

### Splitting the table

When the table grows past one screen of keys, split it **per screen or feature**, one file each, and let the type system keep the languages complete. Zhess has eight tables (`common`, `board`, `game`, `start`, `platform`, `clock`, `assist`, `room`), registered in one index file:

```ts
// src/lib/i18n/define.ts: German, French and Spanish must have exactly the keys of English.
export type Table<E extends Strings> = { en: E } & { [L in Exclude<Language, "en">]: Record<keyof E, string> };
export function defineTable<E extends Strings>(table: Table<E>): Table<E> { return table; }

// src/lib/i18n/start.ts
export default defineTable({ en: { "start.engine.ready": "Computer ready offline", … }, de: { … }, fr: { … }, es: { … } });
```

Keys carry the table's name as a prefix (`start.engine.ready`), so they are unique across tables; a test fails on a duplicate. A missing translation is a type error, not a runtime fallback to English.
