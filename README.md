[![make.fun](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/BaleineGu/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/BaleineGu/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

# Forecast GTA VI's launch-day Twitch viewers hour by hour

Entry for the make.fun bounty `gta6-twitch-ccu-forecast-01` ($250, 20 places
paid, deadline 2026-11-19T01:00:00Z), by @guinrm.

## What it does

`main.py` (Python 3.13, standard library only, no network) reads one case
file at a time from the input directory and writes one forecast file per
case to the output directory: 23 rows, hours 2–24 of the UTC release day,
five quantiles (0.05, 0.25, 0.5, 0.75, 0.95) of that hour's concurrent
Twitch viewers in the Grand Theft Auto VI category, anchored on the
case's hour-1 print.

## The model

The case file carries every comparable launch's full release day. The
program learns the shape of a launch day from those days, as ratios to
hour 1 — and the shape depends strongly on how big hour 1 was (across
the kit's comparables, log hour 1 and log peak/hour 1 correlate at about
−0.94: already-busy categories stay flat, nearly-empty ones explode as
regions unlock). Two views of that dependence are blended per hour, as
a geometric mean of their quantiles:

1. **Neighbours** — the 12 comparables whose log hour-1 count is closest
   to this case's, and the empirical quantiles of their log ratios.
2. **Regression** — a per-hour linear fit of log ratio on log hour-1
   across all comparables, with its residual spread.

The median ratio is clipped to [0.02, 50]× hour 1 and the 5%–95% band is
never narrower than a factor of ~1.5×, so a strange comparable set cannot
produce an absurd or collapsed forecast. With no usable comparables it
falls back to flat persistence. Everything is computed from the case
file at run time, so the same program works on the practice cases and
on the final GTA VI case.

## Validation

Leave-one-out scoring on the kit's 37 comparable launch days, using the
contest's own score (mean pinball loss on the log scale):

| model | mean pinball, all 37 | mean pinball, hour-1 ≥ 100k |
| --- | --- | --- |
| house baseline (flat persistence) | 0.93 | 0.26 |
| this blend | **0.41** | **0.24** |

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

With the kit's practice inputs, e.g. from the kit directory:

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 /path/to/entry/main.py
```

Two runs write identical bytes. Each case runs well inside the 8-second
limit (a few dozen comparables, a few hundred arithmetic operations).

## License

MIT — see `LICENSE`. Viewer data in the kit comes from SullyGnome
(hourly Twitch category averages); see the kit's `input/NOTICE.txt`.
