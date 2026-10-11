# freqtrade maintenance validation

Recorded 2026-10-12T01:32:46.361256+08:00. Branch `feature/ayu_develop`; upstream default `upstream/develop` is merged: **true**. Branch/tag synchronization records are retained in the workspace maintenance evidence. Feature branch publication is coordinated separately.

Runtime: **CPython 3.14.8**, uv **0.13.0**, build backend `hatchling.build`. The complete optional dependency environment includes Torch, which has no Python 3.15 wheel. Python 3.14.8 is selected.

Existing suite: **4612 passed, 35 skipped, 363 deselected in 231.36s (0:03:51)** (exit `0`). Full existing tests pass, including the existing security audit. LightGBM uses eval_X/eval_y with encoded labels, XGBoost forests use parallel trees in one boosting round, and test-owned loops/files/sessions are closed. multidict is constrained to the fixed release >=6.9.1 for CVE-2026-104874. No new test cases, skips or warning filters were added.

No test cases were added. Existing assertions and strict warning behavior remain active; repaired fixtures own their loops, sessions, files and configuration paths. Existing upstream platform skips and configured deselections are reported exactly.

## Final build commands

| Check | Exact command | Exit | Seconds |
| --- | --- | ---: | ---: |
| lock | `uv lock --upgrade` | 0 | 1.75 |
| sync | `uv sync --all-extras --frozen --python 3.14.8` | 0 | 0.07 |
| build | `uv build --python 3.14.8` | 0 | 1.64 |
| compile | `.venv/bin/python -m compileall -q src` | 0 | 0.13 |

## Existing test and compatibility evidence

| Check | Exact command | Exit | Result |
| --- | --- | ---: | --- |
| lock-3.15 | `uv lock --upgrade --python 3.15` | 0 | Check completed; see exact exit code. |
| sync-3.15 | `uv sync --all-extras --python 3.15` | 2 | Check completed; see exact exit code. |
| tests | `/Users/alex/workspaces/trading/freqtrade/.venv/bin/python -m pytest tests` | 2 | 363 deselected, 1 error in 14.54s |
| latest-sync | `uv sync --all-extras --python 3.14` | 0 | Check completed; see exact exit code. |
| tests-2 | `.venv/bin/python -m pytest tests` | 1 | 149 failed, 4453 passed, 35 skipped, 363 deselected, 10 errors in 437.34s (0:07:17) |
| tests-3 | `.venv/bin/python -m pytest tests` | 1 | 17 failed, 4589 passed, 35 skipped, 363 deselected, 6 errors in 348.74s (0:05:48) |
| tests-4 | `.venv/bin/python -m pytest tests` | 1 | 13 failed, 4594 passed, 35 skipped, 363 deselected, 5 errors in 649.46s (0:10:49) |
| final-tests | `.venv/bin/python -m pytest tests` | 1 | 9 failed, 4598 passed, 35 skipped, 363 deselected, 5 errors in 203.16s (0:03:23) |
| dependency-audit | `.venv/bin/python -m pip_audit --progress-spinner=off --ignore-vuln CVE-2026-4539 --skip-editable` | 1 | Check completed; see exact exit code. |
| cleanup-tests | `.venv/bin/python -m pytest tests/rpc/test_rpc_apiserver.py tests/rpc/test_rpc_emc.py tests/test_log_setup.py tests/test_pip_audit.py` | 0 | 129 passed, 1 skipped in 13.87s |
| final-current-tests | `.venv/bin/python -m pytest` | 1 | 1 failed, 4610 passed, 35 skipped, 363 deselected, 1 error in 180.87s (0:03:00) |
| fixture-final-tests | `.venv/bin/python -m pytest` | 1 | 1 failed, 4610 passed, 35 skipped, 363 deselected, 1 error in 242.31s (0:04:02) |
| final-clean-tests | `.venv/bin/python -m pytest` | 0 | 4612 passed, 35 skipped, 363 deselected in 231.36s (0:03:51) |

Raw logs remain in the workspace maintenance evidence directory. The [portable verification record](maintenance/verification.json) retains exact command arrays, exits, timing, runtime inventory and log filenames. Earlier failed attempts remain recorded and are not treated as successful final validation.

## Installed dependency versions

This is the selected environment after locked synchronization; isolated wheel build dependencies are resolved by the build backend and recorded in build output.

