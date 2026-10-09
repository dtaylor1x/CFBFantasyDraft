# Dynasty Fantasy Draft

This standalone fantasy-draft application reads a College Football dynasty save and provides a commissioner view of a simulated snake or NIL-pool draft. The draft runs in memory until you explicitly write the completed rosters to a save.

This source folder has the 1.0.3-era scripts and model. On this computer, the separately installed desktop application is still version 1.0.2; updating these source files does not replace its executable. The existing source was preserved in `fantasy-draft-source-before-update-2026-10-08.zip`.

## Setup

```powershell
npm install
```

## Desktop commissioner app

```powershell
npm start
```

To regenerate the Windows installer and portable executable with `build/icon.png`, run `npm run build`. Both artifacts are written to `dist/`; the portable executable does not require installation.

This opens a dedicated Fantasy Draft window. The home screen leads to draft setup, where you select a dynasty save. Selection immediately creates and verifies a backup beside that file, before parsing it. Choose alphabetical, randomized, or highest-team-NIL snake order, or the non-snake **NIL pool** order. Choose **Create new save** or **Write to selected save** for the completed draft. For a new save, enter its `DYNASTY-` filename suffix in setup; an existing filename is rejected before the draft begins. Team NIL is the sum of `CurrentNILCompensation` for rostered players in the save. The next screen lets you drag teams for snake order or edit each team's starting NIL pool for NIL-pool order. The draft room offers **Sim pick**, **Sim round** (or the next team-count picks in NIL mode), and **Sim to end**. After all picks, **Write drafted rosters** performs the selected save action following another confirmation. The desktop window runs its own private local server, separate from any browser session. This is a development launch; `npm install` is required to restore Electron before running it from this source folder.

For browser-based development or snapshot restoration, run `npm run web` and open `http://127.0.0.1:4173`. Browser file selection remains simulation-only because the browser cannot grant the app a writable source path or create a backup beside it. Refreshing that page keeps its current in-memory draft while the server is running.

The draft uses rostered players and teams from the save. In snake mode, teams pick in the confirmed first-round order, then reverse order in each even round. In NIL-pool mode, the team with the highest remaining NIL pool picks next, even if it picked immediately before. For overall pick `n`, that team's pool is charged the `n`th-highest `BaseNILValue` from the initial player pool, not the selected player's NIL value. Zero or negative balances remain eligible; reaching 85 drafted players removes a team. For shortened pools, teams first receive a shared minimum roster size before any team receives an extra slot. The full sequence is deterministic from the starting pools and ranked NIL charges and is reconstructed on snapshot restore. Both modes enforce each converted position group's per-team base count, global higher-slot count, and a team's total higher-slot budget, preventing extra selections from crowding out required base positions. Every team takes its highest scoring available player. Each player–team pairing has a draft-specific, stable multiplier from `0.99×` to `1.01×` applied to the final score, adding small variation between teams and drafts. Ties use overall, player ID, then name for a repeatable order. The multiplier and confirmed order persist in a draft snapshot.

On save load, the simulator reads `output/overall-progression-model.json` and calculates each player's predicted overall for the next three seasons. The score calculation is isolated in [`src/draft-score.mjs`](src/draft-score.mjs). It derives years in college from class year plus a previous redshirt, estimates likely years remaining using an 80% declaration chance in each eligible season at 89+ overall, and forces departure after the player's final eligible season. The optional five-year eligibility setting adds a year for eligible non-redshirt upperclassmen; it also adjusts their projection class and enables the years-remaining board filter. This produces fractional expected years. It then computes current and future starter scores and applies position values and team depth adjustment. An in-waiting player's score interpolates between adjacent projected starting seasons when the incumbent's remaining years are fractional. The board shows the score for the team currently on the clock; pick history preserves the score at the time of selection. Select a player row to inspect the calculation.

`LT` and `RT` share `OT` depth, `LG` and `RG` share `OG`, `LE` and `RE` share `DE`, all three linebacker positions share `LB`, and `FS` and `SS` share `S`. Each group has a configurable starter count and backup multiplier in `src/draft-score.mjs`. Currently WR, LB, and CB have three starter slots; OT, OG, DE, DT, and S have two; all other groups have one. Every starter slot can have one in-waiting player.

