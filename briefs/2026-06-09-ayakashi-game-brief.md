# Game Brief #001 - Ayakashi
**Date Issued:** 2026-06-09
**Issued By:** Tiger
**Assigned To:** Zacke
**Target Submission Date:** 2026-06-13 (end of week 1)

---

## Game Overview

**Working Title:** Ayakashi (Dark Yokai Slots)
**Game Type:** Slots
**Concept Summary:** A dark Japanese yokai slot set in a cursed shrine. Players spin through supernatural spirits and sacred artifacts across a 5x5 reel grid. Two super symbols deliver distinct bonus mechanics - a free spin multiplier and a 3x3 grid reroll.

---

## Mechanics

**Core Loop:** Standard slot spin. 5 reels x 5 rows. Wins evaluated on paylines (TBD - confirm with Zacke: ways-to-win or fixed lines).

**Win Conditions:** Matching symbols on active paylines left to right.

**Symbol Set:**

| Symbol | ID | Tier | Notes |
|--------|----|------|-------|
| Fox Tail / Kitsune | H1 | High | |
| Oni Mask | H2 | High | |
| Dragon Serpent | H3 | High | |
| Tengu Warrior Mask | H4 | High | |
| Sake Vessel | H5 | High | |
| Bamboo | L1 | Low | |
| Paper Lantern | L2 | Low | |
| Shuriken | L3 | Low | |
| Torii Gate | L4 | Low | |
| 5th Low Symbol | L5 | Low | |
| Kitsune Spirit Orb | W | Wild | Substitutes all standard symbols |
| Ancient Temple Bell | S | Scatter | Triggers free spins |
| Ofuda Talisman | X | Super | Free spin multiplier (see below) |
| Oni Kanabo (iron club) | X2 | Super | 3x3 grid reroll (see below) |

---

## Special Features

**Free Spins**
Triggered by 3+ Scatter (S) landing anywhere on the grid. Free spin count TBD by Zacke (suggest 8/12/15 for 3/4/5 scatters).

**Ofuda Multiplier (X symbol)**
- When Ofuda lands during free spins, it applies a multiplier to that spin's win
- Multiplier value TBD - suggest x2, x3, x5 with weighted probability
- Confirm with Zacke: do multiple Ofuda stack in one spin?

**Oni Kanabo Reroll (X2 symbol)**
- When the iron club lands, all symbols in the 3x3 grid surrounding it are rerolled
- Rerolled symbols are drawn fresh - can produce wins or higher value symbols
- Applies in both base game and free spins
- Edge/corner positions use available cells only, no wrap-around
- Kanabo itself stays in place after triggering

**RTP Target:** 97%
**Volatility:** Medium-High

---

## Visual Direction

**Theme:** Dark Japanese yokai. Demon Slayer colour richness meets traditional woodblock art. Every symbol belongs in the same cursed shrine.

**Colour Palette:**
- Backgrounds: near-black, deep indigo, dark crimson
- High symbols: rich jewel tones - crimson, jade, silver, violet, rose gold
- Specials: gold (W), moonlight silver (S), burning gold (X), volcanic red-black (X2)

**References:** Demon Slayer, Dororo, Jujutsu Kaisen. Traditional Japanese lacquerwork and ink textures.

**Art Status:** All 14 symbols generated and SDK-ready. See briefs/2026-06-09-ayakashi-tom-handoff.md for full asset manifest.

**Animation:** Tiger handling all Spine animations in parallel. Tom does not need to wait - static sprites cover all states until Spine files are delivered.

---

## Technical Requirements

**Platform:** Stake Engine
**Grid:** 5x5 (5 reels, 5 rows)
**Format:** Refer to resources/stake-requirements.md

**SDK Config Changes from Reference Game:**
- INITIAL_BOARD: 5 columns x 7 rows (5 visible + 2 padding top/bottom)
- BOARD_DIMENSIONS: { x: 5, y: 5 } (auto-calculated from INITIAL_BOARD)
- Add X and X2 to symbol map and math config
- L5 is a standard sprite symbol (not the Spine multiplier from the reference game)

**Bet Replay:** Required. Zacke to confirm event IDs and drop structure to Tom at 12pm CET before Tom's build session.

**Known Constraints:**
- Stateless - no jackpots, no continuation rounds
- Kanabo reroll result must be pre-simulated (not live RNG at runtime)
- Ofuda multiplier values must be defined in game_config

---

## Acceptance Criteria

Ollie uses these during review.

- [ ] 5x5 grid renders correctly on desktop and mobile
- [ ] All 14 symbols display: H1-H5, L1-L5, W, S, X, X2
- [ ] Wild (W) substitutes all standard symbols correctly
- [ ] Scatter (S) triggers free spins on 3+ hits
- [ ] Ofuda (X) applies multiplier during free spins
- [ ] Kanabo (X2) rerolls correct 3x3 grid on land
- [ ] Bet Replay works - all states replayable via URL params
- [ ] Sweepstakes language file present (sweeps_en)
- [ ] Mobile layout correct (portrait and landscape)
- [ ] No Stake branding in assets
- [ ] RTP simulation passes math validation in ACP

---

## Notes for Zacke

- X2 (Kanabo) reroll must be pre-computed in simulation and included in the play response, not computed client-side
- Confirm whether Ofuda multipliers stack if multiple X land in one free spin
- Confirm free spin count (suggest 8/12/15 for 3/4/5 scatters)
- Confirm payline structure - recommend ways-to-win for 5x5 grid (3125 ways) over fixed lines
- Drop event structure to Tom in writing at 12pm CET on build day
