# Tom's Flow — Per Game

## What You Receive Before Starting

- `config.ts` from Zacke — complete, drop in as-is. Do not edit paytable or reel strip values.
- BookEvent structure from Zacke — list of custom event names + their payload shapes
- Art assets from Tiger — symbols, background, packed atlases

---

## Steps

**1. Copy the base app**
```bash
cp -r apps/lines apps/new-game-name
```

**2. Drop in Zacke's config.ts**
Replace the existing one. Done. Don't touch the values inside.

**3. Drop in the art assets**
Replace everything in `assets/sprites/symbolsStatic/` and `assets/spines/` with the new art from Tiger.

**4. Update asset keys in constants.ts**
`SYMBOL_INFO_MAP` needs to point to the correct asset filenames. If Tiger named a file `dragon.webp` instead of `h1.webp`, update the key here. Also adjust size ratios if the new symbols have different proportions.

**5. Wire any new BookEvents**
For each custom event Zacke defined (anything beyond the standard `reveal`, `winInfo`, `setWin` etc), add a handler in `bookEventHandlerMap.ts` and build the animation component that responds to it.

For a straight reskin with no new mechanics, this step is skipped entirely.

**6. Update mock data in Storybook**
Drop Zacke's BookEvent examples into `stories/data/` so each story has realistic test data to play against.

**7. Test story by story in Storybook**

| Story | What to verify |
|-------|---------------|
| `MODE_BASE/book/random` | Reels spin and land, wins display, round completes cleanly |
| `MODE_BASE/bookEvent/reveal` | Symbols land in correct positions |
| `MODE_BASE/bookEvent/winInfo` | Winning positions highlight in sequence |
| `MODE_BONUS/book/random` | Full bonus round plays through |
| `COMPONENTS/Symbol/symbols` | Every symbol renders correctly in every state |

When every story passes, the game is done.

**8. Push to GitHub**
Cloudflare auto-deploys. Drop the preview URL in the group chat for Ollie to review next morning. Move straight to the next game.

---

## Reskin vs New Mechanic

| Task | Reskin | New mechanic |
|------|--------|-------------|
| config.ts | Drop in from Zacke | Drop in from Zacke |
| Art assets | New art | New art |
| constants.ts | Update asset keys + size ratios | Update asset keys + size ratios |
| bookEventHandlerMap.ts | Unchanged | Add new event handlers |
| New animation components | None | Build per new event |
| Storybook mock data | Same structure | New event examples from Zacke |

A clean reskin is one session. New mechanics add time proportional to the animation complexity.

---

## Key Rules

- All assets must load from the Stake Engine CDN — no external URLs, no Google Fonts
- Bet Replay is mandatory — every submission needs it
- Never pulled into bounce fixes — always move to the next game
- Flag Tiger early if art or the handoff package is missing before your session starts

---

_See also: `docs/setup-frontend.md` for environment setup and the full event system explanation_