Before a group's starter slots fill, its picks use the original raw selection score, with any configured depth factor for that slot. Afterward, each candidate gets two raw scores. The in-waiting score evaluates projected overall only in the seasons after an unfilled starter slot is expected to open and scales by the square root of the candidate's remaining years after that starter leaves. The backup score starts with original raw selection score. Each receives its depth factor before the larger score is position adjusted. If backup wins, the waiting slot stays open; if waiting wins, that slot is consumed. A new save starts a new in-memory draft.

## Writing completed rosters

The team dropdown in the draft room is always alphabetical, independent of draft order. Save writing is available in the desktop app only after every player has been picked. The writer validates that the selected save still matches its verified backup and that its Team roster references exactly match the draft pool. It updates each player's `TeamIndex`, each team's `Player[]` roster references, and every team depth chart (including specialty slots), then reopens the written save and verifies all assignments. Players who change teams have their team-tenure counter reset, retain their current/base NIL compensation, and each team's `NILProgramPointsSpent` is recalculated from its drafted roster.

After the last pick in either draft mode, [`src/roster-normalizer.mjs`](src/roster-normalizer.mjs) balances `LT/RT`, `LG/RG`, `LE/RE`, `LOLB/MLB/ROLB`, and `FS/SS` within each team. Higher-overall players receive distinct starter spots first and keep their drafted side when it fits; the highest backup from an overfilled side moves first. Any odd extra spot favors the side with more drafted players. Middle/outside linebacker moves also map `PlayerType` into an archetype valid for the destination role; `Power Rusher` maps to middle-linebacker `Field General`, and middle-linebacker `Field General` maps to outside-linebacker `Pass Coverage`. The completed draft view shows final positions and jersey numbers. Within each team, offense and defense have separate jersey-number pools, so they may share a number. Specialists have a third pool. If players on the same side share a number, the highest-overall player keeps it and lower-overall players receive an unused number from the configurable position ranges in that module. When a restricted position has no free number, a flexible-position player may also be renumbered to open one. The save writer writes and verifies position, `PlayerType`, and `JerseyNum` changes.

**Create new save** uses the name entered during setup and writes `DYNASTY-{name}` without an extension beside the original; it will not overwrite an existing file. **Write to selected save** stages and verifies the output, then replaces the selected save with a same-volume rename. On a write/verification failure, the app attempts to restore the verified backup. The original backup is retained in either mode. A game load and week advance are still necessary to validate the save in-game.

`POSITION_DEPTH_RULES` supports an optional `depthMultipliers` array indexed by players already drafted in that group. WR currently uses `[1, 1, 0.85, 0.75]`: the first two WRs have full value, the third gets `0.85`, and the fourth gets `0.75`. Later WRs decline by the rule's `backupMultiplier` of `0.7` per additional pick. Groups without a schedule retain their original backup and in-waiting behavior.

## Scheme multipliers

Each team uses `CurrentOffensiveScheme` and `CurrentDefensiveScheme` from the save's `Team` table. All schemes seen in the save are listed in [`src/scheme-multipliers.mjs`](src/scheme-multipliers.mjs) for tuning. Under a scheme, set a **converted position group** (`OT`, `OG`, `DE`, `LB`, `S`, or a position that does not convert) to an object of **1-based draft slots** and factors. For example, `OFF_SPREAD: { WR: { 3: 1.15, 4: 1.15 } }` makes Spread teams value WR3 at `0.85 × 1.15` and WR4 at `0.75 × 1.15`, alongside the normal position factor. Missing schemes, groups, and slots have factor `1`. Raw left/right position keys are not used. The player detail panel shows the team's schemes and applied factor.

