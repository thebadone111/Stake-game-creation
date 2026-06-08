# Game Maths Guide — Best Practices & Industry Standards

_Maintained by Tiger. Use this document when reviewing game math with AI or internally._
_Last updated: 2026-06-05_

---

## Purpose

This guide exists so that every game's math can be checked — by a human or an AI — against a consistent set of rules before submission. When asking an AI to review a game's math, share this document alongside the game's `game_config.py`, `gamestate.py`, and the simulation analysis report.

**The prompt to use when checking with AI:**

> "Review this game's math against the standards in maths-guide.md. Flag any violations, near-misses, and design concerns. Be specific about which metric fails and by how much."

---

## Part 1 — Platform Hard Rules (Stake Engine)

These are non-negotiable. Violating any of them causes automatic rejection at ACP validation.

| Rule | Requirement | Notes |
|------|-------------|-------|
| RTP range | 90%–98% | Both ends are hard limits. Target 97% for all games. |
| Multi-mode RTP alignment | All bet modes within 0.5% of each other | e.g. BASE at 97% → all modes must be 96.5%–97.5% |
| Hit rate | Better than 1 in 20 bets (>5%) | This is the floor — most games should be 15%+ |
| Max win achievability | Max win must be possible at least 1 in 10,000,000 rounds | Must be reachable, not just theoretically possible |
| Win range gaps | No gaps in win distribution | Cannot jump from 5× to 500× with nothing between |
| Min simulations | 100,000 per bet mode | 1,000,000+ preferred for accuracy |
| Stateless | Each round fully independent | No state carries between bets — no jackpots, no continuation |
| Bet Replay | Must be implementable for all bet modes | Every simulation must be replayable by event ID |

---

## Part 2 — Volatility Tiers

Volatility is a design choice, not an accident. Decide the tier before writing any code and design toward it.

| Metric | Low | Medium | High |
|--------|-----|--------|------|
| Hit rate | 30–40% | 20–30% | 10–20% |
| Average win (winning spins only) | 1–3× bet | 3–10× bet | 10–50× bet |
| Max win target | 500–1000× | 1000–5000× | 5000×+ |
| Dead spin runs | Rare (1–3 in a row) | Occasional (3–6) | Common (6–15+) |
| Player feel | Steady, lots of small returns | Balanced | Long dry spells, rare big hits |
| Best for | Casual players, longer sessions | Broad audience | High-stakes, bonus buyers |

**Recommended starting point for first games:** Medium. Easier to balance, broader audience, less risk of the dead spin problem hurting engagement scores.

**Island Combat note:** Naturally sits at medium-high due to the binary boss fight mechanic. Target 35–45% overall hit rate (counting shield saves) with average wins of 3–5× bet on winning rounds.

---

## Part 3 — RTP Split (Base Game vs Bonus)

The 97% RTP must be distributed sensibly between the base game and any bonus feature. If the split is wrong, one half of the game is worthless.

| Component | Target contribution to total RTP |
|-----------|----------------------------------|
| Base game | 55–65% |
| Bonus / free spins / feature | 35–45% |

**Example at 97% RTP:**
- Base game contributes 59% → players get reasonable returns just spinning
- Bonus contributes 38% → triggering the bonus is genuinely exciting and worth waiting for

**Buy-bonus modes** (if implemented) are an exception — bonus contribution can be 80%+ since the player is paying a 100× premium specifically to access the feature.

**Red flags:**
- Base game < 50% → players grinding to trigger a bonus get terrible effective RTP and churn
- Bonus < 25% → triggering the bonus barely feels different from base game
- Bonus > 60% in a non-buy-bonus game → game is essentially unplayable without the feature

---

## Part 4 — Win Distribution (The Curve)

RTP is an average. The shape of wins around that average determines how the game feels. Every game should have a smooth, continuous distribution — no gaps, no cliffs.

**Target distribution for winning spins:**

