# Abstract and minimal experiment design (draft v1, 2026-10-01)

Target: ICLR 2027 workshop paper (~Feb 2027). Scope: 2D analytic potentials with exact references, plus
released alanine dipeptide checkpoints. No model training. Framing accepted 2026-10-01: reported sampler
evaluations are low-dimensional and qualitative; this paper makes them quantitative.

## Working title

**Marginals hide failures: quantitative evaluation of molecular samplers beyond the Ramachandran plot**

## Abstract (draft, ~220 words)

Machine-learned samplers such as Boltzmann generators and diffusion models are usually evaluated by
overlaying low-dimensional projections of their samples on a reference, most often the phi/psi
Ramachandran plot of alanine dipeptide, together with energy histograms and effective sample size.
These projections are marginals: the Ramachandran plot summarizes a ~60-dimensional distribution in two
dimensions, and visual agreement says little about joint structure, state populations or rare states.
We show that this practice can hide failures. On two-dimensional analytic potentials, where the
Boltzmann density and state populations are known exactly, we construct samplers whose 1D marginals
match the target to within sampling noise while their joint distribution, metastable-state populations
or minor-mode coverage are wrong, and we measure which commonly reported metrics detect each failure.
We then apply the same protocol, without retraining, to released alanine dipeptide checkpoints from
flow-based Boltzmann generators and diffusion samplers, scoring each against reference simulations
matched to its training conditions. We report per-state population errors with bootstrap confidence
intervals, recall of the rare positive-phi states as a function of sample budget, full-dimensional
divergences in internal coordinates, and the gap between marginal and joint scores, alongside
negative controls that preserve marginals by construction. [RESULT SENTENCE: to be written once the
numbers exist, e.g. "Models that are visually indistinguishable on the Ramachandran plot differ by
X in rare-state population."] We release the evaluation code and propose a minimal quantitative
reporting standard for future sampler papers.

Notes on the abstract:
- The bracketed sentence stays empty until Experiment 3 runs. Do not pre-commit to a direction.
- Kinetics is deliberately left out of the headline. Dynbench (bioRxiv, Sept 2026) already covers kinetics
  for trajectory generators; our differentiators are equilibrium samplers, per-state populations,
  rare-state recall and exact analytic references. Kinetics (Experiment 4) is on hold.

## Contributions (what the paper claims)

1. A demonstration, with exact ground truth, that marginal-based metrics pass samplers that are wrong in
   joint structure, populations or mode coverage.
2. A small quantitative protocol (5 numbers per model, below) with error bars and built-in negative controls.
3. The protocol applied to released alanine dipeptide checkpoints, with no retraining.
4. Open evaluation code (working name `samplerbench`).

## Metrics: the protocol

| # | Metric | Detects | Note |
|---|---|---|---|
| M1 | Marginal scores as papers report them (1D torsion W1/JSD, Ramachandran JSD, energy W2) | baseline "what papers see" | reproduce, don't invent |
| M2 | Joint divergence: MMD and classifier two-sample test (C2ST AUC) on full internal coordinates | wrong correlations | 2D: exact density, so also KL |
| M3 | Marginal-vs-joint gap = M2 relative to its noise floor, minus M1 relative to its | the core "hidden failure" number | noise floor from split-half reference |
| M4 | Per-state population error with bootstrap 95% CI, raw and reweighted, with ESS beside it | wrong weights | fixed shared state definitions |
| M5 | Rare-state recall curve: P(at least k samples in minor state) vs sample budget N, against the i.i.d. expectation | mode collapse / under-weighting | positive-phi states for ala2 |

Negative controls run through every metric:
- **Copula shuffle**: permute each coordinate of reference samples independently. Marginals perfect, joint destroyed. M2/M3 must flag it; M1 won't.
- **Geometry corruption at fixed phi/psi**: perturb bond lengths/angles while keeping torsions. Same Ramachandran, bad energies.
- **Split-half reference**: the noise floor every metric is reported against.

## Experiments

### Exp 1: Controlled failures on 2D analytic potentials (core, no training)
- Potentials: Muller-Brown (3 unequal wells), a symmetric/asymmetric double well, and a three-hole
  potential with two pathways. Truth by grid integration: exact density, populations.
- "Samplers" are constructed, not trained, so each failure is exactly controlled:
  (a) exact samples (positive control); (b) copula shuffle (right marginals, wrong joint);
  (c) correct modes, wrong weights, with marginals matched; (d) minor mode dropped;
  (e) widened/narrowed wells at fixed populations.
- Sweep: barrier height via temperature (kT), and sample budget N.
- Output: a table of which metric flags which failure (headline Figure 1), and detection power vs N.

### Exp 2: Negative controls on alanine dipeptide reference data (no models)
- Apply copula shuffle and geometry corruption to the reference MD. Shows that the Ramachandran plot,
  torsion marginals and (for shuffle) energy histograms miss failures that M2/M4 catch on a real molecule.
- Cheap and fully under our control, so it de-risks the paper if Exp 3 hits environment trouble.

### Exp 3: Released alanine dipeptide checkpoints (no retraining)
From /mnt/project-files/baselines/checkpoint-shortlist.md:
- **Must have**: transferable-samplers (ECNF++ and TarFlow, ala2) and BoltzNCE (AD2). Two flow BGs and one
  diffusion/NCE sampler is enough for a workshop paper.
- **Nice to have**: FAB (internal-coordinate flow, a contrast to Cartesian models); Timewarp in equilibrium (MCMC) mode.
- Two-for-One is coarse-grained, so it needs its own reference. Defer to the MLSB version.
- For each model: draw N samples at several budgets, compute M1-M5 against the reference matched to its
  training MD, and report the Ramachandran plot next to the numbers.
- Also check chirality: Cartesian flows can generate mirror-image (D-) residues, and a flipped chirality
  shifts phi sign, so it could masquerade as positive-phi population. Report the fraction before and after
  filtering. (Inferred from the TBG paper filtering chirality; verify per model.)

### Exp 4: exact kinetics on 2D (ON HOLD, user decision 2026-10-01)
- Exact implied timescales from the discretized Smoluchowski generator. Show that barrier height and
  transition-region density from equilibrium samples predict the rate error (TST argument). Include only
  if time allows; otherwise this belongs in the MLSB version alongside Timewarp kinetics.

## Key design decision to settle first

Each ala2 checkpoint was trained on different MD (force field, solvent, temperature). Each one must be
scored against its own matching reference, and cross-model comparisons are then only qualitative.
Before running anything, record per checkpoint: training data source, force field, solvent, temperature,
and whether that MD is downloadable. If two checkpoints share a dataset, that pair gives the clean comparison.

## Order of work (no dates; durations depend on available hours)

1. Exp 1 on CPU: grid references, constructed samplers, metric code. This alone carries the main claim.
2. Exp 2: download the reference ala2 MD, run negative controls.
3. Settle the reference-matching table, then set up transferable-samplers and BoltzNCE environments.
   Mirror BoltzNCE weights early (university file server).
4. Exp 3 on GPU, logged to W&B.
5. Write the result sentence, figures, and the reporting checklist.

## Go / no-go checks
- After step 1: if marginal metrics flag the constructed failures as readily as joint ones, the thesis is
  weak on 2D. Reframe toward population/rare-state error rather than the joint gap.
- After step 3: if no checkpoint runs cleanly, Exp 1 + Exp 2 still make a complete workshop paper
  ("evaluation pitfalls with exact references"), with Exp 3 moved to MLSB.

## Open questions for the user
- Whether FAB is worth the extra environment.
