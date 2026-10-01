# Benchmark systems with known ground truth, and tool ideas

Draft, 2026-10-01. Items marked (memory) are from prior knowledge and were not re-checked online in this pass.

## 1. Test systems, ordered by how exactly we know the answer

### Tier A: analytic potentials (truth computed exactly, no sampling error)
For 2D potentials, the Boltzmann density, state populations, *and* kinetics are exact: discretize the overdamped Langevin (Smoluchowski) generator on a grid, get a rate matrix, and read off eigenvalues (implied timescales), committors and mean first-passage times.

| System | What is known exactly | What it stresses |
|---|---|---|
| Müller-Brown (2D, 3 minima) | populations, free energy, rates, committor | unequal wells, curved transition path |
| 1D/2D double well, Prinz 4-well potential (memory: used by DeepGenMSM and ITO papers) | full spectrum of the transfer operator | implied-timescale recovery, simplest sanity check |
| Three-hole / triple-well 2D (Metzner-Schütte style) (memory) | two competing pathways, entropic channel | pathway coverage, not just end states |
| Correlated double well in d dims (rotate a 1D double well plus Gaussian dims) | exact joint; marginals can be made identical between "right" and "wrong" models | the marginal-vs-joint gap directly |
| Gaussian mixtures with unequal weights (Grenioux et al. 2025) | exact mode weights | population error vs dimension and separation |

Key knob: lower the temperature to raise barriers and make minor states rarer. Plotting every metric against barrier height (in kT) gives a "where does each method class break" curve, likely the paper's headline figure.

### Tier B: small molecules with converged references
| System | Known states and reference | Notes |
|---|---|---|
| Alanine dipeptide | C7eq/C5, alpha_R, and the rare positive-phi states (C7ax/alpha_L, ~1-2% in vacuum-like settings) (memory). Reference: long MD (mdshare 3x250 ns explicit water), umbrella-sampling dF between phi states (4.10 ± 0.26 kT, quoted in BoltzNCE) | The standard. Rare positive-phi state is the natural mode-coverage test. Slowest process is the phi flip (memory: ~ns in explicit water) |
| Ala tri/tetrapeptide; 2AA/4AA datasets (Timewarp, MDGen) | MD references released with those papers | middle rung; transferable checkpoints exist |
| Chignolin (CLN025) | DESRES 106 us trajectory (Lindorff-Larsen 2011) with folded, misfolded, unfolded states; many published MSMs (memory) | stretch goal; reference kinetics exist |
| Deca-alanine | helix-coil free energy along end-to-end distance (Jarzynski / steered-MD literature) (memory) | good for populations; kinetics reference is weaker |

**Gotcha for Tier B:** "ground truth" means truth *for one force field, solvent and temperature*. Every checkpoint must be compared against MD run with exactly its training setup (e.g. vacuum vs implicit vs explicit alanine dipeptide differ a lot). The sibling repo-vetting thread should record this per checkpoint.

**Design flag:** Boltzmann generators and BoltzNCE produce independent equilibrium samples with no time axis, so implied timescales can only be measured on dynamic models (generative MSMs, ITO/BoPITO, Timewarp, MDGen-style). For equilibrium-only samplers, the kinetics column should use kinetics-relevant equilibrium quantities instead: barrier height along the reaction coordinate and probability density in the transition-state region (which sets transition-state-theory rates).

## 2. Tool ideas (an evaluation library, working name `samplerbench`)

1. **Ground-truth oracle.** For each Tier A potential: grid-exact density, state populations, and the discretized generator giving exact implied timescales, committors, MFPTs. For Tier B: a loader for the reference MD plus precomputed MSM.
2. **Fixed, shared state definitions.** One core-set (or committor-based) partition per system, used by every method, so population errors are comparable. Bootstrap error bars.
3. **Joint-vs-marginal gap score.** Compute the same divergence (MMD, sliced Wasserstein, classifier two-sample test) on 1D marginals and on the full joint; report the gap.
4. **Built-in negative controls.** (a) "Copula shuffle": permute each coordinate of the reference samples independently, which keeps marginals perfect and destroys the joint; every joint metric must flag it. (b) For trajectories, shuffle frame order, which keeps the ensemble and destroys kinetics. (c) Split-half reference MD as the noise floor for every metric.
5. **Population error, raw and reweighted.** Per-state population error before and after importance reweighting, with ESS shown next to it so a good-looking reweighted number backed by 3 effective samples is visible.
6. **Rare-state recall curve.** Probability of drawing at least k samples in a state of population p as a function of sample budget N, against the i.i.d. expectation. Reads as "mode recall at budget N".
7. **Kinetics module.** For dynamic models: MSM on generated trajectories with the reference discretization, implied timescales vs lag, Chapman-Kolmogorov test, MFPT and flux. For equilibrium samplers: barrier height and transition-region density as above.
8. **Cost accounting.** Energy evaluations and GPU-seconds per effectively independent sample, including training and reweighting, so speedup claims are normalized across classes.
9. **Temperature / barrier sweep driver.** Runs a method across a ladder of Tier A barrier heights and logs everything to W&B.
10. **Scorecard.** One table/radar per method: marginal, joint, populations, rare-state recall, kinetics, cost.

## 3. Lit search so far (paused at user request)
See gap-log.md.