| Win size | % of all winning spins |
|----------|----------------------|
| 0.2–0.5× bet | 20–25% (micro wins — keeps hit rate up) |
| 0.5–2× bet | 30–35% (small wins — frequent, reassuring) |
| 2–5× bet | 20–25% (medium wins — satisfying) |
| 5–20× bet | 10–15% (big wins — exciting moments) |
| 20–100× bet | 3–6% (very big wins — memorable) |
| 100×+ bet | 0.5–2% (massive wins — rare, shareable) |

**Rules for the distribution:**
1. No gap between any two adjacent tiers larger than 3× (e.g. top of a tier at 5× means bottom of next tier must be at most 15×, not 500×)
2. Each tier must be reachable via normal gameplay, not just a theoretical combination
3. The 100×+ tier must have at least a few distinct win values within it — not just one single max-win outcome

**How to check:** After a 100k simulation run, plot the win distribution from the analysis report. It should look like a smooth declining curve from left (small wins, high frequency) to right (large wins, low frequency) with no sudden drops or missing sections.

---

## Part 5 — Hit Rate Standards

Hit rate is how often a spin returns any non-zero win. The 5% platform minimum is a floor no serious game should approach.

**Industry norms by volatility:**

| Volatility | Minimum hit rate | Comfortable target |
|------------|-----------------|-------------------|
| Low | 30% | 35–40% |
| Medium | 20% | 25–30% |
| High | 12% | 15–20% |
| Any (platform floor) | 5% | Never design this low |

**For custom game formats (Island Combat, Gacha, etc.):**
Count any non-zero return as a hit — including consolation payouts on losses. If your game has a binary win/lose mechanic, you almost certainly need consolation payouts to stay above 15% hit rate.

---

## Part 6 — Dead Spin Tolerance

A dead spin is a round that returns zero. Research across the industry consistently shows player drop-off accelerates after 3–5 consecutive dead spins. For longer-format rounds (RPG sequences, gacha reveals) the tolerance is even lower because each dead spin costs the player more time.

**Guidelines:**

- **Standard slots:** Design so that runs of 6+ consecutive dead spins have probability < 1%
- **Long-format rounds (Island Combat, Gacha):** Design so that runs of 4+ consecutive dead spins have probability < 1% — these rounds are 5–10 seconds each, so 4 in a row is 20–40 seconds of nothing

**How to achieve this:**
1. Add consolation payouts — small returns (0.2–0.5× bet) on otherwise losing rounds when certain conditions are met (e.g. collected high-value modifiers but lost boss fight)
2. Near-miss feedback — even if payout is zero, a close near-miss (2 scatters, almost-full modifier stack) maintains engagement
3. Keep losing animations short — if a round is going to return zero, resolve it quickly. Save the long dramatic sequences for wins.

**Check:** From simulation output, calculate the probability of N consecutive zero-win rounds. Flag if P(6+ consecutive zeros) > 1% for slots, or P(4+ consecutive zeros) > 1% for long-format games.

---

## Part 7 — Multi-Mode RTP Alignment

Games with multiple bet modes (BASE/ELITE/RAID or equivalent) must have all modes within 0.5% RTP of each other.

**Common ways this goes wrong:**

1. **Higher cost modes have too many bonus islands/phases** — more phases = more multiplier accumulation = higher expected value. Compensate by reducing the base payout scalar for higher modes.

2. **Buy-bonus modes designed independently** — if you tune BASE and BONUS separately without checking alignment, they'll drift. Always check both after the optimisation run.

3. **Shield/consolation mechanics apply differently** — if shield is more common in one mode, it shifts RTP. Ensure mechanics that affect RTP are mode-aware.

**Process:**
1. Run all modes in the same simulation run (do not run separately)
2. Check analysis report for per-mode RTP figures
3. If any mode is outside ±0.25% of target, adjust base payout scalar for that mode and re-run
4. Do not rely solely on the optimiser to fix large misalignments — fix the design first

---

## Part 8 — Max Win Design

**Platform requirement:** Max win must be achievable at least 1 in 10,000,000 rounds.

**Industry standards:**

