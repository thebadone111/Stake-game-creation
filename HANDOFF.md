# Session Handoff — 2026-06-27

_Updated at the end of each session. Read this at the start of the next one._

---

## Team Split (As Of This Session)

| Track | Owner | Branch | Focus |
|-------|-------|--------|-------|
| Ayakashi 1 (polish) | Zacke | `ayakashi1` | Polish + upgrade v1 for resubmission or v2 |
| Ayakashi 2 | Max | `final-dev` | Art overhaul, animation pipeline, new direction |

Both branches in `thebadone111/Ayakashi`.

---

## What We Did This Session

### Animation Pipeline — Validated End-to-End
First RunComfy SVI Pro run completed. Avatar idle animation generated (kitsune fox-spirit, 5 passes).

- **Critical bug fixed**: script was only overriding prompt node 366 (Pass 1). Passes 2–5 ran with the default "Adepta Sororitas biker" prompt. Fixed to override nodes 366–370.
- **Outputs**: `art/generated/avatar-wan-svi-idle-avatar-2026-06-27/` — Pass_1 through Pass_5 + final stitched Render (4.4MB). Quality peaks at Pass 4.
- **Script**: `art/wan_svi_idle_avatar.py` — handles polling, avatar upload, pass-by-pass download.
- **Upload timing rule**: instance must be on Standby BEFORE submitting the job. ComfyUI locks on boot. Manual: enable instance in RunComfy dashboard → wait for Standby → THEN run script.

### Two-Tier Animation System Locked
- **Tier 1 — Character/complex**: RunComfy SVI Pro (Wan 2.2 + Stable Video Infinity). Script ready, deployment live.
- **Tier 2 — FX/particles**: fal.ai Wan or Hailuo → 16 frames → 4×4 WebP sprite sheet → PixiJS AnimatedSprite. Template: `art/petal_animate.py` + `art/petal_bake.py`.

Full reference doc: `web-sdk/ANIMATION_PIPELINE.md`

### Decisions Locked (Do Not Revisit)
- No style LoRA — assets not good enough
- No pixi-filters post-processing — looks computed/digital, kills the I2V painterly feel
- No audio-reactive visuals — audio quality insufficient
- Simple FX (sparks, embers) in PixiJS code, not AI-generated
- Manual background removal by Max — BiRefNet unreliable on foxfire/translucent

### Future Tech (Planned, Not This Session)
- GLSL mouse-reactive foxfire shader (Navier-Stokes fluid via WebGL/PixiJS filter, mouse as `vec2` uniform)
- Rapier WASM physics for coin FX
- GSAP for Svelte UI transitions
- Spine 2D (very far future)
- RunPod for next game (~$0.60/hr vs $2.50/hr RunComfy, same A6000)

### Git Restructured
- `ayakashi1` — frozen at `eec5bb7` "Version 1 submitted"
- `final-dev` — Ayakashi 2, new direction (commit `c248b57`)

### Docs Cleaned (`D:\Claude playground\Stake game creation`)
Full cleanup: removed Nils/Tom/Ollie files, rewrote CLAUDE.md, README.md, todo.md, workflow.md, art-pipeline.md, submission-checklist.md, team profiles. Now reflects 2-person team and actual current pipeline.

---

## Current State of Ayakashi 2 (final-dev)

**Working:**
- Full game loop — base game, tumble, free spins, Ofuda multiplier, Kanabo smash
- Art direction locked: painterly dark-fantasy anime, Edo-period yokai folklore
- Avatar idle animation generated (not yet integrated)
- Petal animations generated and baked to sprite sheets

**Known Issues:**
| Issue | Priority |
|-------|----------|
| Reel frame too small | High — visual, next session |
| Symbols too small, quality upgrade needed | High — visual, next session |
| Overlay canvas resize (WinCelebration, BonusTriggerAnimation, FreeSpinsScreen) | High — bug |
| FS retrigger: no animation plays | Medium — needs at least a pulse/flash |
| Anticipation doesn't scale with scatter count | Medium |
| Ofuda charm: 2x and 5x feel identical | Medium |
| MAX win tier: no distinct climax | Medium |
| Audio: all mining-game placeholders | High |
| Bitmap fonts: mining-game placeholders | High |

