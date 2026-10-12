# Fusion Equilibrium Challenge

My entry for the [Fusion Equilibrium Challenge](https://fusion-equilibrium-challenge.sophelio.io/) (NeurIPS 2026 competition track): reconstruct the 2-D poloidal flux map ψ(R,Z) of a tokamak plasma from coil currents and Thomson scattering, without magnetic sensors. The model is trained on DIII-D and applied zero-shot to MAST.

**Goal:** a valid, physics-informed entry in both challenges, plus a clean public repo. Winning is not the goal.

## Objectives

| # | Objective | Done when | Due |
| --- | --- | --- | --- |
| O1 | Get on the leaderboard | Starter baseline scored on Codabench, both machines included | Oct 14 |
| O2 | Beat the baseline | Local composite score above the baseline on held-out DIII-D shots | Oct 16 |
| O3 | Add physics | Coil vacuum-flux (Green's function) features in the model | Oct 18 |
| O4 | Qualify for Challenge 2 | R²ψ > 0.6 on DIII-D and MAST predictions submitted | Oct 18 |
| O5 | Final phase | Best model submitted in the blind phase | Oct 26 |
| O6 | Portfolio | README, figures and a one-page write-up | Oct 31 |

## Key facts

| Phase | Window | Submissions |
| --- | --- | --- |
| Development (public leaderboard) | until Oct 18, 2026 | 5 per day, 100 total |
| Final (blind, leaderboard hidden) | Oct 19–26, 2026 | 3 total |

Score per machine:

```math
S = 0.55\,R^2_\psi + 0.15\,R^2_{q95,\beta_N} + 0.10\,(1 - D_{LCFS}) + 0.20\,\text{Consistency}
```

- Submit per shot only ψ (T × 65 × 65), q95 and βN, on `efit_times`, in test-stream order.
- Challenge 2 score is G_ratio = S_MAST / S_DIII-D, counted only for entries with R²ψ > 0.6 on DIII-D.
- The scorer fixes the global sign of ψ per machine; the ~3× amplitude difference between machines is not fixed.
- Fitting ψ outside the plasma with coil Green's functions alone reaches R² ≈ 0.94, so physics features pay off.

Rules:

- Never use `efit_*` columns or `magnetics_dsep` as inputs.
- Never round or truncate ψ before submitting.
- The read token in the submission is fine-grained, read-only, for the predictions repo only.

## Progress

### M0 Setup (Oct 13, ~2 h)

- [ ] Clone the starter kit; run `uv sync` and `uv sync --group pytorch`
- [ ] `git lfs install && git lfs pull` for the 6 demo shots
- [ ] Make a Hugging Face write token and run `huggingface-cli login`
- [ ] Register on Codabench competition 17456
- [ ] Run `example_usage.py`; plot ψ contours for one DIII-D and one MAST demo shot

Done when: I can load a training row and see its flux map.

### M1 First submission (Oct 14, ~3 h)

- [ ] Run `experiments.py --quick`, then `--n-shots 50 --epochs 50`
- [ ] Check the scorer: `local_score.py --mode perfect` gives 1.0, `--mode zeros` gives 0.0
- [ ] Score the baseline on held-out shots (`--n-shots 20 --skip 7000`) and write the number down
- [ ] Create the private predictions repo (`submission_skeleton.py --max-shots 5 --repo ...`)
- [ ] Make a fine-grained read token for that repo only
- [ ] Full build with `--max-shots 0`; upload `submission_pointer.zip`

Done when: the entry shows on the leaderboard with both machines.

### M2 Data pipeline (Oct 15, ~3 h)

- [ ] Read `MODELING_GUIDE.md`
- [ ] Stream 300–500 training shots; save features to disk, not raw signals
- [ ] Resample coil currents and Ip onto `efit_times` (read `magnetics_time` per shot; Ip has its own time base)
- [ ] Normalize inputs per machine (MAST P-coils are kA·turn, other units differ too)
- [ ] Split train and validation by shot, never by frame

Done when: feature array and ψ array (frames × 65 × 65) are saved.

### M3 PCA model (Oct 16, ~3 h)

- [ ] PCA on ψ; choose the number of modes from reconstruction R² (start near 20)
- [ ] Ridge regression from features to PCA coefficients, then a small MLP
- [ ] Predict q95 and βN from the same features plus simple Thomson summaries (core Te, ne)
- [ ] Local score; submit if it beats the baseline

Done when: the local composite beats the baseline.

### M4 Physics features (Oct 17–18, ~4 h)

- [ ] Green's function of a circular filament for each coil → vacuum ψ on the 65×65 grid
- [ ] Feed vacuum ψ to the model and predict the remaining plasma part
- [ ] Add a plasma-current filament term for Ip
- [ ] Test on the 3 MAST demo shots (the only MAST ground truth); also check R² of −ψ
- [ ] Last development submission before the Oct 18 cutoff

Done when: R²ψ > 0.6 on DIII-D and the MAST demo shots look like a plasma.

### M5 Final phase (Oct 19–26, ~2 h)

- [ ] Pick the model by local score, not by the public leaderboard alone
- [ ] Submit it once; keep 2 submissions for fixes
- [ ] Revoke the read token after Oct 26

### M6 Write-up (by Oct 31, ~3 h)

- [ ] README results section: approach, scores, true vs predicted ψ plots
- [ ] One-page write-up: what worked, what did not, MAST transfer

## Results

| Model | Local S (DIII-D) | R²ψ | Leaderboard S (DIII-D) | G_ratio |
| --- | --- | --- | --- | --- |
| Starter baseline | | | | |

## Links

- [Starter kit](https://github.com/Sophelio/fusion-equilibrium-challenge-starter)
- [Dataset](https://huggingface.co/datasets/Sophelio/fusion-equilibrium-challenge)
- [Codabench competition](https://www.codabench.org/competitions/17456/)

Data: DIII-D and MAST via Sophelio / General Atomics, CC BY 4.0.
