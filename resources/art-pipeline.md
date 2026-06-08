# Art Pipeline

_How we generate and source art assets for game submissions._

---

## Overview

We use AI generation as the primary art source, with free asset libraries as a fallback for things that are too time-consuming to generate. Animation tools come in when a game needs skeletal or deformed character movement (mainly 3-star track).

| Layer | Tool | Use Case |
|-------|------|----------|
| Base art generation | ComfyUI + Flux 2 | Symbols, backgrounds, UI elements, consistent characters |
| Cloud compute | RunComfy | Full ComfyUI in the cloud — same workflow JSONs, better GPU |
| Sprite sheet generation | SEELE AI | Animated sprite sheets from generated stills |
| Free asset backup | Kenney, OpenGameArt, Itch.io | UI elements, particles, anything we can't generate well |
| Skeletal animation | Spine | Character rigs for 3-star games — not needed for reskins |
| Portrait / idle animation | Live2D | Mesh-based character animation — consider for visual novel formats |

## Cloud Setup — RunComfy

**Primary cloud provider.** RunComfy runs full ComfyUI in the browser — same node editor, same workflow JSON format as local. Build and iterate locally on the 4060 (free), export the workflow JSON, run final-quality assets on RunComfy.

**Models to use:**
- **Flux 2 Flex** — general symbols, backgrounds, UI elements
- **Flux Kontext Pro** — character consistency across multiple symbols (wild, scatter, character appearances)
- **Recraft V4 Pro** — if Flux output isn't landing the right illustration style

**Workflow:**
1. Design and test nodes locally in ComfyUI on the 4060
2. When happy with the workflow, export as `workflow.json`
3. Upload to RunComfy, switch to a higher-tier GPU, generate final assets
4. Download PNGs, run through TexturePacker for spritesheets

**Cost:** ~$3–5 per game at ~60 images. Negligible relative to pipeline output.

---

## 1. ComfyUI + Flux1

**What it is:** ComfyUI is a node-based workflow tool for AI image generation. Flux1 (from Black Forest Labs) is the model — currently the strongest open model for consistency, detail, and following prompts accurately.

**Why it's the primary tool:** Flux1 reduces character identity drift by ~40% compared to older models. With the right workflow, the same character looks consistent across symbols, win animations, and background appearances — which matters for cohesive slot art.

**What it handles well:**
- Symbol art (high1, high2, low1–low4, wild, scatter)
- Background art
- UI frames and decorative elements
- Character sheet generation (multiple angles of the same character)
- General illustration for any theme

**Key techniques:**
- **Flux.1 Kontext** — the recommended approach for character consistency. Retains identity across poses and lighting changes without needing complex ControlNet setups.
- **IPAdapter** — for consistency using a reference image
- **ControlNet** — for controlling pose and composition
- **LoRA** — fine-tune for a specific art style or character

**Hardware note:** Flux Dev (the higher-quality variant) needs 16GB+ VRAM for best results. Flux Schnell runs on less but at lower quality. If local hardware is a bottleneck, RunComfy or ThinkDiffusion are cloud options.

**What it doesn't handle well:**
- Sprite sheets with consistent frames across multiple animation states — use SEELE for this
- Pixel art — SpriteCook is better suited (see below)