| Game type | Typical max win |
|-----------|---------------|
| Low volatility slot | 500–1000× |
| Medium volatility slot | 1000–5000× |
| High volatility slot / Megaways | 5000–50000× |
| Custom format (first games) | 500–2000× |

**Guidance for first games:** Target 1000–2000× max win. High max wins require more complex math to distribute wins correctly, and reviewers will scrutinise whether the game actually delivers on it. A well-designed 1000× max win game scores better than a poorly distributed 10000× game.

**Max win must be reachable without a perfect storm of improbable events.** If achieving max win requires 5 independent rare events each with 1% probability, the combined probability is 0.00001% — effectively unreachable and will fail platform validation.

---

## Part 9 — Game-Specific Checklists

### Standard Slot (SDK Stack)

- [ ] RTP 96.5%–97.5% on all modes
- [ ] Hit rate 15%+ (aim for 20–30% for medium volatility)
- [ ] Win distribution smooth — no gaps, check plot from analysis report
- [ ] Base game / bonus RTP split 60/40
- [ ] Max win 1000–5000× and achievable 1 in 10M
- [ ] Dead spin runs: P(6+ consecutive) < 1%
- [ ] All modes within 0.5% RTP of each other
- [ ] Bet Replay implementable for all modes including bonus trigger

### Island Combat (Hybrid Stack — RPG format)

- [ ] RTP 96.5%–97.5% on BASE, ELITE, RAID
- [ ] Overall hit rate 35–45% (wins + shield saves + consolation payouts)
- [ ] Consolation payouts exist for high-modifier losing rounds (0.2–0.5× bet)
- [ ] Dead spin runs: P(4+ consecutive) < 1%
- [ ] Modifier combinations produce smooth win distribution — no gaps
- [ ] RAID mode (5 islands) has higher multiplier potential but aligned RTP via lower base scalar
- [ ] Boss win probability varies by modifier stack but overall RTP stays at target
- [ ] Losing animation shorter than winning animation

### Gacha / CS2 Case (Custom format)

- [ ] RTP 96.5%–97.5%
- [ ] Rarity tier payout ranges overlap slightly (no gap between bottom of Rare and top of Uncommon)
- [ ] Hit rate defined — every pull returns something (technically 100% hit rate but payout can be < 1× bet)
- [ ] Payout multipliers per rarity tier produce smooth overall distribution
- [ ] Legendary/Mythic tier achievable within max win requirements

---

## Part 10 — How to Use AI to Review Math

When asking an AI (Claude or otherwise) to review a game's math, provide:

1. **This document** (maths-guide.md)
2. **`game_config.py`** — symbols, modifiers, bet modes, RTP target
3. **`gamestate.py`** — the run_spin() logic
4. **Analysis report output** — the per-mode RTP, hit rate, and win distribution figures from the simulation run

**Suggested prompt:**

> "I'm building a game for Stake Engine. Review the attached game_config.py and gamestate.py against the standards in maths-guide.md. Check: RTP target and multi-mode alignment, hit rate against volatility tier, win distribution for gaps, dead spin probability, base/bonus RTP split, and max win achievability. Flag any violations with the specific metric and how far off it is. Then flag any design concerns that aren't hard violations but might cause low review scores."

**What AI is good at checking:**
- Logic errors in run_spin() (e.g. a condition that never triggers)
- Whether modifier math produces the expected multiplier ranges
- Whether stated win probabilities produce the claimed RTP
- Whether the distribution design will have gaps

**What AI cannot check without simulation output:**
- Whether the optimiser actually hit the RTP target
- Actual dead spin run probabilities (needs the full distribution data)
- Whether the win distribution curve is smooth (needs the analysis report numbers)

Always run the simulation and provide the output — don't rely on AI to validate math that hasn't been run yet.

---

_See also: `resources/stake-requirements.md` — full platform rules_
_See also: `docs/math-sdk-notes.md` — SDK technical reference_
_See also: `process/review-checklist.md` — Ollie's full pre-submission checklist_
