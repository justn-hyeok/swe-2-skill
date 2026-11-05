# SWE-2 tuning A/B plan, fixed before any test-half result

Arm A: the benchmark instruction as given. Arm B: the same instruction plus tune_rules.md.
Rules were written after reading only dev-half Hard failures and non-Hard sample failures.

Primary measure: tune_test.txt (26 Hard tasks, failures not read), 2 rounds per arm, official fresh re-grade.
Pairs are (task, round). Both arms run the same tasks in the same batch window, in a seeded random arm order.

Adopt only if B passes more pairs than A and a one-sided exact sign test on discordant pairs gives p < 0.05.
Otherwise record "no effect confirmed" and do not put the rules in the skill as tuning.
Secondary, reported but not deciding: median seconds per arm, patch bytes, and dev-half rerun passes.
The dev rerun is a pipeline check only and never counts as evidence.

Environment failures retry once. Graded failures never retry. Stop on 3+ limit hits in a job or an all-empty job.
