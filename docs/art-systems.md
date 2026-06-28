# Art Generation Systems — Reference Guide

_For Max + Zacke. Current as of 2026-06-27. We are actively experimenting — this doc reflects what's validated, what's in progress, and what's next._

---

## Philosophy

We use AI art as a production tool, not a magic button. Every generation needs:
1. A strong prompt (style-locked, specific)
2. A human eye picking from candidates
3. Manual cleanup (background removal, compositing)
4. Integration into the game engine

The goal is to master these systems well enough to produce studio-quality output reliably. We are not there yet — we are learning.

---

## System 1 — Text-to-Image via fal.ai (T2I)

**What it is:** Cloud API for AI image generation. We run two models in parallel and pick from 6 candidates (2 models × 3 each).

### FLUX 1.1 Pro Ultra
| | |
|---|---|
| **API** | `fal-ai/flux-pro/v1.1-ultra` |
| **Cost** | ~$0.06/image |
| **Speed** | ~15–25s |
| **Strengths** | Restraint and minimalism. Best cuttability — tends to give clean edges, clear separation between subject and background. Museum-artifact register. Hard ink-edged masks. |
| **Weaknesses** | Can be too restrained — sometimes lacks the colour drama the Ayakashi palette needs. Can underprompt on atmospheric elements. |
| **Validated on** | Symbols (h1–h5), avatar variants, backgrounds |

### Seedream v4
| | |
|---|---|
| **API** | `fal-ai/bytedance/seedream/v4/text-to-image` |
| **Cost** | ~$0.05/image |
| **Speed** | ~20–30s |
| **Strengths** | Most prompt-literal of the three. Best at integrating foxfire/atmospheric elements into the composition. Rich palette. When FLUX is too plain, Seedream delivers the drama. |
| **Weaknesses** | Can overdo detail — backgrounds become busy, edges harder to cut. More hallucination-prone on complex prompts. |
| **Validated on** | Symbols (h1–h5), avatar, atmospheric backgrounds |

### Recraft v3
| | |
|---|---|
| **API** | `fal-ai/recraft/v3/text-to-image` |
| **Cost** | ~$0.04/image |
| **Speed** | ~10–15s |
| **Strengths** | Painterly illustration register — feels hand-crafted rather than AI. Good for environmental backgrounds and ambient layers. |
| **Weaknesses** | **1000-character prompt cap** — forces you to write concise prompts, which can hurt on complex symbol descriptions. Not suitable for cut-out assets (edges are too painterly). |
| **Validated on** | Background layers (bg_bg, bg_fg), used as 3rd option |

### Workflow
```
python art/symbol_gen.py        # symbols (FLUX + Seedream parallel)
python art/avatar_ab.py         # avatar
python art/bg_gen.py            # backgrounds
python art/bg_more.py           # backgrounds (Recraft variant, compact prompt)
```
Scripts write candidates to `art/generated/`. Contact sheet built automatically for easy comparison.

### What We Don't Know Yet
- Whether FLUX Kontext (image-conditioned generation) gives better character consistency than prompting alone
- How much of the style variance is prompt vs model vs seed
- Optimal denoise value for I2I passes on our existing assets

---

## System 2 — Image-to-Image / Upscaling

**What it is:** Taking an existing image and upgrading it — sharpening, enhancing detail, or transforming it while preserving the composition.

### AuraSR 4x (fal.ai)
| | |
|---|---|
| **API** | `fal-ai/aura-sr` |
| **Cost** | ~$0.05/image |
| **Speed** | ~10s |
| **Strengths** | Simple, reliable 4x upscale. Good edge preservation. No GPU setup required. |
| **Weaknesses** | Dumb upscaler — it enlarges, it doesn't enhance. Won't fix poor detail or add new texture. Won't change composition. |
| **Validated on** | Symbol upscaling for final export |

### FLUX Kontext (To Explore)
| | |
|---|---|
| **What it is** | FLUX-based img2img — take a reference image + prompt, generate a variation that preserves the key elements |
| **Why it matters** | Could be used to: (1) upgrade symbol quality while keeping identity, (2) bake the reel frame into the background, (3) generate new poses of the avatar from the existing one |
| **Status** | **Not yet tested on our assets.** Available via fal.ai (`fal-ai/flux-pro/v1.1-ultra` with image input) or natively in ComfyUI |
| **Key parameter** | Denoise strength — 0.2–0.4 for subtle upgrade, 0.6–0.8 for heavy transformation |