**Resources:**
- Consistent Character Creator 3.8 workflow: [runcomfy.com](https://www.runcomfy.com/comfyui-workflows/consistent-character-creator-3-8-in-comfyui-hyperrealistic-consistent-ai-characters)
- Flux.1 Kontext guide: [comfyui.org](https://comfyui.org/en/solving-character-consistency-with-flux1-kontext)

---

## 2. SEELE AI — Sprite Sheet Generation

**What it is:** AI tool purpose-built for animated sprite sheets. Takes a reference image (your ComfyUI output) and generates walk cycles, idle, attack, run, and jump animations as transparent PNG sprite sheets ready for use in PixiJS/Unity/Unreal.

**Why it pairs with ComfyUI:** ComfyUI generates the character art. SEELE turns it into an animated sprite sheet. The two tools together cover the full pipeline from "character concept" to "production-ready spritesheet."

**What it handles well:**
- Walk cycles, idle, attack, run, jump — one-click presets
- Transparent PNG output with consistent frame registration (no jitter between frames)
- Reduces frame-by-frame animation time from 45+ minutes to under 5 minutes
- Output is optimised for game engines

**Workflow:**
1. Generate base character art in ComfyUI
2. Upload the still to SEELE
3. Select animation type and frame count
4. Download the sprite sheet PNG
5. Run through TexturePacker to create the atlas for PixiJS

**Note on SpriteCook:** SpriteCook ([spritecook.ai](https://www.spritecook.ai)) is an alternative that focuses on pixel art sprites with a similar one-click animation approach. Worth trying for pixel-art-style games but SEELE is the primary tool for our illustration style.

---

## 3. Free Asset Libraries

Use these when the time cost of generating something cleanly outweighs just grabbing a solid free asset. UI chrome, particles, generic icons, and certain backgrounds often fall into this category.

### Kenney ([kenney.nl](https://kenney.nl))
- **License:** CC0 — fully public domain, no attribution required, unlimited commercial use
- **Volume:** 60,000+ assets — 2D sprites, 3D models, audio, fonts
- **Formats:** PNG and SVG (SVG means you can rescale or recolour easily in Inkscape or Illustrator)
- **Best for:** UI elements, buttons, particle effects, generic game objects, placeholder art while waiting on generated assets
- **Quality bar:** Only use assets that look intentional and fit the game's aesthetic — don't drop in Kenney art that looks mismatched

### OpenGameArt ([opengameart.org](https://opengameart.org))
- Community-contributed. License varies by asset — check each one (CC0, CC-BY, GPL are all common)
- Good for: background tiles, particle systems, UI elements
- Less consistent quality than Kenney — filter carefully

### Itch.io ([itch.io/game-assets](https://itch.io/game-assets))
- Mix of free and paid packs from indie creators
- Some very high-quality packs, especially for specific themes (fantasy, cyberpunk, Japanese)
- Check the license on each pack — many are free for commercial use, some require attribution

**General rule:** Any free asset we use needs to match or be close to the game's visual quality. Mismatched asset styles are a quick path to a low review score.

---

## 4. Spine — Skeletal Animation

**What it is:** Industry-standard 2D skeletal animation tool from Esoteric Software. Used in hundreds of shipped games. Animates a character by rigging a layered image with bones, rather than drawing frame by frame.

**When we use it:** Mainly for 3-star track games where character animation quality is a differentiator. Not needed for reskins — symbol animations in reskins are handled in PixiJS directly.

**How it integrates:** The Stake Engine frontend SDK supports Spine via PixiJS. Spine exports a `.json` + `.atlas` + `.png` runtime that PixiJS-spine loads directly.

**What it handles well:**
- Smooth, natural character movement (walk, attack, idle, react)
- Easy to change timing and easing without redrawing frames
- Single rig reused across multiple animation states
- Great for characters that appear multiple times in different contexts

**Learning curve:** Moderate. Not a tool for someone picking it up on the day of a build. Budget time for initial setup and practice before it's needed in production.

**Cost:** Paid license. Essential Software Spine standard license — check current pricing at [esotericsoftware.com](https://esotericsoftware.com).

---

## 5. Live2D

**What it is:** A different approach to character animation. Instead of a skeleton/bone rig, Live2D deforms the original 2D illustration directly — warping, stretching, and layering parts of the flat artwork to simulate movement.

**How it differs from Spine:**
- Spine: separate layered sprites + bones. Looks like animation.
- Live2D: deforms one illustration. Looks like the original art is breathing and moving.

**What it's good at:**
- Subtle idle animations on characters — breathing, blinking, hair movement
- Face expressions and lip sync
- Visual novel / dialogue-heavy formats
- The "vtuber" aesthetic — high fidelity 2D character presence

**When we'd use it:** Potentially for non-slot formats (Island Combat, Gacha) where a character needs to feel alive but doesn't need full skeletal walk/attack cycles. Less useful for standard slot symbol animation.

**When we wouldn't:** Reskins. Any game on the 1-day pipeline. The rig setup time is too high for fast turnaround work.

**Cost:** Paid. Free version available with limitations. Full SDK licensing needed for commercial game use.

---

## Art Checklist per Submission

Before handing off to Tom:

- [ ] All symbols generated and exported as transparent PNG (high1, high2, low1–low4, wild, scatter minimum)
- [ ] Background art (usually BG layer + optional FG overlay)
- [ ] Spritesheet(s) processed in TexturePacker — atlas + json ready
- [ ] Tile assets: BG tile + FG tile (combined under 3MB) + Provider logo
- [ ] No external fonts — any custom font hosted locally in the project
- [ ] No external image URLs — all assets served from Stake Engine CDN or bundled

---

## Asset Format Standards

| Asset | Format | Notes |
|-------|--------|-------|
| Symbols | Transparent PNG | Export at 2x resolution for retina |
| Backgrounds | PNG or WEBP | Full canvas size — usually 1920×1080 |
| Spritesheets | PNG atlas + JSON | TexturePacker format for PixiJS |
| Tile BG + FG | PNG | Under 3MB combined — Stake submission requirement |
| Provider logo | PNG | Check exact spec in `resources/stake-requirements.md` |
| Fonts | WOFF2 | Host locally — no Google Fonts CDN |

---

_See also: `resources/stake-requirements.md` — asset size limits and submission specs_
_See also: `docs/setup-frontend.md` — how Tom integrates assets into the SDK_
