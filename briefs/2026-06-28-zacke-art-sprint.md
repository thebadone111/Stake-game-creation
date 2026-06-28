# Zacke — Art Sprint Brief
_Meeting date: 2026-06-28_

---

## The Goal

We are not rushing to submit a new game. The goal for this sprint is to:

1. **Learn the art generation systems properly** — both Max and Zacke, together
2. **Upgrade Ayakashi 1's art** without regenerating from scratch
3. **Run experiments** that tell us what these tools can actually do
4. **Build ComfyUI workflows** we can reuse on every future game

This is an investment in the studio's capability, not a production sprint. Quality and learning over speed.

---

## Step 0 — Set Up ComfyUI Locally (First Thing)

Before anything else, get ComfyUI running on your machine. This is the foundation for most of what's below.

**Install:**
```bash
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
pip install -r requirements.txt
python main.py
# Opens in browser at http://127.0.0.1:8188
```

**Then install ComfyUI-Manager** (makes installing custom nodes much easier):
```bash
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager.git
```

**First workflow to build:** Basic T2I — Load Checkpoint → CLIP Text Encode → KSampler → VAE Decode → Save Image. Spend time understanding what each node does. This is the skeleton everything else is built on.

**What GPU do you have?** This determines what you can run locally:
- 8GB VRAM: FLUX Schnell (fast, lower quality), most upscalers, Rembg Ultra — fine for experiments
- 12GB+: FLUX Dev, standard I2I workflows
- 24GB+: Wan I2V locally (unlikely to match RunComfy but possible)

For anything that needs more power, we use RunComfy (already live) or RunPod (Max setting up).

---

## Task 1 — Rembg Ultra Experiment

**Goal:** Find out if automated background removal is good enough on our symbols to replace manual cutting.

**Why it matters:** Max is manually cutting every symbol. If Rembg Ultra handles our ink-edged subjects cleanly, the pipeline gets 5× faster.

**How to run it:**
1. Install the `comfyui-rembg` custom node via ComfyUI-Manager
2. Load one of the h1–h5 source images (with background)
3. Run Rembg Ultra (BiRefNet-Large model)
4. Compare the edge to Max's manually cut version

**What we're testing:**
- Clean ink edges on solid subjects — expected to pass
- Any foxfire/atmospheric glow elements — expected to fail or need help
- Whether the result is production-quality or needs cleanup

**Document the result:** Screenshots of pass/fail cases. This decides whether we automate background removal going forward.

---

## Task 2 — Symbol Upgrade via I2I (Not Regen)

**Goal:** Take the existing Ayakashi 1 symbols (h1–h5) and run an upgrade pass that improves quality without changing the character identity.

The current symbols were generated early in the project when the pipeline was less refined. They're good enough but could be crisper, more detailed, better colour-graded.

**Method — low denoise img2img:**
1. In ComfyUI, load an existing symbol (e.g. h1 — the kitsune)
2. VAE Encode it
3. KSampler with **denoise 0.25–0.40** and the original style prompt
4. VAE Decode → compare to original

A low denoise value preserves the composition and identity while allowing the model to sharpen details and fix small quality issues. **Do not exceed 0.5** or the model will start changing the character.

**Denoise calibration (run this first):**
Generate the same symbol at denoise 0.2, 0.3, 0.4, 0.5, 0.6. Save all. Max reviews which range looks best. This calibration informs every future upgrade pass.

**For each symbol, try:**
- FLUX Kontext (img2img with prompt) at best denoise value
- AuraSR 4x after the I2I pass (upscale the already-upgraded version)

**Deliver:** A contact sheet comparing original vs upgraded for each of h1–h5. Max picks final versions.

---

## Task 3 — Reel Frame Integration Experiment

**Goal:** Instead of the reel frame sitting as a flat digital overlay, make it look like it belongs in the scene — shadows, organic edges, atmospheric elements wrapping around it.

**Current situation:**
- The reel frame is a separate `.webp` Sprite rendered as a layer between the background and the board symbols
- The background has zero awareness that the frame exists — it's just a flat image underneath
- Result: the frame looks pasted on, not part of the world

