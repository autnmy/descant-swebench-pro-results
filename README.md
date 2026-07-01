# Descant — SWE-Bench Pro results

Reproducible results for **Descant** — an autonomous, multi-model engineering
system — on the public **SWE-Bench Pro** benchmark (731 tasks).

## Scope of this repository

This repository is a public result artifact for the first target release of **Descant**,
Soprano 1, on SWE-bench Pro.

It is intended to make the benchmark output inspectable and reproducible against the
official grading harness. It does not include the Descant runtime, orchestration system,
prompts, model-routing logic, planning/review loops, validation strategy, or other
implementation details.

The goal is to disclose the benchmark artifact, not the architecture.

## Result

**633 / 731 resolved = 86.59%**, verified with the official
[`SWE-bench_Pro-os`](https://github.com/scaleapi/SWE-bench_Pro-os) harness on
native amd64.

| System | Resolved |
|---|---:|
| **Descant** (multi-model) | **633 / 731 — 86.59%** |
| Mythos&nbsp;5 / Fable&nbsp;5 | 80.3% |
| Opus&nbsp;4.8 | 69.2% |
| GPT-5.5&nbsp; | 58.6% |
| Gemini 3.1 Pro | 54.2% |

*Comparison rows are published third-party SWE-Bench Pro results, shown for context.*

## What's in this repo

| Path | Contents |
|---|---|
| `predictions.json` | The final diff for **all 731** tasks, in the SWE-Bench Pro input format (`[{instance_id, patch, prefix}]`). Run this through the official harness to reproduce 633/731. |
| `winning-diffs/` | The **633** passing diffs, one file per `instance_id`, for direct inspection. |
| `resolved.txt` | The 633 resolved `instance_id`s. |
| `grades/resolved.json` | `instance_id → true/false` for all 731 (our official-harness grade). |

The 731 tasks span **11 real GitHub repositories** across Go / Python / JS / TS:

| Repo | Lang | Tasks | | Repo | Lang | Tasks |
|---|---|--:|---|---|---|--:|
| ansible/ansible | py | 96 | | future-architect/vuls | go | 62 |
| internetarchive/openlibrary | py | 91 | | navidrome/navidrome | go | 57 |
| flipt-io/flipt | go | 85 | | element-hq/element-web | js | 56 |
| qutebrowser/qutebrowser | py | 79 | | NodeBB/NodeBB | js | 44 |
| gravitational/teleport | go | 76 | | tutao/tutanota | ts | 20 |
| protonmail/webclients | js | 65 | | | | |

Each task is a distinct `(repo, base_commit, problem, hidden tests)` instance graded on a frozen amd64 Docker image.

## Reproduce

Using the official harness ([`scaleapi/SWE-bench_Pro-os`](https://github.com/scaleapi/SWE-bench_Pro-os))
and the public dataset ([`ScaleAI/SWE-bench_Pro`](https://huggingface.co/datasets/ScaleAI/SWE-bench_Pro),
split `test`, 731 tasks). Follow the harness README to install deps and produce the raw-sample file,
then evaluate `predictions.json` directly (its `[{instance_id, patch, prefix}]` shape is the format the
harness expects):

```bash
python swe_bench_pro_eval.py \
    --raw_sample_path=swe_bench_pro_full.csv \
    --patch_path=predictions.json \
    --output_dir=results \
    --scripts_dir=run_scripts \
    --num_workers=100 \
    --dockerhub_username=jefzda
# => 633 / 731 resolved
```

Grade on **native amd64** — the per-task Docker images (`dockerhub_tag`, hosted under the public
`jefzda` namespace) are amd64. For local Docker add `--use_local_docker` (and `--docker_platform
linux/amd64` on a non-amd64 host, though emulation can produce false failures — see the note below).

## How it was run

Descant Soprano 1 produced one final diff for each of the 731 SWE-bench Pro tasks. The final submitted diffs in this repository are the artifacts being evaluated.

### Clarification on timeouts and amd64 grading

During local validation, two operational issues caused an initial undercount:

1. Some project test suites were unreliable under arm64/Rosetta emulation even when the same diff passed on native amd64.

2. Some Descant runs were interrupted by an external wall-clock cap before the system completed its normal review-and-revise phase.

In both cases, the relevant correction was operational: grade on the benchmark’s native amd64 target, and allow the already-started single Descant pass to finish its normal pipeline.

No task was manually edited after generation. No task was selectively re-attempted after seeing a failed grade. The final score is computed from the submitted diffs included in this repository.

## A note on secret-scanning

The diffs touch the upstream projects' own **test files**, so a few of them contain test
fixtures that pattern-match as secrets — e.g. dummy passwords like `password = '123456'` and the
literal string `-----BEGIN RSA PRIVATE KEY-----` inside a teleport test assertion (a header string,
no key material). These are public test fixtures in those upstream repos, not real credentials. A
[`.github/secret_scanning.yml`](.github/secret_scanning.yml) excludes `winning-diffs/` and
`predictions.json` from secret-scanning alerts; if push protection still flags one on first push,
it is safe to allow.

## About

"Descant (multi-model)" is an autonomous software engineering system. Software engineering discipline
and processes are applied to continuously improve the codebase beyond just the task at hand. The 
core pipeline picks up an issue, plans, implements, runs an independent review pass, and resolves — orchestrated
deterministically. This repo is the result artifact of the pipeline only; the full system capabilites are
not demonstrated here.  

## License & provenance

- This repository's **original content** (README, result manifests, grades) is MIT-licensed
  (see [LICENSE](LICENSE)). The patches under `winning-diffs/` and in `predictions.json` are
  **derivative works** of their upstream open-source projects and remain under those projects'
  licenses — see [NOTICE](NOTICE).
- Benchmark + dataset: **SWE-Bench Pro** — harness
  [`scaleapi/SWE-bench_Pro-os`](https://github.com/scaleapi/SWE-bench_Pro-os), data
  [`ScaleAI/SWE-bench_Pro`](https://huggingface.co/datasets/ScaleAI/SWE-bench_Pro) (public,
  731 tasks). The dataset is **not** redistributed here — pull it from the source; see its
  license for terms.
