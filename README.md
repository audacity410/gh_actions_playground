# GitHub Actions limit tests (ADO → GHA migration)

All tests are `workflow_dispatch` only. Nothing runs on push.

Setup: create repo secret `GHA_LIMIT_TEST_SECRET` (any value) for the secrets-inheritance check in t01.

| Test | What it proves | Expected |
|---|---|---|
| t01 | Caller + 9 reusable levels (10 total); outputs bubble up; caller `env` does not propagate; `secrets: inherit` reaches level 9 | Pass |
| t02 | Caller + 10 reusable levels (11 total) | Fail at startup |
| t03 | 50 unique reusable workflows | Pass |
| t04 | 51 unique reusable workflows | Fail at startup |
| t05 | 55 calls to the same reusable workflow | Pass (counts as 1) |
| t06 | 9 nested + 41 leaf = 50 unique across the tree | Pass |
| t07 | 9 nested + 42 leaf = 51 unique across the tree | Fail at startup |
| t08 | 10 nested composite actions, output bubbles up | Pass (verify) |
| t09 | 11 nested composite actions | Fail at runtime (runner-enforced) |
| t10 | Self-recursive composite at depths 8–12 | Highest passing depth = real limit |
| t11 | 256-job static matrix (costs ~256 billed minutes) | Pass |
| t12 | Dynamic `fromJSON` matrix, default 257 entries | Fail at expansion; ≤256 passes |
| t13 / t14 | 25 / 26 `workflow_dispatch` inputs | Pass / invalid file |
| t15 | 100 `workflow_call` inputs | Probe (no confirmed cap) |
| t16 | Job output ~900 KB vs ~1.1 MB; 900 KB through `env:` | Over rejected; env hits E2BIG |
| t17 | `$GITHUB_STEP_SUMMARY` 900 KiB vs 1.1 MiB | Over rejected |
| t18 / t19 | Expression in `uses:` (ADO dynamic template pattern) | Invalid file |
| t20 | YAML anchors for in-file reuse | Pass |
| t21 | Cross-repo reusable workflow using a `./` local composite (`second-repo/`) | Fail: resolves to caller repo |

For t03/t04/t06/t07, leave `execute=false` to test the limits without spending minutes; the limit is checked when the workflow tree loads, not when jobs run.

Before real use, pin `actions/checkout` to a full commit SHA.
