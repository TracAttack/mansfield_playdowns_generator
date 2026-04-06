# 28-Team Tournament Bracket Tool — User Guide

## Overview

This tool takes a ranked list of 28 teams, each assigned to a league shift, and automatically builds a tournament bracket. It ensures that teams playing each other in **Round 1** and **Round of 16** are never from the same shift — preventing scheduling conflicts. It then scores each valid bracket by how closely teams are placed to their seeded positions and how evenly shifts are distributed across the four quadrants of the bracket.

---

## Getting Started

Open `bracket.html` in any modern web browser (Chrome, Firefox, Edge, Safari). No installation, server, or internet connection required after the page loads.

---

## Step 1 — Paste Your 28 Teams

On the opening screen, paste your 28 teams into the text box, **one team per line, ranked 1 through 28** (rank 1 = strongest team first).

### Input Format

Each line must contain:
- The team's **shift** (day + time)
- The team's **name** (or name + seed identifier)

The parser is flexible. It accepts any of these formats:

```
Shift [Seed-Name]
Name  Shift
Shift Name
```

### Valid Shift Labels

| Shift | Accepted spellings |
|---|---|
| Tue Early | Tue Early, Tuesday Early |
| Tue Late | Tue Late, Tuesday Late |
| Wed Early | Wed Early, Wednesday Early |
| Wed Late | Wed Late, Wednesday Late |
| Thr Early | Thr Early, Thu Early, Thursday Early |
| Thr Late | Thr Late, Thu Late, Thursday Late |

### Example Input

```
Tue Late [1-Dave Dickinson]
Wed Late [2-Justin Ramm]
Wed Early [3-Kroy Nernberger]
Wed Late [4-Ken Neidhart]
Thr Late [5-Patrick Roe]
Thr Late [6-Steve Day]
Tue Late [7-Nick DeJongh]
Wed Late [8-Allan Veler]
Thr Early [9-Mike Fraboni]
Tue Early [10-Jon Orum]
Thr Late [11-Jesse Peterson]
Tue Early [12-Wesley Henry]
Tue Early [13-Douglas Jr Pahl]
Thr Early [14-Tim Schimick]
Wed Late [15-Matthew Hamilton]
Thr Early [16-Troy Mayne]
Tue Late [17-Matt Aro]
Tue Early [18-Darryl Mataya]
Wed Early [19-David Brown]
Wed Late [20-Paul Ryan]
Wed Early [21-David Solheim]
Thr Early [22-Chuck Soldner]
Thr Late [23-Dan Backes]
Wed Early [24-Mason Tikkanen]
Thr Early [25-Rob Wixson]
Thr Late [26-Bill McGlynn]
Wed Early [27-Robert Blakley]
Wed Late [28-Ben McPhee]
```

Click **Parse Teams** when done.

---

## Step 2 — Review & Fix Teams

The tool shows all 28 teams with their detected names and shifts. You can:

- Edit any **name** directly in the text field
- Change any **shift** using the dropdown
- Fix any errors flagged in the right column (missing name, missing shift, duplicate name)

Teams ranked 1–4 are highlighted in green — they receive **byes** in Round 1.

Click **Find Brackets** when all 28 teams show ✓.

---

## Step 3 — Find Solutions

Click **Find Solutions** to start the search. The tool runs a depth-first search (DFS) to find valid bracket arrangements and scores each one. It returns up to **100 results at a time**, sorted by total score.

Click **Next 100 Results** to continue searching for more arrangements.

---

## The Bracket Structure

The bracket is divided into **4 quadrants**, each containing:

- **1 bye team** (seeds 1–4, fixed)
- **3 Round 1 matchups** (6 teams per quadrant)

### Fixed Matchups (slots never move)

| Quadrant | Bye | R1 Pair A | R1 Pair B | R1 Pair C |
|---|---|---|---|---|
| Q1 | Seed 1 | 16 vs 17 | 9 vs 24 | 8 vs 25 |
| Q2 | Seed 4 | 13 vs 20 | 5 vs 28 | 12 vs 21 |
| Q3 | Seed 3 | 14 vs 19 | 6 vs 27 | 11 vs 22 |
| Q4 | Seed 2 | 15 vs 18 | 10 vs 23 | 7 vs 26 |

