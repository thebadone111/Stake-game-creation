# Game Ideas

_Last updated: 2026-06-27 | Living document — add and evolve ideas here_

---

## Current Status
- **Ayakashi** (yokai 5x5 slot) — submitted to Stake (`ayakashi1` branch)
- **Ayakashi 2** — in development on `final-dev` branch, art + animation overhaul in progress

## Strategy

1. **High-quality original games** — polish and quality first. Ayakashi is the template. Build on it.
2. **Unexplored formats** — gacha, collectible, RPG-skin mechanics. No competition on Stake Engine. Higher ceiling, higher risk.
3. **Reskins as fast follow-ups** — once the pipeline is proven, reskins of top performers can be produced quickly.

Priority: finish Ayakashi 2 to a high quality bar, then move to CS2 Case Opening or the next original concept.

---

## Tier 1: Reskins — Japanese Theme

**Why Japanese?** Three proven performers in the top 20 with no single studio owning the theme. Stake's audience (crypto-native, gaming-crossover) responds to it. Anime/manga assets are widely available and cheaper to produce than Western fantasy.

---

### Reskin A — Based on Dojo Duel (Titan Gaming, #10 — $24.5M/month)
**Theme direction:** Underground fighting tournament. Rival martial artists, each with a distinct fighting style. Neon-lit arena, crowd atmosphere.

**Mechanical changes to make it distinct:**
- Same ways-style win mechanic but add a "finishing move" bonus trigger — when a fighter symbol fills a full reel, it expands and locks for a re-spin
- Add a "training mode" buy-bonus equivalent — costs 10x, starts with wilds locked

**Why this theme hasn't been done:** Dojo Duel is clean and restrained. A grittier, more hype-driven underground fighting aesthetic (think Street Fighter × Yakuza) is adjacent but clearly different.

---

### Reskin B — Based on Harakiri (Colorful Play, #13 — $21M/month)
**Theme direction:** Yokai hunters — Edo-period ghost hunters tracking supernatural creatures. Dark, atmospheric, folklore-based.

**Mechanical changes to make it distinct:**
- Keep scatter-triggered free spins but add a "yokai seal" mechanic — scatters leave a persistent multiplier on the grid for the duration of free spins
- Each yokai type has a different multiplier value (Kappa 1x, Oni 3x, Tengu 5x, Kitsune 10x)

**Why this theme hasn't been done:** Harakiri is samurai/dark aesthetic. Yokai/folklore is a distinct visual direction with a lot more character variety to design. Could also do anime style, AI models usually better at Anime style design.

---

### Reskin C — Based on MIKO (Paperclip Gaming, #16 — $20.3M/month)
**Theme direction:** Anime idol group — J-pop performance theme, stage lights, choreography. Lighter, more colourful, broader appeal.

**Mechanical changes to make it distinct:**
- Cluster pays instead of lines (MIKO likely uses ways) — groups of matching idols explode and cascade
- "Encore" free spin trigger — when triggered, global multiplier increases by +1 each cascade (same pattern as scatter-pays example game in math SDK)

**Why this theme hasn't been done:** MIKO is shrine maiden — spiritual, restrained. An idol/performance theme is louder, more colourful, and targets a different segment of the anime audience.

---

## Tier 2: New Formats — Unexplored Territory

_None of these exist on Stake Engine as of June 2026. Higher build complexity, higher reward if it works._

---

### Idea 1 — CS2 Case Opening (Loot Box Slot)

**Competition:** `fluxgaming-case-opening` already exists on Stake. We can build a better version — more polished animations, more cases, better rarity feel. The format is proven.

**Concept:** Players open cases exactly like CS2/Counter-Strike. Pick a case, pay the fee, watch the scroll animation, item revealed with a rarity tier. Payout = item value relative to bet.

**Why it works on Stake:**
- Maps perfectly to pre-simulated outcomes — each simulation IS an item drop with a known rarity and payout multiplier
- Stake's audience is heavily CS2/crypto crossover — this is their native visual language
- The "scroll" animation (items fly past before landing) is more exciting than reels

