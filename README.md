# ML Sampler Benchmark

A benchmark paper, not a new method. It asks whether ML samplers that look good on marginals
(Ramachandran plots, torsion W2) actually get the joint distribution, metastable populations,
rare states and kinetics right.

**Methods:** flow Boltzmann generators (transferable-samplers, bgflow), diffusion samplers
(BoltzNCE, Two-for-One), generative MSMs / transfer operators (Timewarp, ITO).
**Systems:** Müller-Brown and other 2D analytic potentials, alanine dipeptide, chignolin / deca-alanine.
**Rule:** use released checkpoints; only the 2D toy models are trained (minutes on CPU).

## Where things are

| Path | What |
|---|---|
| [`docs/project-summary.md`](docs/project-summary.md) | Current state of the project: goal, scope, decisions, venues |
| [`docs/decisions.md`](docs/decisions.md) | Dated decision log |
| [`docs/baselines/`](docs/baselines/) | Checkpoint shortlist and per-model notes |
| [`docs/lit-review/`](docs/lit-review/) | Gap log, benchmark systems, tool ideas |
| [`docs/plan/`](docs/plan/) | Project plan |
| [`src/samplerbench/`](src/samplerbench/) | Evaluation library |
| `configs/`, `scripts/`, `notebooks/` | Experiment configs, run scripts, exploration |

The `docs/` folder is kept in sync with the project's Claude workspace notes.

## Setup

```bash
uv venv && uv pip install -e ".[dev]"
pytest
```
