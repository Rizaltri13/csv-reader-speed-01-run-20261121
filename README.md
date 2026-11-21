# Read broken CSV files faster

[![make.fun](https://make.fun/badge/csv-reader-speed-01-run-20261121.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![Rank](https://make.fun/badge/csv-reader-speed-01-run-20261121/Rizaltri13/csv-reader-speed-01-run-20261121/rank.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![Reward](https://make.fun/badge/csv-reader-speed-01-run-20261121/Rizaltri13/csv-reader-speed-01-run-20261121/reward.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)

A make.fun bounty entry for
[csv-reader-speed-01-run-20261121](https://make.fun/bounty/csv-reader-speed-01-run-20261121):
a faster drop-in for csv-reader 1.0.0. `solve.js` answers every case's
`{text, header}` with exactly the `{columns, rows, errors}` csv-reader
returns — same broken-row policy (BOM drop, LF/CRLF record ends, stray CRs,
quoted fields with doubled quotes, quote junk, unterminated quotes,
short/long rows with errors) — and `main.ts` is the program the harness
runs: it reads every case from `/input/cases.json` and writes
`/output/answers.json`.

## How it is fast

One left-to-right pass, no per-character loop on the common path.
Unquoted fields are found with `indexOf` (next comma / next line feed)
and taken as a single slice; quoted fields are found with `indexOf`
(next quote) and taken as slices, joined only when a doubled quote or a
line break inside them forces it. Nothing is built one character at a
time, and no regular expression runs anywhere. `main.ts` turns V8's
incremental marking off before importing `solve.js`, as the house driver
does.

## Run it

```sh
MAKEFUN_INPUT_DIR=./practice/input MAKEFUN_OUTPUT_DIR=./out node main.ts
```

Node 22 or later, no dependencies, no network. The program is pure and
deterministic: two runs on the same input write byte-identical answers.

## Benchmark (practice set, 6 cases, ~10 MB of CSV)

Whole-program wall time on this machine, 7 runs each:

| entry | median |
| --- | --- |
| this entry | 752 ms |
| house baseline (csv-reader 1.0.0) | 1619 ms |

About **2.1× faster** than the baseline end to end; the parser alone
runs about 3.8× faster (the rest is JSON parsing and serialisation,
identical for both). The kit scorer passes all 6 practice cases exactly
(`cases_passed` 6/6), and the answers are byte-identical to the
baseline's.

## Correctness

Beyond the practice set, the parser was checked against csv-reader
1.0.0 on 8,074 differential fuzz cases — random texts, structured
broken-row cases (short/long rows, stray CRs, quote junk, blank lines,
unterminated quotes, CRLF/LF mixes, BOM) and targeted edges — with zero
mismatches.
