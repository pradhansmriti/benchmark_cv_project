# Baseline shortlist: repos and checkpoints (vetted 2026-10-01)

Criteria: maintained, checkpoints for alanine dipeptide (or similar), license, ease of running.
Source: each repo's README and its Hugging Face / file host listings, read on 2026-10-01.
GitHub's API and commit history were blocked from this environment, so "maintained" is judged from
checkpoint upload dates and paper recency, not from last-commit dates.

## Summary

| Class | Pick | Backup | Why |
|---|---|---|---|
| Flow Boltzmann generator | **transferable-samplers** (TarFlow / ECNF++) | bgflow (retrain), FAB | Only BG with released weights for ala2 *and* chignolin |
| Diffusion / score sampler | **BoltzNCE** | Two-for-One (DFF) | Named in the plan, weights for ala2; Two-for-One covers chignolin |
| Generative MSM / transfer operator | **Timewarp** | ITO, MDGen | Only one with pretrained weights usable on ala2 |

Muller-Brown: no repo ships 2D checkpoints. Models on a 2D potential train in minutes on CPU,
so this is the one place to train (using each method's own code), not a from-scratch implementation.

## 1. Flow-based Boltzmann generators

**transferable-samplers** — recommended
- Repo: https://github.com/transferable-samplers/transferable-samplers (~50 stars, MIT core; some
  third-party components under non-commercial licenses, see NOTICE). Fine for academic work.
- Covers: Sequential Boltzmann Generators (ICML 2025), Prose / amortized transferable flows (NeurIPS 2025),
  Transferable BG baseline (NeurIPS 2024).
- Weights: https://huggingface.co/transferable-samplers/model-weights (updated Nov 28, 2025).
  `single_system/` has ECNF++ and TarFlow for Ace-A-Nme (alanine dipeptide), AAA, Ace-AAA-Nme, AAAAAA,
  plus a TarFlow for GYDPETGTWG (chignolin, 457 MB). `transferable/` holds the Prose models.
- Data: ManyPeptidesMD and SBG data on the same HF org (updated Dec 2025).
- Running: Python 3.11 + uv, optional flash-attention, a `.env` with SCRATCH_DIR; quickstart eval command given.
- Bonus: ECNF++ (exact-likelihood equivariant flow) and TarFlow (fast, large) give two BG flavours from one codebase.

**bgflow** (the plan's named example) — backup / Muller-Brown
- https://github.com/noegroup/bgflow, MIT, ~200 stars, self-described alpha. No pretrained weights;
  ships an alanine dipeptide training notebook. Best used for the Muller-Brown flow. Also a
  dependency of BoltzNCE (with bgmol).

**FAB (fab-torch)** — optional second BG
- https://github.com/lollcat/fab-torch, MIT, ~70 stars. Flow weights for alanine dipeptide on HF,
  MD reference data on Zenodo (10.5281/zenodo.6993124). Authors now point users to their JAX version.
  Internal-coordinate flow, so it's a useful contrast to the Cartesian ECNF/TarFlow models.

## 2. Diffusion / score-based samplers

**BoltzNCE** — recommended
- Repo: https://github.com/RishalAggarwal/BoltzNCE (MIT, 91 commits, 6 stars). Paper: NeurIPS 2025, arXiv 2507.00846.
- Weights: https://bits.csb.pitt.edu/files/BoltzNCE/saved_models_bits/ (uploaded Sep 19, 2025): Graphormer
  energy models plus small flow-matching vector-field models for AD2 (alanine dipeptide) and AA2 (dipeptides).
- Data on OSF (links in README). `./install_things.sh` covers most deps, but bgflow + bgmol are a manual install
  with no pinned versions. That's the likely friction point.
- Risks: small, young repo with one main author. Weights are on a university server, so mirror them early.

**Two-for-One / Denoising Force Field (Microsoft)** — recommended second diffusion model, covers chignolin
- https://github.com/microsoft/two-for-one-diffusion, MIT, ~70 stars (JCTC 2023).
- `saved_models/` in the repo: alanine dipeptide (4 folds), chignolin, Trp-cage, BBA, villin, protein G.
- Coarse-grained. It samples iid **and** the learned score can drive Langevin dynamics, so you also get
  kinetics from a diffusion model. That fits the paper's thesis well.
- Conda env file provided.

## 3. Generative MSMs / learned transfer operators

**Timewarp (Microsoft)** — recommended
- https://github.com/microsoft/timewarp, MIT, ~70 stars, 13 commits (static research release).
- Weights + data: HF dataset `microsoft/timewarp` (1.36 TB total; download selectively).
  `2aa_best_model.pt` (426 MB, transferable across dipeptides, so it runs on alanine dipeptide) and `4aa_best_model.pt` (4.8 GB),
  plus AD / 2AA / 4AA MD folders.
- Conditional flow over a large time step, so it gives a learned propagator (implied timescales) and an
  MCMC-corrected equilibrium sampler. The authors note it only works for 2-4 residue peptides.
- Conda env file; old pinned stack, so expect some env work.

**ITO: Implicit Transfer Operator learning (Olsson group)** — backup, cleanest "generative MSM" framing
- https://github.com/olsson-group/ito, MIT, ~20 stars, NeurIPS 2023. Multiple-lag-time conditional diffusion.
- **No pretrained weights.** Ala2 is the primary reproducible system. Chignolin data needs a DESRES request.
- Follow-up BoPITO (https://github.com/olsson-group/BoPITO, MIT, ICLR 2025) adds a Boltzmann prior and
  ships Prinz-potential + ala2 scripts, also with no weights. Training ala2 is a modest GPU job if Timewarp proves unusable.

**MDGen (Jing et al.)** — optional, tetrapeptides only
- https://github.com/bjing2016/mdgen, MIT, ~220 stars. Weights on HF (`bjing-mit/mdgen`: forward_sim,
  interpolation, upsampling, inpainting, atlas). Trained on tetrapeptides, so it's not a drop-in for ala2.

**DeepGenMSM (Wu, Mardt, Noé 2018)** — skip. https://github.com/markovmodel/deep_gen_msm is old paper code
with no maintained stack.

## Things that will matter in Phase 3

1. **Ground truth must match each checkpoint's training MD.** These models were trained on different
   alanine dipeptide datasets (force field, solvent, temperature, and coarse-graining for Two-for-One).
   Each one needs scoring against its own reference data, or retraining on one shared dataset.
   This is the main design decision for the benchmark.
2. Mirror the BoltzNCE weights (university file server) and pull only the Timewarp subfolders you need.
3. bgflow/bgmol show up as a dependency in several stacks. Pin a single version early.
4. For reference MSMs and implied timescales from MD, use `deeptime` (license not checked here).
