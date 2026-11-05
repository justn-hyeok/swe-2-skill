---
name: swe-2
description: Run coding work on SWE-2 directly with headless Devin CLI, and write the failure-scenario spec it needs to do well. Use when the user names SWE-2 or Devin as the worker, wants cheap bulk implementation, or asks which SWE-2 tier or prompt to use. Not for vague requests or as a chat head.
---

# SWE-2

SWE-2 is Cognition's model family in Devin CLI. Default to `swe-2-medium`, the
only tier benchmarked at length here. Self-serve use was documented as free through
2026-11-30, after an earlier extension, with list pricing of $3 input and $15
output per million tokens afterwards. Run `devin models` to see what it costs today.
Evidence and limits are in [bench-evidence.md](references/bench-evidence.md).

It is strong when requirements are written down and weak when they are not:

| Test | Passed |
|---|---|
| SWE-bench Pro V2, 100 sampled public tasks, detailed PR description | 93/100 |
| SWE-bench Pro V2, Hard-51, detailed PR description | 39/51 |
| The author's own hard tasks, mid prompt | 0/8 |
| Same tasks, spec with owner's decisions and failure scenarios | 21/24 runs |

These tests differ and are not directly comparable. The SWE-bench fixes are public, so
recall cannot be ruled out. The 21/24 specs' author had read the fix, so it is an upper bound.

It reports "Done" with the same confidence when it failed: all 10 failures read from the
SWE-bench runs ended that way. Never accept a run on its report.

## Good and bad fits

| Give it | Do not give it |
|---|---|
| Work you can specify, including hard multi-file bugfixes and features | Requests you cannot yet specify: it builds its own reading and verifies that |
| Implementation after a stronger model or the owner settled the design | Design decisions, or the only check on its own work |
| Reviews where you name what to check | Vague "take a look" reviews (a mid prompt passed 3/7 and 1/3) |
| Bulk work where you verify every result yourself | Anything accepted on its own report |
| | A chat head or persona role (it introduces itself as Devin) |

## Before sending private code

Cognition's security docs say data may train models by default and describe opt-out for paid
plans only. They do not address free use or the CLI. Check your Data Controls settings first.

## Writing the prompt

SWE-2 fixes what it understood and proves it with its own tests, so it does not notice a
requirement it missed. Copy this and fill in every section. Delete a section only when it
truly has nothing to say.

```markdown
Work only in <worktree>. Do not commit, push, open a PR, or deploy.

## Goal
<One sentence: the result someone can observe.>

## Decisions already made
<What the code cannot show: "money is never turned into a percentage",
"an unbound task cannot read results", exact error classes and wire formats.>

## Where it applies
<Files or subsystems, plus every caller or entry point that must reach the new behaviour.>

## Required behaviour
1. <One behaviour per line, with exact names, values, formats, and order.>

## Failure scenarios
- <Crash, kill, panic, retry, concurrent call, boundary value, stored or migrated state>: <expected result>

## Must not change
- <Invariants, and any fix you refuse, such as "no fixed timeouts".>
- <Project rules this work touches: lints, forbidden features, dependency limits.>
- If the work cannot be done without breaking one of these, stop and report instead of changing the rule.

## Done means
- Build what you changed, run <commands>, and report the actual output.
- Do not edit tests or checks. If one looks wrong, report it.
- Remove anything your run produced that is not part of the change, such as databases, dumps, and logs.
```

Why these sections earn their place:
- **Decisions** matter most. They separated 0/8 from 21/24.
- **Where it applies** targets a seen failure: a new package was added, while the callers
  the reference fix changed were left untouched.
- **Exact values** target another: a stated "PNG stays PNG" was missed, and tests check exact output.
- **Must not change:** it delivers what you list. In one real fix, a prompt without those
  lines got a 10-minute timeout where the listed one got parent-exit detection. In another,
  a spec requiring `unsafe` missed the repo's `forbid` lint, and both runs relaxed the lint.
- **Done means:** one failed run ended with a package that did not compile.

Before sending it, check the spec against the code. One spec named "a VM with that name
already exists" as a blocked plan, and the code does not block that. Search the repo for rules
your required approach could collide with, then write the acceptance check and see it fail.

