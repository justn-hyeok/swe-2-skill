# swe-2-skill

An agent skill for running coding work on [SWE-2](https://devin.ai) through headless Devin CLI, plus the benchmark behind its advice. Built for my own use, measured on my own account. Not affiliated with Cognition.

## Results

| Test | `swe-2-medium` passed |
|---|---|
| SWE-bench Pro V2, 100 sampled public tasks | 93/100 (95% interval about 86-97%) |
| SWE-bench Pro V2, Hard-51 | 39/51 (about 63-86%) |
| Same 26 Hard tasks, plain prompt | 41/52 |
| Same 26 Hard tasks, plus 7 working rules | 39/52, about 20% slower |

These are self-run under the benchmark's locked protocol, with official re-grading in a fresh sandbox. They are not leaderboard entries. No other model ran through this setup, the harness differs from published runs, and the public fixes mean recall cannot be ruled out. Full limits are in [bench-evidence.md](skills/swe-2/references/bench-evidence.md).

The practical finding: it does well when requirements are written down, it reports "Done" just as confidently when it failed, and generic working rules did not help.

## Install

Copy `skills/swe-2` into your agent's skills directory, for example `~/.claude/skills/swe-2`. It needs Devin CLI, logged in.

Free self-serve use was documented through 2026-11-30, and pricing changes after that. Run `devin models` to check. The skill also notes what Cognition's docs say about training on your data: read that before sending private code.

## Data

`data/` has one row per run for the SWE-bench runs and the tuning A/B, plus the rules and the plan fixed before results. Rounds on my own repositories are summarized in the evidence file, with no raw data here. Patches and transcripts are not included.

Published scores come from [Scale's leaderboard](https://labs.scale.com/leaderboard/swe_bench_pro_public_v2) and [V2 post](https://labs.scale.com/blog/swe-bench-pro-v2), read on 2026-11-04.
