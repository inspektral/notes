# Python Environments and Packaging for ML Engineers

This guide covers the tools and standards that make up the Python packaging ecosystem as of 2026. It is written for junior ML engineers who need to stop breaking their environments and start understanding why things break. The topics covered are virtual environments, pip, wheels, conda, mamba, uv, pyproject.toml, pylock.toml, and the particular headache of CUDA and PyTorch installation. Each section explains what the tool does, when it matters, and where it falls short.

The most important thing to know upfront: virtual environments are non-negotiable, a system-wide CUDA installation is almost never necessary for ML work, and the ecosystem has never been in better shape — or more confusing.

---

## 1. Virtual Environments

A virtual environment is a self-contained directory holding a Python interpreter and its own installed packages, isolated from the system Python and from other projects. Without one, installing `numpy==1.24` for project A can silently break project B, which needs `numpy==1.26`. On Linux, running `sudo pip install` can break OS-level tools that depend on system Python. Virtual environments eliminate this class of problem entirely.

### venv and virtualenv

Python ships `venv` in the standard library since 3.3. It requires no installation and works everywhere. The third-party `virtualenv` is faster due to caching and supports older Python versions. For 2025–2026, `venv` is sufficient for most work. Reach for `virtualenv` only if environment creation speed matters (CI pipelines) or Python 2 compatibility is needed, which it shouldn't be.

### Commands

```bash
# Create
python3 -m venv .venv

# Create with upgraded pip (Python 3.9+)
python3 -m venv .venv --upgrade-deps

# Activate
source .venv/bin/activate          # Linux/macOS
.venv\Scripts\Activate.ps1         # Windows PowerShell

# Deactivate
deactivate

# Delete — environments are disposable, recreate from requirements
rm -rf .venv
```

When activated, the shell prompt shows `(.venv)` and `which python` points to the environment's interpreter. Activation prepends `.venv/bin` to `$PATH`. Inside the environment, `lib/python3.x/site-packages/` holds installed packages, and `pyvenv.cfg` records the base Python path.

### Common mistakes

Never commit `.venv/` to Git. It contains absolute paths and can be hundreds of megabytes. Add it to `.gitignore` and commit `requirements.txt` or `pyproject.toml` instead. Never move or copy a virtual environment; hardcoded paths will break. Always run `python -m pip install` rather than bare `pip` to guarantee the correct interpreter is used. Set `PIP_REQUIRE_VIRTUALENV=true` in the shell configuration to prevent accidental global installs:

```bash
export PIP_REQUIRE_VIRTUALENV=true  # Add to ~/.bashrc or ~/.zshrc
```

This is the single most effective guard against the "I just broke my system Python" class of incident.

---

## 2. pip

pip is the default Python package installer. It downloads packages from PyPI (500,000+ packages), resolves dependencies, builds from source when necessary, and installs into the environment's `site-packages`.

### Dependency resolution

Since version 20.3, pip uses a backtracking resolver based on the `resolvelib` library. It models dependency resolution as a constraint satisfaction problem: identify requirements, query PyPI for candidate versions, fetch metadata, check mutual compatibility, backtrack if conflicts are found. This process is NP-hard in the worst case, which explains why complex ML dependency trees can take minutes to resolve. pip 25.1 (April 2025) upgraded to resolvelib 1.1.0 and added a `ResolutionTooDeep` error for graphs that are too tangled.

### Installation sources

```bash
pip install numpy                              # From PyPI
pip install numpy==1.26.4                      # Exact version
pip install "numpy>=1.24,<2.0"                 # Version range
pip install git+https://github.com/user/repo@v1.0.0  # Git tag
pip install git+https://github.com/user/repo@abc123  # Git commit
pip install ./my-package/                      # Local directory
pip install -r requirements.txt                # From requirements file
pip install -e ".[dev]"                        # Editable install with extras
```

### requirements.txt and pip freeze

`pip freeze` dumps every package in the environment — direct and transitive dependencies alike — with no way to tell them apart. Removing a direct dependency leaves its transitive dependencies as orphans in the freeze output. This is a well-known problem. `pip-tools` solves it cleanly: direct dependencies go in `requirements.in`, then `pip-compile` resolves the full tree into a pinned `requirements.txt` annotated with `# via` comments showing the dependency chain:

```bash
pip install pip-tools
pip-compile requirements.in            # Resolve and pin
pip-sync requirements.txt              # Sync environment exactly, removing extras
pip-compile --upgrade requirements.in  # Upgrade all
```

