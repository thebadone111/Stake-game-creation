# Backend Dev Environment Setup

_For Zacke and Nils. Get this done before the first game build._

---

## What You're Setting Up

The Stake Engine math SDK is a Python framework that pre-simulates every possible game outcome and outputs static files. Those files are what the RGS serves at runtime — no live RNG ever runs. Your job is to write the game logic in Python, run the simulation, and produce the output files that Tom's frontend consumes.

---

## System Requirements

Install these before anything else.

**Python 3.12+**
Specifically 3.12 — not just any Python 3. Check what you have:
```bash
python --version
```
If it's not 3.12+, download from python.org or use pyenv.

**Rust + Cargo**
The RTP optimisation step is written in Rust, not Python. Required even if you never touch Rust code directly.
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# restart your terminal after, then verify:
cargo --version
```

**Git**
```bash
git --version
```

**Make** (recommended)
Used for the SDK's shorthand commands. On Windows, install via Chocolatey (`choco install make`) or use the manual commands listed below as alternatives.

---

## Clone the Math SDK

```bash
git clone git@github.com:StakeEngine/math-sdk.git
cd math-sdk
```

---

## Setup

**With Make:**
```bash
make setup
```

**Manual (Windows or no Make):**
```bash
python -m venv env
# Windows:
env\Scripts\activate.bat
# Mac/Linux:
source env/bin/activate

pip install -r requirements.txt
pip install -e .
```

This creates a virtual environment, installs dependencies, and installs the SDK itself in editable mode so imports work from anywhere inside the repo.

---

## Verify — Run the Fifty-Fifty Example

This is the simplest game in the SDK (50% win at 2x, 50% loss). Run it first to confirm everything is working before touching any real game code.

```bash
make run GAME=fifty_fifty
```

Or manually:
```bash
python games/fifty_fifty/run.py
```

After it runs, check that this folder exists and has files in it:
```
math-sdk/games/fifty_fifty/library/
```

If it's there, setup is working.

---

## Run a Real Example — Lines Game

Once fifty_fifty works, run the lines example — this is the closest thing to the reskin games you'll be building.

```bash
make run GAME=0_0_lines
```

Then open and read through a few entries in:
```
games/0_0_lines/library/books/books_base.jsonl
```

Each line is one simulated round. It looks like this:
```json
{
  "id": 42,
  "payoutMultiplier": 450,
  "events": [
    { "type": "reveal", "board": [["high1","low2","wild"],["high1","high1","low1"],["high1","low2","scatter"]] },
    { "type": "winInfo", "wins": [{ "line": 0, "symbol": "high1", "count": 3, "amount": 450 }] },
    { "type": "setWin", "amount": 450 },
    { "type": "finalWin", "payoutMultiplier": 450 }
  ]
}
```

Also look at the lookup table:
```
games/0_0_lines/library/lookup_tables/lookUpTable_base.csv
```

Each row is `simulation_id, probability_weight, payout_multiplier`. The server picks a row weighted by probability, then fetches the matching events from the jsonl file.

Also explore the other example games:
- `0_0_ways` — ways-style win calculation
- `0_0_cluster` — cluster pays with tumble
- `0_0_scatter` — cascading scatter with global multiplier (closest to Island Combat)

---

## Project Structure

```
math-sdk/
  games/
    0_0_lines/          ← example: lines slot
    0_0_ways/           ← example: ways slot
    0_0_cluster/        ← example: cluster pays
    0_0_scatter/        ← example: cascading scatter
    0_0_expwilds/       ← example: expanding wilds
    fifty_fifty/        ← minimal example for RGS testing
    template/           ← copy this for new games
  src/
    calculations/       ← board, lines, ways, scatter, cluster, tumble logic
    config/             ← config file generation
    events/             ← pre-built event functions
    executables/        ← reusable game logic
    state/              ← game state tracking
    wins/               ← win/wallet management
    write_data/         ← file output and compression
  utils/
    analysis/           ← win distribution analysis
    game_analytics/     ← PAR sheet / hit-rate output
  tests/                ← PyTest win calculation tests
  optimization_program/ ← Rust RTP balancer (compiled automatically)
```

---

## Per-Game File Structure

When you start a new game, copy the template:
```bash
cp -r games/template games/my-game-name
```

Inside your game folder:
```
games/my-game-name/
  game_config.py       ← symbols, paytable, bet modes, RTP target
  gamestate.py         ← run_spin() — the single simulation entry point
  game_executables.py  ← optional: reusable logic groupings
  game_calculations.py ← optional: custom calculation overrides
  game_events.py       ← optional: custom event types
  run.py               ← simulation parameters (num sims, dev vs prod mode)
  library/             ← auto-generated on run (don't edit manually)
    books/             ← uncompressed jsonl output (dev)
    books_compressed/  ← zstd-compressed jsonl (production)
    configs/           ← config.json files for RGS
    lookup_tables/     ← CSV weight tables
    publish_files/     ← what gets uploaded to ACP
```

---

## Dev Run vs Production Run

**Dev run (fast, use this while building):**
Edit `run.py`:
```python
num_threads = 1
compression = False
num_sim_args = { "base": 100, "bonus": 100 }
run_conditions = {
    "run_sims": True,
    "run_optimization": False,   # skip Rust step
    "run_analysis": False
}
```
Run time: seconds. Output is readable uncompressed jsonl. Use this to check your event logic looks right.

**Production run (when logic is confirmed):**
```python
num_sim_args = { "base": int(1e5), "bonus": int(1e5) }   # 100k minimum, 1M preferred
run_conditions = {
    "run_sims": True,
    "run_optimization": True,    # Rust RTP balancer runs
    "run_analysis": True         # generates PAR sheet
}
```
Run time: minutes to an hour depending on complexity. Produces compressed output in `library/publish_files/` — these are the files that go to ACP.

---

## What Gets Uploaded to ACP

After a production run, `library/publish_files/` contains:
- `index.json` — lists all bet modes, their cost multipliers, and which files to use
- `lookUpTable_<mode>_0.csv` — weighted probability table per mode
- `books_<mode>.jsonl.zst` — compressed simulation events per mode

All three are required. The mock RGS also reads these same files locally.

---

## Math Validation Rules

Check these before declaring math done. The ACP auto-validates and rejects if any fail.

| Rule | Requirement |
|------|------------|
| RTP | 96.5%–97.5% (target 97%) |
| Multi-mode alignment | All modes within 0.5% of each other |
| Hit rate | Better than 1 in 20 bets (>5% — aim for 15%+) |
| Max win achievability | Reachable at least 1 in 10,000,000 rounds |
| Win range | No gaps — smooth distribution, no jumps |
| Min simulations | 100,000 per mode for production |

See `docs/maths-guide.md` for full standards and how to check them.

---

## Key Docs

- `docs/math-sdk-notes.md` — detailed SDK reference
- `docs/maths-guide.md` — industry standards and validation checklist
- `docs/mock-rgs-spec.md` — spec for the local test server
- `docs/stake-engine-md-doc/math/` — official SDK docs (offline mirror)

---

## First Things to Try

1. Run `fifty_fifty` — confirm setup works
2. Run `0_0_lines` — read 10 lines of the jsonl output
3. Copy the template, rename a few symbols, run a dev sim — see how output changes
4. Read `docs/maths-guide.md` before writing your first real game
