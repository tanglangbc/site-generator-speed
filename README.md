# site-generator-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/tanglangbc/site-generator-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/tanglangbc/site-generator-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0 static site generator.
`solve.js` is a dependency-free ES module whose `run(input)` returns exactly
the answer makefun-site 1.0.0 returns for every case's input — the output tree
as a JSON object of path → text, byte for byte — but roughly 3× faster on CPU
time.

## How it is faster

Same pipeline, same semantics, less work per byte:

- Inline Markdown renders in one native regex scan per string; the plain runs
  between special characters are copied verbatim (they hold no escapable
  character), instead of calling `escapeHtml` once per character.
- Markdown is lexed and rendered in a single pass over the lines, classified
  by `charCodeAt` checks instead of a battery of per-line regexes, with no
  token objects allocated.
- Front matter is parsed by index scanning instead of per-line regexes.
- `slugify` takes a pure-ASCII fast path (NFKD is the identity on ASCII).
- Templates render into an array joined once instead of concatenating a
  string per node; a post's page reuses its already-built card.
- Word counting uses the native `\S+` match.

Nothing is cached from one case to the next; the build reads no clock, no
environment, no randomness and no file.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=output node main.ts
```

`main.ts` is the harness driver: it reads every case from
`$MAKEFUN_INPUT_DIR/cases.json`, answers each with `run` from `solve.js`,
and writes `$MAKEFUN_OUTPUT_DIR/answers.json`.

## Benchmark (local, practice set, 3 cases)

| implementation | CPU time / case |
|---|---|
| makefun-site 1.0.0 (house baseline) | ~221 ms |
| this entry | ~72 ms |

About **3.1× faster** than the house baseline, with identical output on all
3 practice cases (verified against the kit's `practice/reference/expected.json`).

---

<sub>Entered on [make.fun](https://make.fun/?ref=NVRADX&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
