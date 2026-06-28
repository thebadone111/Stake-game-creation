# Art Pipeline

Full pipeline for generating and processing art assets. Validated on Ayakashi (2026-06-27).

---

## Still Images

### Models

Two primary models, run in parallel, 3 candidates each = 6 options per asset. Max picks from contact sheet.

| Model | fal.ai ID | Strengths |
|-------|-----------|-----------|
| FLUX 1.1 Pro Ultra | `fal-ai/flux-pro/v1.1-ultra` | Minimalism, restraint, best cuttability |
| Seedream v4 | `fal-ai/bytedance/seedream/v4/text-to-image` | Most prompt-literal, best palette + foxfire integration |
| Recraft v3 (3rd option) | `fal-ai/recraft/v3/text-to-image` | Painterly backgrounds. **Has a 1000-char prompt cap** — use compact prompt variant |

Cost: ~$0.27 per asset. AuraSR 4x upscale: ~$0.05 extra.

### Scripts
- `art/fal_generate.py` — shared queue, contact sheet builder, resume-by-prompt-hash
- `art/symbol_gen.py` — high symbol driver (h1–h5 with mask descriptions)
- `art/avatar_ab.py` — cuttable avatar driver
- `art/upscale.py` — AuraSR 4x
- `art/bg_gen.py` / `art/bg_more.py` — background generation (Recraft variant uses bg_more.py)
- `art/i2v_test.py` — Wan + Hailuo I2V comparison

### Workflow
1. Run appropriate driver script
2. Review contact sheet (`fal_generate.build_contact_sheet`), Max picks 1
3. AuraSR upscale if needed, or re-roll
4. Max cuts backgrounds manually (see below)
5. Place final assets in `web-sdk/apps/lines/static/assets/`

### Background Removal
**Manual by Max. Do not chain rembg/BiRefNet/alpha-matting onto gens.**

BiRefNet binary segmenter cuts foxfire tails and translucent elements as background — unreliable on any glow/flame/mist. Manual cutting is faster than cleanup. Job is to deliver clean-cuttable sources from the generation step.

