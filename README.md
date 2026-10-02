# prison-rankup-eco-preset

![Minecraft](https://img.shields.io/badge/Prison-A-Z_%2B_Prestige-6E4AFF?logo=minecraft&logoColor=white) ![Economy](https://img.shields.io/badge/economy-calibrated-success) ![License](https://img.shields.io/badge/License-MIT-green)

> Rank ladder, mine composition and sell prices calibrated as one system rather
> than three independent files.

## The calibration problem

A prison economy breaks in a specific, predictable way: rank costs grow
linearly while mining income grows exponentially (better mines, better enchants,
more multipliers). Early ranks take hours, late ranks take seconds, and the
ladder collapses.

This preset holds **time-per-rank roughly constant** across A-Z by matching the
cost curve to the income curve.

```yaml
ladder:
  base-cost: 5000
  growth-factor: 1.55      # each rank costs ~1.55x the previous
```

Calibration target: **25-40 minutes of active mining per rank**, at that rank's
own income level, from A through Z.

## No rank multiplier — deliberately

```yaml
multipliers:
  prestige-additive: 0.08      # additive, not compounding
  max-total-multiplier: 10.0
```

Mine composition **already** scales income: mine-a is 70% stone, mine-z is 30%
diamond blocks. Adding a rank sell multiplier on top of that compounds two
exponentials and produces hyperinflation by rank T.

Prestige bonus is additive for the same reason: prestige 20 earns 2.6x, not
4.6x. The cap is a backstop, not the intended operating point.

## Mine resets: percentage, not timer

```yaml
reset:
  trigger-percent-remaining: 25
  max-interval-seconds: 900     # ceiling so an unused mine still refreshes
  async-block-placement: true
  blocks-per-tick: 8000
```

A timer resets a half-full mine (wasting blocks) or leaves an empty one (players
stand idle). Resetting at 25% remaining keeps the mine continuously useful.

**Async placement is not optional at scale.** A 100x100x30 mine is 300,000
blocks. Placing those in one tick is a guaranteed watchdog timeout, so resets
are sliced across ticks.

## Price structure

Prices are set against **mine composition**, not vanilla rarity:

| Material | Price | Share of its mine |
|---|---|---|
| `STONE` | 2.0 | 70% of mine-a |
| `IRON_ORE` | 24.0 | 30% of mine-f |
| `DIAMOND_ORE` | 240.0 | 20% of mine-m |
| `ANCIENT_DEBRIS` | 2200.0 | 5% of mine-t |
| `NETHERITE_BLOCK` | 24000.0 | 10% of mine-z |

A block that is 70% of an early mine must be worth far less per unit than one
that is 5% of a late mine, or the early ladder out-earns the late one and the
whole economy inverts.

Block forms pay a small premium over 9 raw units, which rewards reaching the
mines that contain them.

## Permission structure

Ranks inherit linearly — `rank-b` parents `rank-a`, and so on. A permission
granted at rank A does not need repeating 26 times, and removing it from A
removes it everywhere above.

```yaml
tracks:
  prison-ranks:
    groups: [rank-a, rank-b, ..., rank-z]
```

## Files

| File | Contents |
|---|---|
| `config/ranks.yml` | A-Z costs, prefixes, prestige settings |
| `config/mines.yml` | Composition per mine, reset behaviour |
| `config/sell-prices.yml` | Prices and multiplier policy |
| `config/permissions-tracks.yml` | LuckPerms track and inheritance |

Written for Prison-style plugin suites. Key names may need mapping to your
specific plugin, but **the cost and price curves are the transferable part** —
those are the hard thing to get right.

## License

MIT — see [LICENSE](LICENSE).