The compiled output makes it clear what was chosen deliberately and what was pulled in automatically. `pip-sync` ensures the environment matches the lockfile exactly.

### Editable installs

An editable install (`pip install -e .`) links a package's source into the Python path so changes are reflected immediately without reinstalling. Modern editable installs use PEP 660: the build backend creates a `.pth` file in `site-packages` pointing to the source directory. Changes to Python source take effect instantly; changes to `pyproject.toml` (new dependencies, entry points) require re-running the install command.

### Private registries and dependency confusion

`--index-url` replaces PyPI with a private registry. `--extra-index-url` searches both. The latter is vulnerable to dependency confusion attacks: pip searches all indexes and picks the highest version number, so an attacker can publish a malicious package with a higher version on public PyPI. Mitigate with `--index-url` exclusively, exact version pins, or `--no-index --find-links` for maximum control.

---

## 3. Wheels

A wheel (`.whl`) is a pre-built binary distribution format defined by PEP 427. It is a ZIP archive that gets extracted directly into `site-packages` — no compilation, no arbitrary code execution, no compiler required. A source distribution (sdist, `.tar.gz`) contains raw source and must be built before installation, potentially requiring a C compiler, header files, and time. pip always prefers wheels.

### Filename anatomy

The filename `numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.whl` encodes: package name, version, Python implementation and version (`cp312` = CPython 3.12), ABI tag, and platform tag (`manylinux_2_17_x86_64` = Linux with glibc ≥ 2.17 on x86_64). A pure Python wheel like `requests-2.31.0-py3-none-any.whl` works on any Python 3 interpreter, any ABI, any platform.

### manylinux

Linux binary compatibility is complex because distributions ship different system library versions. The manylinux standard (PEP 600) solves this by defining a minimum glibc version. Wheels tagged `manylinux_2_17_x86_64` work on any Linux with glibc ≥ 2.17. The `auditwheel` tool inspects built wheels and bundles required shared libraries. `cibuildwheel` automates cross-platform wheel building in CI.

When no compatible wheel exists on PyPI, pip downloads the sdist, builds it locally, and caches the resulting wheel. To build wheels for a project: `python -m build` produces both sdist and wheel in `dist/`.

---

## 4. conda

conda is fundamentally different from pip. It is a language-agnostic package manager and environment manager that treats Python itself as just another package. Where pip handles only Python packages, conda manages the full stack: Python, C/C++ libraries, CUDA toolkits, MKL, compilers, R packages, and more. This makes it indispensable for ML work involving GPU acceleration.

### How it differs from pip

conda uses a SAT solver for dependency resolution, encoding the entire dependency graph as a boolean satisfiability problem and solving it in one pass. This is mathematically rigorous — conda proves a valid solution exists before installing anything. pip's backtracking resolver processes dependencies more or less serially and can produce subtly broken environments. The tradeoff: conda's solver can be slower, though this has been largely addressed by the libmamba integration (see Section 5).

conda packages are pre-compiled binaries (`.conda` or `.tar.bz2`) served from channels. The two channels that matter are `defaults` (maintained by Anaconda Inc., subject to commercial licensing for organizations with 200+ employees) and `conda-forge` (community-driven, free, larger selection, more current). For 2025–2026, use conda-forge as the primary channel.

### Commands and environment.yml

```bash
# Create
conda create -n myenv python=3.12 numpy pandas -c conda-forge

# Activate / deactivate
conda activate myenv
conda deactivate

# Install
conda install -c conda-forge scikit-learn matplotlib

# Export (cross-platform, direct dependencies only)
conda env export --from-history > environment.yml
```

A well-structured `environment.yml`:

```yaml
name: ml-project
channels:
  - conda-forge
dependencies:
  - python=3.12
  - numpy=1.26
  - pandas=2.2
  - scikit-learn=1.5
  - pytorch=2.3
  - pytorch-cuda=12.4
  - pip>=23.0
  - pip:
    - some-pypi-only-package==1.2.3
```

### Critical rules for mixing conda and pip

Pin major.minor versions, not patch. Use one primary channel with `conda config --set channel_priority strict`. Install all conda packages first, pip packages last. Never use pip to upgrade a conda-installed package. After using pip inside a conda environment, avoid running `conda install` again — recreate the environment instead. Violating these rules is the single most common cause of broken conda environments.