**Mechanics:**
- 6 rarity tiers: Consumer (white), Industrial (light blue), Mil-Spec (blue), Restricted (purple), Classified (pink), Covert (red/yellow)
- Each tier maps to a payout multiplier range
- "StatTrak™" equivalent as a rare modifier — same item but shown with a kill counter animation, +50% multiplier
- Multiple case types as bet modes — each case has a different item pool / volatility profile
- Bounces, some cases spin multiple times, to keep engagement high

**Build stack:** SDK stack (it's fundamentally a slot with a different visual). The "scroll" is just a reel strip dressed differently.

**Visual complexity:** Medium. Items need art, scroll animation, case opening animation, rarity flare effects. Manageable without a dedicated animator if AI-generated item art is used carefully.

---

### Idea 2 — Island Combat (Stateless RPG)

**Core insight:** This looks like a clicker/idle RPG but is technically a standard stateless slot with a combat animation skin. Single `/wallet/play` call. Fully compatible with the math SDK. No multi-request complexity.

**Concept:** Player bets → watches their hero clear 3 islands collecting modifiers → boss fight plays out → win or loss revealed. Everything is pre-calculated. The player watches a story, not a random number.

**The bet-as-damage mechanic:**
Bet size scales the displayed numbers cosmetically — €5 bet = 5 base damage, €50 bet = 50 base damage. Boss HP scales proportionally. Win rate stays identical across all bet sizes (RTP compliance). Players *feel* the connection between bet size and power without it affecting probability.

**A round step by step:**
```
Player bets €5
  → POST /wallet/play (single call)

Server returns pre-simulated events:
  { type: "islandClear", island: 1, modifier: "attackUp" }
  { type: "islandClear", island: 2, modifier: "shield" }
  { type: "islandClear", island: 3, modifier: "critChance" }
  { type: "bossFight", playerHP: 18, bossHP: 0, result: "win" }
  { type: "finalWin", amount: 4.5 }

Frontend animates the sequence (~8 seconds)
Player watches — all outcomes already determined
```

**Modifiers (drop from island clears, affect payout multiplier):**
| Modifier | Effect |
|---------|--------|
| Attack Up | 2x multiplier on boss win |
| Critical Chance | 3x multiplier on boss win |
| Shield | Second chance if boss win — re-fight at 1x |
| Double Strike | Two hits per round in boss fight animation |
| Rage | 5x multiplier but lose shield protection |

**Math structure (Python):**
Maps directly to the cascading tumble pattern in `0_0_scatter`. Islands = tumble phases, modifiers = multiplier symbols, boss outcome = final evaluation. The three islands before the boss mirror the scatter/freegame buildup structure already in the example games.

**Bet modes:**
- BASE (1x): 3 islands, 1 boss — standard run
- ELITE (2x): 4 islands, harder boss, higher max win
- RAID (10x): 5 islands, legendary boss, cinematic fight, max win cap increased

**Stateless compatibility:** Fully compatible. The entire run is one simulation. The "clicker" feel is purely animation pacing.

**Build stack:** Hybrid (Phaser). Multi-scene flow: island scenes → boss scene → results screen. Does not fit the slot SDK structure.

**Visual complexity:** High. Needs hero sprite, enemy sprites per island, boss sprites, attack animations. Flag for post-designer-hire. This is game 4 or 5.

**Scalability:** Later entries can introduce hero classes as separate games — Warrior (high damage, low modifier chance), Mage (low damage, high modifier multipliers), Rogue (crit-heavy, binary win/loss). Each is a separate submission with the same engine.

---

### Idea 3 — Gacha Pull with Collectible PNG

**Concept:** Players pull for collectible character cards. Each pull reveals a character with a rarity tier and a payout multiplier. The card is a real downloadable PNG — the player owns it aesthetically. High-rarity cards have significantly higher payouts.

**Core loop:**
- Place bet → pull animation (card flip/reveal) → character card revealed with rarity flash → win credited → option to download the card PNG

**Rarity tiers and payout ranges:**
| Tier | Colour | Payout range | Pull rate |
|------|--------|-------------|-----------|
| Common | Grey | 0.1x–0.5x | ~60% |
| Uncommon | Green | 0.5x–2x | ~25% |
| Rare | Blue | 2x–10x | ~10% |
| Epic | Purple | 10x–50x | ~4% |
| Legendary | Gold | 50x–500x | ~1% |
| Mythic | Animated | 500x–max win | <0.1% |

**Collection appeal:** Design 100+ unique characters across the rarity tiers. Players naturally want to "complete" the set — this is retention the slot format doesn't have.

**Downloadable PNG:**
The card PNG is a static asset hosted on the Stake CDN. When a player gets a result, a download button appears. The PNG includes the character art, rarity border, pull date, and a unique serial number visible on the card face.

**Stateless compatibility:** Each simulation maps to a specific card (character + rarity + serial number). The serial number on the PNG face is unique per simulation ID, not per player — acknowledge this in the game design (the card is the art, not a proof of uniqueness).

---

### Idea 3b — Gacha PNG + Off-Platform NFT Claims

**Extension of Idea 3.** For Legendary and Mythic tier pulls, the downloaded PNG can carry two off-platform hooks that work without the game ever making an external request:

**Hook A — Embedded URL in PNG metadata (XMP/EXIF):**
- Each high-rarity PNG has a URL embedded in its metadata pointing to an NFT marketplace or claim portal
- Players who open image properties or use a reader see the link
- The game's JavaScript never makes an external call — it's static metadata in an image file
- Stake's XSS policy covers JavaScript calls, not image metadata — this is almost certainly fine
- Could also include a discount code for NFT purchases embedded in the metadata for high-rarity pulls

**Hook B — Visible serial number + off-platform claim portal:**

The cleaner, more player-visible approach:

```
1. Pre-generate card PNGs before launch
   Each card has a unique serial number visible in the card artwork itself
   e.g. "XF-4291-7K" printed on the card border

2. Publish a signed lookup table off-platform:
   serial_number → NFT token ID → claimed (true/false)

3. Player wins a Legendary pull
   Game shows the card + serial number on screen
   Download button serves the static PNG from Stake CDN

4. Player visits our separate NFT claim site (no connection to the game)
   Enters or uploads their serial number
   Site verifies it’s unclaimed, mints/transfers the NFT to their wallet
   Marks serial as claimed — prevents double-claim

5. The game makes zero external calls
   Entirely off-platform — clean separation
```

**The simulation collision problem and how to handle it:**
The same simulation can be served to multiple players over the game’s lifetime. The first person to claim a given serial number gets the NFT. Subsequent players with the same simulation get the PNG but find the NFT already claimed. Solutions:
- Match NFT supply to expected simulation frequency (if sim #42 hits ~once per 10,000 bets and you expect 1M lifetime bets, pre-mint ~100 tokens for that simulation)
- Communicate clearly in-game: "First claim wins the NFT — the card is always yours to keep"

**What to build first:** Idea 3 (PNG + serial number, no NFT infrastructure yet). Prove the format and get approval. NFT claim portal is a separate off-platform build that can launch alongside or after.

**Honest flags:**
- Combining gambling payouts with off-platform asset claims is legally untested in many jurisdictions — flag for legal review
- Stake’s terms may have opinions on off-platform promotion — check on first ACP login
- The NFT layer needs to be in the original submission (post-approval changes to mechanics are not allowed)
> **This is our magnus opus. Long-term project. Do not start until Idea 3 (base gacha) is live and approved.**
>
> Full build order: reskin slots (games 1–3) → base gacha approved → smart contract + claim portal → NFT-enabled resubmit.
>
> Full technical flow documented in conversation history — re-read before kicking off Phase 1.
---

## Backlog / Rough Ideas

- **Wheel of Fortune variant** — spin a wheel with a known prize ring, but the "power" of the spin is random. More theatrical than slots.
- **Fishing game** — cast into a pond, reel in fish of varying sizes. Popular in Asian mobile gaming markets. Maps to single-outcome reveals.
- **Monster collection** — similar to gacha but combat-focused. Roll a monster, fight a dungeon enemy. Hybrid idle/gacha.
- **Trading card pack opening** — similar to CS2 case but card-game aesthetic. Could tie into the PNG collectible system.

---

_See also: `docs/how-games-work-hybrid-stack.md` for the Phaser stack that handles non-slot formats_
_See also: `/memories/game-ideas.md` for high-level vision notes_