### Real-ESRGAN Anime (ComfyUI)
| | |
|---|---|
| **What it is** | Upscaler specifically trained on anime/illustration art |
| **Why it matters** | Adds crisp line quality and anime-appropriate sharpening rather than generic photography upscaling |
| **Models** | `4x-AnimeSharp`, `4x-UltraSharp` |
| **Status** | **Planned — not yet in workflow.** Requires ComfyUI setup. |

### Standard img2img (ComfyUI)
| | |
|---|---|
| **What it is** | Run VAE encode → KSampler with low denoise on an existing image |
| **Why it matters** | The core tool for upgrading and transforming existing art. Reel frame integration experiment relies on this. |
| **Status** | **Learning — both Max and Zacke need to build this in ComfyUI** |

---

## System 3 — ComfyUI (Node-Based Workflow)

**What it is:** Open-source, node-based AI image pipeline. The same tool that RunComfy runs in the cloud. Gives full control over every step of the process — models, samplers, upscalers, ControlNet, post-processing.

### Why We Need It
fal.ai is great for one-click T2I but is a black box. ComfyUI lets us:
- Build custom I2I upgrade workflows (reel frame integration, symbol enhancement)
- Run Rembg Ultra for background removal
- Use Real-ESRGAN anime upscaler
- Experiment with ControlNet for composition control
- Build the full Wan SVI Pro workflow locally (when hardware is sufficient)
- Mix and chain models in ways the API doesn't expose

### Local vs RunComfy vs RunPod
| | Local | RunComfy | RunPod |
|---|---|---|---|
| **Cost** | Free (electricity) | $2.50/hr (A6000) | ~$0.60/hr (A6000) |
| **Control** | Full | Full (GUI) | Full (self-hosted) |
| **Setup** | 1-2 hours | Zero (browser) | ~2-4 hours |
| **GPU** | What you have | A6000 (48GB) | A6000 (48GB) |
| **Good for** | I2I experiments, upscalers, masks, small workflows | Final render quality, Wan SVI Pro animations | All production runs — planned for next game |
| **Status** | **To set up on both machines** | Live (ayakashi-wan-idle deployment) | Planned — Max setting up for next game |

### What to Learn First (Recommended Order)
1. Basic node graph: `Load Image → VAE Encode → KSampler → VAE Decode → Save Image`
2. T2I with FLUX — same results as fal.ai but full control
3. I2I at low denoise — the core upgrade tool
4. Rembg Ultra node — batch background removal
5. Real-ESRGAN node — anime upscaler
6. ControlNet — composition and pose control

