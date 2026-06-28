# Frontend Brief — Ayakashi (game ID: `0_0_lines`)
_Prepared by Tiger for Tom. Branch: `5x5test`_

---

## Files You Need

| What | Path in repo |
|------|-------------|
| Game config (symbols, paylines, paytable) | `games/0_0_lines/game_config.py` |
| Game library (JSONL books served by RGS) | `games/0_0_lines/library/books/` |
| Published configs | `games/0_0_lines/library/publish_files/` |
| Lookup tables | `games/0_0_lines/library/lookup_tables/` |
| Freegame reel strip | `games/0_0_lines/reels/FR0.csv` |
| Base reel strip | `games/0_0_lines/reels/BR0.csv` |
| Frontend SDK docs | `docs/stake-engine-md-doc/` |

The library/books are what the RGS serves — you won't read them directly, but they need to be deployed alongside the frontend.

---

## Board Layout

- **Grid:** 5 reels × 5 visible rows (5×5)
- **Padding:** enabled — each reel has 1 hidden symbol above and 1 below the visible area
- **Total symbols per reel in events:** 7 (1 padding top + 5 visible + 1 padding bottom)
- **Row indexing in events:** row `0` = top padding, rows `1–5` = visible, row `6` = bottom padding

> **Important for rendering:** When an event gives `row: 3`, that means the 3rd visible row (0-indexed as row 2 internally, but events are +1 offset). Always subtract 1 to get the visible grid position.

---

## Symbols

| Symbol | Type | Description |
|--------|------|-------------|
| H1 | High pay | Highest value regular symbol |
| H2 | High pay | |
| H3 | High pay | |
| H4 | High pay | |
| L1 | Low pay | |
| L2 | Low pay | |
| L3 | Low pay | |
| L4 | Low pay | |
| L5 | Low pay | Lowest value regular symbol |
| W  | Wild | Substitutes for all pay symbols. In freegame only, carries a random **multiplier** value (see below) |
| S  | Scatter | 3/4/5 on board triggers free spins. Does not pay on lines |
| M  | FS Multiplier | At free spin trigger, multiplies the total FS count awarded |
| X  | Exploder (bomb) | On any win, destroys a 3×3 area centred on itself |

### Wild (W) multiplier values — freegame only
In the freegame, each W on the board is assigned a random multiplier from this weighted table:

| Multiplier | Weight |
|-----------|--------|
| 2×  | 60 |
| 3×  | 80 |
| 4×  | 50 |
| 5×  | 20 |
| 10× | 15 |
| 20× | 10 |
| 50× | 5  |

The multiplier value is attached to the symbol in the `reveal` event — see `symbol.multiplier` in the board data.

### M (fsMultiplier) table
Number of M symbols visible on the board when scatters trigger → total FS multiplier:

| M symbols | Multiplier |
|-----------|-----------|
| 1 | 2× |
| 2 | 3× |
| 3 | 5× |
| 4 | 10× |
| 5 | 20× |

---

## Paytable

Pays are in **bet multiples** (e.g. `10` = win 10× the bet).

| Symbol | 3-of-a-kind | 4-of-a-kind | 5-of-a-kind |
|--------|------------|------------|------------|
| W  | 10 | 20 | 50 |
| H1 | 10 | 20 | 50 |
| H2 | 3  | 5  | 15 |
| H3 | 2  | 3  | 10 |
| H4 | 1  | 2  | 8  |
| L1 | 1.0 | 1  | 5 |
| L2 | 0.6 | 0.7 | 3 |
| L3 | 0.6 | 0.7 | 3 |
| L4 | 0.4 | 0.5 | 2 |
| L5 | 0.3 | 0.3 | 1 |

