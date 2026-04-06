# Migration Guide: freqtrade

**Branch**: `feature/ayu_develop`
**Standard**: See `docs/PYTHON_MODERN_STANDARD.md` in the trading workspace root.

## Overview

This project has been modernized on the `feature/ayu_develop` branch to use the 2026 Python tooling stack. When syncing from upstream (default branch), the following changes must be re-applied if upstream overwrites them.

## What Changed

### 1. Build System (pyproject.toml)
- **Build backend**: `hatchling` (was: `hatchling` -- already migrated)
- **PEP 621 metadata**: All project metadata in `[project]` table
- **Dependencies**: Managed by `uv`, lockfile in `uv.lock`

### 2. Removed Legacy Files
The following files were removed (upstream may re-add them on sync):
- `requirements.txt`
- `requirements-dev.txt`
- `requirements-freqai.txt`
- `requirements-freqai-rl.txt`
- `requirements-hyperopt.txt`
- `requirements-plot.txt`

If these reappear after a sync, delete them again. All configuration is in `pyproject.toml`.

### 3. Source Layout
- **Layout**: `src/` layout
- **Package moved**: `freqtrade/` -> `src/freqtrade/`
- **Import unchanged**: `import freqtrade` still works

If upstream adds files to the old location, move them to `src/freqtrade/`.

### 4. Tooling Configuration (in pyproject.toml)

#### Ruff (linting + formatting)
```toml
[tool.ruff]
line-length = 100

[tool.ruff.lint]
extend-select = ["C90", "B", "F", "E", "W", "UP", "I", "A", "TID", "C4", "SIM", "PERF", "TC", "YTT", "S", "PTH", "RUF", "ASYNC", "NPY"]

[tool.ruff.lint.per-file-ignores]
"src/freqtrade/freqai/**/*.py" = ["S311"]
"tests/**.py" = ["S101", "S104", "S311", "S105", "S106", "S110"]
"src/freqtrade/templates/**.py" = ["RUF100"]

[tool.ruff.lint.isort]
known-first-party = ["freqtrade", "freqtrade_client"]
```

#### Pyright (type checking)
```toml
[tool.pyright]
include = ["src/freqtrade", "ft_client"]
pythonPath = ["src"]
pythonVersion = "3.13"
typeCheckingMode = "off"
```

#### Pytest
```toml
[tool.pytest.ini_options]
minversion = "9.0"
pythonpath = ["src"]
testpaths = ["tests"]
addopts = ["-ra", "-q", "--strict-markers", "--import-mode=importlib", "--dist", "loadscope"]
xfail_strict = true
filterwarnings = ["error"]
asyncio_mode = "auto"
asyncio_default_fixture_loop_scope = "function"
```

Changes from initial migration:
- Added `minversion`, `testpaths`, `xfail_strict`, `filterwarnings` per standard
- Added `-ra`, `-q`, `--strict-markers`, `--import-mode=importlib` to addopts
- Added `asyncio_default_fixture_loop_scope`

### 5. File Reorganization
- `MIGRATION_GUIDE.md` moved from repo root to `docs/MIGRATION_GUIDE.md`

### 6. Python Version
- `.python-version` set to `3.13`
- `requires-python = ">=3.13"` in pyproject.toml

## After Upstream Sync Checklist

When merging upstream changes into `feature/ayu_develop`:

1. **Delete re-added legacy files**: `requirements.txt`, `requirements-*.txt`
2. **Check pyproject.toml**: Upstream may modify `[project]` metadata (version bumps, new deps). Merge those changes but keep `[build-system]`, `[tool.ruff]`, `[tool.pyright]`, `[tool.pytest]` sections intact.
3. **Check source layout**: If upstream adds new modules to the old `freqtrade/` path, move them to `src/freqtrade/`.
4. **Re-lock**: Run `uv lock` to update `uv.lock` with any new/changed dependencies.
5. **Verify**: Run `uv sync && uv run python -c "import freqtrade" && uv run pytest` (if tests exist).

## Quick Commands

```bash
uv sync                                    # Install all deps
uv run python -c "import freqtrade"        # Verify import
uv run pytest                              # Run tests
uv run ruff check .                        # Lint
uv run ruff format .                       # Format
uv lock                                    # Re-generate lockfile
```
