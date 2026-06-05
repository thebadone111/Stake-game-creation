# Project Todo List

_Last updated: 2026-06-05_

---

## 🔴 High Priority — Blocking

| # | Task | Owner | Notes |
|---|------|-------|-------|
| 1 | **Register team account on stake-engine.com and get ACP access** | Tiger | Nothing can be submitted without this. Check if it requires an application or is self-serve. |
| 2 | **Build Mock RGS server (Flask on AWS EC2)** | Zacke/Tiger | Unblocks all local dev and Ollie's review workflow. See `docs/workflow.md` for spec. ACP is assumed production-only — local testing depends on this. |
| 3 | **Set up Cloudflare Pages for review builds** | Tiger | Connect GitHub → Cloudflare Pages auto-deploy. Ollie gets a preview URL per push. Points `rgs_url` at EC2 mock RGS. |

---

## 🟠 Research — Do This Week

| # | Task | Owner | Notes |
|---|------|-------|-------|
| 4 | Read math SDK quickstart + setup docs | Zacke | `/docs/math/quick-start` and `/docs/math/setup` — must understand before first brief |
| 5 | Read frontend SDK intro, flowchart, and task breakdown | Zacke | `/docs/front-end/flowchart` and `/docs/front-end/task-breakdown` — confirm SDK is workable |
| 6 | Browse Stake.com existing games | Tiger | 30 min — identify gaps and saturated types before briefing first game |
| 7 | Research asset generation without a designer | Tiger | Midjourney/SD for art, Rive or Spine for animation — what hits the 2-star bar? |
| 8 | Clarify ACP staging vs production when account is live | Tiger | Assumed production-only for now — confirm with Stake Engine on first login |

---

## 🟡 Setup — Before First Brief

| # | Task | Owner | Notes |
|---|------|-------|-------|
| 9 | Set up Python math SDK dev environment | Zacke | Clone `git@github.com:StakeEngine/math-sdk.git`, install Python 3.12+, Rust/Cargo, run `make setup`, run `fifty_fifty` example |
| 10 | Set up frontend SDK dev environment | Zacke | Get web-sdk clone URL from ACP, install Node 18.18.0 + pnpm 10.5.0, run `pnpm install`, launch Storybook for `lines` example |
| 11 | Deploy Mock RGS to EC2 | Zacke/Tiger | Flask app reading math output files — see `docs/workflow.md`. Needs to be live before Zacke starts first game. |
| 12 | Create `briefs/` directory and issue first brief | Tiger | Brief template is ready — just needs the folder and a dry run |
| 13 | Update Ollie's review checklist with mandatory platform requirements | Ollie | Bet Replay, tile assets, Stake.us language file, UI requirements, general disclaimer |
| 14 | Define provider/team name for tile asset naming convention | Tiger | Used in all asset filenames e.g. `ProviderName-Logo.png` |
| 15 | Hire Designer/Animator | Tiger | Tile assets required per submission — blocking quality not just polish |

---

## 🟢 First Game — When Setup Is Done

| # | Task | Owner | Notes |
|---|------|-------|-------|
| 16 | Issue first game brief | Tiger | Use `process/game-brief-template.md` — save to `briefs/2026-XX-XX-game-name.md` |
| 17 | Build first game (SDK stack — slot) | Zacke | First game validates the full pipeline end-to-end. Develop locally against mock RGS. |
| 18 | Zacke pushes to GitHub — Cloudflare deploys preview | Zacke | Tiger + Ollie get a review URL pointing at mock RGS |
| 19 | Tiger + Ollie local review pass | Tiger/Ollie | Test against mock RGS preview URL. Ollie uses updated review checklist. |
| 20 | Fix any bounces, re-deploy | Zacke | Repeat 18–19 until locally approved |
| 21 | Prepare tile assets | Tiger/Designer | BG + FG (max 3MB combined) + Provider logo |
| 22 | Write game submission blurb | Tiger | Short description of theme and mechanics — required with every submission |
| 23 | Prepare Stake.us language file (`sweeps_en.json`) | Zacke | Swap gambling terms — required for Stake.us eligibility |
| 24 | Tiger submits math files + frontend build to ACP | Tiger | Upload from Zacke's GitHub repo. Log in `submissions/log.md`, update `pipeline.md`. |

---

## 🔵 Ongoing / Recurring

| # | Task | Cadence | Owner |
|---|------|---------|-------|
| 25 | Update `pipeline.md` | Daily | All |
| 26 | Log submission outcome in `submissions/log.md` | Per submission | Tiger |
| 27 | Brief next game immediately after submission | Daily | Tiger |
| 28 | Add notes to `docs/` as team learns the SDK | Ongoing | Zacke/Tiger |

---

## ✅ Done

| # | Task | Completed | Notes |
|---|------|-----------|-------|
| — | Project structure and process templates set up | 2026-06-05 | pipeline, briefs template, review checklist, submission checklist all in place |
| — | `stake-requirements.md` populated from official docs | 2026-06-05 | Full technical requirements documented |
| — | Stack guides created (SDK and Hybrid) | 2026-06-05 | `resources/stack-stake-sdk.md` and `resources/stack-hybrid.md` |
| — | Stake Engine docs cloned locally | 2026-06-05 | `docs/stake-engine-md-doc/` — 64 markdown files, fully searchable |
| — | Docs notes files created for all SDK sections | 2026-06-05 | rgs, math, frontend, approval notes all populated from docs repo |
| — | How-games-work explainers written | 2026-06-05 | SDK stack and hybrid stack plain-English overviews in `docs/` |
| 8 | Stake Engine docs cloned locally | 2026-06-05 | Done — `docs/stake-engine-md-doc/` |
| — | `game_ideas.md` populated with full concept library | 2026-06-05 | 3 Japanese reskins, CS2 case opening, Island Combat RPG, Gacha PNG, Gacha NFT magnus opus |
| — | Payout model researched and documented | 2026-06-05 | 10% GGR perpetual royalty. 96% RTP target. Avg game ~$636/mo royalties. See `CLAUDE.md`. |
