# Stake Game Creation — Working Memory

## Project Goal
Submit one game per day to the Stake platform. A dedicated review team handles bouncing before submission.

## Team
| Name | Role | Notes |
|------|------|-------|
| Tiger | Operations Lead | Coordinates pipeline, does art (AI-generated) when no designer, supports where needed |
| Ollie | Review Lead | Checklist-based QA before submission; leads bounce fixes with Tiger + backend support |
| Zacke | Backend Lead | Math SDK lead — game_config, gamestate, simulation runs |
| Nils | Backend Helper | Pairs with Zacke on math; flexes to frontend/art support when math is done |
| Tom | Frontend Lead | Leads frontend build, Storybook, animations, asset integration |
| TBD | Designer / Animator | Actively recruiting — critical for 3-star quality; Tiger covers art until hired |

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
- `resources/art-pipeline.md` — art tools (ComfyUI + Flux1, SEELE, Kenney, Spine, Live2D) and asset checklist
- `docs/` — technical notes for Zacke (RGS API, math SDK, frontend SDK, approval guidelines)
- `docs/stake-engine-md-doc/` — offline mirror of stake-engine.com/docs (64 markdown files)
- `team/` — individual team member profiles

## Working Hours
- Zacke, Nils, Ollie: mornings only (CET)
- Tom: evenings only (Taiwan time) = 6pm–11pm TWN / 12pm–5pm CET
- Tiger: full day, both timezones — bridges Europe morning and Tom's evening

## Workflow (Daily Cadence)

**Day before:** Tiger generates AI art assets for next game (ComfyUI + Flux 2 locally, RunComfy for final quality). Assets must be ready before build day starts — art is the critical path.

**European morning (9am–12pm CET):**
1. Morning sync (15 min) — Tiger briefs the day. Zacke + Nils start math. Assets confirmed ready for Tom.
2. Zacke + Nils build math (game_config + gamestate) — ~2 hours
3. Tiger handles tile assets, submission blurb, and art prep in parallel
4. Zacke + Nils finish math → flex to spritesheet processing (TexturePacker), Bet Replay event IDs
5. At 12pm CET: Zacke drops event structure to Tom in writing. European team done for the day.

**Tom's evening (6pm–11pm TWN / 12pm–5pm CET):**
6. Tom comes online — receives assets, brief, and event structure
7. Tom builds frontend. Tiger assists on frontend during this window.
8. Tom pushes to GitHub → Cloudflare deploys

**Next European morning (9am CET):**
9. Ollie runs full checklist review against Cloudflare preview
10. If approved → Tiger submits to ACP, logs in `submissions/log.md`, updates `pipeline.md`
11. If bounced → Tiger + Ollie handle all fixes. **Tom is never pulled into bounce fixes — he starts the next game.**

**If bounced (avg initial rating < 2.0 stars):**
- Tiger + Ollie own all fixes — Tom is protected, always on the next game
- Zacke + Nils support if fixes are backend-related
- 3-day resubmission window — treat as urgent
- Tiger resubmits once fixed

## Current Status
- Project setup: 2026-06-05
- Games submitted: 0
- Designer/Animator: HIRING
- ACP account: REGISTERED ✅ — Team name: **maxiai** — ACP: stake-engine.com/teams/maxiai
- Game concepts: ready in `game_ideas.md` (3 reskins + CS2 case + Island Combat + Gacha NFT)
- Payout model: 10% GGR perpetual royalty. At 97% RTP, avg game ~$477/month. Top games $3.75K–$7.5K/month.
- Payouts: monthly, closes UTC midnight, invoice on payments page, funds sent within 12h. Wallet address must be set.

## Timezones
- Tiger, Zacke, Nils, Ollie: Europe (CEST, UTC+2)
- Tom: Taiwan (UTC+8) — 6 hours ahead
- Tiger works across both timezones, focus on Tom's overlap window
- Overlap window: 9am–1pm CET (3pm–7pm Taiwan) — only time all parties available simultaneously

## General Cadence (flexible — not strict times)

The rough flow each day tends to be:
- Europe works in the morning — math gets built, previous game gets reviewed, fixes handled, submission done
- Tom works in his evening (Taiwan) — frontend build, pushes to GitHub, Cloudflare deploys
- Tiger bridges both — assists Tom on frontend, coordinates handoffs, generates art

Math is typically done before Tom starts, so he has everything ready when he comes online. Ollie reviews the Cloudflare preview and either ships or writes bounce notes. Tiger and Ollie handle all fixes — Tom always moves forward onto the next game.

## Weekly Cadence (3 reskins + 3-star track)
| Day | Europe (morning) | Tom (evening) |
|-----|-----------|---------------|
| **Mon** | Reskin 1 math. Ship anything held from Fri. | Reskin 1 frontend |
| **Tue** | Review + ship Reskin 1. 3-star math prep. | 3-star session |
| **Wed** | Reskin 2 math | Reskin 2 frontend |
| **Thu** | Review + ship Reskin 2. 3-star math prep. | 3-star session |
| **Fri** | Reskin 3 math | Reskin 3 frontend |
| **Mon** | Review + ship Reskin 3. New cycle begins. | — |

**3-star track:** Runs Tue + Thu evenings indefinitely. No deadline — rolls until ready. Likely 4–8+ sessions (2 weeks to a month+). Never blocks the reskin pipeline.

## Ramp-Up Schedule
| Date | Milestone |
|------|-----------|
| 2026-06-06 (Sat) | Tiger meets Tom — confirm frontend lead |
| 2026-06-08 (Mon) | Full team meeting + dev environment setup. Everyone runs test game by end of day. |
| 2026-06-10 (Tue) | First game build starts |
| Week of 2026-06-08 | **Target: 1 game submitted** |
| Week of 2026-06-15 | **Target: 3 games submitted** |
| Week of 2026-06-22+ | **Steady state: 3 games/week + 3-star track 2 days/week** |

## Pre-Monday Blockers
- [ ] Mock RGS deployed on EC2 (Tiger — today)
- [ ] Tom confirmed (Tiger — Saturday)
- [ ] Web SDK clone URL pulled from ACP (Tiger — before Monday)
- [ ] Tom's dev environment ready: Node 18.18.0, pnpm 10.5.0 (Tom — before Monday)
- [ ] First game brief written and ready to assign at Monday meeting (Tiger)

## Notes for Claude
- I am the operations lead assistant and research assistant for this project
- Always check `pipeline.md` for current game status before making recommendations
- One game per day is the target cadence — flag anything that risks the pipeline
