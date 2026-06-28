# Stake Game Studio

Two-person studio building casino games for the Stake Engine platform (Stake.com / Stake.us).

## Team
- **Max** (Tiger) — art, frontend, submissions, creative direction
- **Zacke** — math SDK, game logic, Python

No formal roles. Both collaborate on design and direction based on what's needed.

ACP: [stake-engine.com/teams/maxiai](https://stake-engine.com/teams/maxiai)

---

## Games

| Game | Status | Branch |
|------|--------|--------|
| Ayakashi (yokai slots, v1) | Submitted | `ayakashi1` (frozen) |
| Ayakashi 2 | In development | `final-dev` |

---

## Platform
**Stake Engine** — pre-simulated outcomes, 97% RTP target, stateless architecture, Bet Replay mandatory.
Review: 3 independent reviewers, avg ≥2.0 stars → published. 10% GGR perpetual royalty.

---

## Tech Stack
- **Math**: Python math-sdk — generates CSV probability tables + event configs
- **Frontend**: PixiJS + Svelte 5 + SvelteKit monorepo (web-sdk)
- **Deploy**: GitHub → Cloudflare Pages (auto), Mock RGS on EC2 for testing

---

## Folder Structure

```
Stake game creation/
  CLAUDE.md                    ← working memory for Claude (tech decisions, locked choices, context)
  README.md                    ← this file
  pipeline.md                  ← live tracker — current game status
  todo.md                      ← current priority list
  game_ideas.md                ← concept library

  team/
    tiger.md                   ← Max's profile
    zacke.md                   ← Zacke's profile

  process/
    game-brief-template.md     ← brief template for each new game
    submission-checklist.md    ← pre-ACP self-review checklist
    workflow.md                ← how a game gets built start to finish

  resources/
    art-pipeline.md            ← full art + animation pipeline (fal.ai + RunComfy)
    stake-requirements.md      ← platform specs and technical limits
    stack-stake-sdk.md         ← SDK stack guide (slots)
    stack-hybrid.md            ← Hybrid stack guide (non-slots, Phaser 3)

  docs/
    setup-backend.md           ← math SDK dev environment setup
    setup-frontend.md          ← web-sdk dev environment setup
    mock-rgs-spec.md           ← Flask mock RGS spec
    maths-guide.md             ← math standards + validation rules
    rgs-notes.md               ← Carrot RGS API reference
    math-sdk-notes.md          ← math SDK reference
    frontend-sdk-notes.md      ← frontend SDK reference
    approval-notes.md          ← submission quality notes
    how-games-work-sdk-stack.md
    how-games-work-hybrid-stack.md
    stake-engine-md-doc/       ← offline mirror of stake-engine.com/docs (64 files)

  briefs/                      ← per-game brief files
  submissions/
    log.md                     ← submission history
```

---

## Game Concepts

See `game_ideas.md` for the full library. Current focus is Ayakashi 2.

Highest priority future games:
- **CS2 Case Opening** — loot box slot, SDK stack, proven format on Stake
- **Yokai Hunters** — Harakiri-based reskin, dark folklore aesthetic
- **Island Combat** — stateless RPG skin, Phaser 3 hybrid stack
- **Gacha PNG** — collectible card pulls, eventually with NFT layer

---

## Art Pipeline Overview

See `resources/art-pipeline.md` for the full pipeline.

**Stills**: FLUX 1.1 Pro Ultra + Seedream v4 via fal.ai. AuraSR 4x upscale. Manual background removal by Max.

**Character animations (Tier 1)**: RunComfy SVI Pro — Wan 2.2 + Stable Video Infinity. ~$2–5/run.

**FX/particles (Tier 2)**: fal.ai Wan or Hailuo → 16-frame 4×4 WebP sprite sheets → PixiJS AnimatedSprite.

---

## Platform Key Rules
- All outcomes pre-simulated — no live RNG, fully stateless
- Bet Replay mandatory (URL param `bet=<event_id>`)
- Target RTP: 97% (range 96.5–97.5%)
- Tile assets: BG + FG under 3MB combined + Provider logo
- Stake.us: include `sweeps_en.json` language file
- No external URLs in game code (strict XSS policy)
- Original games only, no Stake branding, no IP infringement
