---
name: review-carnival-house-rules
description: Review a change to a carnival game for house rules 1 to 3 in AGENTS.md, that every game registers on the midway, reports scores through recordScore from src/carnival, and draws with theme and canvasFont. Use as one lens in a code review run on changes under src/games/.
---

# Review lens: Carnival House Rules

Review whether each game under `src/games/` that the change touches keeps the first three house rules in the root `AGENTS.md`.

## Scope

- **Registration (rule 1)** A game lives in its own folder, `src/games/<game-id>/`, its `game.ts` has its `CarnivalGame` as the default export, and the folder name and the game's `id` match. `src/games/index.ts` finds every `./*/game.ts` and registers its default export, so this is the only way a game gets a booth on the midway.
- **Scores (rule 2)** A game reports its score by calling `recordScore(game.id, score)` from `src/carnival` once, when a round ends. It never reads or writes `localStorage` or `sessionStorage` itself, and keeps no high score table of its own.
- **Theme (rule 3)** A game takes its colours, fonts and sizes from `theme` and `canvasFont` in `src/carnival/theme.ts`. It has no hex, `rgb()`, `hsl()` or named CSS colour, and no font string, of its own.

Code outside `src/games/` is out of scope. `src/carnival/` defines the theme and the scores module, so its literal colours, font strings and storage calls are the rules' source, not breaches of them. `src/midway/palette.ts` has its own scenery colours by design.

## Method

Read the house rules in the root `AGENTS.md` first. They are the standard; this lens only says how to check them.

For each game folder the change touches, read its `game.ts` whole, not only the changed lines: a rule can break through a line the change removed, such as the only `recordScore` call or the `export default`.

- **Registration** Check that `game.ts` sits directly in `src/games/<game-id>/`, that its default export is the game's `CarnivalGame`, and that the `id` it exports equals the folder name. A game written somewhere `./*/game.ts` does not match, such as `src/games/<game-id>.ts` or `src/games/<game-id>/src/game.ts`, is never found.
- **Scores** Follow every path that ends a round, and check that each calls `recordScore` once with the game's `id`, guarded so a loop running past the end does not record it again. `src/games/water-cannon/game.ts` is the precedent: it records when `state.phase` turns `'over'`, behind a `scoreRecorded` flag. Search the folder for `localStorage`, `sessionStorage`, `indexedDB` and `document.cookie`.
- **Theme** Search the folder's non-test files for colour literals (`#` followed by three to eight hex digits, `rgb(`, `rgba(`, `hsl(`, `hsla(`, and CSS colour names assigned to `fillStyle`, `strokeStyle`, `shadowColor` or a style property) and for font strings (`px` next to a family name, or anything assigned to `ctx.font` that is not a `canvasFont(...)` call).

## Threshold

Report a breach of rules 1 to 3 in code under `src/games/` that the change adds or leaves broken.

Do not report:

- Code outside `src/games/`, for the reasons in Scope.
- A value derived from the theme: arithmetic on `theme.canvas` or `theme.font` sizes, a colour from `theme.colors` with an alpha or offset applied, or a string built from `canvasFont`.
- Test data in `game.test.ts`, such as a score or an id written as a literal.
- Anything lint, the type-checker or the tests already enforce. `src/games/index.test.ts` fails when a game folder is not registered, and `src/games/index.ts` throws on a missing default export or an `id` that differs from its folder, so `npm run check` already fails on those. Report them only where the code would get past that check, such as a game file the `./*/game.ts` pattern does not match.
- Rules 4 to 7 in `AGENTS.md`. Other lenses and the reviewers own them.

## Reporting

- Name the rule by number, the file and line, and the rule's own words from `AGENTS.md`.
- State the consequence as the code has it, not worse:
  - A game file `./*/game.ts` does not match gets no booth, and nothing fails to say so.
  - A `game.ts` with no default export gets no booth. Today `src/games/index.ts` throws while loading, which stops the whole midway from loading, not only that booth.
  - An `id` that differs from the folder name breaks the house rule. Today `src/games/index.ts` throws on it while loading, so the midway does not load.
  - A game that skips `recordScore` never shows a best score on its booth card. One that calls it more than once a round fills the shared table with duplicates. One that writes storage itself bypasses the shared table, and the midway never shows those scores.
  - A colour or font of the game's own makes that booth look unlike the rest of the carnival and stops following theme changes. Nothing breaks for the player.
- Give the fix in the carnival's terms: the `theme.colors` key or `canvasFont(size, weight)` call to use, the `recordScore(game.id, score)` call and where a round ends, or the folder and export to move to.