### Use Miniforge, not Anaconda

Miniforge is the recommended installer for 2025–2026. It is community-maintained, defaults to conda-forge, includes both conda and mamba, and carries no commercial licensing restrictions. Anaconda's licensing changes (enforced since 2024) have led many universities and companies to ban Anaconda and Miniconda entirely. Miniforge avoids this.

```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

For reproducibility beyond `environment.yml`, use `conda-lock`, which generates fully resolved lock files with exact versions and hashes per target platform.

---

## 5. mamba and micromamba

mamba was created because conda's original Python-based SAT solver became a bottleneck as conda-forge grew. Environments that took 45+ minutes to solve with classic conda solved in 2–5 minutes with mamba. The key innovation: the solver was reimplemented in C++ using libsolv (from openSUSE's package management), with parallel downloads and better error messages.

### The libmamba integration

The critical fact for 2025–2026: conda ≥ 23.10 (November 2023) uses the libmamba solver by default. A recent conda installation already gets mamba-level solving speed without changing any commands. The standalone `mamba` CLI still offers extras — notably `mamba repoquery` for dependency tree inspection — and Mamba 2.0 (mid-2024) added mirror support, OCI registry support, and TUF-based supply chain security.

### micromamba

micromamba is a statically-linked C++ executable (~5 MB) that reimplements conda with zero dependencies — no Python, no base environment. It is designed for Docker containers and CI pipelines:

```dockerfile
FROM mambaorg/micromamba:latest
COPY environment.yml /tmp/environment.yml
RUN micromamba create -f /tmp/environment.yml -y
```

### When to use what

Use conda commands directly for daily work — the libmamba solver is already there. Use the standalone mamba CLI for `repoquery` and Mamba 2.0 features. Use micromamba for Docker images and CI where minimal footprint matters. All three read the same package format and environment files.

---

## 6. uv

uv is the most significant new tool in the Python packaging ecosystem. Built in Rust by Astral (the company behind ruff), it launched in February 2024 as a pip replacement and expanded in August 2024 into a comprehensive project manager. As of March 2026 it is at version 0.10.10 with 81,000+ GitHub stars.

### Speed

The performance numbers are not subtle. With a warm cache, dependency resolution runs up to 337x faster than pip-compile, and package installation runs ~77x faster than pip-sync. Even cold-cache operations are 2–10x faster. This comes from native Rust implementation, parallel downloads, HTTP range requests for metadata (avoiding full wheel downloads), a global cache with Copy-on-Write and hardlinks, and the PubGrub CDCL SAT solver.

### The unified toolchain

uv replaces pip, pip-tools, virtualenv, pyenv, and pipx with a single binary:

```bash
# Virtual environments (80x faster than python -m venv)
uv venv --python 3.12

# pip-compatible interface
uv pip install requests
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt

# Project management
uv init my-project --lib
uv add requests pandas torch
uv add --dev pytest ruff
uv lock
uv sync
uv run python train.py    # No manual activation needed

# Python version management
uv python install 3.12
uv python pin 3.12

# Tool management (pipx replacement)
uvx ruff check .
uv tool install black
```

`uv run` is worth highlighting: it verifies the lockfile is current, syncs the environment, and executes the command, all automatically.

### uv.lock and export

uv generates a `uv.lock` file in human-readable TOML capturing dependencies for all platforms and Python versions. It is uv-specific but can be exported:

```bash
uv export --format requirements.txt    # For pip compatibility
uv export --format pylock.toml         # PEP 751 standard
```

### The limitation for ML engineers

uv cannot manage non-Python dependencies. It cannot install CUDA, cuDNN, system libraries, or compilers — that remains conda's domain. The common pattern for ML work is conda/mamba for the base environment and CUDA, then uv for Python packages. That said, uv added `--torch-backend` support in late 2025, which auto-detects the GPU and installs the correct PyTorch CUDA variant:

```bash
uv pip install torch --torch-backend=auto   # Detects CUDA automatically
uv pip install torch --torch-backend=cu128  # Explicit CUDA 12.8
```

This does not replace a proper CUDA toolkit for all use cases, but for PyTorch specifically it removes a significant pain point.

---

## 7. pyproject.toml

`pyproject.toml` is the standard configuration file for Python projects, replacing the old `setup.py`, `setup.cfg`, and `MANIFEST.in`. Three PEPs define it: PEP 518 introduced the file and the `[build-system]` table, PEP 517 standardized the build backend interface, and PEP 621 standardized project metadata in the `[project]` table.

### Structure

```toml
[build-system]
requires = ["hatchling>=1.27"]
build-backend = "hatchling.build"

