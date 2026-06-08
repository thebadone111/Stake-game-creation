# Mock RGS — Specification

_Built by Tiger/Zacke. Deployed on AWS EC2. Used for all local dev and review builds._

---

## What It Is

A lightweight Flask API that mimics the Stake Engine Carrot RGS (Remote Gaming Server). It reads the math SDK's output files and serves pre-simulated outcomes exactly as the production RGS would — so Tom can develop against it locally and Ollie can review against it via the Cloudflare preview URL.

**It is not a production server.** No auth, no real money, no database. Sessions are held in memory and reset on restart. That's fine — it's purely for development and internal review.

---

## Tech Stack

- **Python 3.12+** with **Flask**
- **gunicorn** for production serving on EC2
- **zstandard** library for reading `.jsonl.zst` compressed math output
- **HTTPS** via Cloudflare Tunnel (simplest) or Let's Encrypt + nginx

Install dependencies:
```bash
pip install flask gunicorn zstandard
```

---

## How It Works

1. On startup, the server scans a `games/` folder for uploaded math SDK output files
2. When a player authenticates, a session is created in memory with a fake balance
3. When a player bets, the server reads the game's lookup table CSV, picks a simulation by weighted random, reads the corresponding events from the compressed jsonl file, and returns them
4. Bet Replay reads a specific simulation by ID directly

The server never generates outcomes — it only serves pre-computed ones from the math output files.

---

## Folder Structure

```
mock-rgs/
  app.py              ← Flask application
  requirements.txt
  games/
    my-game-name/
      base/
        index.json
        lookUpTable_base_0.csv
        books_base.jsonl.zst
      bonus/
        index.json
        lookUpTable_bonus_0.csv
        books_bonus.jsonl.zst
```

These files come directly from the math SDK's `library/publish_files/` folder after a simulation run. Just copy them in.

---

## API Endpoints

---

### `POST /wallet/authenticate`

Called once when the game loads. Creates a session and returns player config.

**Request headers:**
```
Content-Type: application/json
```

**Request body:**
```json
{
  "sessionID": "test-session-abc123",
  "gameId": "my-game-name",
  "lang": "en",
  "device": "desktop"
}
```

**Response — fresh session (no active round):**
```json
{
  "balance": {
    "amount": 100000000,
    "currency": "USD"
  },
  "config": {
    "minBet": 10000,
    "maxBet": 1000000000,
    "stepBet": 10000,
    "defaultBetLevel": 1000000,
    "betLevels": [
      10000,
      20000,
      50000,
      100000,
      200000,
      500000,
      1000000,
      2000000,
      5000000,
      10000000
    ],
    "jurisdiction": {
      "socialCasino": false,
      "disabledFullscreen": false,
      "disabledTurbo": false
    }
  },
  "round": null
}
```

**Response — session with interrupted round (player reconnected mid-game):**
```json
{
  "balance": {
    "amount": 99000000,
    "currency": "USD"
  },
  "config": { "...": "same as above" },
  "round": {
    "id": "round-xyz789",
    "modeId": "base",
    "betAmount": 1000000,
    "payoutMultiplier": 0,
    "state": {
      "events": [
        { "type": "reveal", "board": [["high1","low2","wild"],["scatter","high1","low1"],["low2","low1","high1"]] },
        { "type": "winInfo", "wins": [] },
        { "type": "finalWin", "payoutMultiplier": 0 }
      ],
      "currentEvent": 1
    }
  }
}
```

The `round.state.currentEvent` index tells the frontend where to resume animation from.

**Money format reminder:**
All amounts are integers with 6 implied decimal places. `100000000` = $100.00. `1000000` = $1.00. `10000` = $0.01.

---

### `POST /wallet/play`

Called when the player places a bet. Debits the balance, picks a simulation, returns events.

**Request body:**
```json
{
  "sessionID": "test-session-abc123",
  "betAmount": 1000000,
  "modeId": "base"
}
```

**Response — loss (zero payout):**
```json
{
  "balance": {
    "amount": 99000000,
    "currency": "USD"
  },
  "round": {
    "id": "round-xyz789",
    "modeId": "base",
    "betAmount": 1000000,
    "payoutMultiplier": 0,
    "state": {
      "events": [
        {
          "type": "reveal",
          "board": [
            ["low2", "low1", "high2"],
            ["low1", "low2", "low1"],
            ["high2", "low1", "low2"]
          ]
        },
        { "type": "finalWin", "payoutMultiplier": 0 }
      ]
    }
  }
}
```