Within each quadrant, **Pair A** winner plays the bye in Round of 16. **Pair B** and **Pair C** winners play each other in Round of 16.

> **The slot numbers are fixed.** The tool assigns *teams* to slots — so a team ranked 9th may be placed into slot 9 (exact) or moved to a nearby slot if a clash forces it.

---

## Clash Rules

The tool enforces two hard constraints:

**Round 1 clash** — the two teams playing each other in a Round 1 game must not be on the same shift.

**Round of 16 clash** — teams who could meet in the Round of 16 must not be on the same shift. This includes:
- The bye seed vs either team from Pair A
- Pair B winner vs Pair C winner (all 4 combinations)

No bracket solution will violate either of these rules. QF (quarterfinal) shift conflicts are tracked and reported but do not disqualify a result.

---

## How Teams Are Placed (DFS with Backtracking)

Teams are placed in **3 phases** in this order:

1. **Phase 1** — Bye seeds 1–4 (fixed, always exact)
2. **Phase 2** — Upper seeds 5–16 (ascending order: 5, 6, 7 … 16)
3. **Phase 3** — Lower seeds 28–17 (descending order: 28, 27, 26 … 17)

For each team, the tool only considers slots **within ±3 of that team's home slot** (e.g. seed 9 can only go into slots 6–12). This keeps placements close to seed while focusing the search.

The tool greedily picks the first available slot with no R1 or R16 clash. If no clean slot exists within the window, it **backtracks** to a previous team and tries the next available option — following the same phase order in reverse.

---

## Scoring

Each result is scored on two dimensions:

### 1. Seed Fidelity (max 28 pts)

How close each team is to their natural seed position:

| Slot difference | Points |
|---|---|
| 0 — exact | 1.0 |
| 1–3 slots off | 0.5 |
| 4 slots off | 0.3 |
| 5 slots off | 0.2 |
| 6 slots off | 0.1 |
| 7+ slots off | 0.0 |

### 2. Shift Distribution (max 28 pts)

For each shift, 1 point is awarded per team **if** that shift's teams are well-distributed across the four quadrants:

| Teams in shift | Passes if… |
|---|---|
| 1 | Always passes |
| 2 | Both teams in different quadrants |
| 3 | All 3 teams in different quadrants |
| 4 | All 4 teams in different quadrants |
| 5 | All 4 quadrants represented, no quadrant has more than 2 |
| 6 | All 4 quadrants represented, no quadrant has more than 2 |

If a shift does **not** pass, every team in that shift scores **0** distribution points.

### Total Score

```
Total = Seed Fidelity + Distribution Score
Max   = 28 + 28 = 56 pts
```

Results are sorted by total score, highest first.

---

## Reading the Results Table

Each row in the results table shows:

| Column | Description |
|---|---|
| # | Result rank (1 = best total score) |
| Fidelity | Seed fidelity sub-score |
| Distrib. | Distribution sub-score |
| Total | Combined score |
| Score % | Total as % of maximum (56) |
| Slots 1–28 | Color-coded cell showing which team seed occupies each slot |
| View → | Click to open the full bracket view |

### Slot Color Legend

| Color | Meaning |
|---|---|
| Dark green | Team placed in exact home slot |
| Light green | 1–3 slots from home |
| Amber | 4 slots from home |
| Orange | 5 slots from home |
| Salmon | 6 slots from home |
| Red | 7+ slots from home |

---

## Viewing a Result

Clicking **View →** on any result opens the full detail view, which includes:

1. **Bracket SVG** — the full visual bracket with all teams placed, color-coded by shift
2. **Score panel** — fidelity and distribution sub-scores with breakdown tables
3. **R1 & R16 Clash Check** — every possible R1 and R16 matchup per quadrant with ✅ / 🔴 indicators
4. **Full Placement Audit** — every team listed in placement order (Phase 1 → 2 → 3), showing which slot was chosen, how far from home, and — for moved teams — every candidate slot that was considered and why it was skipped

