[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/Zhoya0077/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/Zhoya0077/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

# site-generator-speed-01 — fast drop-in for makefun-site 1.0.0

Entry for the make.fun bounty
[site-generator-speed-01-run-20261122](https://make.fun/bounty/site-generator-speed-01-run-20261122)
("Build a static site faster, 22 November 2026", $100, top 10 × $10).

`solve.js` is a drop-in replacement for makefun-site 1.0.0: `run(input)`
returns exactly the output tree that `build()` returns for the same input,
byte for byte — every page, index and tag page, `build-report.txt`, and
every static file. `main.ts` is the harness driver that answers each case
in `/input/cases.json` and writes `/output/answers.json`.

## Why it is faster

The reference spends ~85% of its CPU time in Markdown rendering, mostly
calling `escapeHtml` once per character. This entry keeps every rule and
every decision of the reference and only changes how the work is done:

- **Inline Markdown** scans for the next special character (`\`, backtick,
  `*`, `[`) with one regex and escapes whole plain runs at once, instead of
  escaping character by character.
- **Block rendering** is fused into a single scan that renders straight
  into a chunk array — no token objects, no per-block string concatenation —
  and dispatches on each line's first character instead of running six
  regexes per line.
- **slugify** takes an ASCII fast path (lowercase + one regex) and falls
  back to the exact reference algorithm (NFKD + combining-mark removal +
  character loop) for non-ASCII text.
- **Template rendering** pushes chunks into an array joined once, reuses
  one scope/loop object per for-loop execution, and builds every card, page
  and tag object once per source/tag instead of rebuilding them per render.
- **Front matter** parsing follows the reference exactly (it is ~1% of the
  time, so no divergence risk was taken there).

No I/O, no clock, no environment, no randomness: the answer is a pure
function of the input, and each case is answered alone (nothing is cached
from one case to the next). Node 22, JavaScript only, nothing installed, no
WebAssembly, no native addon, no prebuilt binary.

## Local verification

Practice set (3 cases from the kit), scored byte-identical against the
reference with the kit scorer, and timed with `process.cpuUsage` on
`run()` (median of 9 interleaved reps, Node v24.22.1 on this VM; a second
5-rep run confirmed the same margin, 61.1% faster overall):

| case | baseline | this entry | faster |
|------|----------|------------|--------|
| g-0001 | 160,961 µs | 56,837 µs | 64.7% (2.83×) |
| g-0002 | 261,331 µs | 111,077 µs | 57.5% (2.35×) |
| g-0003 | 290,824 µs | 108,160 µs | 62.8% (2.69×) |
| total | 713,116 µs | 276,074 µs | 61.3% (2.58×) |

Correctness: kit scorer **3/3 pass** on the practice set; **9/9 pass** on
three freshly drawn sets (random seeds, compared against the oracle's
expected answers); two end-to-end `main.ts` runs produce byte-identical
`answers.json` (deterministic).

## License

MIT — see `LICENSE`.