**The experiment — I2I composite:**

1. **Make a composite image:**
   - Take `bg_bg.webp` (the background painting)
   - Overlay `reel_frame.webp` on top of it at the correct position
   - Flatten into a single image — this is your I2I input
   
2. **Run img2img at low denoise (0.3–0.45) with a prompt:**
   ```
   painterly dark-fantasy anime, Edo-period Japanese yokai shrine, 
   lacquered wooden frame integrated into misty forest scene, 
   foxfire glow casting light on frame edges, atmospheric depth, 
   masterful composition
   ```
   
3. **Review the output:**
   - Does the frame look like it grew out of the background?
   - Are there shadows, lighting, atmospheric blending?
   - Or is it just a slightly blurred version of the composite?

4. **Try variations:**
   - Vary denoise (0.2 / 0.35 / 0.5)
   - Try FLUX Kontext with the composite as the reference image
   - Try inpainting: mask only the areas where frame meets background, let the model blend those edges

**The game integration:** If the I2I produces good integration, the resulting image replaces `bg_bg.webp`. The reel frame sprite in-game can then be removed or made very subtle — the background carries the frame.

**This could be a big visual upgrade.** The gap between "frame looks pasted on" and "frame is part of the world" is the gap between amateur and studio work.

**Deliver:** 3–5 candidate integrated backgrounds. Max reviews.

---

## Task 4 — Background Layer Upgrade

The current background (`bg_bg.webp`) is one painting. `bg_effect` (glow/light pass) and `bg_mist` are currently not being loaded in the game.

**4a — Upgrade bg_bg:**
Run the same I2I upgrade pass as the symbols. Low denoise, same painterly dark-fantasy prompt. Should sharpen the painting quality.

**4b — Regenerate bg_effect and bg_mist as proper layers:**
These were dropped because the new `bg_bg` carries its own atmosphere. But having them as separate additive layers gives us runtime animation (fade, pulse, breathe). Worth regenerating properly:
- `bg_effect.webp` — a glow/light pass (soft foxfire light bloom). Render mode: Additive.
- `bg_mist.webp` — atmospheric mist that drifts. Render mode: Normal, 50–60% opacity.

Generate these at low denoise from the upgraded bg_bg so they match its palette.

---

## Task 5 — AuraSR Upscale Pass on All Art

After the I2I upgrade on symbols and backgrounds, run AuraSR 4x on everything before final export.

```bash
python art/upscale.py   # (check script for input/output paths, update as needed)
```

This is a simple terminal operation. Run it on the full asset set once I2I upgrades are done.

---

## Experiments to Document

For each experiment, save:
- Input image
- Settings used (model, denoise, prompt)
- Output image
- Short note on what worked / what didn't

We want to build institutional knowledge about what these tools actually do on our art style. Commit the results to `art/experiments/` with a dated folder.

---

## What Not To Do

- **Don't regen symbols from scratch** unless an I2I upgrade makes it clearly worse than starting over
- **Don't rush a new math build** — the math for Ayakashi 1 is already solid. Polish art first.
- **Don't submit until the art passes our own quality bar** — we are setting the standard, not chasing the review timeline
- **Don't use BiRefNet on foxfire elements** — confirmed unreliable. Use manual cutting or test Rembg Ultra first.

---

## Sync Points with Max

Max is working on Ayakashi 2 (`final-dev` branch) in parallel — same art upgrade work on the new version.

**Share freely:**
- Experiment results (good and bad) — we are both learning
- ComfyUI workflows as `.json` files — anything that works for Ayakashi 1 is directly reusable
- Prompt discoveries — what makes FLUX vs Seedream behave differently

**Key reference:** `D:\Claude playground\Stake game creation\docs\art-systems.md` — read this. Covers all tools, pros/cons, what's validated vs what we're figuring out.

---

## Priority Order

1. ComfyUI local setup
2. Rembg Ultra experiment (quick, informs everything else)
3. Denoise calibration on one symbol (sets the baseline for all I2I work)
4. Symbol upgrade pass (h1–h5)
5. Reel frame integration experiment
6. Background upgrade
7. AuraSR final upscale pass on everything
