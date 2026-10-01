# Project summary

_Last synced: 2026-10-01_

## Goal
A benchmark of ML molecular samplers that goes beyond marginals: joint KL/MMD, metastable
state populations, implied timescales and mode coverage.

## Current target (decided 2026-10-01)
A safe, finished paper for **ICLR 2027 workshops (~Feb 1 2027, soft target)** that can be built on later.
Leading candidate: **"Marginals hide failures"**: controlled failures on 2D analytic potentials plus
released alanine dipeptide checkpoints, with no training of molecular models.
The full benchmark (chignolin / deca-alanine, kinetics across classes) moves to **MLSB / LMRL at NeurIPS 2027 (~Sept 2027)**.

## Methods and baselines
See [baselines/checkpoint-shortlist.md](baselines/checkpoint-shortlist.md).
- Flow BG: transferable-samplers (TarFlow / ECNF++); backup bgflow, FAB
- Diffusion: BoltzNCE; second pick Two-for-One (covers chignolin)
- Generative MSM: Timewarp; backup ITO / BoPITO

## Closest prior work
Dynbench (bioRxiv, Sept 2026) already benchmarks kinetics for protein trajectory generators but not
Boltzmann generators, per-state populations or analytic potentials. See [lit-review/gap-log.md](lit-review/gap-log.md).

## Open design questions
- Ground truth must match each checkpoint's training MD (force field, solvent, temperature, coarse-graining).
- Equilibrium-only samplers have no time axis, so their "kinetics" column uses barrier height and
  transition-region density instead of implied timescales.

## Set aside
Capsid assembly, learned-CV weighted ensemble, PLBAP kinetics extension.

## Tools
W&B for tracking, cloud GPU for checkpoint evaluation.
