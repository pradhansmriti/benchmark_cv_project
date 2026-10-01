# Gap log (partial, search paused 2026-10-01)

## Closest prior work (read these first)
- **Dynbench**: Liu, Qian, Chi, "Ensemble tests mask missing dynamics in protein conformational generators", bioRxiv 2026-09-04 (https://www.biorxiv.org/content/10.64898/2026.09.02.748111v1.full). Evaluates trajectory/ensemble protein generators (MDGen, MarS-FM, ConfRover, BioEmu, AlphaFlow, Str2Str, P2DFlow...) on ATLAS/mdCATH with ensemble, temporal (ITS, VAMP-2, transition matrices) and geometry layers. Shows frame shuffling leaves ensemble scores unchanged but kinetic pass rate drops 0.963 -> 0.004. Uses alanine dipeptide as a positive control only. Does NOT evaluate Boltzmann generators / reweighted samplers, per-state population error, or analytic potentials. **Overlaps our kinetics axis; must be cited and differentiated.**
- Grenioux, Noble, Gabrié, "Improving the evaluation of samplers on multi-modal targets", ICLR 2025 workshop (arXiv 2504.08916). Argues for mode-weight error as the key metric; synthetic Gaussian mixtures only, no molecules, no kinetics.
- Grenioux, Noble, "Diffusion-based Annealed Boltzmann Generators: benefits, pitfalls and hopes", arXiv 2601.21026. Shows "mode blindness" of score matching (wrong relative mode weights); synthetic only.
- Blessing et al., "Beyond ELBOs", ICML 2024 (arXiv 2406.07423). Large variational-sampler benchmark incl. mode-coverage metric; synthetic/Bayesian targets, no molecules (memory for details).
- Fu et al., "Forces are not enough", TMLR 2023 (arXiv 2210.07237). Same argument structure for ML force fields: test-set error doesn't predict simulation quality. Good framing precedent.

## Metrics reported by method papers
| Paper | Systems | Reports | Missing |
|---|---|---|---|
| BoltzNCE (Aggarwal et al., NeurIPS 2025, arXiv 2507.00846) | ALDP | Ramachandran, dF between phi states, E-W2, T-W2, NLL | rare-state coverage, joint divergence, kinetics |
| Sequential BG (Tan et al. 2025, arXiv 2502.18462) | ALDP to hexa-Ala, chignolin | ESS, E-W1, T-W2, Ramachandran | populations, kinetics |
| Transferable BG (Klein, Noé, NeurIPS 2024) | ALDP, 2AA dipeptides | ESS, NLL, Ramachandran, energies, dF along phi, TICA | per-state populations, kinetics |
| VT-DIS (Zhang, Midgley, Hernández-Lobato, TMLR 2025) | DW4, LJ13, ALDP | ESS | nearly everything else |
| Autoregressive BG (arXiv 2606.27361) | peptides to chignolin | E-W2 headline | populations, joint, kinetics |
| MDGen (Jing et al., NeurIPS 2024) | tetrapeptides, proteins | torsion JSD, MSM populations and fluxes, autocorrelation, ITS | — (dynamic model; already kinetics-aware) |
| MarS-FM (Kapuśniak et al., ICLR 2026) | proteins to 500 res | RMSD, Rg, secondary structure (per abstract) | not checked in full |

## Working read of the gap
Equilibrium samplers (BGs, diffusion/flow samplers) are scored on marginals (Ramachandran, torsion W2), energy W2 and ESS. Per-state population error, rare-state recall and joint divergence are rarely reported on molecules; mode-weight work exists but only on synthetic targets. The kinetics axis is now partly covered for protein trajectory generators by Dynbench (Sept 2026).

## Still to check
FAB, iDEM, Timewarp, ITO/BoPITO, BioEmu, DeepGenMSM, "No trick, no treat" (arXiv 2502.06685), energy-based diffusion MD (Plainer et al.), and a forward-citation sweep of Dynbench.
