# Stack Guide: Stake Engine SDK (Python + PixiJS + Svelte)

<!-- 
  Use this stack for: Slot games — reels, lines, ways, scatter, tumble, cluster, bonus rounds.
  Best choice when speed matters and the game type fits the slot format.
-->

---

## When to Use This Stack

- Game type is a **slot** (reels, paylines, bonus rounds, free spins, tumble mechanics, etc.)
- Priority is shipping fast — the SDK handles RGS integration, file format, and math output wiring
- Zacke wants pre-built slot components rather than building from scratch

---

## Stack Overview

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Math | Python (Stake Engine Math SDK) | Simulate all outcomes, generate CSV + config files |
| Renderer | PixiJS (via SDK) | 2D WebGL rendering — handles sprites, animations, particles |
| UI / Logic | Svelte (via SDK) | Component framework, wires math output to frontend |
| Dev tooling | Storybook | Build and test UI components in isolation |
| Hosting | Stake Engine CDN | Automatic once files are uploaded to ACP |

---

## Math Layer (Python)

The math framework pre-computes every possible outcome before any player ever bets.

### What you define in Python:
- Symbol definitions and weights
- Reel strips / board configuration
- Pay tables (lines, ways, scatter, cluster)
- Bonus round triggers and behaviour
- Bet modes and cost multipliers
- RTP target and volatility

### What the SDK generates automatically:
- All outcome CSV files (simulation number, probability, payout multiplier)
- Backend config files for the RGS
- Lookup tables
- Simulation acceptance reports

### Key SDK docs:
- Math Quickstart: `https://stake-engine.com/docs/math/quick-start`
- Math File Format: `https://stake-engine.com/docs/math/math-file-format`
- Game Structure: `https://stake-engine.com/docs/math/high-level-structure/game-structure`

---

## Frontend Layer (PixiJS + Svelte)

### Structure
The SDK provides a structured project layout — do not deviate from it without good reason:
- `src/` — game source (see File Structure docs)
- Components built in Svelte, rendered via PixiJS
- Events drive animation — math output fires events, frontend listens and animates

### How animation works:
1. Math output defines "events" (e.g. `SPIN_RESULT`, `BONUS_TRIGGER`, `WIN_DISPLAY`)
2. Frontend subscribes to events and plays corresponding PixiJS animations
3. New animations = add a new event in math + handle it in frontend

### Pre-built components available:
- Reel rendering and spin mechanics
- Win line displays
- Symbol animations
- UI shell (balance, bet selector, spin button)

### Key SDK docs:
- Frontend Intro: `https://stake-engine.com/docs/front-end`
- Flowchart: `https://stake-engine.com/docs/front-end/flowchart`
- Task Breakdown: `https://stake-engine.com/docs/front-end/task-breakdown`
- Adding New Events: `https://stake-engine.com/docs/front-end/adding-new-events`
- File Structure: `https://stake-engine.com/docs/front-end/file-structure`

---

## RGS Integration

Handled automatically by the SDK. The frontend communicates with the RGS via:

```
POST /wallet/authenticate  — on game load
POST /wallet/play          — on each bet
POST /wallet/end-round     — when animations complete
```

The SDK's NPM client (`@StakeEngine/ts-client`) wraps these calls — no manual implementation needed.

---

## Mandatory Features (must be in every game regardless of stack)

- [ ] **Bet Replay** — accept `replay=true` URL param, fetch from RGS, disable betting UI
- [ ] **Game Tile Assets** — BG image, FG image, Provider logo (see `stake-requirements.md`)
- [ ] **Jurisdiction language file** — `sweeps_<lang>` file swapping gambling terms for Stake.us
- [ ] **Small bet denomination support** — min bet must include $0.01, $0.02, $0.05 levels
- [ ] **Mobile support** — handle `device=mobile` URL param

---

## Build & Submission Checklist

1. Write and test Python math model
2. Run SDK simulation — verify outcome distribution and RTP
3. Build frontend in Svelte/PixiJS using generated math output
4. Test in Storybook
5. Test Bet Replay for all bet modes (normal win, big win, max win, loss, bonus trigger)
6. Prepare tile assets (BG, FG, logo — max 3MB combined)
7. Write submission blurb (theme + mechanics description)
8. Prepare `sweeps_<lang>` language file
9. Upload to ACP
10. Submit for review

---

## Pros & Cons

| Pros | Cons |
|------|------|
| Fastest path to a slot game | Only well-suited for slots |
| Math → frontend wiring is automatic | Svelte layer is Stake-specific — limited reuse outside platform |
| Pre-built reel/win components | If SDK has bugs or limitations, debugging their abstraction |
| RGS API calls handled for you | Less flexibility for non-standard game formats |
| Storybook dev environment | |

---

## Usage Example

```
# Typical project flow
1. Clone math SDK template
2. Define symbols, reels, paytable in Python
3. Run: python simulate.py → generates /output/ folder
4. Clone frontend SDK template
5. Point frontend at /output/ folder
6. Customise visuals, add animations
7. Build: npm run build → generates /dist/ folder
8. Upload /dist/ to ACP
```

---

_See also: `stack-hybrid.md` for non-slot game types_
_See also: `stake-requirements.md` for full platform requirements_
