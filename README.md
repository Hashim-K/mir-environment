# mir-environment

Reproducible conda environments for the MSc thesis MIR project.

## Environments

| File                     | Use                                | GPU       |
| ------------------------ | ---------------------------------- | --------- |
| `environment.yml`        | Laptop — full stack                | CUDA 12.4 |
| `environment-daic.yml`   | DAIC HPC — headless, A40-optimised | CUDA 12.4 |
| `environment-webapp.yml` | Webapp / CI — lightweight          | CPU only  |

---

## Laptop Setup

```bash
# 1. Create and activate
conda env create -f environment.yml
conda activate MIR

# 2. Finish tricky installs (madmom, BeatNet)
mir-setup

# 3. Verify
python -m mir_env.verify_installation
```

---

## DAIC Setup

```bash
# 1. Set path to mir-core (adjust to your workspace layout)
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core

# 2. Create env (do this on a login node, not a compute node)
conda env create -f environment-daic.yml
conda activate MIR-daic

# 3. Finish tricky installs
mir-setup

# 4. Verify
python -m mir_env.verify_installation
```

Add to your `~/.bashrc` on DAIC:

```bash
export MIR_DATA_ROOT=/tudelft.net/your-group/datasets
export MIR_OUTPUTS_ROOT=/tudelft.net/your-group/outputs
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core
```

### Recommended SLURM header for A40 nodes

```bash
#SBATCH --partition=general
#SBATCH --gres=gpu:a40:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=12:00:00
```

---

## Webapp / CI Setup

```bash
conda env create -f environment-cpu.yml
conda activate MIR-cpu
```

---

## Notes

- `numpy<2.0` is a hard constraint — both madmom and BeatNet require it
- `numba` must come from conda-forge, not pip — the pip wheel lacks proper LLVM bindings on some platforms
- `madmom` has no PyPI release — installed from GitHub master
- `BeatNet` is installed from `Hashim-K/beatnet` fork with `--no-deps` to avoid its stale numba pin
