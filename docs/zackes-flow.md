# Zacke's Flow — Per Game

## Receives
- Game brief from Tiger — theme, symbol names, which reference game to base math on, any special mechanics, target volatility

---

## Steps

**1. Copy math template**
```bash
cp -r games/template/ games/new-game-name/
```

---

**2. game_config.py**

Defines the full structure of the game:

- **Symbol names** — match the brief (H1=Champion, H2=Sensei, etc.) and must match asset filenames Tom will use
- **Paytable** — how much each symbol pays for 3/4/5 of a kind. Start from the reference game's paytable, adjust to theme. Can't be arbitrary — has to produce 97% RTP when combined with the reel strips.
- **Reel strips** — the virtual reel each column spins through (~200 symbols long). Controls how often each symbol lands. H1 appears 2–3 times (rare), L1 appears 20+ times (common), scatter appears 4–6 times across reels.
- **Bet modes** — base game (cost: 1.0) + bonus buy (cost: 100x). Keep it simple for reskins.
- **Special symbol properties** — wild (substitutes), scatter (triggers free spins), multiplier, etc.

---

**3. gamestate.py**

The `run_spin()` function — the actual game logic:
- Pick a random stop position on each reel strip
- Read the 3 visible symbols at that position
- Evaluate which paylines hit and calculate win amounts
- Check if scatter count triggers free spins
- Handle any special mechanics (cascades, multipliers, wilds)
- Output the full BookEvent sequence for that outcome

For a new mechanic, this is where the complexity lives. For a reskin it's a light edit or untouched.

---

**4. Dev run**

In `run.py`, set:
```python
num_sim_args = {"base": int(1e4), "bonus": int(1e4)}
num_threads = 24
rust_threads = 32
```
```bash
make run GAME=new-game-name
```
10k sims. Fast. Inspect `library/configs/event_config_base.json` — do the BookEvents look right? Are win amounts calculating correctly? Do free spins trigger? Does the event sequence make sense?

---

**5. Validate RTP + Volatility**

Check `library/stats_summary.json` and `library/statistics_summary.json` against `docs/maths-guide.md`:
- RTP: 97% on all modes, within 0.5% of each other
- Hit rate: 15%+ (aim 20–30% for medium volatility)
- Base/bonus RTP split: ~60% base / ~40% free spins
- Win distribution: no gaps > 3× between adjacent tiers
- Dead spin runs: P(6+ consecutive zeros) < 1%

If RTP is off, adjust reel strips (add/remove symbol occurrences) or paytable values and re-run. This is iterative — expect 2–4 rounds for a reskin, more for a new mechanic.

---

**6. Production run**

Once logic and RTP confirmed, set `run.py` to:
```python
num_sim_args = {"base": int(1e6), "bonus": int(1e6)}
```
```bash
make run GAME=new-game-name
```
1M sims. Re-validate `stats_summary.json` — the 1M figures are authoritative, 10k can be noisy especially on bonus volatility stats.

---

**7. Handoff to Tom**

Four files/folders to deliver:

- **`library/publish_files/`** — full folder. Contains compressed books (`.jsonl.zst`), lookup tables (`.csv`), and `index.json`. Tom's mock RGS reads directly from these.
- **`library/configs/config_fe_<game_id>.json`** — frontend game config. Grid dimensions, paylines, RTP, bet modes, symbol list. Tom does not edit this.
- **`library/configs/event_config_base.json`** — base game BookEvent structure. Tom copies examples from this into his Storybook data files.
- **`library/configs/event_config_bonus.json`** — bonus BookEvent structure (if applicable).

Also deliver:
- **Custom BookEvent specs** (if any) — for any non-standard events: name, payload shape, when it fires, edge cases. Symbol names in event configs must exactly match sprite names in Tiger's spritesheet.
- **Bet Replay event IDs** — specific simulation IDs for: normal win, big win, max win, loss, bonus trigger. Ollie uses these during review. Without them the checklist can't be completed.

---

**8. Flex work**

Math done → assist with:
- TexturePacker (spritesheet processing for Tom's assets)
- Answering Tom's questions about event structure
- Math-related bounce fixes if a game gets rejected

---

## New Mechanics vs Reskins

For a reskin, steps 2 and 3 are light edits — copy reference math, rename symbols, tweak paytable slightly, run sims to confirm RTP. Total math time ~2 hours.

For a new mechanic:
- Step 2 has to design the probability impact before coding
- Step 3 requires implementing the full logic loop
- Step 5 takes many more iterations — you're calibrating unknown math
- BookEvent structure has to be agreed with Tom upfront before either side starts
- Platform constraint: **everything must be pre-simulatable**. No mechanics that depend on previous round state, no progressive jackpots, no live decisions. If the full outcome can't be computed in one shot at build time, it won't work on Stake Engine.

This is why new mechanics live on the 3-star track (Tue/Thu, no deadline) and reskins get the daily cadence.

---

## Goal Timeline

Handoff should reach Tom before 12pm CET. Tom comes online around 12pm CET / 6pm Taiwan — if the handoff is ready when he starts, the pipeline runs smoothly.
