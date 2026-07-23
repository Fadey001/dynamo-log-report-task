# dynamo/log-report

A repaired Terminal-Bench 2 (Harbor) task. The exercise itself is small. Parse an Apache
style access log at /app/access.log into a three key JSON summary at /app/report.json. The
work here was fixing the authoring, not the parsing.

## Verification

```
harbor run -p log-report -a oracle     # Mean 1.000, reward 1
harbor run -p log-report --agent nop   # Mean 0.000, reward 0
```

Both gates pass on harbor 0.20.0.

## Defects found and fixed

| # | File | Defect | Fix |
|---|---|---|---|
| 1 | `task.toml` | `artifacts` was a string, not a TOML array | Now an array |
| 2 | `task.toml` | Declared `/app/out.json` while the solution writes `/app/report.json` | Both point at `/app/report.json` |
| 3 | `task.toml` | `verification_explanation` said only "Check the report file.", which described the broken check | Rewritten to describe the real value checks |
| 4 | `environment/Dockerfile` | `FROM python:latest` was not an approved base, and `:latest` is never allowed | Pinned `python:3.13-slim-bookworm` by sha256 digest |
| 5 | `environment/Dockerfile` | `COPY solution_hint.py` leaked a full reference implementation into the agent image | Removed the COPY and deleted the file |
| 6 | `tests/test_outputs.py` | Only checked that the file existed and was not empty, so an empty object scored reward 1 | Checks the produced values, their types, and the exact key set |
| 7 | `tests/test_outputs.py` | Had no link to the instruction criteria, because none existed | Five tests, each docstring naming the criterion it covers |
| 8 | `tests/test.sh` | Wrote the reward to `/app/reward.txt` | Writes `/logs/verifier/reward.txt` |
| 9 | `tests/test.sh` | Never produced a CTRF report | Passes `--ctrf /logs/verifier/ctrf.json` |
| 10 | `instruction.md` | Never gave the output path | Gives `/app/report.json` as an absolute path |
| 11 | `instruction.md` | Never gave the output format | Names the three exact keys |
| 12 | `instruction.md` | Had no success criteria and just asked the agent to save its findings | Five numbered criteria that match the tests one for one |

## Four way consistency

The path in `instruction.md`, the `artifacts` entry in `task.toml`, what the verifier checks,
and what `solution/solve.sh` writes all agree on /app/report.json.

## Verifier honesty

I ran three adversarial cases on top of the two the brief asks for. The original verifier
passed two of them. The repaired one fails all three, which is the correct result.

| Case | Original verifier | Repaired verifier |
|---|---|---|
| No file written, which is the nop agent | 0 | 0 |
| An empty object written to the right path | 1 | 0 |
| Correct shape but `total_requests` set to 7 | 1 | 0 |

## Expected values

The access.log baked into the image has 6 non-empty lines and 3 distinct client IPs. The most
requested path is /index.html with 3 hits, against 2 for /about.html and 1 for /api/login.

## Known open item

The checklist says the `category`, `subcategory`, `task_objective`, and `artifact_type` values
have to come from `.dynamo/diversity-taxonomy.toml`. That file was not in the broken bundle,
so I kept the original author's values. They are well formed snake_case, but I could not check
them against the real taxonomy.
