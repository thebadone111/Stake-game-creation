# Frontend Dev Environment Setup

_For Tom. Get this done before the first game build._

---

## What You're Setting Up

The Stake Engine web SDK is a TurboRepo monorepo built on PixiJS + Svelte 5. It provides pre-built reel components, RGS API wiring, a state machine for the bet flow, and a Storybook environment for building and testing everything without a live server.

You won't be building from scratch — you'll copy one of the example apps and customise it. Storybook is your primary dev environment. You rarely need a live RGS connection during development.

---

## System Requirements

**Node 18.18.0 — exact version required**
The SDK is pinned to this version. Using anything else will likely cause issues.

Use nvm to manage Node versions:
```bash
# Install nvm if you don't have it
# Mac/Linux: https://github.com/nvm-sh/nvm
# Windows: https://github.com/coreybutler/nvm-windows

nvm install 18.18.0
nvm use 18.18.0

# Verify
node --version   # should print v18.18.0
```

**pnpm 10.5.0 — exact version required**
```bash
npm install -g pnpm@10.5.0

# Verify
pnpm --version   # should print 10.5.0
```

**Git**
```bash
git --version
```

---

## Get the Clone URL

The web SDK repo is private — the clone URL comes from the ACP.

Log in at `stake-engine.com/teams/maxiai`, find the web SDK repository link, and copy it. Ask Tiger if you can't find it.

---

## Clone and Install

```bash
git clone <URL from ACP>
cd web-sdk
pnpm install
```

`pnpm install` will take a few minutes the first time — it's installing dependencies across the entire monorepo.

---

## Verify — Run Storybook

```bash
pnpm run storybook --filter=lines
```

This launches the Storybook dev server for the `lines` example game. Open the URL it prints (usually `http://localhost:6006`).

If you see the Storybook UI with the lines game components listed, setup is working.

Try clicking through:
- `MODE_BASE > book > random` — plays a random full round
- `COMPONENTS > Symbol > symbols` — shows all symbols in all states

---

## Run Dev Mode (optional)

If you want to run the game itself (not Storybook) against the mock RGS:
```bash
pnpm run dev --filter=lines
```

This requires the mock RGS to be running. Storybook works without it.

---

## Project Structure

The SDK is a monorepo. The parts you'll work in most:

```
web-sdk/
  apps/
    lines/      ← copy this for a lines/ways slot
    ways/       ← ways game example
    cluster/    ← cluster pays example
    scatter/    ← scatter/cascading example
  packages/
    pixi-svelte/          ← core PixiJS + Svelte integration (rarely touch)
    components-ui-pixi/   ← pre-built UI components (restyle, don't copy as-is)
    components-ui-html/   ← HTML modals and overlays
    utils-slots/          ← reel helpers
    utils-sound/          ← Howler audio wrapper
    utils-layout/         ← canvas sizing and responsive positioning
    utils-xstate/         ← game state machine
```

When you start a new game, copy whichever example app is closest:
```bash
cp -r apps/lines apps/my-game-name
```

---

## Inside a Game App

```
apps/lines/src/
  routes/
    +page.svelte            ← entry point — sets up contexts, renders <Game />
  components/
    Game.svelte             ← root game component
    Symbol.svelte           ← individual symbol
    Background.svelte       ← background layer
  game/
    bookEventHandlerMap.ts  ← THE KEY FILE — routes server events to animations
    typesBookEvent.ts       ← TypeScript types for all incoming events
    typesEmitterEvent.ts    ← TypeScript types for all internal animation events
    context.ts              ← context setup/retrieval helpers
    eventEmitter.ts         ← event emitter instance
  stories/
    data/
      base_books.ts         ← mock base game data for Storybook
      bonus_books.ts        ← mock bonus data
      base_events.ts        ← individual event mocks
      bonus_events.ts
    *.stories.svelte        ← Storybook story files
```

The two files you'll edit most often for a new game:
- `bookEventHandlerMap.ts` — wires each BookEvent from the server to frontend animations
- `Symbol.svelte` — how each symbol looks and animates

---

## The Event System — Key Concept

