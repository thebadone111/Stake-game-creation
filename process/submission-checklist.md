# Submission Checklist

Pre-ACP self-review. Run through this before submitting any game.

---

## Self-Review
- [ ] All game flows work: base game, win, big win, bonus trigger, free spins, retrigger, max win
- [ ] No softlocks or infinite loops
- [ ] Bet Replay works with documented event IDs (normal win, big win, max win, bonus trigger, loss)
- [ ] Mobile layout correct (resize browser or DevTools device mode)
- [ ] No console errors — especially no external URL calls (strict XSS policy)
- [ ] All Storybook stories pass
- [ ] Game brief acceptance criteria all met
- [ ] Final build packaged correctly (see `resources/stake-requirements.md`)

## Assets
- [ ] Tile assets ready: `GameTitle-BG.png`, `GameTitle-FG.png`, `ProviderName-Logo.png`
- [ ] BG + FG tile assets under 3MB combined
- [ ] `sweeps_en.json` included in build (required for Stake.us eligibility)
- [ ] General disclaimer text in game rules popup
- [ ] No external fonts or image URLs in the build

## ACP Submission
- [ ] Math files uploaded (`library/publish_files/`)
- [ ] Frontend build uploaded (`dist/`)
- [ ] Tile assets uploaded
- [ ] Submission blurb written and attached
- [ ] Stake reference number recorded in `submissions/log.md`
- [ ] `pipeline.md` updated