[project]
name = "ml-trainer"
version = "1.0.0"
description = "A machine learning training framework"
readme = "README.md"
license = "Apache-2.0"
requires-python = ">=3.10"
dependencies = [
    "numpy>=1.24,<2.0",
    "pandas>=2.0",
    "torch>=2.1",
    "transformers>=4.35",
]

[project.optional-dependencies]
gpu = ["nvidia-cuda-runtime-cu12", "nvidia-cudnn-cu12"]
serving = ["fastapi>=0.100", "uvicorn[standard]>=0.20"]

[project.scripts]
ml-train = "ml_trainer.cli:train"

# PEP 735 dependency groups — dev-only, NOT published to PyPI
[dependency-groups]
test = ["pytest>=7.0", "pytest-cov>=4.0"]
lint = ["ruff>=0.4", "mypy>=1.8"]
dev = [
    {include-group = "test"},
    {include-group = "lint"},
]

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

The `[build-system]` section declares what is needed to build the package. The `[project]` section holds all metadata. The `[tool]` section is for tool-specific configuration (ruff, pytest, mypy, etc.). Dependency groups (PEP 735, accepted 2024) provide a standard way to define development dependencies that are not published to PyPI — unlike optional dependencies (extras), which end users can install.

### Build backends

setuptools holds ~79% market share on PyPI. It is the most mature option with full C/C++ extension support. Hatchling (~6.5%) is fast and extensible, recommended by scientific-python for new projects. Flit-core (~3.8%) is minimalist, suited for simple pure-Python packages. Maturin handles Rust extensions via PyO3. Meson-python powers NumPy and SciPy builds. For most ML projects, setuptools or hatchling are the correct choice.

---

## 8. pylock.toml

PEP 751, authored by Brett Cannon, was accepted on March 31, 2025. It gives Python its first standard lock file format after years of fragmentation. Before pylock.toml, every tool had its own incompatible format: `poetry.lock`, `uv.lock`, `pdm.lock`, `Pipfile.lock`, and pip-tools' compiled `requirements.txt`. This forced tools like Dependabot, cloud providers, and security scanners to support every format individually.

### Contents

pylock.toml is a TOML file where each dependency gets a `[[packages]]` entry with exact version, source URL, mandatory file hashes, file sizes, upload timestamps, and dependency relationships. No resolver is needed at install time — everything is pre-resolved. Multiple platforms are supported via environment markers.

```toml
lock-version = '1.0'
created-by = 'pip'
requires-python = '>=3.10'

[[packages]]
name = 'requests'
version = '2.31.0'
requires-python = '>=3.8'

[[packages.wheels]]
name = 'requests-2.31.0-py3-none-any.whl'
url = 'https://files.pythonhosted.org/packages/.../requests-2.31.0-py3-none-any.whl'
hashes = {sha256 = 'abc123...'}
```

### Adoption as of early 2026

pip ≥ 25.1 has an experimental `pip lock` command. uv supports `uv export --format pylock.toml`. PDM ≥ 2.24 can use pylock.toml instead of pdm.lock. Pipenv 2026.1.0 detects and prioritizes pylock.toml. pip-audit ≥ 2.9 scans it for vulnerabilities. Poetry support is not yet implemented. The standard brings Python in line with JavaScript (`package-lock.json`), Rust (`Cargo.lock`), and Go (`go.sum`).

---

## 9. CUDA and GPU Packages

The CUDA stack has three layers: the NVIDIA GPU driver (host system), the CUDA Toolkit (libraries like cuBLAS, cuDNN, cuFFT), and ML framework bindings (PyTorch, TensorFlow, JAX compiled against specific CUDA versions). The critical insight: modern ML frameworks bundle CUDA runtime libraries inside their pip/conda packages. A compatible NVIDIA driver on the host is needed, but not a system-wide CUDA Toolkit installation.

### PyTorch installation

PyTorch publishes separate wheel repositories for each CUDA variant. The `--index-url` flag selects the repository:

```bash
# CUDA 12.8 (latest as of PyTorch 2.7.0)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128

# CPU only
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

# With uv (auto-detects GPU)
uv pip install torch --torch-backend=auto

# With conda
conda install pytorch torchvision pytorch-cuda=12.4 -c pytorch -c nvidia
```

