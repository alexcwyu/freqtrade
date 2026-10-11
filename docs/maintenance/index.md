# freqtrade: portable verification records

Full raw logs are retained in the workspace maintenance evidence, not distributed here. Hashes identify those original logs. A successful build or selected test subset does not imply a full test-suite pass. Commands containing <workspace> describe the original execution environment.

[Machine-readable records](verification.json) · [Repository validation summary](../validation.md)

## Executed commands

| Check | Exit | Duration | Command | Outcomes |
| --- | ---: | ---: | --- | --- |
| build | 0 | 2.34s | `uv build --python 3.14` | Successfully built dist/freqtrade-2025.1.tar.gz<br>Successfully built dist/freqtrade-2025.1-py3-none-any.whl |
| cleanup-tests | 0 | 18.92s | `.venv/bin/python -m pytest tests/rpc/test_rpc_apiserver.py tests/rpc/test_rpc_emc.py tests/test_log_setup.py tests/test_pip_audit.py` | 129 passed, 1 skipped in 13.87s |
| compile | 0 | 0.71s | `<workspace>/freqtrade/.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| current-sync | 0 | 9.94s | `uv sync --all-extras --python 3.14` | See exit status; no test-count assertion. |
| dependency-audit | 1 | 1.73s | `.venv/bin/python -m pip_audit --progress-spinner=off --ignore-vuln CVE-2026-4539 --skip-editable` | See exit status; no test-count assertion. |
| final-build | 0 | 3.09s | `uv build --python 3.14.8` | Successfully built dist/freqtrade-2026.10.dev0.tar.gz<br>Successfully built dist/freqtrade-2026.10.dev0-py3-none-any.whl |
| final-build-2 | 0 | 1.64s | `uv build --python 3.14.8` | Successfully built dist/freqtrade-2026.10.dev0.tar.gz<br>Successfully built dist/freqtrade-2026.10.dev0-py3-none-any.whl |
| final-clean-tests | 0 | 239.04s | `.venv/bin/python -m pytest` | 4612 passed, 35 skipped, 363 deselected in 231.36s (0:03:51) |
| final-compile | 0 | 0.14s | `.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| final-compile-2 | 0 | 0.13s | `.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| final-current-tests | 1 | 187.17s | `.venv/bin/python -m pytest` | 1 failed, 4610 passed, 35 skipped, 363 deselected, 1 error in 180.87s (0:03:00) |
| final-lock | 0 | 0.52s | `uv lock --upgrade` | See exit status; no test-count assertion. |
| final-lock-2 | 0 | 1.75s | `uv lock --upgrade` | See exit status; no test-count assertion. |
| final-sync | 0 | 0.1s | `uv sync --all-extras --frozen --python 3.14.8` | See exit status; no test-count assertion. |
| final-sync-2 | 0 | 0.07s | `uv sync --all-extras --frozen --python 3.14.8` | See exit status; no test-count assertion. |
| final-tests | 1 | 213.71s | `.venv/bin/python -m pytest tests` | 9 failed, 4598 passed, 35 skipped, 363 deselected, 5 errors in 203.16s (0:03:23) |
| fixture-final-tests | 1 | 252.29s | `.venv/bin/python -m pytest` | 1 failed, 4610 passed, 35 skipped, 363 deselected, 1 error in 242.31s (0:04:02) |
| freqai-diagnostic | 1 | 28.5s | `.venv/bin/python -m pytest tests/freqai/test_freqai_interface.py -x` | 1 failed, 3 passed, 7 skipped in 12.15s |
| freqai-diagnostic-2 | 1 | 37.09s | `.venv/bin/python -m pytest tests/freqai/test_freqai_interface.py -x` | 1 failed, 5 passed, 7 skipped in 19.58s |
| freqai-diagnostic-3 | 1 | 28.47s | `.venv/bin/python -m pytest tests/freqai/test_freqai_interface.py -x` | 1 failed, 5 passed, 7 skipped in 13.94s |
| freqai-diagnostic-4 | 1 | 31.2s | `.venv/bin/python -m pytest tests/freqai/test_freqai_interface.py -x` | 1 failed, 5 passed, 7 skipped in 16.61s |
| freqai-diagnostic-5 | 0 | 53.92s | `.venv/bin/python -m pytest tests/freqai/test_freqai_interface.py -x` | 25 passed, 14 skipped in 42.28s |
| latest-sync | 0 | 1.56s | `uv sync --all-extras --python 3.14` | See exit status; no test-count assertion. |
| lock-3.14 | 0 | 0.61s | `uv lock --upgrade --python 3.14` | See exit status; no test-count assertion. |
| lock-3.15 | 0 | 1.95s | `uv lock --upgrade --python 3.15` | See exit status; no test-count assertion. |
| security-sync | 0 | 0.71s | `uv sync --all-extras --python 3.14.8` | See exit status; no test-count assertion. |
| strategies-build | 0 | 0.88s | `uv build --python 3.15` | Successfully built dist/freqtrade_strategies-0.1.0.tar.gz<br>Successfully built dist/freqtrade_strategies-0.1.0-py3-none-any.whl |
| strategies-compile | 0 | 0.13s | `<workspace>/freqtrade-strategies/.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| strategies-final-build | 0 | 1.26s | `uv build --python 3.15.0` | Successfully built dist/freqtrade_strategies-0.1.0.tar.gz<br>Successfully built dist/freqtrade_strategies-0.1.0-py3-none-any.whl |
| strategies-final-build-2 | 0 | 0.95s | `uv build --python 3.15.0` | Successfully built dist/freqtrade_strategies-0.1.0.tar.gz<br>Successfully built dist/freqtrade_strategies-0.1.0-py3-none-any.whl |
| strategies-final-compile | 0 | 0.08s | `.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| strategies-final-compile-2 | 0 | 0.08s | `.venv/bin/python -m compileall -q src` | See exit status; no test-count assertion. |
| strategies-final-lock | 0 | 0.2s | `uv lock --upgrade` | See exit status; no test-count assertion. |
| strategies-final-lock-2 | 0 | 0.18s | `uv lock --upgrade` | See exit status; no test-count assertion. |
| strategies-final-sync | 0 | 0.03s | `uv sync --all-extras --frozen --python 3.15.0` | See exit status; no test-count assertion. |
| strategies-final-sync-2 | 0 | 0.02s | `uv sync --all-extras --frozen --python 3.15.0` | See exit status; no test-count assertion. |
| strategies-lock-3.15 | 0 | 0.24s | `uv lock --upgrade --python 3.15` | See exit status; no test-count assertion. |
| strategies-sync-3.15 | 0 | 0.57s | `uv sync --all-extras --python 3.15` | See exit status; no test-count assertion. |
| strategies-tests | 5 | 0.56s | `<workspace>/freqtrade-strategies/.venv/bin/python -m pytest tests` | See exit status; no test-count assertion. |
| sync-3.14 | 0 | 7.88s | `uv sync --all-extras --python 3.14` | See exit status; no test-count assertion. |
| sync-3.15 | 2 | 0.04s | `uv sync --all-extras --python 3.15` | See exit status; no test-count assertion. |
| tests | 2 | 41.88s | `<workspace>/freqtrade/.venv/bin/python -m pytest tests` | See exit status; no test-count assertion. |
| tests-2 | 1 | 457.4s | `.venv/bin/python -m pytest tests` | 149 failed, 4453 passed, 35 skipped, 363 deselected, 10 errors in 437.34s (0:07:17) |
| tests-3 | 1 | 358.03s | `.venv/bin/python -m pytest tests` | 17 failed, 4589 passed, 35 skipped, 363 deselected, 6 errors in 348.74s (0:05:48) |
| tests-4 | 1 | 699.84s | `.venv/bin/python -m pytest tests` | 13 failed, 4594 passed, 35 skipped, 363 deselected, 5 errors in 649.46s (0:10:49) |
