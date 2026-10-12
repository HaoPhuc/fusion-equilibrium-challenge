# Fusion Equilibrium Challenge

Predict the plasma's magnetic flux map ψ(R,Z) from coil currents and Thomson data, with no magnetic sensors. Train on DIII-D, then test zero-shot on MAST.

Goal: a valid, physics-informed entry in both challenges and a clean repo. Not trying to win.

Links: [Starter kit](https://github.com/Sophelio/fusion-equilibrium-challenge-starter), [Dataset](https://huggingface.co/datasets/Sophelio/fusion-equilibrium-challenge), [Codabench](https://www.codabench.org/competitions/17456/), [Rules](https://fusion-equilibrium-challenge.sophelio.io/)

## Roadmap

| | Milestone | Date | Time | Done when |
|---|---|---|---|---|
| M0 | Setup | Oct 13 | 2 h | I can load a shot and plot ψ |
| M1 | First submission | Oct 14 | 3 h | Baseline is on the leaderboard |
| M2 | Data pipeline | Oct 15 | 3 h | Features and ψ saved to disk |
| M3 | PCA model | Oct 16 | 3 h | Beats the baseline locally |
| M4 | Physics features | Oct 17–18 | 4 h | R²ψ > 0.6 on DIII-D |
| M5 | Final phase | Oct 19–26 | 2 h | Best model submitted |
| M6 | Write-up | Oct 31 | 3 h | Results and plots in this README |

The development phase ends Oct 18.

## Tasks

### M0 Setup
- Clone starter kit, `uv sync` and `uv sync --group pytorch`
- `git lfs install && git lfs pull` (demo shots)
- Hugging Face write token, `huggingface-cli login`
- Register on Codabench
- Run `example_usage.py`, plot ψ for one DIII-D and one MAST shot

### M1 First submission
- `experiments.py --quick`, then `--n-shots 50 --epochs 50`
- Check scorer: `--mode perfect` gives 1.0, `--mode zeros` gives 0.0
- Score baseline: `local_score.py --n-shots 20 --skip 7000`
- Create private predictions repo (`submission_skeleton.py --max-shots 5`)
- Fine-grained read token for that repo only
- Full build (`--max-shots 0`), upload `submission_pointer.zip`

### M2 Data pipeline
- Read `MODELING_GUIDE.md`
- Stream 300–500 training shots, save features (not raw signals)
- Resample coils and Ip onto `efit_times`
- Normalize inputs per machine
- Split train/validation by shot, not by frame

### M3 PCA model
- PCA on ψ (start with about 20 modes)
- Ridge regression to PCA coefficients, then small MLP
- q95 and βN from same features plus Thomson summaries
- Score locally, submit if better

### M4 Physics features
- Coil Green's functions to get vacuum ψ on the 65×65 grid
- Model predicts the plasma part on top of vacuum ψ
- Add a plasma-current filament for Ip
- Check on the 3 MAST demo shots (also try −ψ)
- Last dev submission before Oct 18

### M5 Final phase
- Pick model by local score
- Submit once, keep 2 spare
- Revoke read token after Oct 26

### M6 Write-up
- Fill in Results below with plots
- One-page summary: what worked, what didn't, MAST transfer

## Results

| Model | Local S | R²ψ | Leaderboard S | G_ratio |
|---|---|---|---|---|
| Starter baseline | | | | |

Data: DIII-D and MAST via Sophelio / General Atomics, CC BY 4.0.