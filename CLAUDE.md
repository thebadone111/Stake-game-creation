# Stake Game Studio — Working Context

## Team
- **Max** (handle: Tiger) — art generation, frontend development, ACP submission, overall creative direction. Primary Claude user.
- **Zacke** — math SDK, game logic, Python simulation. No formal hierarchy — both collaborate on design and direction.

No other team members. Previous team (Tom, Ollie, Nils) is no longer active.

## ACP
- Team name: **maxiai** — stake-engine.com/teams/maxiai

## Games

| Game | Status | Branch | Owner |
|------|--------|--------|-------|
| Ayakashi (v1) | Submitted | `ayakashi1` (frozen at `eec5bb7`) | Zacke — polish + upgrade pass |
| Ayakashi 2 | In development | `final-dev` | Max — art overhaul, animation pipeline |

Both branches are in the same repo: `thebadone111/Ayakashi`
Max's local path: `C:\Users\tiger\Desktop\Stake\game-1\Ayakashi`

## Tech Stack
- **Math**: Python math-sdk (`math/` in repo)
- **Frontend**: PixiJS + Svelte 5 + SvelteKit monorepo (`web-sdk/`)
- **Deploy**: Cloudflare Pages (auto from GitHub push), Mock RGS on EC2 for testing

## Art Pipeline (Locked)

### Still Images
- **FLUX 1.1 Pro Ultra** (`fal-ai/flux-pro/v1.1-ultra`) — minimalism, restraint, best cuttability
- **Seedream v4** (`fal-ai/bytedance/seedream/v4/text-to-image`) — most prompt-literal, best palette + foxfire
- **Recraft v3** (`fal-ai/recraft/v3/text-to-image`) — painterly backgrounds (3rd option, 1000-char prompt cap)
- 2 models × 3 candidates = 6 options per asset. Max picks from contact sheet.
- AuraSR 4x upscale ($0.05) where needed
- Scripts: `art/fal_generate.py`, `art/symbol_gen.py`, `art/avatar_ab.py`, `art/upscale.py`
- **Background removal: manual by Max** — BiRefNet unreliable on foxfire/translucent. Do not chain rembg onto gens.

### Character Animations — Tier 1 (RunComfy SVI Pro)
- Wan 2.2 I2V A14B + Stable Video Infinity 2.0 + LightX2V
- Deployment: `274fbd0c-e4fc-4097-848d-24552911e37b` (ayakashi-wan-idle)
- Script: `art/wan_svi_idle_avatar.py`
- **CRITICAL**: Enable instance to Standby in RunComfy dashboard FIRST. Upload avatar.webp while idle. THEN run script.
- Must override ALL 5 prompt nodes (366–370). Node 97 = LoadImage. Only overriding 366 = passes 2–5 use default prompt.
- ~$2–5/run, ~14–32min. Quality peaks at pass 4 — Ctrl+C after pass 4 to skip final render if needed.
- Planned upgrades (not yet implemented): 3 passes, RIFE interpolation, Real-ESRGAN upscale, 1080p input, dev vs hero workflow variants.

### FX/Particle Animations — Tier 2 (fal.ai sprite sheets)
- Wan 2.2 ($0.40/5s) or Hailuo 02 ($0.27/6s) — run both in parallel, pick best
- 16 frames → 4×4 WebP sprite sheet → PixiJS AnimatedSprite ping-pong loop
- Template: `art/petal_animate.py` + `art/petal_bake.py`
- Pick Wan for motion realism (avatar reactions), Hailuo for ambient layers where file size matters

### Style
- "painterly dark-fantasy anime, Edo-period rural Japanese yokai folklore…"
- Do NOT use Demon Slayer / ufotable named anchors (de-IP'd 2026-06-26)
- See `web-sdk/STYLE_GUIDE.md` and `web-sdk/PROMPT_GUIDE.md`
- One prompt per ROLE, not per asset. Strip bg language for cut-out assets.
- Low symbols: Yuji Syuku font over generated washi background (not individually AI-generated)

## Key Decisions (Do Not Revisit)
- **No style LoRA** — assets not good enough to justify it
- **No pixi-filters post-processing stack** — looks computed/digital, conflicts with I2V painterly aesthetic
- **No audio-reactive visuals** — audio quality insufficient
- **Manual background removal** — do not chain rembg/BiRefNet/alpha-matting onto gens
- **I2V is the animation mechanism** — sprite sheets are the runtime format, not a separate concept
- **Simple FX in code** — embers, sparks, simple particles should be PixiJS code, not AI-generated

## Future Tech (Planned, Not Implemented)
- **GLSL mouse-reactive foxfire shader** — Navier-Stokes fluid in WebGL via PixiJS filter, mouse position as `vec2` uniform
- **Rapier WASM physics** — physically simulated coins that bounce and stack (replaces particle emitters for coin FX)
- **GSAP** — for Svelte UI transitions (programmatic, no manual markup required)
- **RunPod for next game** — ~$0.60/hr vs $2.50/hr RunComfy, same A6000, self-hosted ComfyUI
- **IPAdapter** — under consideration for cross-clip animation identity consistency (not confirmed)
- **Spine 2D** — very far future for skeletal character animation

## Ayakashi 2 — Current Priorities
See `todo.md`.

## Key Files
- `pipeline.md` — current game status tracker
- `todo.md` — priority list
- `game_ideas.md` — concept library
- `submissions/log.md` — submission history
- `process/workflow.md` — how a game gets built (2-person flow)
- `process/submission-checklist.md` — pre-ACP checks
- `resources/art-pipeline.md` — full art + animation pipeline
- `resources/stake-requirements.md` — platform specs
- `docs/` — SDK references, math guide, approval notes
- `docs/stake-engine-md-doc/` — offline mirror of stake-engine.com/docs (64 files)
- `team/` — team profiles
