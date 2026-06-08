# Tom — Frontend Lead

## Role
Frontend Lead. Owns the frontend build — PixiJS/Svelte SDK, Storybook, animations, and asset integration.

## What This Looks Like Day-to-Day
- Receive the game brief, art assets, and event structure from Tiger/Zacke
- Build the frontend using the Stake Engine web SDK (PixiJS + Svelte 5)
- Wire BookEvents to EmitterEvents in `bookEventHandlerMap.ts`
- Build and test all animations via Storybook
- Integrate art assets (spritesheets, backgrounds, UI)
- Push to GitHub — Cloudflare Pages auto-deploys for review
- Implement Bet Replay (mandatory for every submission)

## Focus Areas
- Storybook-first — every BookEvent story passing is the definition of done
- Animation quality — aim for solid 2-star on reskins, higher on 3-star sessions
- Asset integration and spritesheet management
- Mobile layout support
- Bet Replay implementation (non-negotiable — no approval without it)

## Working Style
- Works Taiwan evenings
- Tiger is available during this time and can assist on frontend
- Goal: push to GitHub before the end of each session so Ollie has something to review the next European morning
- Tom always moves to the next game — bounce fixes are handled by Tiger + Ollie

## Notes
- Never pulled into bounce fixes — always focused on the next game
- Goal is to have art + brief + event structure waiting when the session starts — flag Tiger early if anything is missing
- All assets must load from the Stake Engine CDN — no external URLs, no Google Fonts
- See `docs/setup-frontend.md` for environment setup
- See `docs/frontend-sdk-notes.md` for SDK reference
- See `docs/maths-guide.md` for quality standards