**Response — small win:**
```json
{
  "balance": {
    "amount": 99500000,
    "currency": "USD"
  },
  "round": {
    "id": "round-abc456",
    "modeId": "base",
    "betAmount": 1000000,
    "payoutMultiplier": 50,
    "state": {
      "events": [
        {
          "type": "reveal",
          "board": [
            ["low1", "low1", "low1"],
            ["high2", "low2", "wild"],
            ["scatter", "low1", "high1"]
          ]
        },
        {
          "type": "winInfo",
          "wins": [
            { "line": 0, "symbol": "low1", "count": 3, "amount": 50 }
          ]
        },
        { "type": "setWin", "amount": 50 },
        { "type": "finalWin", "payoutMultiplier": 50 }
      ]
    }
  }
}
```

**Response — bonus trigger (free spins):**
```json
{
  "balance": {
    "amount": 98000000,
    "currency": "USD"
  },
  "round": {
    "id": "round-def789",
    "modeId": "base",
    "betAmount": 1000000,
    "payoutMultiplier": 1250,
    "state": {
      "events": [
        {
          "type": "reveal",
          "board": [
            ["scatter", "high1", "low2"],
            ["high1", "scatter", "low1"],
            ["low2", "wild", "scatter"]
          ]
        },
        { "type": "freeSpinTrigger", "count": 10 },
        { "type": "updateFreeSpin", "current": 10, "total": 10 },
        {
          "type": "reveal",
          "board": [["high1","high1","high1"],["low1","scatter","low2"],["high1","wild","high1"]]
        },
        { "type": "winInfo", "wins": [{ "line": 0, "symbol": "high1", "count": 5, "amount": 1000 }] },
        { "type": "setWin", "amount": 1000 },
        { "type": "updateFreeSpin", "current": 9, "total": 10 },
        { "type": "updateFreeSpin", "current": 8, "total": 10 },
        { "type": "freeSpinEnd" },
        { "type": "setTotal", "amount": 1250 },
        { "type": "finalWin", "payoutMultiplier": 1250 }
      ]
    }
  }
}
```

**Error response — insufficient balance:**
```json
{
  "error": "ERR_IPB",
  "message": "Insufficient player balance"
}
```
HTTP status: 400

**How simulation selection works internally:**
1. Read `lookUpTable_{modeId}_0.csv` for the game
2. Pick a row by weighted random (using `probability_weight` column)
3. Get the `simulation_id` from that row
4. Find the matching line in `books_{modeId}.jsonl.zst` by `id`
5. Return the `events` array from that simulation

---

### `POST /wallet/end-round`

Called after all animations complete. Closes the round and applies winnings.

**Request body:**
```json
{
  "sessionID": "test-session-abc123"
}
```

**Response:**
```json
{
  "balance": {
    "amount": 100250000,
    "currency": "USD"
  }
}
```

The balance here reflects the win being credited. Internally: `new_balance = balance_after_debit + (betAmount * payoutMultiplier / 100)`.

Note: some bet modes have `auto_close_disabled = True` in the math config. In that case the frontend calls end-round manually at a specific point in the animation, not at the end. The mock RGS doesn't need to handle this differently — just always accept the call and return updated balance.

---

### `POST /wallet/balance`

Optional. Used for periodic balance refreshes if the frontend needs them.

**Request body:**
```json
{
  "sessionID": "test-session-abc123"
}
```

**Response:**
```json
{
  "balance": {
    "amount": 100250000,
    "currency": "USD"
  }
}
```

---

### `GET /bet/replay/{game}/{version}/{mode}/{event}`

Returns a specific simulation by ID. No auth required — URLs are publicly shareable.

**Example URL:**
```
GET /bet/replay/my-game-name/1/base/42
```

Parameters:
- `game` — game ID (folder name in `games/`)
- `version` — game version (use `1` for mock, actual versions come from ACP)
- `mode` — bet mode (`base`, `bonus`, etc.)
- `event` — simulation ID to retrieve

**Response:**
```json
{
  "payoutMultiplier": 450,
  "costMultiplier": 1.0,
  "state": {
    "events": [
      {
        "type": "reveal",
        "board": [
          ["high1", "low2", "wild"],
          ["high1", "high1", "low1"],
          ["high1", "low2", "scatter"]
        ]
      },
      {
        "type": "winInfo",
        "wins": [
          { "line": 0, "symbol": "high1", "count": 3, "amount": 450 }
        ]
      },
      { "type": "setWin", "amount": 450 },
      { "type": "finalWin", "payoutMultiplier": 450 }
    ]
  }
}
```

**Error — simulation not found:**
```json
{
  "error": "ERR_NOT_FOUND",
  "message": "Simulation 9999 not found in mode base for game my-game-name"
}
```
HTTP status: 404

---

