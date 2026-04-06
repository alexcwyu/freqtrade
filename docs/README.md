# Freqtrade -- Project Documentation

> **Last Updated**: 2026-04-06T16:25:30Z  \
> **Git Hash**: `46f73cacb`

> **Version**: 2026.4-dev | **License**: GPLv3 | **Python**: >= 3.13

## Overview

Freqtrade is a free and open-source cryptocurrency trading bot written in Python. It is designed to support algorithmic trading on all major cryptocurrency exchanges through the CCXT library. Freqtrade provides a complete trading ecosystem: strategy development, backtesting, hyperparameter optimization, live/dry-run trading, and machine learning integration (FreqAI).

The bot operates as an event-driven system with a throttled main loop, executing user-defined strategies against real-time or historical market data. It supports spot trading, margin trading, and futures (long and short positions) with configurable leverage.

**Official Documentation**: [https://www.freqtrade.io](https://www.freqtrade.io)

## Key Features

- **Multi-Exchange Support** -- Unified exchange abstraction via CCXT supporting 25+ exchanges including Binance, Kraken, OKX, Bybit, Gate.io, Kucoin, Bitget, and more. Each exchange has a dedicated adapter for exchange-specific behavior.

- **Strategy Framework** -- Python-based strategy interface (`IStrategy`) with methods for indicator computation, entry/exit signal generation, custom stoploss, position sizing, and DCA (Dollar Cost Averaging) via `adjust_trade_position`.

- **Backtesting Engine** -- High-fidelity historical simulation that processes OHLCV candles tick-by-tick, supporting multi-strategy runs, detailed trade analysis, and signal/rejected-signal export.

- **Hyperparameter Optimization** -- Optuna-based hyperopt with parallel execution (joblib), supporting optimization of buy/sell parameters, ROI tables, stoploss values, and trailing stop configurations. Multiple loss functions available (Sharpe, Sortino, Calmar, MaxDrawDown, etc.).

- **FreqAI (Machine Learning)** -- Integrated ML pipeline supporting PyTorch, XGBoost, LightGBM, CatBoost, and reinforcement learning. Automated feature engineering, model training, and prediction within the strategy loop.

- **Remote Control** -- Telegram bot integration, REST API with web UI (FreqUI), Discord notifications, and webhook support. Full bot control and monitoring from any device.

- **Risk Management** -- Configurable stoploss (fixed, trailing, custom), ROI tables, pair locking, protections (cooldown, max drawdown, stoploss guard), and position adjustment limits.

- **Plugin Architecture** -- Extensible pairlist handlers (volume-based, market cap, performance filters, spread filters) and protection plugins with a resolver-based dynamic loading system.

- **Data Management** -- Historical data download, multiple storage formats (JSON, Parquet), data conversion utilities, and an analysis toolkit for post-backtest investigation.

- **Futures & Leverage** -- Full support for futures trading with cross/isolated margin modes, funding fee tracking, liquidation price calculation, and leverage configuration.

## Architecture Overview

Freqtrade follows a layered architecture with clear separation of concerns:

```
CLI/Commands --> Worker --> FreqtradeBot --> Strategy
                                |
                    +-----------+-----------+
                    |           |           |
                Exchange   DataProvider  RPCManager
                    |           |           |
                  CCXT      History/    Telegram/API/
                            Candles     Webhook/Discord
```

The `Worker` class manages the bot lifecycle (start, stop, reconfigure) and throttles the main trading loop. `FreqtradeBot` is the central orchestrator that coordinates all components: exchange interaction, strategy execution, trade management, and notifications.

See [architecture.md](architecture.md) for detailed component breakdowns and diagrams.

## Component Summary

| Component | Location | Description |
|-----------|----------|-------------|
| **FreqtradeBot** | `src/freqtrade/freqtradebot.py` | Main orchestrator -- trade lifecycle, order management, DCA |
| **Worker** | `src/freqtrade/worker.py` | Bot lifecycle manager with throttled main loop |
| **Strategy** | `src/freqtrade/strategy/` | `IStrategy` interface for user-defined trading logic |
| **Exchange** | `src/freqtrade/exchange/` | CCXT-based exchange abstraction with 25+ adapters |
| **DataProvider** | `src/freqtrade/data/dataprovider.py` | Candle data, orderbook, and ticker access for strategies |
| **Persistence** | `src/freqtrade/persistence/` | SQLAlchemy models for Trade, Order, PairLock |
| **RPC** | `src/freqtrade/rpc/` | Telegram, REST API, webhooks, Discord integration |
| **Plugins** | `src/freqtrade/plugins/` | Pairlist handlers and protection plugins |
| **Backtesting** | `src/freqtrade/optimize/backtesting.py` | Historical strategy simulation engine |
| **Hyperopt** | `src/freqtrade/optimize/hyperopt/` | Optuna-based parameter optimization |
| **FreqAI** | `src/freqtrade/freqai/` | Machine learning integration pipeline |
| **Wallets** | `src/freqtrade/wallets.py` | Balance tracking and position management |
| **Configuration** | `src/freqtrade/configuration/` | JSON config loading, validation, and schema |
| **Commands** | `src/freqtrade/commands/` | CLI command implementations (trade, backtest, etc.) |

## Documentation Index

| Document | Description |
|----------|-------------|
| [architecture.md](architecture.md) | System architecture, component diagrams, plugin system |
| [workflow.md](workflow.md) | Trading lifecycle, key workflows with sequence diagrams |
| [state-management.md](state-management.md) | Trade/order state machines, bot states, locks |
| [development.md](development.md) | Setup, project structure, strategy development guide |

## Quick Start

A minimal strategy with RSI entry/exit signals and a dry-run backtest command. No exchange API keys needed for backtesting.

```python
# File: user_data/strategies/QuickRSI.py
from pandas import DataFrame
from freqtrade.strategy import IStrategy
import talib.abstract as ta


class QuickRSI(IStrategy):
    INTERFACE_VERSION = 3
    timeframe = "1h"
    minimal_roi = {"0": 0.05}       # 5% take-profit
    stoploss = -0.10                 # 10% stoploss
    startup_candle_count = 30
    can_short = False

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe["rsi"] = ta.RSI(dataframe, timeperiod=14)
        return dataframe

    def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe.loc[
            (dataframe["rsi"] < 30) & (dataframe["volume"] > 0),
            "enter_long",
        ] = 1
        return dataframe

    def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe.loc[
            (dataframe["rsi"] > 70) & (dataframe["volume"] > 0),
            "exit_long",
        ] = 1
        return dataframe
```

Run a backtest (no API keys needed):

```bash
# Download sample data
freqtrade download-data --exchange binance --pairs BTC/USDT --timeframes 1h --days 90

# Create a minimal config for backtesting
freqtrade new-config --config config_backtest.json

# Run the backtest
freqtrade backtesting \
    --config config_backtest.json \
    --strategy QuickRSI \
    --timerange 20240101-20240401 \
    --dry-run-wallet 1000
```

## FreqAI Overview

FreqAI is Freqtrade's integrated machine learning pipeline that enables adaptive, self-training strategies. It automates the full ML lifecycle within the trading loop: feature engineering, model training, prediction, and model management.

### Supported Models

| Model | Class | Framework | Use Case |
|-------|-------|-----------|----------|
| LightGBM Regressor | `LightGBMRegressor` | LightGBM | Price/return prediction |
| LightGBM Classifier | `LightGBMClassifier` | LightGBM | Direction classification |
| XGBoost Regressor | `XGBoostRegressor` | XGBoost | Price/return prediction |
| XGBoost Classifier | `XGBoostClassifier` | XGBoost | Direction classification |
| XGBoost RF Regressor | `XGBoostRFRegressor` | XGBoost | Random forest regression |
| XGBoost RF Classifier | `XGBoostRFClassifier` | XGBoost | Random forest classification |
| PyTorch MLP Regressor | `PyTorchMLPRegressor` | PyTorch | Deep learning regression |
| PyTorch MLP Classifier | `PyTorchMLPClassifier` | PyTorch | Deep learning classification |
| PyTorch Transformer | `PyTorchTransformerRegressor` | PyTorch | Transformer-based prediction |
| Reinforcement Learner | `ReinforcementLearner` | Stable-Baselines3 | RL-based trading agent |
| RL Multiproc | `ReinforcementLearner_multiproc` | SB3 + multiprocessing | Parallel RL training |

Multi-target variants (`LightGBMRegressorMultiTarget`, `XGBoostRegressorMultiTarget`, `LightGBMClassifierMultiTarget`) predict multiple outputs simultaneously.

### How It Works

1. **Feature engineering**: Define features in strategy's `feature_engineering_expand_all()` and `feature_engineering_expand_basic()` methods
2. **Automated training**: FreqAI trains models on a rolling window of historical data, retraining at configurable intervals
3. **Prediction**: Model predictions appear as `&-prefixed` columns in the strategy DataFrame (e.g., `&-s_close`)
4. **Data preprocessing**: The `FreqaiDataKitchen` handles feature scaling, NaN removal, and outlier detection via a `datasieve` pipeline
5. **Model persistence**: The `FreqaiDataDrawer` manages model serialization, metadata, and staleness detection

### Minimal FreqAI Config

```json
{
    "freqai": {
        "enabled": true,
        "identifier": "my_model",
        "model_training_parameters": {
            "n_estimators": 800
        },
        "data_split_parameters": {
            "test_size": 0.33
        },
        "feature_parameters": {
            "include_corr_pairlist": ["BTC/USDT", "ETH/USDT"],
            "include_timeframes": ["5m", "15m", "1h"],
            "indicator_periods_candles": [10, 20, 50]
        },
        "train_period_days": 30,
        "backtest_period_days": 7
    }
}
```

See the official FreqAI documentation at [freqtrade.io/en/latest/freqai/](https://www.freqtrade.io/en/latest/freqai/) for complete configuration options.

## Quick Start (CLI Commands)

```bash
# Install
pip install -e .

# Create a new configuration
freqtrade new-config --config config.json

# Create a new strategy
freqtrade new-strategy --strategy MyStrategy

# Download historical data
freqtrade download-data --exchange binance --pairs BTC/USDT ETH/USDT --timeframes 5m 1h

# Backtest a strategy
freqtrade backtesting --config config.json --strategy MyStrategy --timerange 20230101-20231231

# Optimize parameters
freqtrade hyperopt --config config.json --strategy MyStrategy --hyperopt-loss SharpeHyperOptLoss -e 500

# Run in dry-run mode (paper trading)
freqtrade trade --config config.json --strategy MyStrategy

# Start the web UI server
freqtrade webserver --config config.json
```

## Links

- **GitHub**: [https://github.com/freqtrade/freqtrade](https://github.com/freqtrade/freqtrade)
- **Documentation**: [https://www.freqtrade.io](https://www.freqtrade.io)
- **Discord**: [https://discord.gg/p7nuUNVfP7](https://discord.gg/p7nuUNVfP7)
- **FreqUI**: [https://github.com/freqtrade/frequi](https://github.com/freqtrade/frequi)