Getting those decisions:
- Ask the owner, or read them from the issue, docs, or existing behaviour. If they are
  unknown, resolve them before delegating. Do not have another model guess them: a
  DeepSeek spec from the code alone passed only the 1 weakly graded task of 8.
- Do not add a self-review step. In 18 runs a forced review pass made real code edits
  twice and raised median time by half.
- Do not add generic working rules such as a forced checklist or throwaway tests for every
  item. On 26 hard tasks, plain passed 41/52 and rules 39/52, and rules took about 20% longer.

## Accepting the result

Check every run with something it did not write: your own test or a stated acceptance
command. When that check fails, one fresh run with the same spec is cheap to try while it is free,
but expect a small gain: on 26 hard tasks, 22 passed in two runs against 79% per run, and 4 failed
both. After that, change the spec. Retrying only helps when you own the check.

Read the changed-file list too. Runtime output can land in the diff: Redis dumps made up most
of the 2 largest of 100 benchmark patches. A passing patch is not yet a mergeable one.

Test on the platform that matters. A fix verified only on Linux failed on macOS.

## Running it

Give each run its own worktree, because this mode can write anywhere. Keep the spec and
every output file outside it, or they land in the diff.

```bash
git -C <repo> worktree add <worktree> -b swe-2/<task> <base>
cd <worktree> && devin -p --model swe-2-medium --permission-mode dangerous \
  --respect-workspace-trust false --prompt-file <out>/spec.md --export <out>/session.md \
  </dev/null > <out>/stdout.txt 2> <out>/stderr.txt
echo $? > <out>/exit.txt
```

Long tasks: run it in the background. Hard benchmark tasks took a median of about 6 minutes
and up to 49. On macOS there is no `timeout`, so use `perl -e 'alarm 3000; exec @ARGV' devin ...`.

Then read the run, in this order:
1. `<out>/exit.txt`, and `<out>/stdout.txt` for `rejected a tool call`. Exit 0 alone proves little.
2. `git status --short`, including untracked files, then `git diff`. To see new files in the
   diff, run `git add -N .` first.
3. Your own acceptance check, run yourself. Not its report.
4. Remove a run you will not keep: `git -C <repo> worktree remove --force <worktree>`, then
   delete its branch.

For two attempts, repeat with a second worktree and output directory. Continue the latest
session in the same worktree with `devin -c -p "<next instruction>"` and the same flags.

Run SWE-2 in parallel. It is free right now, so use it freely and do not ration runs: split work
into independent tasks and start one run per task, each in its own worktree, instead of queuing them
or packing them into one big run. Keep tasks apart when they touch the same files, because their diffs
will conflict when you merge them. Benchmarks ran 4 at once with no limit errors. Check every result as usual.

## Traps

- **`--sandbox` with `-p`:** it replaces `--permission-mode dangerous` with its own
  mode, so writes needing confirmation get rejected and the run stalls partway.
  Read the output for `rejected a tool call`, not just the exit code.
- **Model choice:** pass `--model` on every run. Never edit
  `~/.config/devin/config.json` per run; every Devin session reads that file, and
  swapping it once changed the model of other running workers.
- **Which model ran:** the file from `--export` is JSON with `agent.model_name`, such as
  `SWE-2 Medium`. Check it when it matters.
- **Devin logs:** `~/.local/share/devin/cli/logs` carries session tokens. Never share or
  commit it. `--export` output is the file to keep.
- **Paid tiers:** `*-lightning` and `swe-1-7` cost money. Check `devin models`.
- **Usage:** no token log appears. About 270 benchmark runs over two days hit no limit; no
  fair-use limit is documented here, so still watch for limit errors.
- **Hook files:** project hooks in `.devin/config.json` nest under `"hooks"`, while
  `.devin/hooks.v1.json` has no wrapper key. Blocking from `Stop` works in Devin
  3000.11.234 but is undocumented.
- **Reported elsewhere, not measured here** (one author's
  [evaluation](https://gist.github.com/wtfsayo/6d20600fed8ebb95476af8f165cafc28/)):
  it edits a check that blocks it unless told not to; large read-heavy tasks stall, so
  keep subtasks to about 10 files or have it write partial findings to a file; treat a
  `max_tokens` turn with one output token as failed and retry narrower; state hard
  limits as an allowed tool list.
