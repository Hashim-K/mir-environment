# mir-environment

Reproducible conda environments for the MSc thesis MIR project.

## Environments

| File                     | Use                                | GPU       |
| ------------------------ | ---------------------------------- | --------- |
| `environment.yml`        | Laptop — full stack                | CUDA 12.4 |
| `environment-daic.yml`   | DAIC HPC — headless, A40-optimised | CUDA 12.4 |
| `environment-delftblue.yml` | DelftBlue — headless GPU training | CUDA 12.4 |
| `environment-apptainer.yml` | Common Apptainer runtime base     | CUDA 12.4 |
| `environment-webapp.yml` | Webapp / CI — lightweight          | CPU only  |

---

## Laptop Setup

```bash
# 1. Create and activate
conda env create -f environment.yml
conda activate MIR

# 2. Verify
python -m mir_env.verify_installation
```

---

## DAIC Setup

```bash
# 1. Load miniconda (DAIC system module)
module use /opt/insy/modulefiles
module load miniconda

# 2. Set path to mir-core (adjust to your workspace layout)
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core

# 3. Create env (do this on a login node, not a compute node)
conda env create -f environment-daic.yml
conda activate MIR-daic

# 4. Verify
python -m mir_env.verify_installation
```

Add to your `~/.bashrc` on DAIC:

```bash
module use /opt/insy/modulefiles
module load miniconda
export MIR_DATA_ROOT=/tudelft.net/your-group/datasets
export MIR_OUTPUTS_ROOT=/tudelft.net/your-group/outputs
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core
```

> **Note:** DAIC's `/tudelft.net` mounts are Windows-based and have pip
> compatibility issues. Keep conda envs in `$HOME` (the default) — do not
> relocate them to project storage.

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
conda env create -f environment-webapp.yml
conda activate MIR-webapp
```

---

## DelftBlue Setup

Install Miniconda or Miniforge in `$HOME` first. Then create the environment on
a login node:

```bash
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core
conda env create -f environment-delftblue.yml
conda activate MIR-delftblue
python -m mir_env.verify_installation
```

On DelftBlue worker nodes, do not rely on `conda init`. Source `conda.sh`
directly in your job script before activating the environment:

```bash
module load openssh
module load git
unset CONDA_SHLVL
source "$HOME/miniconda3/etc/profile.d/conda.sh"
conda activate MIR-delftblue
```

---

## Common Apptainer Runtime

`environment-apptainer.yml` is the shared headless GPU runtime used by the
common Apptainer build in `msc-thesis`.

The intended flow is:

```bash
cd /path/to/msc-thesis
./scripts/build-apptainer.sh
./scripts/apptainer-exec.sh python -m mir_env.verify_installation
```

---

## Notes

- `numpy<2.0` is a hard constraint — both madmom and BeatNet require it
- `numba` must come from conda-forge, not pip — the pip wheel lacks proper LLVM bindings on some platforms
- `madmom` has no PyPI release — installed from GitHub master
- `BeatNet` is installed from `Hashim-K/beatnet` fork (stale numba pin removed in fork)