## Error Codes Reference

| Code | HTTP Status | Meaning |
|------|------------|---------|
| `ERR_IS` | 400 | Invalid or expired session — session not found |
| `ERR_IPB` | 400 | Insufficient balance |
| `ERR_GLE` | 400 | Gambling limits exceeded (mock: not implemented, return if needed) |
| `ERR_LOC` | 400 | Invalid player location (mock: not implemented) |
| `ERR_VAL` | 400 | Invalid request parameters |
| `ERR_GEN` | 500 | Generic server error |
| `ERR_NOT_FOUND` | 404 | Simulation ID not found (replay only) |

---

## Session Management

Sessions are stored in a Python dict in memory:

```python
sessions = {
  "test-session-abc123": {
    "balance": 100000000,     # starting balance in integer format
    "currency": "USD",
    "gameId": "my-game-name",
    "currentRound": None      # or round object if mid-round
  }
}
```

Sessions reset on server restart. That's fine — it's a test server. For testing reconnection flow, restart the server mid-round and re-authenticate to see the round resumption response.

**Default starting balance:** $100.00 = `100000000`

---

## Loading Game Files

On startup (or lazily on first request for a game), load the math output files:

```python
import csv
import zstandard as zstd
import json

def load_game(game_id, mode_id):
    base_path = f"games/{game_id}/{mode_id}"
    
    # Load lookup table into memory
    lookup = []
    with open(f"{base_path}/lookUpTable_{mode_id}_0.csv") as f:
        reader = csv.reader(f)
        for row in reader:
            lookup.append({
                "id": int(row[0]),
                "weight": int(row[1]),
                "payout": int(row[2])
            })
    
    # Load all simulations into memory (for small games)
    # For large production files, use line-indexed seeking instead
    simulations = {}
    dctx = zstd.ZstdDecompressor()
    with open(f"{base_path}/books_{mode_id}.jsonl.zst", "rb") as f:
        with dctx.stream_reader(f) as reader:
            for line in reader.read().decode().strip().split("\n"):
                sim = json.loads(line)
                simulations[sim["id"]] = sim
    
    return {"lookup": lookup, "simulations": simulations}
```

**Weighted random selection:**
```python
import random

def pick_simulation(lookup):
    total_weight = sum(row["weight"] for row in lookup)
    r = random.randint(0, total_weight - 1)
    cumulative = 0
    for row in lookup:
        cumulative += row["weight"]
        if r < cumulative:
            return row["id"]
```

---

## Deploying on EC2

**Recommended setup:**
1. Launch an EC2 instance (t3.micro is enough — it's a test server)
2. Open port 443 (HTTPS) in the security group
3. Install Python, pip, and the dependencies
4. Clone the mock-rgs repo onto the instance
5. Use a **Cloudflare Tunnel** for HTTPS — simplest option, free, no certificate management:
   ```bash
   # Install cloudflared, then:
   cloudflared tunnel --url http://localhost:5000
   ```
   This gives you a public HTTPS URL like `https://random-words.trycloudflare.com` that proxies to your Flask app.

**Running the server:**
```bash
# Development:
flask run --host=0.0.0.0 --port=5000

# Production (use gunicorn):
gunicorn -w 2 -b 0.0.0.0:5000 app:app
```

**Uploading game files:**
After each math SDK run, copy `library/publish_files/` to the EC2 instance:
```bash
scp -r math-sdk/games/my-game/library/publish_files/ ec2-user@<ip>:~/mock-rgs/games/my-game/base/
```

Or push to a shared GitHub repo and pull on the EC2 instance.

---

## Using the Mock RGS in the Frontend

The `rgs_url` is passed as a URL parameter when launching the game. In dev/review:

```
https://my-game.pages.dev/index.html
  ?sessionID=test-session-abc123
  &lang=en
  &device=desktop
  &rgs_url=https://your-cloudflare-tunnel-url.trycloudflare.com
```

Never hardcode the `rgs_url` in the frontend — always read it from the URL params.

---

## Bet Replay IDs to Document

For every submission, document these simulation IDs from the math output for Ollie's review:

| Type | Simulation ID | Notes |
|------|--------------|-------|
| Normal win (small) | — | A win in the 1–5× range |
| Big win | — | A win in the 20–100× range |
| Max win | — | Highest payout simulation |
| Loss | — | Zero payout |
| Bonus trigger | — | Free spins / bonus round trigger |

These let Ollie test Bet Replay for specific outcomes without relying on random simulation results.

---

_See also: `docs/rgs-notes.md` — full Carrot RGS API reference_
_See also: `docs/setup-backend.md` — math SDK setup (produces the files this server reads)_