W substitutes for all pay symbols. W pays use the W row above (not the substituted symbol's pay).

---

## Free Spins

### Trigger (base game)
- 3 scatters → 8 free spins
- 4 scatters → 12 free spins
- 5 scatters → 15 free spins

### Retrigger (during free spins)
- 2 scatters → +3 free spins
- 3 scatters → +5 free spins
- 4 scatters → +8 free spins
- 5 scatters → +12 free spins

### M symbol
Fires **after** the scatter count is set, before free spins begin. Multiplies `tot_fs` in place. The `fsMultiplier` event tells you the original count, multiplier, and new total.

### Wincap
**2000× the bet.** Once reached, `wincap` event fires and no further wins are added.

---

## Game Mechanics

### Tumble / Cascade
1. Board is revealed (`reveal` event)
2. Lines are evaluated — if any wins, `winInfo` fires
3. Winning symbols (+ any X bomb area) are marked for removal
4. `tumbleBoard` fires — shows what exploded and what fell in
5. New symbols drop in from the reel strip above
6. Lines are re-evaluated on the new board
7. Loop continues until no wins OR wincap reached

### X Bomb (Exploder)
- Only activates when a win occurs on the same spin
- Destroys all 9 positions in a 3×3 grid centred on the X symbol
- Affected symbols are removed in the same tumble as the winning line symbols
- If X is on the edge/corner, the 3×3 area is clipped to the board boundary

### Paylines
50 lines total. Pays left-to-right, 3 or more matching symbols starting from reel 0 (leftmost). W substitutes on any line.

---

## Event Reference

Every event has this base shape:
```json
{ "index": 0, "type": "eventType", ...fields }
```

`index` increments by 1 per event across the whole round.

### Amount encoding
All `amount` fields are **integer cents** — i.e. `win_in_bet_multiples × 100`.
- Amount `100` = 1× bet
- Amount `2000` = 20× bet
- Amount `200000` = 2000× bet (wincap)

---

### `reveal`
Fires at the start of every spin (base and free).

```json
{
  "index": 0,
  "type": "reveal",
  "board": [
    [ {"name":"H1"}, {"name":"W","wild":true,"multiplier":3}, ... ],
    ...
  ],
  "paddingPositions": [45, 112, 88, 203, 17],
  "gameType": "basegame",
  "anticipation": false
}
```

- `board[reel][row]` — reel 0–4, row 0–6 (0=top padding, 1–5=visible, 6=bottom padding)
- `paddingPositions` — reel strip index of the top padding symbol for each reel (used for Bet Replay)
- `gameType` — `"basegame"` or `"freegame"`
- `anticipation` — `true` if 2 scatters visible (show anticipation animation on reel 3+)
- Symbol fields: `wild`, `scatter`, `multiplier`, `fsMultiplier`, `exploder` — only present if `true`
- In freegame, W symbols carry `"multiplier": N` (the randomly assigned multiplier value)

---

### `winInfo`
Fires after every evaluation (initial board + after each tumble that produced wins).

```json
{
  "index": 1,
  "type": "winInfo",
  "totalWin": 300,
  "wins": [
    {
      "symbol": "H1",
      "kind": 3,
      "win": 1000,
      "positions": [
        {"reel": 0, "row": 2},
        {"reel": 1, "row": 2},
        {"reel": 2, "row": 2}
      ],
      "meta": {
        "winWithoutMult": 500,
        "overlay": {"reel": 2, "row": 3}
      }
    }
  ]
}
```

- `totalWin` — total win from this evaluation in cents
- `wins[].symbol` — symbol name
- `wins[].kind` — number of matching symbols (3, 4, or 5)
- `wins[].win` — win amount in cents (after multipliers, capped at wincap)
- `wins[].positions` — grid positions of the winning symbols (row is +1 offset — row 1 = top visible row)
- `wins[].meta.winWithoutMult` — pre-multiplier win (show base value separately if desired)
- `wins[].meta.overlay` — position to display the multiplier badge (present when W multiplier applies)

---

### `tumbleBoard`
Fires immediately after `winInfo`, before the board updates.

```json
{
  "index": 2,
  "type": "tumbleBoard",
  "explodingSymbols": [
    {"reel": 0, "row": 2},
    {"reel": 1, "row": 2},
    {"reel": 2, "row": 2}
  ],
  "newSymbols": [
    [{"name":"L3"}],
    [{"name":"H2"}],
    [{"name":"W","wild":true}],
    [],
    []
  ]
}
```

- `explodingSymbols` — all positions being removed (winning line symbols + X bomb area combined)
- `newSymbols[reel]` — array of symbols falling in from the top for each reel, in top-to-bottom order
- Empty array `[]` means no new symbols on that reel (no symbols were removed there)

---

### `setTumbleWin`
Updates the running tumble win counter banner (the accumulating total across cascades).

```json
{ "index": 3, "type": "setTumbleWin", "amount": 300 }
```

---

### `updateTumbleWin`
Updates the tumble win banner with the full spin win so far.

```json
{ "index": 4, "type": "updateTumbleWin", "amount": 1300 }
```

---

### `setWin`
Updates the main win ticker (per-spin).

```json
{
  "index": 5,
  "type": "setWin",
  "amount": 1300,
  "winLevel": "standard"
}
```

- `winLevel` — `"standard"`, `"bigWin"`, `"megaWin"`, `"epicWin"` — drives win celebration animation

---

### `setTotalWin`
Updates the total round win (base + freegame combined).

```json
{ "index": 6, "type": "setTotalWin", "amount": 1300 }
```

---

### `freeSpinTrigger`
Fires when scatters trigger free spins from the base game.

```json
{
  "index": 7,
  "type": "freeSpinTrigger",
  "totalFs": 24,
  "positions": [
    {"reel": 0, "row": 3},
    {"reel": 2, "row": 1},
    {"reel": 4, "row": 2}
  ]
}
```

- `totalFs` — final free spin count **after** M multiplier has been applied
- `positions` — reel positions of the scatter symbols (row +1 offset)

---

### `fsMultiplier`
Fires between scatter detection and `freeSpinTrigger` when M symbols are on the board.

```json
{
  "index": 7,
  "type": "fsMultiplier",
  "symbolCount": 2,
  "multiplier": 3,
  "originalFs": 8,
  "totalFs": 24,
  "positions": [
    {"reel": 1, "row": 2},
    {"reel": 3, "row": 4}
  ]
}
```

- `symbolCount` — number of M symbols visible
- `multiplier` — the multiplier applied (2/3/5/10/20)
- `originalFs` — free spin count before multiplication
- `totalFs` — free spin count after multiplication
- `positions` — reel positions of the M symbols

---

### `enterBonus`
Fires when the game transitions into free spin mode.

```json
{ "index": 8, "type": "enterBonus", "reason": "freegame" }
```

---

### `updateFreeSpin`
Fires at the start of each free spin.

```json
{
  "index": 9,
  "type": "updateFreeSpin",
  "amount": 1,
  "total": 24
}
```

- `amount` — current free spin number (1-indexed, increments each spin)
- `total` — total free spins awarded

---

### `freeSpinRetrigger`
Fires when scatters land during free spins (retrigger).

```json
{
  "index": 10,
  "type": "freeSpinRetrigger",
  "totalFs": 32,
  "positions": [{"reel": 1, "row": 2}, {"reel": 3, "row": 4}]
}
```

---

### `freeSpinEnd`
Fires at the end of all free spins.

```json
{
  "index": 11,
  "type": "freeSpinEnd",
  "amount": 45000,
  "winLevel": "endFeature"
}
```

- `amount` — total freegame win in cents
- `winLevel` — always `"endFeature"` — use for the freegame summary screen

---

### `wincap`
Fires when total win reaches 2000× the bet. No further wins are added after this.

```json
{ "index": 12, "type": "wincap", "amount": 200000 }
```

---

### `finalWin`
Last event in every round. The single source of truth for what the player is paid.

```json
{ "index": 13, "type": "finalWin", "amount": 45000 }
```

---

## Full Event Sequence Reference

### Base game — no win
```
reveal → finalWin
```

### Base game — win with tumble
```
reveal
→ winInfo → tumbleBoard → setTumbleWin
→ winInfo → tumbleBoard → setTumbleWin   (repeat per cascade)
→ updateTumbleWin → setWin → setTotalWin
→ finalWin
```

### Base game — scatter trigger
```
reveal
→ [tumble loop if wins]
→ updateTumbleWin → setWin → setTotalWin
→ fsMultiplier (if M symbols present)
→ freeSpinTrigger
→ enterBonus
→ [free spin loop]
→ freeSpinEnd
→ setTotalWin
→ finalWin
```

### Free spin (single spin within freegame)
```
updateFreeSpin
→ reveal
→ winInfo → tumbleBoard → setTumbleWin   (repeat per cascade)
→ updateTumbleWin → setWin → setTotalWin
→ freeSpinRetrigger (if 2+ scatters land)
```

### Wincap hit
```
... [normal sequence] ...
→ wincap
→ finalWin
```

---

## Key Notes for Frontend

1. **Row offset:** All row positions in events are `+1` from the internal grid. Row `1` = top visible row. Subtract 1 when mapping to your grid array.

2. **Amount is always integer cents.** Divide by 100 to get bet multiples for display.

3. **tumbleBoard fires before the board visually updates.** Play the explosion animation on `explodingSymbols`, then drop in `newSymbols` from the top.

4. **winInfo fires for every cascade evaluation**, not just the first. Listen for it in a loop until `updateTumbleWin` fires (that signals the cascade chain is done).

5. **W multiplier is on the symbol object in `reveal`.** If `symbol.multiplier` is present and > 1 on a wild in freegame, display the multiplier badge on that cell.

6. **X bomb**: the `explodingSymbols` array in `tumbleBoard` includes both the winning line positions AND the 3×3 bomb area combined — you don't need to calculate the bomb area yourself.

7. **fsMultiplier fires before freeSpinTrigger.** Animate the M symbols first, show the multiplied count, then transition to freegame.

8. **Anticipation:** `reveal.anticipation = true` means 2 scatters are visible. Show a heightened animation on the remaining reels while they spin.

9. **Wincap:** once `wincap` fires, freeze the win counter at 200000 (2000× bet). Do not process any more `winInfo` or `setWin` events after it.

---

## Math Summary (for display/UI calibration)

| Stat | Value |
|------|-------|
| RTP | 97% |
| Hit rate | ~29% (1 in 3.4 spins) |
| Max win | 2000× bet |
| Free spin trigger rate | ~1 in 200 base spins |
| Avg freegame win | ~74× bet |
| Paylines | 50 |
| Grid | 5×5 |

---

_Questions → Tiger. Math/SDK questions → Zacke._
