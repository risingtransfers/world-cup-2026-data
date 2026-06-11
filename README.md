# World Cup 2026 Player Data

Open dataset covering all **48 national squads — 1,363 players** — at the 2026 FIFA World Cup, with per-90 performance stats and AI-computed player similarity examples.

Maintained by [Rising Transfers](https://risingtransfers.com), an AI football intelligence platform built on 56,000+ player profiles.

## What's inside

| File | Contents |
|---|---|
| `data/squads.csv` | All 1,363 squad players: name, nation, position, club, age, and AI-estimated transfer value |
| `data/per90_stats.csv` | Per-90 metrics for **1,181** players in 2025-26 (computed; minimum 450 league minutes) |
| `data/dna_similarity_examples.json` | AI similarity examples: top-5 stylistically similar players for 20 star players |

> Looking for the full player-similarity dataset (hundreds of players)? See [football-player-similarity](https://github.com/risingtransfers/football-player-similarity).

## Quick start

Every row in `squads.csv` and `per90_stats.csv` includes a `slug` — use it to open the full AI profile on Rising Transfers:

https://risingtransfers.com/en/players/{slug}/alternatives


**Example — players who play like Mbappé:**

https://risingtransfers.com/en/players/k-mbappe/alternatives

Browse all 48 squads with live group tables at the [World Cup 2026 Hub](https://risingtransfers.com/en/world-cup).

## Methodology

- **Transfer value estimates** (`rt_value_estimate_eur`) are produced by Rising Transfers' AI valuation model — not third-party market consensus figures.
- **Per-90 metrics** are computed from seasonal aggregates. Players with fewer than 450 league minutes in 2025-26 are excluded from `per90_stats.csv` to reduce small-sample noise.
- **Similarity scores** come from Player DNA, a semantic embedding of playing style. Scores range 0–1; only top-N results are published (no raw 768-dim vectors).

See [docs/data_dictionary.md](docs/data_dictionary.md) for column definitions.

## License

Released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution:

> Data: Rising Transfers — risingtransfers.com

## Related datasets from Rising Transfers

- [football-player-similarity](https://github.com/risingtransfers/football-player-similarity) — "who plays like X" for hundreds of players (evergreen)
- [football-data-glossary](https://github.com/risingtransfers/football-data-glossary) — football analytics metrics & methodology reference