### Style
Locked style prefix: "painterly dark-fantasy anime, Edo-period rural Japanese yokai folklore..."
- No Demon Slayer / ufotable named anchors (de-IP'd 2026-06-26)
- See `web-sdk/STYLE_GUIDE.md` and `web-sdk/PROMPT_GUIDE.md`
- One prompt per ROLE, not per asset. Strip background language for cut-out assets.

### Low Symbols (L1–L5)
Kanji characters rendered via Yuji Syuku font over a single generated washi paper background. Not individually AI-generated. Hallucination-free, visually uniform, much cheaper.

---

## Character Animations — Tier 1 (RunComfy SVI Pro)

For all character/complex animations. Validated 2026-06-27 on avatar idle.

### Tech
- Model: Wan 2.2 I2V A14B + Stable Video Infinity 2.0 + LightX2V
- Deployment: `274fbd0c-e4fc-4097-848d-24552911e37b` (ayakashi-wan-idle)
- Hardware: A6000 on RunComfy @ $2.50/hr → ~$2–5/run
- Time: ~14–32min (5 passes)
- Script: `art/wan_svi_idle_avatar.py`

### How to Run
1. **Enable instance to Standby in RunComfy dashboard first** — do NOT submit cold
2. Upload avatar.webp to the instance while it is idle (before submitting)
3. Run: `python art/wan_svi_idle_avatar.py`
4. Script polls every 3s, uploads avatar on boot, monitors logs for pass completions, downloads pass-by-pass
5. Quality peaks at pass 4 — Ctrl+C after pass 4 to skip final render if needed
6. Outputs saved to `art/generated/avatar-wan-svi-idle-avatar-YYYY-MM-DD/`

**CRITICAL upload timing**: Do NOT submit job first and upload after. Instance locks on boot. There is only ~90s between boot and when LoadImage executes — not reliable when submitting cold. Keep instance on Standby, upload first, then submit.

### Node Overrides (Must Override ALL)
```python
body = {'overrides': {
    '97':  {'inputs': {'image': 'avatar.webp'}},      # LoadImage
    '366': {'inputs': {'prompt': IDLE_PROMPT}},        # Pass 1
    '367': {'inputs': {'prompt': IDLE_PROMPT}},        # Pass 2
    '368': {'inputs': {'prompt': IDLE_PROMPT}},        # Pass 3
    '369': {'inputs': {'prompt': IDLE_PROMPT}},        # Pass 4
    '370': {'inputs': {'prompt': IDLE_PROMPT}},        # Pass 5
}}
```

Overriding only node 366 = passes 2–5 run with the workflow default "Adepta Sororitas biker" prompt.

### Outputs
- `Pass_1_00002.mp4` through `Pass_5_00002.mp4` — per-pass clips
- `Render_00002.mp4` — final stitched output (~4.4 MB)

### Planned Upgrades (Not Yet Implemented)
- Drop to 3 passes (not 5)
- RIFE frame interpolation (VHS_RIFE_VFI node)
- Real-ESRGAN anime per-frame upscale (4x-AnimeSharp)
- 1080p input resolution (currently lower)
- Two workflow variants: **dev** (fast, LightX2V kept) vs **final/hero** (drop LightX2V, full quality)

### Next Game: RunPod
RunPod self-hosted ComfyUI: ~$0.60/hr on same A6000 hardware vs $2.50/hr RunComfy. Max is setting this up for the next game. 4× cheaper, full control over the workflow.

---

## FX / Particle Animations — Tier 2 (fal.ai Sprite Sheets)

For simple FX, ambient particles, environmental elements. Validated on petal animations.

### Models
| Model | Cost | Best For |
|-------|------|----------|
| Wan 2.2 | $0.40/5s | Motion realism — avatar reactions, hero moments |
| Hailuo 02 | $0.27/6s | File size priority — ambient particle layers |

Run both in parallel and pick the better result.

### Workflow
1. Generate still with `symbol_gen.py` or similar
2. Max cuts background, saves as `<asset>-nobg.png`
3. Run `art/petal_animate.py` (template) — submits both Wan + Hailuo in parallel (~7 min for 5 assets × 2 models)
4. Run `art/petal_bake.py` (clone per asset) — mp4 → 16 evenly-spaced frames → 4×4 WebP sprite sheet
5. PixiJS `AnimatedSprite` plays the sheet. Use ping-pong loop for idle states.

### Sprite Sheet Format
- 4×4 grid, 16 frames evenly spaced from the mp4
- ~256px per frame, ~30–500KB at q82 depending on motion
- Ping-pong loop: `[...frames, ...frames.slice(1, -1).reverse()]`

### Simple FX in Code
Embers, sparks, basic particles: use PixiJS particle emitters directly. Do not generate these with AI — code is faster and more controllable.

---

## Asset Format Standards

| Asset | Format | Notes |
|-------|--------|-------|
| Symbols | WebP (transparent) | 200×200px, exported from cut PNG |
| Backgrounds | PNG or WebP | Full canvas — usually 1920×1080 |
| Sprite sheets | 4×4 WebP grid | 16 frames, ~256px/frame |
| Tile BG + FG | PNG | Under 3MB combined — Stake submission requirement |
| Provider logo | PNG | See `resources/stake-requirements.md` for exact spec |
| Fonts | WOFF2 | Host locally — no Google Fonts or external CDN |

---

## Animation Target List

Full list of all animations needed for Ayakashi 2 (from `web-sdk/ANIMATION_PIPELINE.md`):

**Avatar / Character**
- Idle loop ✅ (generated)
- Reel stop reaction
- Tumble reaction
- Wild land / expand
- Kanabo smash
- Bonus trigger
- Free spins intro
- Big win climax

**Symbol Animations**
- H1–H5 land (reel stop)
- Wild expand frame
- Scatter pulse

**FX / Environment**
- Petals ambient ✅ (generated)
- Foxfire glow pulse
- Win line sparkle trail
- Multiplier counter flash (FS retrigger)
- Coin shower (big win)
- Screen shake (kanabo)

**UI**
- Loading screen
- FS counter increment
- Bet size change reaction
