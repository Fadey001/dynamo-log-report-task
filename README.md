# dynamo/log-report

A repaired Terminal-Bench 2 (Harbor) task. The underlying exercise is small: parse an
Apache-style access log at `/app/access.log` into a three-key JSON summary at
`/app/report.json`. The work here was fixing the authoring, not the domain.

## Verification

```
harbor run -p log-report -a oracle     # Mean 1.000  (reward 1)
harbor run -p log-report --agent nop   # Mean 0.000  (reward 0)
```

Both gates pass on harbor 0.20.0.

## Defects found and fixed

| # | File | Defect | Fix |
|---|---|---|---|
| 1 | `task.toml` | `artifacts` was a string, not a TOML array | `artifacts = ["/app/report.json"]` |
| 2 | `task.toml` | Declared `/app/out.json` while the solution writes `/app/report.json` | Aligned on `/app/report.json` |
| 3 | `task.toml` | `verification_explanation` read "Check the report file.", describing the broken existence-only check | Rewritten to describe the real value assertions |
| 4 | `environment/Dockerfile` | `FROM python:latest` was an unapproved base, and `:latest` is never allowed | `python:3.13-slim-bookworm` pinned by `@sha256` digest |
| 5 | `environment/Dockerfile` | `COPY solution_hint.py` leaked a full reference implementation into the agent image | `COPY` removed and the file deleted |
| 6 | `tests/test_outputs.py` | Only asserted the file existed and was non-empty, so `{}` scored reward 1 | Asserts the produced values, their types, and the exact key set |
| 7 | `tests/test_outputs.py` | No correspondence to instruction success criteria, since none existed | Five tests, each docstring naming the criterion it covers, 1:1 |
| 8 | `tests/test.sh` | Wrote the reward to `/app/reward.txt` | Writes `/logs/verifier/reward.txt` |
| 9 | `tests/test.sh` | Never produced a CTRF report | Passes `--ctrf /logs/verifier/ctrf.json` |
| 10 | `instruction.md` | Never stated the output path | States `/app/report.json` as an absolute path |
| 11 | `instruction.md` | Never stated the output schema | Names the three exact keys |
| 12 | `instruction.md` | No success criteria; asked the agent to "save your findings" | Five numbered, unambiguous criteria |

## Four-way consistency

`instruction.md`, the `artifacts` entry in `task.toml`, the verifier assertions, and what
`solution/solve.sh` writes all point at `/app/report.json`.

## Verifier honesty

Three adversarial cases beyond the two required by the brief. The original verifier passed
two of them; the repaired one fails all three.

| Case | Original verifier | Repaired verifier |
|---|---|---|
| No file written (nop agent) | 0 | 0 |
| `{}` written to the right path | 1 | 0 |
| Correct shape, `total_requests: 7` | 1 | 0 |

## Expected values

For the `access.log` baked into the image: 6 non-empty lines, 3 distinct client IPs, and
`/index.html` as the most requested path (3 hits against 2 for `/about.html` and 1 for
`/api/login`).

## Known open item

The checklist requires the `category`, `subcategory`, `task_objective`, and `artifact_type`
values to be drawn from `.dynamo/diversity-taxonomy.toml`. That file was not included in the
broken bundle, so the original author's values were kept. They are well-formed snake_case but
could not be confirmed against the real taxonomy.