| Distribution | Version |
| --- | --- |
| absl-py | 2.5.1 |
| aiofiles | 25.1.0 |
| aiohappyeyeballs | 2.7.1 |
| aiohttp | 3.14.5 |
| aiohttp-fast-zlib | 0.3.0 |
| aiosignal | 1.4.0 |
| alembic | 1.20.0 |
| annotated-doc | 0.0.5 |
| annotated-types | 0.8.0 |
| anyio | 4.15.1 |
| appnope | 1.0.0 |
| argon2-cffi | 25.1.0 |
| argon2-cffi-bindings | 26.1.0 |
| arrow | 1.4.0 |
| ast-comments | 1.3.0 |
| ast_serialize | 0.12.1 |
| asttokens | 3.0.2 |
| async-lru | 2.3.0 |
| attrs | 26.1.0 |
| babel | 2.18.0 |
| beautifulsoup4 | 4.15.0 |
| bleach | 6.4.0 |
| boolean.py | 5.0 |
| Bottleneck | 1.6.0 |
| build | 1.6.1 |
| CacheControl | 0.14.4 |
| cachetools | 7.2.1 |
| ccxt | 4.5.85 |
| certifi | 2026.6.17 |
| cffi | 2.0.0 |
| cfgv | 3.5.0 |
| charset-normalizer | 3.4.7 |
| click | 8.5.0 |
| cloudpickle | 3.1.2 |
| cmaes | 0.13.1 |
| colorlog | 6.12.0 |
| comm | 0.2.3 |
| coverage | 7.16.2 |
| cryptography | 50.0.2 |
| cyclonedx-python-lib | 11.12.0 |
| datasieve | 0.1.9 |
| debugpy | 1.8.22 |
| defusedxml | 0.7.1 |
| distlib | 0.4.3 |
| execnet | 2.1.2 |
| executing | 2.3.0 |
| Farama-Notifications | 0.0.6 |
| fastapi | 0.143.0 |
| fastjsonschema | 2.22.2 |
| filelock | 4.1.1 |
| fqdn | 1.6.0 |
| freqtrade | 2026.10.dev0 |
| freqtrade-client | 2026.9 |
| frozenlist | 1.8.0 |
| fsspec | 2026.9.0 |
| ft-pandas-ta | 0.3.16 |
| grpcio | 1.84.0 |
| gymnasium | 1.4.0 |
| h11 | 0.16.0 |
| httpcore | 1.0.9 |
| httpcore2 | 2.13.1 |
| httpx | 0.28.1 |
| httpx2 | 2.13.1 |
| humanize | 4.16.0 |
| identify | 2.6.20 |
| idna | 3.18 |
| iniconfig | 2.3.1 |
| ipykernel | 7.4.0 |
| ipython | 9.17.1 |
| ipython_pygments_lexers | 1.1.1 |
| ipywidgets | 8.1.9 |
| isoduration | 20.11.0 |
| janus | 2.0.0 |
| jedi | 0.20.0 |
| Jinja2 | 3.1.6 |
| joblib | 1.6.0 |
| json5 | 0.17.3 |
| jsonpointer | 3.2.1 |
| jsonschema | 4.26.0 |
| jsonschema-specifications | 2025.9.1 |
| jupyter | 1.1.1 |
| jupyter-console | 6.6.3 |
| jupyter-events | 0.12.1 |
| jupyter-lsp | 2.3.1 |
| jupyter_builder | 1.2.3 |
| jupyter_client | 8.10.0 |
| jupyter_core | 5.9.1 |
| jupyter_server | 2.21.1 |
| jupyter_server_terminals | 0.5.4 |
| jupyterlab | 4.6.4 |
| jupyterlab_pygments | 0.3.0 |
| jupyterlab_server | 2.28.1 |
| jupyterlab_widgets | 3.0.17 |
| lark | 1.3.1 |
| librt | 0.16.0 |
| license-expression | 30.4.4 |
| lightgbm | 4.7.0 |
| Mako | 1.4.3 |
| Markdown | 3.11 |
| markdown-it-py | 4.2.0 |
| MarkupSafe | 3.0.4 |
| matplotlib-inline | 0.2.2 |
| mdurl | 0.1.2 |
| mistune | 3.3.4 |
| mpmath | 1.3.0 |
| msgpack | 1.2.3 |
| multidict | 7.1.0 |
| mypy | 2.4.0 |
| mypy_extensions | 1.1.0 |
| narwhals | 2.27.1 |
| nbclient | 0.11.0 |
| nbconvert | 7.17.1 |
| nbformat | 5.11.1 |
| nbstripout | 0.9.1 |
| nest-asyncio2 | 1.7.4 |
| networkx | 3.7 |
| nodeenv | 1.11.0 |
| notebook | 7.6.3 |
| notebook_shim | 0.2.4 |
| numexpr | 2.14.2 |
| numpy | 2.5.4 |
| numpy-typing-compat | 20260602.2.5 |
| opentelemetry-api | 1.45.1 |
| optuna | 5.0.0 |
| optype | 0.19.0 |
| orjson | 3.11.9 |
| packageurl-python | 0.17.6 |
| packaging | 26.3 |
| pandas | 3.0.6 |
| pandocfilters | 1.5.1 |
| parso | 0.8.7 |
| pathspec | 1.1.1 |
| pexpect | 4.9.0 |
| pillow | 12.3.0 |
| pip | 26.2.1 |
| pip-requirements-parser | 32.0.1 |
| pip_api | 0.0.35 |
| pip_audit | 2.10.1 |
| platformdirs | 4.13.0 |
| plotly | 7.1.0 |
| pluggy | 1.6.0 |
| pre_commit | 4.6.2 |
| prometheus_client | 0.26.0 |
| prompt_toolkit | 3.0.53 |
| propcache | 0.5.2 |
| protobuf | 7.36.2 |
| psutil | 7.2.2 |
| ptyprocess | 0.7.0 |
| pure_eval | 0.2.4 |
| py-serializable | 2.1.0 |
| pyarrow | 26.0.0 |
| pycoingecko | 3.2.0 |
| pycparser | 3.0 |
| pydantic | 2.14.0 |
| pydantic_core | 2.50.0 |
| Pygments | 2.21.0 |
| PyJWT | 2.15.1 |
| pyparsing | 3.3.3 |
| pyproject_hooks | 1.3.3 |
| pytest | 9.1.1 |
| pytest-asyncio | 1.4.0 |
| pytest-cov | 7.1.0 |
| pytest-mock | 3.16.0 |
| pytest-random-order | 1.2.0 |
| pytest-timeout | 2.4.0 |
| pytest-xdist | 3.8.0 |
| python-dateutil | 2.9.0.post0 |
| python-discovery | 1.6.2 |
| python-json-logger | 4.2.0 |
| python-rapidjson | 1.25 |
| python-telegram-bot | 22.8 |
| pytz | 2026.5 |
| PyYAML | 6.0.3 |
| pyzmq | 27.2.0 |
| questionary | 2.1.1 |
| referencing | 0.37.0 |
| requests | 2.34.2 |
| rfc3339-validator | 0.1.4 |
| rfc3986-validator | 0.1.1 |
| rfc3987-syntax | 1.1.0 |
| rich | 15.0.0 |
| rpds-py | 2026.9.1 |
| ruff | 0.17.0 |
| sb3_contrib | 2.9.0 |
| schedule | 1.2.2 |
| scikit-learn | 1.9.1 |
| scipy | 1.18.1 |
| scipy-stubs | 1.18.1.1 |
| sdnotify | 0.3.2 |
| Send2Trash | 2.1.0 |
| setuptools | 84.0.0 |
| six | 1.17.0 |
| sortedcontainers | 2.4.0 |
| soupsieve | 3.0.1 |
| SQLAlchemy | 2.1.4 |
| stable_baselines3 | 2.9.0 |
| stack-data | 0.6.3 |
| starlette | 1.7.0 |
| sympy | 1.14.0 |
| TA-Lib | 0.8.1 |
| tabulate | 0.10.0 |
| technical | 1.7.0 |
| tensorboard | 2.21.0 |
| tensorboard-data-server | 0.7.2 |
| terminado | 0.18.1 |
| threadpoolctl | 3.7.0 |
| time-machine | 3.5.1 |
| tinycss2 | 1.5.1 |
| tomli | 2.5.0 |
| tomli_w | 1.2.0 |
| torch | 2.14.1 |
| tornado | 6.5.10 |
| tqdm | 4.70.1 |
| traitlets | 5.16.1 |
| truststore | 0.10.4 |
| types-cachetools | 7.0.0.20260713 |
| types-filelock | 3.2.7 |
| types-python-dateutil | 2.9.0.20260807 |
| types-requests | 2.33.0.20261006 |
| types-tabulate | 0.10.0.20261006 |
| typing-inspection | 0.4.4 |
| typing_extensions | 4.16.0 |
| tzdata | 2026.5 |
| uri-template | 1.3.0 |
| urllib3 | 2.8.0 |
| uvicorn | 0.54.0 |
| uvloop | 0.22.1 |
| virtualenv | 21.14.6 |
| wcwidth | 0.9.2 |
| webcolors | 25.10.0 |
| webencodings | 0.6.1 |
| websocket-client | 1.9.2 |
| websockets | 17.2 |
| Werkzeug | 3.1.9 |
| widgetsnbextension | 4.0.16 |
| xgboost | 3.4.1 |
| yarl | 1.25.1 |
| zlib-ng | 1.0.0 |

See [architecture](ayu-develop.md) for source-linked component/sequence/workflow diagrams and [development](development.md) for reproducible uv commands.
