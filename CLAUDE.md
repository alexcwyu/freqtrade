# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Freqtrade is a free and open source crypto trading bot written in Python. It supports all major exchanges via CCXT, can be controlled via Telegram or web UI, and includes backtesting, plotting, money management tools, and strategy optimization by machine learning (FreqAI).

**Important**: Always create PRs against the `develop` branch, not `stable`.

## Setup and Installation

### Quick Setup

```bash
# Automated setup (detects Python 3.11-3.13, uses uv if available)
./setup.sh

# Manual setup with pip
pip install -e .

# Development setup with all dependencies
pip install -e .[dev]

# Install specific feature sets
pip install -e .[plot]          # Plotting capabilities
pip install -e .[hyperopt]      # Hyperparameter optimization
pip install -e .[freqai]        # Machine learning features
pip install -e .[freqai_rl]     # Reinforcement learning
```

### Requirements

- Python >= 3.11
- TA-Lib (technical analysis library)
- SQLite (for persistence)
- System requirements: 2GB RAM, 1GB disk, 2vCPU minimum

## Development Commands

### Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_<file_name>.py

# Run specific test method
pytest tests/test_<file_name>.py::test_<method_name>

# Run with coverage
pytest --cov=freqtrade

# Parallel test execution
pytest -n auto
```

### Code Quality

```bash
# Run pre-commit checks (recommended - install with: pre-commit install)
pre-commit run -a

# Run ruff (linting and formatting)
ruff check .
ruff format .

# Type checking with mypy
mypy freqtrade

# All checks are configured in pyproject.toml
```

### Running the Bot

```bash
# Trade mode (live/dry-run)
freqtrade trade --config config.json

# Backtesting
freqtrade backtesting --config config.json --strategy SampleStrategy

# Hyperparameter optimization
freqtrade hyperopt --config config.json --hyperopt-loss SharpeHyperOptLoss --strategy SampleStrategy

# Download data
freqtrade download-data --exchange binance --pairs BTC/USDT ETH/USDT --timeframes 5m 1h

# Start web server
freqtrade webserver --config config.json

# List available strategies
freqtrade list-strategies

# Create new strategy
freqtrade new-strategy --strategy MyStrategy
```

## Architecture Overview

### Core Components

**freqtradebot.py** (3000+ lines)
- Main bot class that orchestrates all trading operations
- Manages bot lifecycle: initialization, main loop, shutdown
- Handles trade execution: entry, exit, position management
- Implements risk management and order handling
- State machine for different trading states

**Strategy System** (`freqtrade/strategy/`)
- `interface.py`: Base strategy class with core methods
- Key methods to implement:
  - `populate_indicators()`: Add technical indicators to dataframe
  - `populate_entry_trend()`: Define entry signals
  - `populate_exit_trend()`: Define exit signals
  - `custom_exit()`: Custom exit logic
  - `custom_stake_amount()`: Dynamic position sizing
- Strategies inherit from `IStrategy` base class
- Located in `user_data/strategies/` by default

**Exchange Layer** (`freqtrade/exchange/`)
- Unified interface for multiple exchanges via CCXT
- Exchange-specific implementations for: Binance, Kraken, OKX, Bybit, Gate.io, etc.
- Handles: order placement, market data, balance retrieval, WebSocket connections
- `exchange.py`: Main exchange interface
- `exchange_ws.py`: WebSocket support for real-time data

**Data Management** (`freqtrade/data/`)
- `history/`: Historical data loading and management
- `datahandlers/`: Multiple format support (JSON, Parquet)
- `btanalysis/`: Backtesting analysis tools
- `converter/`: Data format conversion utilities

**RPC System** (`freqtrade/rpc/`)
- Multiple communication interfaces:
  - `telegram.py`: Telegram bot for remote control
  - `api_server/`: REST API and WebUI (FastAPI)
  - `webhook.py`: Webhook notifications
- Real-time trade notifications and bot control

**FreqAI** (`freqtrade/freqai/`)
- Machine learning integration for adaptive strategies
- Supported frameworks: PyTorch, XGBoost, LightGBM, CatBoost
- `prediction_models/`: Pre-built ML models
- `RL/`: Reinforcement learning environments
- `base_models/`: Base classes for custom models
- Uses feature engineering and data preprocessing pipelines

**Optimization** (`freqtrade/optimize/`)
- `backtesting.py`: Historical strategy testing
- `hyperopt/`: Hyperparameter optimization using Optuna
- `optimize_reports.py`: Performance analysis
- Supports walk-forward analysis and edge positioning

**Persistence** (`freqtrade/persistence/`)
- SQLAlchemy-based database layer
- `models.py`: Trade, Order, and PairLock models
- `pairlock_middleware.py`: Pair locking logic
- SQLite by default, supports PostgreSQL

### Key Directories

```
freqtrade/
├── commands/          # CLI command implementations
├── configuration/     # Config loading and validation
├── data/             # Data handling and storage
├── enums/            # Type-safe enumerations
├── exchange/         # Exchange integrations
├── freqai/           # Machine learning module
├── optimize/         # Backtesting and hyperopt
├── persistence/      # Database models
├── plot/             # Plotting utilities
├── plugins/          # Plugin system (pairlists, protections)
├── resolvers/        # Dynamic loading of strategies/exchanges
├── rpc/              # Remote control interfaces
├── strategy/         # Strategy interface
├── templates/        # Strategy templates
└── util/             # Utility functions

