# mir-environment

Reproducible conda environments for the MSc thesis MIR project.

## Environments

| File                     | Use                                | GPU       |
| ------------------------ | ---------------------------------- | --------- |
| `environment.yml`        | Laptop — full stack                | CUDA 12.4 |
| `environment-hpc-bootstrap.yml` | Minimal HPC bootstrap env for DAIC and DelftBlue | CPU only |
| `environment-apptainer.yml` | Common Apptainer runtime base     | CUDA 12.4 |
| `environment-webapp.yml` | Webapp / CI — lightweight          | CPU only  |

---

## Laptop Setup

Prefer the parent workspace initializer:

```bash
cd /path/to/msc-thesis
./scripts/workspace/init.sh
```

Manual environment creation is still possible from this repo:

```bash
# 1. Create and activate
conda env create -f environment.yml
conda activate MIR

# 2. Verify
python -m mir_env.verify_installation
```

---

## DAIC Setup

Prefer the parent workspace initializer:

```bash
cd /path/to/msc-thesis
./scripts/workspace/init.sh
```

It creates the lightweight `MIR-hpc` bootstrap env and leaves the full runtime
to Apptainer.

Manual setup:

```bash
# 1. Load miniconda (DAIC system module)
module use /opt/insy/modulefiles
module load miniconda

# 2. Create the lightweight bootstrap env on a login node
conda env create -f environment-hpc-bootstrap.yml
conda activate MIR-hpc

# 3. Verify bootstrap tooling
dvc version
```

Add to your `~/.bashrc` on DAIC:

```bash
module use /opt/insy/modulefiles
module load miniconda
export MIR_DATA_ROOT=/tudelft.net/your-group/datasets
export MIR_OUTPUTS_ROOT=/tudelft.net/your-group/outputs
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core
export MIR_SHARED_ROOT=/tudelft.net/your-group/project-root
export MIR_RUNS_ROOT=/tudelft.net/your-group/project-root/runs
```

> **Note:** DAIC's `/tudelft.net` mounts are Windows-based and have pip
> compatibility issues. Keep conda envs in `$HOME` (the default) — do not
> relocate them to project storage.

The full training/runtime stack should run through the shared Apptainer image,
not through a large native Conda environment on the login node.

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

Prefer the parent workspace initializer after installing Miniconda or Miniforge
in `$HOME`:

```bash
cd /path/to/msc-thesis
./scripts/workspace/init.sh
```

Manual environment creation on a login node:

```bash
conda env create -f environment-hpc-bootstrap.yml
conda activate MIR-hpc
dvc version
```

Recommended shared-storage split:

- keep repos and Conda/Miniforge in `$HOME`
- keep large run outputs, DVC cache, and Apptainer images under your project
  storage root such as `/tudelft.net/staff-umbrella/mirworkspace`

On DelftBlue worker nodes, do not rely on `conda init`. Source `conda.sh`
directly in your job script before activating the environment:

```bash
module load openssh
module load git
unset CONDA_SHLVL
source "$HOME/miniconda3/etc/profile.d/conda.sh"
conda activate MIR-hpc
```

---

## Common Apptainer Runtime

`environment-apptainer.yml` is the shared headless GPU runtime used by the
common Apptainer build in `msc-thesis`.

The intended flow is:

```bash
cd /path/to/msc-thesis
./scripts/apptainer/build.sh
./scripts/apptainer/smoke-test.sh
```

---

## Notes

- `numpy<2.0` is a hard constraint — both madmom and BeatNet require it
- `numba` must come from conda-forge, not pip — the pip wheel lacks proper LLVM bindings on some platforms
- `madmom` has no PyPI release — installed from GitHub master
- `BeatNet` is installed from `Hashim-K/beatnet` fork (stale numba pin removed in fork)
- The HPC bootstrap env keeps `dvc` and `dvc-s3` on Conda to avoid pip builds of `pygit2` on cluster login nodes