**Resources:**
- ComfyUI GitHub: [github.com/comfyanonymous/ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- Recommended starter workflows: [comfyui.org/workflows](https://comfyui.org)
- FLUX Kontext workflow: [comfyui.org/en/solving-character-consistency-with-flux1-kontext](https://comfyui.org/en/solving-character-consistency-with-flux1-kontext)

---

## System 4 — Character Animation (I2V) via RunComfy

**What it is:** Image-to-Video. Take a single still image and generate a short animation clip. Used for all character and complex animations.

### Wan SVI Pro (RunComfy — Tier 1)
| | |
|---|---|
| **Model** | Wan 2.2 I2V A14B + Stable Video Infinity 2.0 + LightX2V |
| **Deployment** | `274fbd0c-e4fc-4097-848d-24552911e37b` (ayakashi-wan-idle) |
| **Script** | `art/wan_svi_idle_avatar.py` |
| **Cost** | ~$2–5/run (~$2.50/hr on A6000) |
| **Time** | ~14–32min (5 passes, sequential) |
| **Output quality** | High — sustained character identity across long sequences, cinematic motion |
| **Strengths** | Best quality available. Multi-pass refinement (each pass refines the previous). Full control via prompt. |
| **Weaknesses** | Expensive relative to simple FX. Slow. Instance must be on Standby before submission (upload timing critical). All 5 prompt nodes must be overridden. |
| **Validated on** | Avatar idle animation (2026-06-27) ✅ |

**Critical workflow note:** Enable instance to Standby in RunComfy dashboard → upload avatar.webp while idle → THEN run script. Submitting cold misses the upload window.

### fal.ai Wan 2.2 (Tier 2)
| | |
|---|---|
| **API** | `fal-ai/wan/v2.2/image-to-video` |
| **Cost** | ~$0.40/5s clip |
| **Strengths** | Simple API, no setup. Faster than SVI Pro for quick FX clips. |
| **Weaknesses** | Single-pass only. Character identity degrades over longer clips. Not suitable for sustained character animations. |
| **Use for** | FX layers, environmental elements, particle animations |

### fal.ai Hailuo 02 (Tier 2)
| | |
|---|---|
| **Cost** | ~$0.27/6s clip |
| **Strengths** | Cheapest per-clip. Smallest file size output. Good enough for ambient overlays. |
| **Weaknesses** | Lower motion quality than Wan. Not for character work. |
| **Use for** | Ambient particle layers where file size matters more than motion quality |

---

## System 5 — Background Removal

**The problem:** Our art style has translucent foxfire, atmospheric glow, and mist elements. Most automated tools see these as "background" and cut them.

### Manual Cutting (Max) — Current Primary Method
| | |
|---|---|
| **Reliability** | 100% — human judgment |
| **Cost** | Time only |
| **Strengths** | Handles foxfire, translucent elements, complex edges perfectly |
| **Weaknesses** | Slowest. Not scalable to 50+ assets |
| **When to use** | Avatar, any symbol with atmospheric/translucent elements |

### BiRefNet / Standard rembg — Not Suitable
Binary segmenter. Cuts foxfire tails and glowing edges as background. Confirmed unreliable on our assets. Do not chain onto gens automatically.

### Rembg Ultra (ComfyUI Node) — To Evaluate
| | |
|---|---|
| **What it is** | ComfyUI node using BiRefNet-Large or BRIA RMBG 2.0 |
| **Performance** | Excellent on hard ink-edged subjects with solid backgrounds. Still unconfirmed on our foxfire/translucent elements. |
| **Status** | **Not yet tested on our assets.** High priority experiment — if it works on our symbols, it massively speeds up the pipeline. |
| **Experiment:** | Generate a clean-background symbol → run Rembg Ultra → compare edge to Max's manual cut |

---

## System 6 — Sprite Sheet Pipeline

**What it is:** Convert I2V animation clips (mp4) into a grid of frames that PixiJS can play back efficiently.

### Validated Pattern (petal_bake.py)
```
mp4 → 16 evenly-spaced frames → 4×4 WebP grid → PixiJS AnimatedSprite
```
- Cost: zero (runs locally in Python/OpenCV)
- Output: ~30–500KB WebP at q82 depending on content
- PixiJS ping-pong loop: `[...frames, ...frames.slice(1,-1).reverse()]`

Scripts: `art/petal_animate.py` (I2V generation) + `art/petal_bake.py` (mp4 → sheet)

Clone `petal_bake.py` for each new animation type — adjust frame size and grid layout as needed.

---

## Experiments Planned (Both of Us)

These are open questions. No right answers yet — we test and see.

| Experiment | Goal | Tool | Status |
|------------|------|------|--------|
| FLUX Kontext on symbols | Test character consistency across h1–h5 using shared reference | ComfyUI or fal.ai | Not started |
| Real-ESRGAN vs AuraSR | Which upscaler gives better anime quality on our symbols? | ComfyUI (ESRGAN) vs fal.ai (AuraSR) | Not started |
| Rembg Ultra on symbol | Does it handle our foxfire/edges cleanly? | ComfyUI | Not started |
| I2I reel frame integration | Bake the reel frame into bg so it looks naturally placed | ComfyUI img2img | Not started — see Zacke brief |
| FLUX Kontext portrait-to-animation | Use Kontext to generate avatar in new pose → feed into SVI Pro | ComfyUI + RunComfy | Not started |
| Denoise calibration | What denoise values work for our art style in img2img? | ComfyUI | Not started |
| Model comparison grid | Same prompt across FLUX / Seedream / Recraft / NoobAI — which fits best? | Mixed | Ongoing informally |

---

## What We Know We Don't Know

- Whether a local ComfyUI workflow can match RunComfy quality for character animations (hardware dependent)
- Ideal prompt length and structure for Seedream vs FLUX — they respond differently
- How much the LoRA + style approach could unify visual consistency (deferred — assets not good enough yet)
- Whether Rembg Ultra can fully replace manual cutting for hard-edged symbols
- Whether FLUX Kontext can maintain kitsune identity well enough across all 5 high symbols

---

## Cost Reference

| Operation | Tool | Cost |
|-----------|------|------|
| T2I image | FLUX 1.1 Pro Ultra | ~$0.06 |
| T2I image | Seedream v4 | ~$0.05 |
| T2I image | Recraft v3 | ~$0.04 |
| 4x upscale | AuraSR | ~$0.05 |
| 5s I2V clip | Wan 2.2 (fal.ai) | ~$0.40 |
| 6s I2V clip | Hailuo 02 (fal.ai) | ~$0.27 |
| Full animation run | RunComfy SVI Pro (5 pass) | ~$2–5 |
| Per-hour GPU | RunComfy A6000 | $2.50 |
| Per-hour GPU | RunPod A6000 (planned) | ~$0.60 |
| ComfyUI local | — | Free |
