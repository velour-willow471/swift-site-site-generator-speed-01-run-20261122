[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/velour-willow471/swift-site-site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/velour-willow471/swift-site-site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

# swift-site

A faster drop-in for makefun-site 1.0.0, written from scratch for the
[make.fun bounty "Build a static site faster, 22 November 2026"](https://make.fun/bounty/site-generator-speed-01-run-20261122).
`solve.js` exports `run(input)`, which returns exactly what makefun-site 1.0.0
returns for every `{ site, templates, files }` input: the whole output tree —
every page's `index.html`, the index and tag pages, `build-report.txt` and
each static file — including its front-matter refusals, slug collisions,
Markdown subset, template dialect and broken-link report. `main.ts` is the
harness driver: it reads `/input/cases.json`, answers each case with `run`,
and writes `/output/answers.json`.

## How it is fast

- Markdown is rendered in one pass over the body's lines, with no token
  objects and no per-character escaping: inline text is copied in whole runs
  between markup characters, and a run is escaped with one regex pass only
  when it holds a character that needs escaping.
- Slugs and long dates are memoised per build (titles, tags and headings
  repeat constantly), and pure-ASCII slugs skip Unicode normalisation.
- Front matter is parsed by hand, line by line, with no regex per line, and
  the body is never re-joined into a string just to be split again.
- Templates are parsed once per build and rendered by an interpreter that
  threads one output array through the whole render and resolves a path's
  first name through a prototype-chained scope, so a loop variable or a root
  value is one property read, not a chain walk.
- Word counts are counted by a table-driven character scan instead of a
  regex match that builds an array of every word, and each page's card is
  built once, not once per render.

## Measured

On the kit's practice set (3 cases, 250–480 Markdown sources each), this
entry's `run` takes about 290 ms of CPU per full set against about 720 ms
for the house baseline on this machine (Node 24, shared CPU — the graded
heat runs Node 22 in a 1-CPU container), roughly a 2.4× speedup. All 3
practice cases match the reference byte-for-byte as JSON values, and the
same holds on 36 further cases drawn from the kit's own generator with
fresh seeds.

## Files

- `solve.js` — the generator (dependency-free ES module)
- `main.ts` — the harness driver (Node 22, `--no-incremental-marking` as the house driver)
- `LICENSE` — MIT