To restore an earlier draft snapshot in the development web server while importing its save's current team schemes, start with `npm run web -- --restore path\to\snapshot.json --restore-save path\to\dynasty-save`. Already completed picks retain their recorded scores; new picks use scheme factors. In the desktop app, a completed snapshot can be paired with `--restore-save` to create a corrected new save without redrafting, but writing is enabled only if every saved pick exactly matches that source save's player records and team assignment. The selected source is backed up before parsing.

## Scheme archetype multipliers

Edit [`src/archetype-multipliers.mjs`](src/archetype-multipliers.mjs) to give a scheme a factor for a converted position group and archetype. All observed offense and defense schemes are listed there. Option offense has QB archetype factors that can be tuned in this file. Archetype factors stack with the existing position, depth, and scheme-depth multipliers; omitted combinations default to `1`. The draft board shows archetypes, and player details show the applied archetype factor. See the [position-by-position archetype reference](docs/archetype-reference.md) for exact save labels and their multiplier keys.

## Teammate archetype matches

Edit [`src/archetype-match-multipliers.mjs`](src/archetype-match-multipliers.mjs) to value a candidate whose archetype is already represented among that team's drafted players in the same converted position group. A matching QB gets `1.025×`, while a matching HB gets `0.95×`; the file contains the current values for the other groups. This factor applies once when there is at least one match, regardless of how many teammates match, and stacks with the other scoring factors. A missing archetype receives no match factor. The player detail panel shows the matching teammate count and multiplier. Restoring a draft snapshot rebuilds match counts from the recorded picks.

## Position maximums

When a save is loaded, the simulator counts every draftable player by converted position group and divides each count by the number of teams. The whole-number quotient is every team's base maximum; the remainder is the number of teams allowed one extra player. For example, 10 WRs and 3 teams gives a base of 3, with one team allowed a fourth WR. Once one team drafts WR4, every other team's WR maximum is 3. A position at its current maximum stays visible in the available-player list as **maxed**, with no draft score, and cannot be selected by the simulation. Player details show its pool count, base, ceiling, and remaining extra-team slots.

The simulator also limits how many extra position slots each team can consume across all groups, based on its total number of snake-draft picks. It reserves a feasible distribution of the remaining extra slots and can reassign those reservations as teams draft. Occasionally, a team's higher slot will be unavailable before the global remainder is exhausted because taking it would leave another team without enough legal picks; player details state this reason. This ensures the full player pool can be assigned. Maximums and extra-slot usage are reconstructed from the player pool and completed picks when restoring a snapshot. The source logic is in [`src/position-maximums.mjs`](src/position-maximums.mjs).

## Export players

```powershell
npm run export -- "C:\path\to\dynasty-save" --output ".\output\players.xlsx"
```

By default, the export includes players assigned to a known team. Add `--all-player-records` to include unassigned/recruit/free-agent records stored in the Player table.

```powershell
npm run export -- "C:\path\to\dynasty-save" --all-player-records
```

The `Players` worksheet contains editable `Draft Score`, `Draft Tier`, and `Draft Notes` columns followed by source fields from the save. The six skill-cap fields are preserved individually. `Skill Cap Total` is their sum; it is intentionally a neutral derived value until the game-facing meaning of each cap value is confirmed.

This step is read-only. It never writes to the dynasty save.

## Train the progression model

Place consecutive exports in `output` using names such as `season-2026.xlsx` through `season-2030.xlsx`, then run:

```powershell
npm run predict
```

This trains a ridge-regression model on one-year player transitions. It uses current overall, class year, development trait, and skill-cap total, including nonlinear and interaction terms. The most recent season receives one-, two-, and three-year iterative forecasts in `output/overall-progression-predictions.xlsx`. The fitted coefficients and scaling values are also saved to `output/overall-progression-model.json`.

The season before the latest export is held out as a temporal test. The output reports model error alongside a no-growth baseline.

## Apply forecasts to the dynasty player pool

After training the model, write its iterative forecasts directly into `output/dynasty-player-pool.xlsx`:

```powershell
npm run apply-predictions
```

This adds or refreshes `Predicted Gain +1` through `Predicted Overall +3` on the `Players` sheet. Forecasts stop after a player's senior season. Use `--input`, `--output`, or `--model` to override the default files.
