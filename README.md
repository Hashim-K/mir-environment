# mir-environment

Reproducible conda environments for the MSc thesis MIR project.

## Environments

| File | Use | GPU |
|------|-----|-----|
| `environment.yml` | Laptop — full stack | CUDA 12.4 |
| `environment-daic.yml` | DAIC HPC — headless, A40-optimised | CUDA 12.4 |
| `environment-cpu.yml` | Webapp / CI — lightweight | CPU only |

## Laptop setup

```bash
conda env create -f environment.yml
conda activate MIR
mir-setup
python -m mir_env.verify_installation
```

## DAIC setup

```bash
export MIR_CORE_PATH=/path/to/msc-thesis/repos/mir-core
conda env create -f environment-daic.yml
conda activate MIR-daic
mir-setup
```

## Notes

- `numpy<2.0` hard constraint — madmom + BeatNet both require it
- `numba` must come from conda-forge, not pip
- `madmom` has no PyPI release — installed from GitHub master
- `BeatNet` installed with `--no-deps` to avoid stale numba pin