Click **← Back to results** to return to the table without losing your search progress.


---

## Prompt Engineering History

This tool was built iteratively through a conversation with Claude (Anthropic). The following is a complete summary of every design decision made, in order, so that an identical implementation can be regenerated from scratch.

---

### 1. Initial setup — React bracket builder

Start with a React artifact (Claude Sonnet in claude.ai). The base app should:

- Accept a paste of 28 teams with names and shifts
- Parse team name and shift from each line (flexible: tab-separated, double-space, or inline)
- Show a review screen where names and shifts can be edited
- Validate: no missing names, no missing shifts, no duplicate names

---

### 2. Bracket structure — fixed slot positions

The bracket has **4 quadrants**, each with 1 bye seed and 3 Round 1 pairs. Slot numbers are **permanently fixed** — the tool assigns teams to slots, not the other way around.

```
Bye seeds:   [1, 4, 3, 2]  (one per quadrant, Q1–Q4)

SLOT_UPPERS: [[16,9,8], [13,5,12], [14,6,11], [15,10,7]]
SLOT_LOWERS: [[17,24,25],[20,28,21],[19,27,22],[18,23,26]]
```

Within each quadrant:
- `si=0` (pair 0) = bye-facing pair — winner plays bye in R16
- `si=1` and `si=2` (pairs 1+2) = cross pair — winners play each other in R16

The SVG bracket renders left side (Q1 top, Q2 bottom) and right side (Q3 bottom, Q4 top) with round columns: Round 1 → Round of 16 → Quarterfinal → Semifinal → Championship.

Each slot box shows: fixed slot number badge, assigned team name, team rank, and a "moved from" label if not in home slot. Boxes are color-coded by shift.

---

### 3. Clash scoring — three tiers per slot

For each candidate slot when placing a team, compute three binary clash scores:

- **r1** — does this team's shift match their direct Round 1 opponent (upper↔lower in same pair)?
- **r16** — does this team's shift match any other team in the same group within the quadrant?
  - Group A (bye-facing): bye + pair 0 upper + pair 0 lower
  - Group B (cross pair): pairs 1 and 2 — all 4 teams can meet each other in R16
- **qf** — does this team's shift match any team in the opposite group (cross-quadrant at QF stage)?

All three are binary (0 or 1). Hard constraints are **r1=0 AND r16=0**. qf conflicts are tracked but do not block placement.

---

### 4. Candidate window — ±3 slots from home

When looking for a slot to place a team, only consider slots **within ±3 of the team's home slot number**, in **phase order**:

- Phase 2 (upper seeds 5–16): scan slots in order 5, 6, 7 … 16
- Phase 3 (lower seeds 28–17): scan slots in order 28, 27, 26 … 17

Filter that ordered list to only slots where `|slot - homeSlot| <= 3`. If no clean slot exists in this window, it is a dead end — do not go outside the window.

---

### 5. DFS with automatic backtracking

Placement follows a **depth-first search** in three phases:

```
Phase 1: Bye seeds 1–4 (fixed, not placed by DFS)
Phase 2: Upper seeds 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16
Phase 3: Lower seeds 28, 27, 26, 25, 24, 23, 22, 21, 20, 19, 18, 17
```

At each step:
1. Generate candidates in phase order within ±3 window
2. Greedily take the **first** candidate with r1=0 and r16=0 (short-circuit)
3. If no clean candidate exists → dead end → **automatically backtrack**

Backtracking: pop the top frame off the stack, re-fetch candidates for that team with restored quads, find the **next** clean candidate after the previously chosen index. If none, pop again. Repeat until a new branch is found or the stack is empty (fully exhausted).

Each stack frame stores: `{teamSeed, candidates, chosenIdx, quadsAfter}`.

`advancePastSolution` continues from a completed solution by treating it as a dead end and backtracking from the final frame.

---

### 6. Results collection — find 100 at a time

