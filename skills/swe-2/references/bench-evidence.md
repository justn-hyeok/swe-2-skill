# SWE-2 bench evidence

One person's measurements up to 2026-11-05. Rounds on the author's own repositories have no public raw data.
SWE-bench Pro rows and the tuning A/B are in this repository's `data/` directory.

## Round 1: bugfix tests visible to the model

| Tier and prompt | Passed | Median time |
|---|---|---|
| `swe-2-medium` strict | 20/20 | 115s |
| `swe-2-medium` strict+verify | 11/11 | 61s |
| `swe-2-medium` loose+plan+verify | 9/11 | 122s |
| `swe-2-medium` loose | 7/11 | 139s |
| `swe-2-medium` mid | 6/11 | 137s |
| `swe-2-high` strict | 32/35 | 169s |
| `swe-2-max` strict | 35/35 | 257s |

Medium with a mid prompt, by kind: bugfix 5/5, feature 0/3, review 1/3.
With a strict prompt: feature 3/3, review 3/3.

## Round 2: bugfix tests hidden, mid prompt, one run per task

`swe-2-medium` passed 28/45 with 2 environment failures excluded, median about
110s. By kind (47 runs, including those 2 failures): bugfix 23/37, feature 2/3,
review 3/7.

| Model | Passed |
|---|---|
| DeepSeek v4.1 flash (Command Code) | 41/45 |
| Sonnet 5.5 (Claude Code) | 34/45 |
| Composer 2.5 (Cursor) | 33/45 |
| Grok 4.6 medium (Cursor) | 29/45 |
| Haiku 5.5 (Claude Code) | 28/45 |
| `swe-2-medium` (Devin) | 28/45 |

By language: Go 5/5, Kotlin 5/6, Swift 4/7, Python 4/7, TS 5/9, Rust 3/8, Java 2/5.
These include the 2 environment failures, so they sum to 47.

## Sealed rounds: history cut off at the task's start

Earlier rounds gave agents a full clone, so a later fix commit was readable. In the
pre-seal tuning runs, 3 used `git log --all`-style commands and all 3 passed. Rounds 1
and 2 above cannot be checked and may be inflated, including DeepSeek's 41/45. Sealed
runs below use a task state with no later commits. A transcript scan of the SWE-2 runs
found no use of later refs or the original repositories. DeepSeek runs were not scanned.

Hard set: 8 multi-file tasks, hidden tests. Strict prompts state behaviours, failure
scenarios, invariants, and required interface names, with no implementation. Their
author read the fix, so they carry its decisions and some of what the tests check.

| Arm | Passed | Median time |
|---|---|---|
| `swe-2-medium`, mid | 0/8 | 266s |
| DeepSeek v4.1 flash, mid | 2/8 | 676s |
| DeepSeek spec from code alone, then `swe-2-medium` | 1/8 | 341s plus the spec |
| `swe-2-medium`, strict, 3 rounds | 21/24 | 337s |
| DeepSeek v4.1 flash, strict | 8/8 | 592s |

DeepSeek shared the Mac with other runs, so its times are not a fair comparison. Its
one-round 8/8 and SWE-2's 7/8 per round differ by one task.

Strict per round was 7/8 each time. One Python task failed once and
one Rust task failed twice, so any pass within the first 2 rounds covered
all 8. One Python task has a weak test and is the only task the DeepSeek spec passed.

Failing tests under mid and the DeepSeek spec mostly check decisions the code does not
reveal, such as money never deriving a percentage. These tests encode the original
fix's choices, so 0/8 means the decisions were not passed on, not that the code was bad.

Mixed set, 19 tasks: mid 10/19 and strict 17/19, with medians of 76s and 55s. Five of
six strict prompts audited in this older set name the implementation, so that score is generous.

Unsealed tuning arm, same 19 tasks: mid 11/19; mid plus working rules and a `Stop`
review hook 12/19, with median time 85s to 128s. Two of those 12 passes had read later
history, and the review pass changed code in only 2 of 18 runs.

## SWE-bench Pro V2, public split: 100 tasks

`swe-2-medium` passed 93/100 (95% interval about 86-97%), 2026-11-04. Raw rows are in
`data/sample100.jsonl`. Tasks are a fixed-seed sample of the 642
public tasks, one run each, with the benchmark's prompt and no spec added. The agent had no network
except model hosts and a 50-minute budget, and patches were re-graded in a fresh sandbox with the
official tooling. This is a self-run, not an entry on Scale's leaderboard.

- Pass by repo: NodeBB 6/6, ansible 11/13, element-web 7/8, flipt 11/12, navidrome 8/8, openlibrary
  12/13, qutebrowser 10/11, teleport 10/11, tutanota 1/1, vuls 9/9, webclients 8/8.
- Time per task: median 160s, mean 265s, max 2391s. No limit hits, environment errors, or empty patches.
- The harness checks gave reference patch 5/5 and empty patch 0/5. Sandbox and fresh-sandbox grades
  agreed on all 100.
