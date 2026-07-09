# Level Progression

How difficulty and rewards scale as `gameState.level` climbs. There is no final level — this loops indefinitely.

## Per-level spawn

- **Enemy count**: `2 + level` total, split roughly half-and-half between the left and right sides of the house.
  - Half of one side's share are ranged `Enemy` (patrol + shoot at player).
  - The other half are `ChaserEnemy` (no ranged attack, homes in on the player, explodes for 10 house damage on contact).
- **Spawn timing**: all queued enemies get a random delay spread across a 5–10 second window after the level banner clears — they trickle in rather than appearing all at once.
- **Speed scaling**: both `Enemy` and `ChaserEnemy` speed = `2 + floor(level / 2)` (patrol speed for `Enemy` is computed slightly differently at construction, then overridden to this value on spawn).
- **Enemy fireball speed** (ranged `Enemy` only): `6 + level * 0.5`.

## Milestones

| Every... | Effect |
|---|---|
| **5 levels** (`level % 5 == 0`) | A `BossSlime` (500 HP) spawns immediately, walking in from off-screen left. Also grants `level / 5` upgrade points. |
| **10 levels** (`level % 10 == 0`) | `maxLives += 1`, and `lives` refills to the new max. |
| **Level 11** (one-time) | Before that level's enemies spawn, the game pauses and plays the Charizard evolution cutscene — see [charizard_cutscene.md](charizard_cutscene.md). Enemies for level 11 spawn only after the cutscene finishes. |

## BossSlime detail

- 500 HP; takes 10× the player's `damage` stat per fireball hit (vs. 1× for normal enemies), worth 500 score on death (vs. 50).
- While HP > 100: periodically "spits" mini-slimes that crawl horizontally along the ground toward the house. Spit count scales with damage taken so far: `floor((500 − health) / 100) + 1`.
- At HP ≤ 100: enters **rage mode** (one-time trigger) — switches to `spawnSkyRain()`, dropping 12–17 mini-slimes from random points above the map toward the house, repeating every 5s instead of the ground-crawl spit. Also triggers a red screen flash and "!! ENRAGED !!" banner.

## Upgrade stats (spent via the Stats menu, 1 upgrade point each)

| Stat | Effect per point |
|---|---|
| Fireball Damage | +1 damage per hit (vs. normal enemies; ×10 multiplier still applies vs. `BossSlime`) |
| Max Lives | +1 max & current lives |
| House Max HP | +20 max HP (and current, on the house that exists *right now* — a fresh `House` next level always starts at `100 + 20 × houseHealth stat`) |
| Move Speed | +1 to the player's effective move speed (base 8) |

## What resets between levels vs. what persists

- **Resets every `spawnEnemies()` call** (i.e., every new level, and every life-loss-to-zero retry of the *same* level): House (full HP for that level's bonus tier), enemies, spawn queue, fireballs, enemy fireballs, mini-slimes.
- **Persists across levels**: score, level number, lives/maxLives, upgrade points, upgrade stats, player position. These are also the fields round-tripped by Export/Import State (see [game_loop.md](game_loop.md)).