Run the DFS in **batched ticks** (50 solutions per setTimeout tick) so the UI stays responsive. For each complete solution:

1. Score it (fidelity + distribution)
2. Add to results array
3. Call `advancePastSolution` to advance to the next

Stop after 100 results per batch. Sort all results by **total score descending**. The "Next 100 Results" button continues from the last DFS position.

---

### 7. Seed fidelity scoring (max 28 pts)

For each of the 28 teams, compare the team's rank to the slot they were placed in:

```
diff = |team_rank - slot_number|

diff = 0  → 1.0 pt  (exact)
diff 1–3  → 0.5 pt
diff = 4  → 0.3 pt
diff = 5  → 0.2 pt
diff = 6  → 0.1 pt
diff ≥ 7  → 0.0 pt
```

Bye seeds (1–4) always score 1.0 (they are never moved).

---

### 8. Shift distribution scoring (max 28 pts)

For each shift, check whether its teams are well-spread across the 4 quadrants:

```
count = 1          → always passes
count 2–4          → all teams must be in different quadrants (maxInAny === 1)
count 5–6          → all 4 quadrants represented AND maxInAny <= ceil(count/4)
```

If a shift **passes**: every team in that shift earns **1 distribution point**.
If a shift **fails**: every team in that shift earns **0 distribution points**.

---

### 9. Total score and sorting

```
Total = fidelityTotal + distTotal
Max   = 28 + 28 = 56
Pct   = round(total / 56 * 100)
```

Results are sorted by `total` descending. The results table shows Fidelity, Distrib., Total, and Score % columns, plus a compact 28-cell color-coded grid showing which team seed occupies each slot.

---

### 10. Results table — 28-column slot grid

Each row in the results table has 28 small colored cells (one per bracket slot, columns 1–28). Each cell shows the **team seed** placed in that slot and is background-colored by fidelity distance:

```
Exact        → dark green  (#EAF3DE / #27500A)
±1–3         → light green (#C0DD97 / #27500A)
±4           → amber       (#FAEEDA / #412402)
±5           → orange      (#FAC775 / #412402)
±6           → salmon      (#F5C4B3 / #501313)
7+           → red         (#FCEBEB / #791F1F)
```

---

### 11. Detail view — per result

Clicking a result opens a full detail page with four sections in order:

1. **Bracket SVG** — full bracket visualization, shift-colored boxes, slot number badges, "moved" labels
2. **Score panel** — total score, sub-scores (fidelity + distribution), fidelity breakdown table (all 28 teams with rank/slot/diff/pts), shift distribution table (per-shift Q1–Q4 counts, quads used, max allowed, pts)
3. **R1 & R16 Clash Check** — per quadrant, every R1 matchup (3 rows) and R16 matchup (6 rows) with ✅ / 🔴
4. **Full Placement Audit** — teams listed in phase order (Phase 1 → 2 → 3) with phase headers. For each team: slot chosen, diff from home, points earned. For moved teams: full candidate table showing every slot in the ±3 window with r1/r16/qf marks, conflict reason, and ✅ chosen / ❌ skipped / — not reached

---

### 12. Output format — standalone HTML

The final deliverable is a **single self-contained HTML file** that:

- Loads React 18, ReactDOM, and Babel standalone from Cloudflare CDN
- Requires no build step, no npm, no server
- Can be hosted on GitHub Pages by naming it `index.html` and enabling Pages from the repo root

To republish: create a GitHub repo, commit `index.html`, enable Settings → Pages → Deploy from branch → main / root.


- **More results = better options.** The first 100 results are found quickly but may not be the best. Click "Next 100 Results" a few times to explore more of the solution space.
- **85% is a strong result.** A total score above 85% (≥ 47.6 pts out of 56) means most teams are very close to their seed and shifts are well distributed.
- **The ±3 window is intentional.** Teams will never be placed more than 3 slots from home. If no valid solution can place a team within that window, the DFS backtracks rather than placing them far away.
- **Bye seeds always score 1.0.** Seeds 1–4 are fixed in their bye positions and are never moved.
