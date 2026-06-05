# Stake Game Creation — Working Memory

## Project Goal
Submit one game per day to the Stake platform. A dedicated review team handles bouncing before submission.

## Team
| Name | Role | Notes |
|------|------|-------|
| Tiger | Operations Lead | Works across all areas as needed |
| Ollie | Review Team Lead | Owns QA and review before submission |
| Zacke | Game Team Lead | Owns game design and build |
| TBD | Designer / Animator | Actively recruiting — will join game team |

## Key Files
- `pipeline.md` — live daily tracker (what's in dev / review / submitted)
- `todo.md` — master priority tracker with blocking/research/setup/first-game tasks
- `game_ideas.md` — game concept library (reskins, CS2 case opening, Island Combat, Gacha NFT magnus opus)
- `submissions/log.md` — full history of submitted games
- `process/game-brief-template.md` — standard brief Zacke uses for each game
- `process/review-checklist.md` — Ollie's review criteria
- `process/submission-checklist.md` — final checks before sending to Stake
- `process/workflow.md` — full 3-phase dev/review/submission workflow with infrastructure diagram
- `resources/stake-requirements.md` — platform specs and technical limits
- `resources/stack-stake-sdk.md` — SDK stack guide (slots, PixiJS + Svelte 5)
- `resources/stack-hybrid.md` — hybrid stack guide (non-slots, Phaser 3)
- `docs/` — technical notes for Zacke (RGS API, math SDK, frontend SDK, approval guidelines)
- `docs/stake-engine-md-doc/` — offline mirror of stake-engine.com/docs (64 markdown files)
- `team/` — individual team member profiles

## Workflow (Daily Cadence)
1. Tiger assigns/briefs new game using `game-brief-template.md`
2. Zacke builds the game
3. Ollie reviews against `review-checklist.md`
4. If approved → Tiger submits via `submission-checklist.md` and logs in `submissions/log.md`
5. If rejected → bounced back to Zacke with notes

## Current Status
- Project setup: 2026-06-05
- Games submitted: 0
- Designer/Animator: HIRING
- ACP account: REGISTERED ✅ — Team name: **maxiai** — ACP: stake-engine.com/teams/maxiai
- Game concepts: ready in `game_ideas.md` (3 reskins + CS2 case + Island Combat + Gacha NFT)
- Payout model: 10% GGR perpetual royalty. At 96% RTP, avg game ~$636/month. Top games $5K–$10K/month.
- Payouts: monthly, closes UTC midnight, invoice on payments page, funds sent within 12h. Wallet address must be set.

## Notes for Claude
- I am the operations lead assistant and research assistant for this project
- Always check `pipeline.md` for current game status before making recommendations
- One game per day is the target cadence — flag anything that risks the pipeline