When installed via pip, PyTorch pulls in `nvidia-cublas-cu12`, `nvidia-cudnn-cu12`, and other CUDA library packages as dependencies. These go into `site-packages` and PyTorch finds them at runtime. On Linux, a bare `pip install torch` from PyPI now defaults to CUDA 12.8 wheels as of PyTorch 2.9.1.

### Driver compatibility

The driver is backward compatible: applications built for older CUDA versions always work on newer drivers. The minimum driver versions for common CUDA toolkit versions:

| CUDA Toolkit | Minimum Linux Driver |
|---|---|
| 12.8 | ≥ 570.26 |
| 12.6 | ≥ 560.28 |
| 12.4 | ≥ 550.54 |
| 12.1 | ≥ 530.30 |
| 11.8 | ≥ 520.61 |

The rule: keep the driver as current as possible. A driver supporting CUDA 12.8 runs applications built for any CUDA 12.x or 11.x version.

A common source of confusion: `nvidia-smi` shows the maximum CUDA version the driver supports. `nvcc --version` shows the compiler version. `torch.version.cuda` shows what PyTorch was compiled against. These can all differ, and that is normal.

### Verification

```python
import torch
print(f"PyTorch: {torch.__version__}")
print(f"CUDA version: {torch.version.cuda}")
print(f"GPU available: {torch.cuda.is_available()}")
print(f"GPU count: {torch.cuda.device_count()}")
print(f"GPU name: {torch.cuda.get_device_name(0)}")
x = torch.rand(3, 3).cuda()  # Functional test
```

If `torch.cuda.is_available()` returns `False` despite a GPU being present, check the driver version with `nvidia-smi`. If `torch.version.cuda` returns `None`, the CPU-only build was installed — reinstall with the correct `--index-url`.

### Other frameworks

JAX uses a plugin architecture: `pip install "jax[cuda12]"` pulls CUDA support via separate plugin packages. TensorFlow uses `pip install tensorflow[and-cuda]` to pull NVIDIA pip packages. Each framework has its own CUDA version requirements — avoid mixing frameworks in the same environment unless the CUDA versions are confirmed compatible.

---

## 10. Choosing Tools and Recommended Workflows

### When to use what

**uv**: pure Python packages, maximum speed, unified workflow (install, lock, run), new projects. This is the default choice for non-CUDA Python work in 2026.

**conda/mamba**: non-Python dependencies (CUDA, cuDNN, MKL, compilers, C libraries), fresh ML workstation setup, teams standardized on conda. Install Miniforge.

**pip**: environments where uv is unavailable, maximum compatibility with existing infrastructure, documentation that assumes pip.

### Workflows by scenario

| Scenario | Approach |
|---|---|
| New ML research project | uv + `--torch-backend=auto` for local dev; Docker for training |
| Team project with GPU needs | Miniforge + conda-forge + conda-lock |
| Production model serving | Docker with pinned NGC base image + uv for Python deps |
| CI/CD pipeline | micromamba or uv in Docker; cache aggressively |
| Quick prototype | `uv run --with torch,transformers python script.py` |
| HPC cluster | Module system for drivers + conda env + pip with `--index-url` |
| Open-source library | pyproject.toml + uv for development + cibuildwheel for releases |

### The hybrid pattern

Many ML teams converge on conda/mamba for the base environment and CUDA toolkit, uv or pip for Python packages. This leverages conda's strength in system-level dependencies while taking advantage of uv's speed for the Python layer:

```bash
mamba create -n ml python=3.12 cuda-toolkit=12.4 -c conda-forge
mamba activate ml
uv pip install torch --torch-backend=auto
uv pip install transformers datasets accelerate
```

### Rules that save hours

1. Always use a virtual environment. Set `PIP_REQUIRE_VIRTUALENV=true`.
2. Pin versions. Use `==` for applications, `>=,<` for libraries. Use lock files.
3. Check the driver first. Run `nvidia-smi` before installing anything GPU-related.
4. Specify `--index-url` explicitly for PyTorch. Defaults change.
5. Do not mix conda and pip carelessly. All conda packages first, pip last. Never conda-install after pip-installing.
6. Use lock files. `uv lock`, `pip-compile`, `conda-lock` — pick one.
7. Treat environments as disposable. Define declaratively, delete and recreate rather than patch.
8. Use containers for production. NGC images eliminate CUDA compatibility issues.