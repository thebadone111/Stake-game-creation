# Development Workflow

Two-person studio. Max and Zacke collaborate based on what's needed — no fixed handoff structure.

---

## Infrastructure

```
LOCAL (Max's machine)                     LOCAL (Zacke's machine)
  web-sdk (PixiJS + Svelte)                math-sdk (Python)
  art/ (generation scripts)
         │                                        │
         └──────────────── git push ──────────────┘
                                  │
                    GITHUB (Ayakashi repo)
                    thebadone111/Ayakashi
                          │
              ┌───────────┴───────────┐
              │                       │
    CLOUDFLARE PAGES          STAKE ENGINE ACP
    Preview URL per push       Production only
    rgs_url → EC2 Mock RGS
              │
         AWS EC2
         Mock RGS (Flask)
```

---

## Math

Zacke (or Max) using Python math-sdk:

1. Copy `games/template/` to `games/<game-name>/`
2. Edit `game_config.py` — symbols, reels, paytable, bet modes, RTP target
3. Edit `gamestate.py` — `run_spin()` logic
4. **Dev run** (fast iteration, catches logic errors):
   ```
   make run GAME=<game-name>   # num_sim_args = int(1e4)
   ```
5. Validate stats in `library/stats_summary.json` against `docs/maths-guide.md`
6. **Production run** (accurate RTP, ~2 hrs):
   ```
   make run GAME=<game-name>   # num_sim_args = int(1e6)
   ```
7. Re-validate production stats — these are authoritative

**Handoff deliverables:**
- `library/publish_files/` — books, LUTs, index.json (upload to ACP math)
- `library/configs/config_fe_<game_id>.json` — frontend game config
- `library/configs/event_config_base.json` — base game BookEvent structure
- `library/configs/event_config_bonus.json` — bonus event structure (if applicable)

---

## Frontend

Max (or Zacke) in web-sdk:

1. Copy example app (`apps/lines` for standard slots)
2. Drop in `config.ts` from math handoff (or run `apps/lines/sync-config.py`)
3. Wire events: `BookEvent` → `EmitterEvent` → PixiJS handler
4. Test in Storybook:
   ```
   pnpm run storybook --filter=lines
   ```
5. Art assets: place in `web-sdk/apps/lines/static/assets/`
6. Push to GitHub → Cloudflare auto-deploys preview URL

**Config sync note:** If board or paylines change in math, regenerate `config.ts`:
```
python apps/lines/sync-config.py
```

---

## Art

Max generates and processes:

1. Run fal.ai generation scripts (see `resources/art-pipeline.md`)
2. Review contact sheet, pick best candidates
3. Cut backgrounds manually (BiRefNet unreliable on foxfire/translucent)
4. Process to final WebP format (200×200px symbols)
5. Place in `web-sdk/apps/lines/static/assets/sprites/`

For animations: see `resources/art-pipeline.md` — RunComfy Tier 1 for characters, fal.ai sprite sheets Tier 2 for FX.

---

## Review

Max self-reviews (no separate reviewer). Use `process/submission-checklist.md` as the checklist.

1. Open Cloudflare preview URL
2. Test all game flows: base game, win, big win, bonus trigger, free spins, retrigger, max win
3. Test Bet Replay with documented event IDs
4. Test mobile view (DevTools device mode or actual mobile)
5. Check console for errors — especially external URL violations

---

## Submission (ACP)

1. Self-review passes `process/submission-checklist.md`
2. Build frontend: `pnpm run build --filter=lines`
3. Prepare tile assets: `GameTitle-BG.png`, `GameTitle-FG.png`, `ProviderName-Logo.png` (BG+FG under 3MB)
4. Write submission blurb (theme + mechanics)
5. Upload math files to ACP (`library/publish_files/`)
6. Upload frontend build to ACP (`dist/`)
7. Upload tile assets
8. Submit review request with blurb
9. Record Stake reference number in `submissions/log.md`
10. Update `pipeline.md`

---

## Per-Game File Structure

```
web-sdk/apps/lines/               ← frontend for lines-type games
  src/
    game/
    components/
    stores/
  static/
    assets/
      sprites/                    ← symbols, avatars, UI
      audio/                      ← sounds
  stories/
    data/                         ← Storybook test event fixtures
  config.ts                       ← generated from math (don't hand-edit)

art/
  <game_scripts>.py               ← generation scripts
  generated/                      ← raw AI outputs
    avatar-wan-svi-idle-avatar-YYYY-MM-DD/

math/games/<game-name>/
  game_config.py
  gamestate.py
  run.py
  library/                        ← generated on sim run
    publish_files/                ← upload to ACP
    configs/
    stats_summary.json
```