There are two completely separate event types. Understanding this before touching code saves a lot of confusion.

**BookEvents** — come FROM the server (math SDK output via RGS)
Examples: `reveal`, `winInfo`, `setWin`, `freeSpinTrigger`, `finalWin`

**EmitterEvents** — internal signals WITHIN the frontend between components
Examples: `reelsStop`, `showWinLines`, `playWinCelebration`, `showMultiplier`

The flow is always:
```
Server response
  → BookEvent received
  → bookEventHandlerMap routes it
  → Handler broadcasts EmitterEvent(s)
  → Svelte components listen, animate, resolve
  → Next BookEvent fires
```

Each EmitterEvent resolves (via a Promise) before the next one fires. This is what makes animations play in sequence even though all events arrived at once.

---

## Adding a New Event (any custom mechanic)

Use this checklist every time — always in this order:

1. Add mock data to `stories/data/base_books.ts` (or `bonus_books.ts`)
2. Add a Storybook story for the new event
3. Add TypeScript type to `typesBookEvent.ts`
4. Add handler to `bookEventHandlerMap.ts`
5. Add EmitterEvent types to `typesEmitterEvent.ts`
6. Subscribe to the EmitterEvent in the relevant Svelte component
7. Build the animation
8. Test via the Storybook story — confirm it resolves cleanly

---

## Storybook Workflow

Storybook is the primary dev environment. You don't need a live game or RGS session to build and test anything — just mock data.

**The rule:** if every Storybook story for every BookEvent resolves correctly, the game is done.

Key story categories:

| Story path | What it tests |
|-----------|---------------|
| `MODE_BASE/book/random` | Full random base round — click Action to play |
| `MODE_BASE/bookEvent/reveal` | Just the reveal animation in isolation |
| `MODE_BONUS/book/random` | Full bonus round |
| `COMPONENTS/Game/component` | Full game with loading screen |
| `COMPONENTS/Symbol/symbols` | Every symbol in every state |

Work Storybook story by story. When they all pass, you're done.

---

## Important Rules

**Asset loading — critical**
All assets must load from the Stake Engine CDN. No external URLs. No Google Fonts. No external image CDNs. Stake's XSS policy will block them at runtime and reviewers check the console. Host fonts locally in the project.

**SDK sample assets**
The backgrounds, symbols, and animations in the example apps cannot be used in a submitted game. They're for reference only. Every submission needs original assets.

**Node + pnpm versions**
Don't upgrade either. The SDK is pinned. If something seems broken, check `node --version` and `pnpm --version` first.

**Svelte 5**
The SDK uses Svelte 5, not Svelte 4. Use Svelte 5 rune syntax: `$state`, `$derived`, `$effect`. Not the old `let`/reactive statements.

---

## Bet Replay — Mandatory

Every game must support Bet Replay. This is non-negotiable — games are not approved without it.

When the URL includes `replay=true`, the game must:
1. Detect the param on load
2. Fetch the specific simulation from `GET {rgs_url}/bet/replay/{game}/{version}/{mode}/{event}`
3. Disable all betting UI
4. Play through the full animation
5. Show a Play Again button (replays the same outcome)

Check whether the SDK provides scaffolding for this in the example apps. If not, build it manually. See `docs/mock-rgs-spec.md` for the replay endpoint format.

---

## Key Docs

- `docs/frontend-sdk-notes.md` — detailed SDK reference and architecture notes
- `docs/mock-rgs-spec.md` — mock RGS endpoints for local testing
- `docs/maths-guide.md` — quality standards Tom should be aware of
- `docs/stake-engine-md-doc/front-end/` — official SDK docs (offline mirror)

---

## First Things to Try

1. Run `pnpm run storybook --filter=lines` — verify setup works
2. Play through `MODE_BASE > book > random` a few times — understand the event flow
3. Open `bookEventHandlerMap.ts` in the lines app and read through it
4. Open `Symbol.svelte` and see how symbol states are handled
5. Change a symbol's win animation and see it update in Storybook immediately
6. Try running `pnpm run storybook --filter=scatter` — see a different game type
