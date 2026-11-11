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

- **Fewest words that work.** Labels are 1–2 words. Hints are one line. Tutorial bodies fit in 200 px at 10 px.
- **Exclamation marks only for success** (`Sudoku Complete!`, `Link copied!`).
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
