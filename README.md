# CFB 27 Fantasy Draft Machine

Create a fantasy draft from a College Football 27 dynasty save. Draft for one or more teams, let the computer make other picks, and write the completed rosters to a dynasty save.

## Get started

1. Run the **Setup** installer in `dist`, or launch the **Portable** executable there without installing it.
2. Select **Start Drafting** and choose your dynasty save. The app creates a verified backup beside the save before reading it.
3. Choose a draft-order method and whether to **Create new save** (recommended) or **Write to selected save**. If creating a new save, enter its name on this screen.
4. If you use a five-year eligibility mod, check **Using 5-year eligibility mod/tool**. If the selected save supports editable transfer settings, you can also choose **Turn off transfers**.
5. On the draft-order screen, search for teams and mark each one you want to control with its **USER/CPU** button. For snake drafts, drag teams to change the first-round order. In NIL-pool mode, adjust starting NIL values instead. Select **Confirm draft order** when ready.

## Draft-order choices

- **Randomized**, **Alphabetical**, and **Team NIL highest first** use a snake draft: the order reverses each round.
- **NIL pool** is not a snake draft. The team with the highest remaining pool picks next; a team may pick twice in a row. The app deducts a ranked NIL amount after each pick, and pools may go below zero.

## In the draft room

- **Available players** shows the remaining pool and the current team's draft scores. Search or filter by position, development trait, eligible seasons, previous team, ratings, size, and skill caps. Select a row to view the player's scouting details.
- When a user-controlled team is on the clock, choose a player and press **Select Player**. The position-coverage row shows how that team's roster is filling out.
- Use **Sim pick**, **Sim round**, or **Sim to next user pick** to advance the draft. **Autosim between user picks** continues to your next selection after you make or simulate a pick. **Sim to end** asks for confirmation because it also simulates any remaining user-team picks.
- **Pick history** shows completed selections. **Team picks** lets you review one team's draft in order.

Computer-controlled teams choose from the available players using team-specific draft scores. Their needs change as their roster fills, so the player at the top of the board can differ by team.

## Save the completed draft

After every player has been selected, press **Write drafted rosters** and confirm. With **Create new save**, the app writes `DYNASTY-<your name>` beside the original and leaves the original unchanged. With **Write to selected save**, it replaces the selected save after staging and verification. The backup remains available in either case.

The app also balances left/right positions and duplicate jersey numbers on drafted teams before writing. Load the resulting dynasty in-game to confirm it works with your particular save. Keeping the original and its backup until you have loaded and advanced the new dynasty is recommended.

