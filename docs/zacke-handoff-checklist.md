# Zacke's Handoff Checklist

Everything Tom needs before he can start. Deliver this in writing before 12pm CET.

---

## 1. Math Output Files — Complete and Ready to Drop In

- [ ] Production sim run complete (`int(1e6)` sims)
- [ ] `library/stats_summary.json` validated against `docs/maths-guide.md`:
  - [ ] RTP 97% ± 0.5% on all modes
  - [ ] All modes within 0.5% RTP of each other
  - [ ] Hit rate 15%+ (20–30% for medium volatility)
  - [ ] Base/bonus RTP split ~60/40
  - [ ] Dead spin runs: P(6+ consecutive zeros) < 1%
  - [ ] Win distribution: no gaps > 3× between adjacent tiers
- [ ] `library/publish_files/` folder complete:
  - [ ] `books_base.jsonl.zst`
  - [ ] `books_bonus.jsonl.zst` (if bonus mode)
  - [ ] `index.json`
  - [ ] `lookUpTable_base_0.csv`
  - [ ] `lookUpTable_bonus_0.csv` (if bonus mode)
- [ ] `library/configs/config_fe_<game_id>.json` generated and correct:
  - [ ] Game name and gameID set correctly
  - [ ] Grid dimensions correct (numReels, numRows)
  - [ ] All bet modes defined with correct cost and max_win
  - [ ] Paylines correct
- [ ] `library/configs/event_config_base.json` generated
- [ ] `library/configs/event_config_bonus.json` generated (if applicable)
- [ ] Symbol names in event configs match Tiger's agreed asset filenames

**How to build this:** `docs/zackes-flow.md` steps 2–6
**RTP standards:** `docs/maths-guide.md`

---

## 2. BookEvent Structure — Written Description

For every event the game outputs, Tom needs the name and exact payload shape. Standard events are already handled by the SDK — only document custom ones.

**Standard events (no need to document, Tom knows these):**
- `reveal`, `winInfo`, `setTotalWin`, `freeSpinTrigger`, `updateFreeSpin`, `freeSpinEnd`, `setWin`, `finalWin`

**Custom events — document each one:**

For each custom event, provide:
- Event name (must match exactly what gamestate.py outputs)
- Payload shape with field names and types
- When it fires in the sequence
- Edge cases (e.g. "this event fires 0–5 times, may not appear at all")

Example format:
```
cascadeExplode
  payload: { positions: Array<{ reel: number, row: number }> }
  fires: after every winning board evaluation
  edge cases: fires 0 times if no cascade occurs; max 5 times per spin

cascadeFill
  payload: { newSymbols: Array<{ reel: number, row: number, name: string }> }
  fires: immediately after each cascadeExplode
  edge cases: always paired with cascadeExplode, never fires alone
```

**How events work:** `docs/how-games-work-sdk-stack.md`  
**How Tom uses this:** `docs/toms-flow.md` step 5  
**Adding new events (frontend side):** `docs/stake-engine-md-doc/front-end/adding-new-events.md`  
**Event structure reference:** `docs/stake-engine-md-doc/math/game-state-structure/events.md`

---

## 3. Bet Replay Event IDs

Ollie needs specific simulation IDs to test Bet Replay during review. Without these, Ollie cannot complete the review checklist and the game cannot be submitted.

Provide one simulation ID for each of the following:

- [ ] Normal win (small, nothing special)
- [ ] Big win (hits the big win celebration threshold)
- [ ] Max win (or as close as available in the sim output)
- [ ] Loss (no win, round ends cleanly)
- [ ] Bonus trigger (scatter lands, free spins activate)

These are IDs from `library/publish_files/` — the specific event identifiers Ollie enters into the replay URL.

**Why this matters:** `docs/stake-engine-md-doc/approval-guidelines/game-replay-requirements.md`  
**How replay works:** `docs/mock-rgs-spec.md` (replay endpoint section)  
**Review checklist context:** `process/review-checklist.md`

---

## 4. Confirm Art Assets Are Ready

Before handing off to Tom, confirm with Tiger that assets are ready and in the right format:

- [ ] All symbol `.webp` files named to match the symbol names in config.ts
- [ ] symbolsStatic atlas packed (TexturePacker output: `.json` + spritesheet `.png`)
- [ ] Spine win animation assets available (`.atlas` + `.json` per symbol, or confirmed reusing existing skeletons)
- [ ] Background asset ready
- [ ] Audio file ready (`sounds.json`)

If TexturePacker hasn't been run yet — Zacke or Nils handles this as flex work after math is done.

**Art pipeline reference:** `resources/art-pipeline.md`  
**Asset format requirements:** `resources/stake-requirements.md`

---

## Quick Reference — What Each File Is For

| File | Owner | Purpose |
|------|-------|---------|
| `game_config.py` | Zacke | Math definition — symbols, paytable, reel strips, bet modes |
| `gamestate.py` | Zacke | Spin logic — what happens each round, BookEvent output |
| `publish_files/` | Zacke → Tom | Compressed books + LUTs. Mock RGS reads these directly. |
| `config_fe_<id>.json` | Zacke → Tom | Frontend game config — grid, paylines, RTP, bet modes. Tom does not edit. |
| `event_config_base.json` | Zacke → Tom | Base game BookEvent examples. Tom copies into Storybook data files. |
| `event_config_bonus.json` | Zacke → Tom | Bonus BookEvent examples (if applicable). |
| `bookEventHandlerMap.ts` | Tom | Maps BookEvents to animations. Only needs editing for custom events. |
| `constants.ts` | Tom | Asset keys, symbol size ratios, spin options |

---

_See also: `docs/zackes-flow.md` for the full step-by-step math process_  
_See also: `docs/toms-flow.md` for what Tom does with this handoff_  
_See also: `process/workflow.md` for the full pipeline from brief to submission_