---

## Next Session — Max (Ayakashi 2)

### 1. Art Overhaul
- Reel frame: make larger, adjust layout
- Symbols: make larger, run I2I upscale pass on existing art
- Evaluate Rembg Ultra: test on a symbol vs manual cut — if crisper edges, run a batch workflow

### 2. Avatar Animation Integration
Passes are downloaded. Need to:
1. Review passes (Pass_4 or Pass_5 likely best), pick one
2. Bake to WebP sprite sheet — clone `art/petal_bake.py`, adjust for avatar dimensions
3. Integrate into web-sdk as AnimatedSprite
4. Test in Storybook

### 3. Remaining Animation Clips (RunComfy Tier 1)
After art overhaul is settled (generate clips from final-quality source art):
- Reel stop (h1–h5 land)
- Tumble reaction
- Wild land / expand
- Kanabo smash
- Bonus trigger
- Free spins intro
- Big win climax

Enable instance to Standby first. Script: `art/wan_svi_idle_avatar.py` (clone per animation type, adjust prompt).

### 4. Code Bugs (In Order)
1. Overlay canvas resize — WinCelebration, BonusTriggerAnimation, FreeSpinsScreen
2. FS retrigger pulse/flash
3. Anticipation scale with scatter count
4. Ofuda visual scaling
5. MAX win climax

### 5. Audio + Fonts
- Replace all audio (current: mining-game placeholder sounds)
- Replace bitmap fonts (current: mining-game placeholders)

---

## Next Session — Zacke (Ayakashi 1 Art Sprint)

Full brief: `briefs/2026-06-28-zacke-art-sprint.md`
Systems reference: `docs/art-systems.md`

**Summary:**
- Set up ComfyUI locally first — foundation for all I2I work
- Rembg Ultra experiment — test automated bg removal on our symbols
- Symbol upgrade via I2I (low denoise img2img) — improve h1–h5 without regenerating
- Reel frame integration — I2I composite to bake frame into background naturally
- Background layer upgrade (bg_bg upgrade, restore bg_effect + bg_mist as proper layers)
- AuraSR final upscale pass on everything

**Not this sprint:** new math builds, new submissions, frontend changes.

**Branch:** `ayakashi1`. Old `gen-*.py` art scripts deleted — use new ComfyUI workflows or borrow scripts from `final-dev`.

---

## Key Technical References

### RunComfy SVI Pro
- Deployment: `274fbd0c-e4fc-4097-848d-24552911e37b` (ayakashi-wan-idle)
- Script: `art/wan_svi_idle_avatar.py`
- **Enable instance to Standby FIRST. Upload avatar.webp while idle. THEN run script.**
- Override nodes: `97` (image), `366–370` (all 5 pass prompts — MUST override all 5)
- ~$2–5/run, ~14–32min on A6000

### fal.ai Generation
- `python art/symbol_gen.py` — symbols (FLUX + Seedream, 6 candidates)
- `python art/avatar_ab.py` — avatar
- `python art/upscale.py` — AuraSR 4x
- `FAL_KEY` must be set in shell each session (not inherited by background processes)

### Sprite Sheet Bake
- Template: `art/petal_bake.py`
- mp4 → 16 evenly-spaced frames → 4×4 WebP grid
- PixiJS ping-pong: `[...frames, ...frames.slice(1, -1).reverse()]`

### Config Sync
After board/payline changes in math: `python apps/lines/sync-config.py`

### Repo
- GitHub: `thebadone111/Ayakashi`
- Max local: `C:\Users\tiger\Desktop\Stake\game-1\Ayakashi`
- Max branch: `final-dev`
- Zacke branch: `ayakashi1`
