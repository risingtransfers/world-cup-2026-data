# World Cup 2026 Player Data

Open dataset covering all **48 national squads — 1,363 players** — at the 2026 FIFA World Cup, with per-90 performance stats and AI-computed player similarity examples.

Maintained by [Rising Transfers](https://risingtransfers.com), an AI football intelligence platform built on 56,000+ player profiles.

## What's inside

| File | Contents |
|---|---|
| `data/squads.csv` | All 1,363 squad players: name, nation, position, club, age, and AI-estimated transfer value |
| `data/per90_stats.csv` | Per-90 performance metrics for the 2025-26 season (computed; minimum 450 minutes played) |
| `data/dna_similarity_examples.json` | AI similarity examples: top-5 stylistically similar players for 20 star players |

> Looking for the full player-similarity dataset (hundreds of players)? See [football-player-similarity](https://github.com/risingtransfers/football-player-similarity).

## Quick start

Every row carries a `slug` you can use to look up the full AI profile for any player:

https://risingtransfers.com/en/players/{slug}/alternatives

Example — find players who play like Mbappé:
[https://risingtransfers.com/en/players/kylian-mbappe/alternatives](https://risingtransfers.com/en/players/k-mbappe/alternatives)

Browse all 48 squads with live group tables at the [World Cup 2026 Hub](https://risingtransfers.com/en/world-cup).

## Methodology

- **Transfer value estimates** are produced by Rising Transfers' AI valuation model, not market consensus figures.
- **Per-90 metrics** are computed from seasonal aggregates; players with fewer than 450 league minutes in 2025-26 are excluded to reduce small-sample noise.
- **Similarity scores** come from Player DNA, a semantic embedding of playing style. Scores range 0–1; only model outputs are published (no raw vectors).

## License

Released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution:

> Data: Rising Transfers — risingtransfers.com

## Related datasets from Rising Transfers

- [football-player-similarity](https://github.com/risingtransfers/football-player-similarity) — "who plays like X" for hundreds of players (evergreen)
- [football-data-glossary](https://github.com/risingtransfers/football-data-glossary) — football analytics metrics & methodology reference