tests/                # Mirror structure of freqtrade/
user_data/            # User strategies, configs, data
├── strategies/       # Custom strategy files
├── data/            # Downloaded market data
└── notebooks/       # Jupyter analysis notebooks
```

## Key Concepts

### Trading Flow

1. **Initialization**: Load config, initialize exchange, load strategy
2. **Main Loop**:
   - Fetch market data (candles, tickers)
   - Run strategy indicators and signals
   - Check for entry opportunities
   - Manage open positions (stop loss, ROI, custom exits)
   - Execute orders via exchange
3. **State Management**: Track trades in database, handle pair locks, maintain wallets

### Strategy Development

Strategies define the trading logic. Minimal strategy structure:

```python
class MyStrategy(IStrategy):
    # Strategy parameters
    timeframe = '5m'

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # Add indicators (RSI, MACD, etc.)
        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # Define entry conditions
        dataframe.loc[conditions, 'enter_long'] = 1
        return dataframe

    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # Define exit conditions
        dataframe.loc[conditions, 'exit_long'] = 1
        return dataframe
```

### Configuration System

- JSON-based configuration in `config.json`
- Schema validation via `config_schema/`
- Can use `freqtrade new-config` for interactive setup
- Supports multiple configs and inheritance

### FreqAI Integration

FreqAI enables ML-based predictions in strategies:

1. Define features in strategy's `feature_engineering_*` methods
2. Configure FreqAI in config (model type, training parameters)
3. Bot automatically trains models on historical data
4. Predictions available in strategy via `dataframe['&*']` columns

## Code Style

### Style Guidelines

- **Line length**: 100 characters (enforced by ruff)
- **Import order**: Managed by isort (2 lines after imports)
- **Type hints**: Required for all public methods (checked by mypy)
- **Docstrings**:
  - Required for all public methods
  - Use double quotes
  - Follow reST format: `:param xxx:`, `:return:`, `:raises:`
  - Multiline docstrings indented to first quote level

### Example

```python
def calculate_profit(
    entry_price: float,
    exit_price: float,
    stake_amount: float
) -> float:
    """
    Calculate profit for a trade.

    :param entry_price: Entry price of the trade
    :param exit_price: Exit price of the trade
    :param stake_amount: Amount invested in the trade
    :return: Profit amount in quote currency
    :raises ValueError: If prices are invalid
    """
    if entry_price <= 0:
        raise ValueError("Entry price must be positive")
    return (exit_price - entry_price) / entry_price * stake_amount
```

## Testing Guidelines

- All new features must include tests
- Tests should mirror the structure of `freqtrade/`
- Use pytest fixtures from `conftest.py`
- Mock external dependencies (exchange, network calls)
- Aim for high coverage on trading logic

## Common Patterns

### Accessing Exchange

```python
# In strategy or bot
self.exchange.fetch_ticker(pair)
self.exchange.create_order(pair, order_type, side, amount, price)
```

### Working with Dataframes

```python
# Strategies work with pandas DataFrames
# Each row is a candle with: date, open, high, low, close, volume
dataframe['rsi'] = ta.RSI(dataframe)
dataframe.loc[
    (dataframe['rsi'] < 30) & (dataframe['volume'] > threshold),
    'enter_long'
] = 1
```

### Database Queries

```python
# In freqtradebot or RPC
from freqtrade.persistence import Trade

trades = Trade.get_open_trades()
trade = Trade.get_trades([Trade.id == trade_id]).first()
```

## Troubleshooting

### Common Issues

- **TA-Lib not found**: Install TA-Lib system library first, then Python wrapper
- **Exchange errors**: Check API keys, permissions, and exchange-specific requirements
- **Strategy not loading**: Ensure strategy class name matches filename
- **Test failures**: Run `pre-commit run -a` to catch style issues

### Debug Mode

```bash
# Verbose logging
freqtrade trade --config config.json --logfile freqtrade.log -vvv

# Dry run for testing without real trades
freqtrade trade --config config.json --strategy MyStrategy --dry-run
```

## Resources

- **Documentation**: https://www.freqtrade.io
- **Discord**: https://discord.gg/p7nuUNVfP7
- **GitHub Issues**: https://github.com/freqtrade/freqtrade/issues
- **Developer Docs**: https://www.freqtrade.io/en/latest/developer/

## Project Structure Notes

- Python package managed via `pyproject.toml` (setuptools backend)
- Version defined in `freqtrade/__init__.py` (auto-appends git commit in dev)
- Optional dependencies grouped: `[plot]`, `[hyperopt]`, `[freqai]`, `[freqai_rl]`, `[dev]`
- Pre-commit hooks configured in `.pre-commit-config.yaml`
- CI/CD via GitHub Actions (`.github/workflows/`)