- Published scores on all 642: Opus 5 99.4, Kimi K3 97.7, GPT-6 Astra 96.9, Sonnet 5 93.15, GPT-5.6
  Terra 92.37, Haiku 5.5 87.23. The interval overlaps most of these, and harnesses differ.
- Hard-51 tasks in this sample: 2/5, too few to rank. Published Hard scores run from 52.9 to 98.
- No web tool call in 100 runs. One run used `git log --all`, which showed a single branch, and it
  passed. Overlap with the reference fix has a median of 0.18, and the highest cases were moved code or
  interface stubs.
- Recall cannot be ruled out. The repositories and fixes are public, and Scale reports Opus 5 at 638/642
  public and 222/272 private. No private-set score exists for SWE-2.
- Passing is not mergeable: the two largest patches (130 and 272 KB) included Redis dump files. Patch
  quality was not reviewed beyond that.

## SWE-bench Pro V2, Hard-51

`swe-2-medium` passed 39/51 (95% interval about 63-86%), 2026-11-05, under the same protocol. 5 tasks came
from the 100-task sample and 46 were new. Raw rows for the 46 are in
`data/hard46.jsonl`.

- Pass by repo includes ansible 4/8, openlibrary 4/8, flipt 6/7, navidrome 2/3, element-web 4/4, and qutebrowser 4/4.
- Time per task: median 342s, mean 468s, max 2961s. No limit hits, environment errors, or empty patches.
- Published Hard scores: Opus 5 98.0, GPT-6 Astra 90.2, Sonnet 5 88.2, GLM-5.3 84.3, Muse Spark Open 82.35,
  Sonnet 5.5 70.59 (mini-swe-agent), Gemini 3.8 Flash 58.8, Haiku 5.5 52.94. Harnesses differ and the interval is wide.
- One task passed in its sandbox and failed the fresh re-grade, on list order (`ansible-c1f2df`).

Failures read: 10, being 6 from a seeded half of Hard-51 and the 4 non-Hard failures in the 100-task sample.
The other 6 Hard failures stay unread as the tuning test set. All 10 reports said "Done".
Examples: a stated requirement was not implemented (`navidrome-69e0`, PNG format), a package did not compile
(`teleport-7744f7`), and a test wanted an order the description never stated (`ansible-c1f2df`).

Tuning A/B, 2026-11-05, plan fixed before results in `data/tune_plan.md`. Plain prompt against plain plus
the 7 rules in `data/tune_rules.md`, on the 26 Hard tasks whose failures were not read, 2 rounds per arm:

| Arm | Passed | Median time |
|---|---|---|
| Plain | 41/52 | 346s |
| With rules | 39/52 | 416s |

Plain alone passed 8 pairs, rules alone passed 6, and 5 failed both (one-sided sign test p = 0.79). No effect was
confirmed, so the rules were not adopted. This does not test them on prompts without a detailed description.

Running twice helps less than it looked. On plain, 22/26 tasks passed at least once against 79% per run, only 3 tasks
changed result between rounds, and 4 failed both. Picking the passing run needs a check you own.

## One real issue, no fix to read

One issue in a private project (cancel handling across Swift and TypeScript), 2026-11-04. An agent wrote the
spec with no existing fix and made the decisions itself, then wrote 30 acceptance checks
before reading any diff. The checks ran on a fake host and reviewer, not real infrastructure.

| Run | Spec | Passed | Failed |
|---|---|---|---|
| 1 | original | 28/30 | review count, IP wait |
| 2 | same, fresh run | 29/30 | review count |
| 3 | one line added for the review count | 28/30 | dry-run refusal, IP wait |

This is one task, graded per check, so it cannot be compared with the per-task bench
scores above. The checks and spec share an author, so wrong decisions would not show.

## Two more real issues, template spec

2026-11-05, base `swe-2-medium`, two fresh runs per issue, with the skill's template. The same author wrote
spec and checks before any diff. Both issues had clear requirements.

| Issue | Checks | Runs passing all |
|---|---|---|
| Public Rust CLI (macOS sysctl parsing) | 12 | 2/2 |
| Private project (one-shot allowance, fake host) | 45 | 2/2 |

Two checks in the second issue were corrected after seeing results, both author errors: a wrong claim that a same-name VM
blocks a plan, and a parser that stopped on an audit log broken by concurrent writes. No check caught the first issue's
lint change, which diff review found. Both reports disclosed it. Two issues cannot establish the template's effect.

## What this does not show

- No other model ran through this SWE-bench setup, so published scores are context, not a head-to-head.
- SWE-2 is not shown to beat other models. Visible tests made round 1 easy for
  every model, and round 2 ranked it level with Haiku at mid.
- High and max are not shown to be worse on harder work. They matched medium on
  easy tasks and were slower; they were not run in round 2.
- Every own-bench strict prompt had an author who read the answer, so 21/24 is an upper bound.
  The real-issue sections are the only specs written with no fix to read.
- Asking a model to list open questions for the owner, rather than guess, is unmeasured.
- Most conditions ran once per task, so a 1 or 2 task gap can be chance.
- Model and harness are paired, so harness effects cannot be separated out.
- Retrying was computed from 3 rounds of 8 tasks, not run as a live retry policy.
