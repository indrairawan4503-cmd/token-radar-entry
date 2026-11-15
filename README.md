# Token Radar (radar-2)

A program that ranks pre-token DeFi protocols by what their token launch would be
worth, and renders the ranking as a static, no-network web dashboard. Entry for the
make.fun bounty **token-radar-01** ("Build the best token launch radar").

## What it does

`main.ts` (Node 22, standard library only) reads the sealed DefiLlama snapshot at
`/input/snapshot.json`, scores every rankable pre-token protocol, and writes:

- `board.json` — 197 ranked rows (all 40 watchlist protocols, plus every other
  rankable pre-token protocol below the floor), each with an implied value and a
  six-step farm plan.
- `data.js` — the same board as a script the dashboard reads.
- `dashboard/` — the static page: sortable/filterable board, a detail view per
  protocol with the model's own inputs and its farm plan, a live-token view set
  apart, and the full methodology.
- `screenshots/` — the board, a protocol's detail view, and a filtered view.
- `dossier.json` — built with `vendor/makefun-board.ts`, the summary the judge reads.

## The model (radar-2)

A protocol is rankable when its record carries no token (no market cap, no
CoinGecko id, no symbol) and it is not a wrapped, staked or bridged product of a
token that already trades — the watchlist's own exclusion rule, applied to the
whole board. Implied value follows only from the snapshot's own figures:

```
implied = round(tvl_usd
  x category_multiple        median market-cap/TVL of the live-token protocols
                             in the same category in this snapshot (>= 3 comps,
                             clamped to [0.08, 0.80]; else the snapshot median)
  x (1 + 0.04 x extra chains, max 5)
  x (1 + clamp(change_7d, -50, 150) / 100 x 0.6)
  x (0.7 + min(age_days, 1095) / 1095 x 0.5))
```

No protocol name is special-cased. The dashboard recomputes every row from the
same figures, so the page and `board.json` always agree.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Then open `out/dashboard/index.html` (it reads `out/data.js` beside it).

## License

MIT — see `LICENSE`.
