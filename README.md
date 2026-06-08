# Stake Game Creation

A game studio building and submitting casino games to the Stake Engine platform (Stake.com / Stake.us). The goal is a steady cadence of shipped games — reskins to keep the pipeline moving, with higher-quality 3-star games built in parallel.

---

## The Team

| Name | Role |
|------|------|
| Tiger | Operations Lead — pipeline, art (AI-generated), ACP submission, assists on frontend |
| Zacke | Backend Lead — math SDK, game logic, simulation |
| Nils | Backend Helper — pairs with Zacke, flexes to frontend/art pipeline when math is done |
| Tom | Frontend Lead — PixiJS + Svelte SDK, Storybook, animations |
| Ollie | Review Lead — checklist QA, bounce fix ownership with Tiger |
| TBD | Designer/Animator — actively hiring |

**Timezones:** Zacke, Nils, Ollie, Tiger are in Europe (CEST). Tom is in Taiwan (UTC+8, 6 hours ahead). Tiger works across both and bridges the handoff.

---

## How a Game Gets Built

The pipeline is follow-the-sun. The rough flow:

1. **European morning** — Zacke + Nils build the math. Tiger preps art, brief, and tile assets.
2. **Handoff** — Zacke drops the event structure to Tom in writing. Art + brief are ready and waiting.
3. **Taiwan evening** — Tom builds the frontend. Tiger assists. Game gets pushed to GitHub → Cloudflare auto-deploys.
4. **Next European morning** — Ollie reviews the Cloudflare preview. Either approves (Tiger submits to ACP) or bounces with notes (Tiger + Ollie fix — Tom always moves forward to the next game).

For a reskin, the goal is to complete all four steps in one day.

---

## Two Stacks

**SDK Stack — for slot games (reels-based)**
- Math: Python math SDK
- Frontend: PixiJS + Svelte 5, Storybook
- Use for: reskins, CS2 case opening, most submissions

**Hybrid Stack — for non-slot formats**
- Math: same Python math SDK
- Frontend: Phaser 3 + TypeScript
- Use for: Island Combat, RPG/adventure formats

---

## Folder Structure

```
Stake game creation/
  CLAUDE.md                    ← working memory (team, cadence, status)
  pipeline.md                  ← live tracker — check this for current game status
  todo.md                      ← master priority tracker
  game_ideas.md                ← concept library (reskins, CS2, Island Combat, Gacha)
  README.md                    ← this file

  team/
    tiger.md
    zacke.md
    nils.md
    tom.md
    ollie.md

  process/
    game-brief-template.md     ← Zacke uses this for every game
    review-checklist.md        ← Ollie's QA criteria
    submission-checklist.md    ← Tiger's pre-ACP checks
    workflow.md                ← full workflow + infrastructure diagram

  resources/
    stake-requirements.md      ← platform specs, tile asset requirements
    stack-stake-sdk.md         ← SDK stack guide (slots)
    stack-hybrid.md            ← Hybrid stack guide (non-slots)
    art-pipeline.md            ← art tools, sprite sheet workflow, free asset sources

  docs/
    setup-backend.md           ← dev environment setup for Zacke + Nils
    setup-frontend.md          ← dev environment setup for Tom
    mock-rgs-spec.md           ← Flask mock RGS spec (endpoints, responses, EC2 deploy)
    maths-guide.md             ← math standards + AI review checklist
    rgs-notes.md               ← Carrot RGS API reference
    math-sdk-notes.md          ← math SDK reference
    frontend-sdk-notes.md      ← frontend SDK reference
    approval-notes.md          ← submission + quality notes
    stake-engine-md-doc/       ← offline mirror of stake-engine.com/docs (64 files)

  submissions/
    log.md                     ← full submission history

  briefs/                      ← per-game brief files (created as games are assigned)
```

---

## Game Concepts

Three tiers, in order of priority:

**Tier 1 — Reskins (start here)**
Japanese-themed reskins of proven top performers. These validate the pipeline and generate royalties quickly.
- Reskin A — Underground fighting tournament (based on Dojo Duel, #10 performer)
- Reskin B — Yokai hunters (based on Harakiri, #13 performer)
- Reskin C — Anime idol group cluster pays (based on MIKO, #16 performer)

**Tier 2 — New Formats**
- CS2 Case Opening — loot box slot, SDK stack, CS2 aesthetic
- Island Combat — stateless RPG skin, Hybrid/Phaser stack
- Gacha PNG — collectible card pulls with downloadable PNGs

**Tier 3 — Magnus Opus**
- Gacha NFT — Gacha PNG + off-platform NFT claim portal. Build last, after Gacha PNG is approved and earning.

---

## Platform Rules (Key Points)

- All outcomes must be **pre-simulated** — no live RNG, fully stateless
- **Bet Replay is mandatory** — games without it will not be approved
- **Target RTP: 97%** (range: 96.5%–97.5%)
- Tile assets required per submission: BG + FG (under 3MB combined) + Provider logo
- Stake.us auto-qualified if `sweeps_en.json` language file is included
- Review: 3 independent reviewers — avg ≥ 2.0 stars → full review; < 2.0 → not published (can resubmit within 3 days)
- Post-approval: only minor visual fixes — math and mechanics are locked

---

## Payout Model

10% GGR perpetual royalty. At 97% RTP, the average game earns roughly $477/month in royalties. Top performers reach $3.75K–$7.5K/month. Payouts close monthly at UTC midnight — set your wallet address in ACP or the balance rolls over.

---

## Getting Started

**Backend (Zacke, Nils):** See `docs/setup-backend.md`

**Frontend (Tom):** See `docs/setup-frontend.md`

**Review (Ollie):** See `process/review-checklist.md`

**ACP:** stake-engine.com/teams/maxiai

**Current pipeline status:** See `pipeline.md`
