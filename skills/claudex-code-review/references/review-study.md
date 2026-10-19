# Multi-model review study

Study date: 2026-10-18. One review per model at `xhigh` found more serious defects than any combination of reviews at `high`, including two reviews per model. False findings were refuted in the cross-examination round, but so were real ones, so a refuted finding is worth probing rather than dropping.

## Method

I replayed eight past changes from a private codebase. Four of them shipped a defect that was found later, and four had no later correction. Every review used the `inspect` mode of the runner in `claudex-loop` 2.2.0, with Codex on `gpt-6.1-sol` and Claude on `claude-opus-5-5`, the same brief, and the same frozen snapshot.

- Each finding was settled by a live probe, either a temporary test against the code or a compile and run of the code before and after the change, or by a match to a defect fixed later.
- A finding counts as serious when the probe showed a silent wrong result, data loss, a crash or hang, or a false refusal. Doc, test, message, and performance findings count as real but not serious.
- Findings from different runs were matched by hand to the defect they describe.
- Each setup below is a union of runs, scored by distinct defects.

## Results

Per run, 90% to 95% of findings were real, for both models.

| Setup, eight changes | Known defects (of 4) | Serious defects | Real defects | Findings to probe |
|---|---|---|---|---|
| Codex `high` | 3 | 16 | 18 | 19 |
| Codex `xhigh` | 3 | 20 | 22 | 23 |
| Codex `high`, two runs | 3 | 22 | 24 | 26 |
| Claude `high` | 1 | 17 | 31 | 34 |
| Claude `xhigh` | 2 | 28 | 46 | 49 |
| Claude `high`, two runs | 2 | 21 | 41 | 46 |
| One of each, `high` | 3 | 24 | 38 | 42 |
| One of each, `high`, two runs each | 3 | 32 | 52 | 59 |
| One of each, `xhigh` | 3 | 34 | 52 | 56 |
| All six runs | 3 | 39 | 67 | 77 |

- A second model added more serious defects than a second run of the same model: 7.75 on average, against 6 for a second Codex run and 4 for a second Claude run.
- Codex found the same 3 of the 4 known defects in every run. Claude found 1 at `high`, 2 in a second `high` run, and 2 at `xhigh`. No run found the fourth.

| Median per run (max) | Wall clock | Input tokens | Output tokens |
|---|---|---|---|
| Codex `high` | 195 s (345 s) | 1.72 M | 8.0 K |
| Codex `xhigh` | 287 s (433 s) | 1.59 M | 12.7 K |
| Claude `high` | 267 s (440 s) | 0.57 M | 24.3 K |
| Claude `xhigh` | 610 s (921 s) | 2.17 M | 55.9 K |

In the cross-examination round, each reviewer was sent the findings that only the other reviewer had reported, 31 in all, without attribution:

- 23 were confirmed, and all 23 were real.
- 6 were refuted: 2 were false, 1 rested on an input that does not compile, and 3 were real test or doc gaps.
- 2 were marked unverifiable: 1 could not be tested, and 1 was a real doc gap.
- No new defect came out of the round, which added about 3.5 minutes with both reviewers in parallel.

In a missed-defect round, each reviewer was given the union of both reviews and asked for defects it did not cover. On the four changes with known defects, the round found 3 more serious defects. A plain second run of each model found 3 as well, with fewer findings to probe (37 against 45).

## What this means for `claudex-code-review`

- Of the two levels tested, `xhigh` seems the better default for both reviewers, at about twice the wall clock of `high`. Claude's median `xhigh` run took longer than 600 s, so I'd pass a `--timeout` above the runner's default at that level.
- A finding refuted in cross-examination should be verified like any other, not dropped.
- Only 2 of the 31 contested findings were false, so there was little for a tournament or judging round to remove.
- A missed-defect round found no more serious defects than a plain second run, so I wouldn't add it as a mode.
- Two runs of each model at `high` found 32 serious defects, fewer than one run of each at `xhigh`, so I wouldn't add a second-run mode either.

## Limits

- Eight changes from one codebase, and one `xhigh` run per model. Claude's recall varied between identical `high` runs, so part of its `xhigh` gain may be run-to-run variance.
- Every reported finding was settled, but a defect that no run reported is counted only if it was one of the four known ones.
- The probes were written by Claude agents. Each verdict rests on observed behavior, but the choice of inputs to try was a judgment.
