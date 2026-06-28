# Todo — Ayakashi 2

_Current priorities for the final-dev branch._

---

## Art Overhaul (Next Session)
- [ ] Reel frame — make larger
- [ ] Symbols — make larger, improve quality
- [ ] I2I upscale pass on existing symbols
- [ ] Rembg Ultra batch workflow for cleaner masks (evaluate vs manual cutting)

## Remaining Animation Clips
All via RunComfy SVI Pro (Tier 1). See `resources/art-pipeline.md` for process.
- [ ] Reel stop (h1–h5 land animations)
- [ ] Tumble
- [ ] Wild land / expand
- [ ] Kanabo smash (mechanic animation)
- [ ] Bonus trigger
- [ ] Free spins intro
- [ ] Big win climax
- [ ] FS retrigger — at minimum a pulse/flash on the counter (currently nothing plays)

## Code Bugs / Incomplete
- [ ] Overlay canvas resize — WinCelebration, BonusTriggerAnimation, FreeSpinsScreen don't resize on window resize
- [ ] Anticipation intensity — should scale with scatter count (2 scatters ≠ 4 scatters visually)
- [ ] Ofuda charm visual scaling — 2x and 5x multiplier feel identical, need distinct visual weight
- [ ] MAX win tier — needs a distinct climax, not just more particles

## Audio
- [ ] Replace all audio — current sounds are placeholder mining-game audio

## Fonts
- [ ] Replace bitmap fonts — mining-game placeholders still in use

## Infrastructure
- [ ] Set up RunPod for next game (~$0.60/hr vs $2.50/hr RunComfy, same A6000 hardware)

---

## Future (Not Blocking Ayakashi 2)
- GLSL mouse-reactive foxfire shader (Navier-Stokes fluid, WebGL via PixiJS filter)
- Rapier WASM physics for coin FX (replaces particle emitters)
- GSAP for Svelte UI transitions (programmatic, no manual markup)
- RIFE frame interpolation + Real-ESRGAN in RunComfy workflow (planned upgrade to animation pipeline)
- IPAdapter evaluation for cross-clip animation identity consistency

---

## Done
- [x] Version 1 submitted to Stake (`ayakashi1` branch frozen)
- [x] Art pipeline locked (fal.ai + RunComfy two-tier system)
- [x] Avatar idle animation generated (5 passes, Wan SVI Pro)
- [x] Git branches split — ayakashi1 (v1) + final-dev (ayakashi2)
- [x] ANIMATION_PIPELINE.md written (web-sdk/)
