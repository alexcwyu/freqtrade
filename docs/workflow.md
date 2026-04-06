# Freqtrade Workflows

## Trading Lifecycle

The main trading loop runs inside the `Worker` class, which throttles execution to align with the configured timeframe. Each iteration calls `FreqtradeBot.process()`, which executes the full trading cycle.

```mermaid
sequenceDiagram
    participant W as Worker
    participant B as FreqtradeBot
    participant E as Exchange
    participant S as Strategy
    participant DP as DataProvider
    participant DB as Database
    participant RPC as RPCManager

    loop Every throttle interval
        W->>B: process()
        B->>E: reload_markets()
        B->>B: update_trades_without_assigned_fees()
        B->>DB: get_open_trades()
        B->>B: _refresh_active_whitelist()
        B->>DP: refresh(pairs, informative_pairs)
        DP->>E: fetch_ohlcv() / WebSocket
        B->>S: bot_loop_start()
        B->>S: analyze(active_pairs)
        S->>S: populate_indicators()
        S->>S: populate_entry_trend()
        S->>S: populate_exit_trend()
        B->>B: manage_open_orders()
        B->>B: exit_positions(trades)
        B->>B: process_open_trade_positions() [DCA]
        B->>B: enter_positions()
        B->>DB: commit()
        B->>RPC: process_msg_queue()
    end
```

## Key Workflows

### New Trade Entry Flow

When the strategy generates an entry signal for a pair, Freqtrade evaluates multiple conditions before placing an order.

```mermaid
sequenceDiagram
    participant B as FreqtradeBot
    participant S as Strategy
    participant PL as PairLocks
    participant W as Wallets
    participant E as Exchange
    participant DB as Database
    participant RPC as RPCManager

    B->>B: enter_positions()
    B->>DB: get_open_trades()
    Note over B: Remove pairs with open trades from whitelist
    B->>PL: is_global_lock()
    alt Global lock active
        B-->>B: Skip all entries
    end

    loop For each pair in whitelist
        B->>B: create_trade(pair)
        B->>DP: get_analyzed_dataframe(pair)
        B->>B: get_free_open_trades()
        B->>S: get_entry_signal(pair)
        alt Signal detected
            B->>PL: is_pair_locked(pair, side)
            alt Pair locked
                B-->>B: Skip pair
            end
            B->>W: get_trade_stake_amount(pair)
            opt Depth of market check enabled
                B->>E: fetch_l2_order_book(pair)
                B->>B: _check_depth_of_market()
            end
            B->>B: execute_entry(pair, stake_amount)
            B->>S: custom_entry_price()
            B->>S: custom_stake_amount()
            B->>S: leverage()
            B->>S: confirm_trade_entry()
            alt Confirmed
                B->>E: create_order(pair, type, side, amount, price)
                B->>DB: Trade.create() + Order.create()
                B->>RPC: send_msg(ENTRY)
            end
        end
    end
```

### Trade Exit Flow

For each open trade, Freqtrade checks multiple exit conditions in priority order.

```mermaid
sequenceDiagram
    participant B as FreqtradeBot
    participant S as Strategy
    participant E as Exchange
    participant DB as Database
    participant RPC as RPCManager

    B->>B: exit_positions(trades)

    loop For each open trade
        B->>W: check_exit_amount(trade)
        alt Insufficient balance
            B->>B: handle_onexchange_order(trade)
        end

        opt stoploss_on_exchange enabled
            B->>B: handle_stoploss_on_exchange(trade)
            alt Stoploss triggered
                B->>RPC: send_msg(EXIT_FILL)
                Note over B: Continue to next trade
            end
        end

        B->>B: handle_trade(trade)
        B->>DP: get_analyzed_dataframe(pair)
        B->>S: get_exit_signal(pair)
        B->>E: get_rate(pair, side=exit)
        B->>S: should_exit(trade, rate)

        Note over S: Check exit conditions in order:
        Note over S: 1. Stoploss (fixed/trailing/custom)
        Note over S: 2. ROI table
        Note over S: 3. Exit signal from strategy
        Note over S: 4. Custom exit logic

        alt Exit condition met
            B->>B: execute_trade_exit(trade, rate, exit_reason)
            B->>S: custom_exit_price()
            B->>S: confirm_trade_exit()
            alt Confirmed
                B->>E: create_order(pair, type, side, amount, price)
                B->>DB: Update trade + Create order
                B->>RPC: send_msg(EXIT)
            end
        end
    end
```

### Stoploss Handling

Freqtrade supports multiple stoploss mechanisms that are evaluated in each iteration.

```mermaid
flowchart TB
    Start[Check Exit Conditions] --> SLType{Stoploss Type?}

    SLType -->|Fixed| Fixed[Calculate fixed SL from entry price]
    SLType -->|Trailing| Trailing[Track highest price, trail SL]
    SLType -->|Custom| Custom[Call strategy.custom_stoploss]
    SLType -->|On Exchange| OnExchange[Manage exchange-side SL order]

    Fixed --> CheckSL{Price <= Stoploss?}
    Trailing --> TrailingLogic{Trailing Logic}
    Custom --> CheckSL

    TrailingLogic --> |Price above offset| Activate[Activate trailing SL]
    TrailingLogic --> |Price below offset| UseFixed[Use initial SL]
    Activate --> UpdateSL[Update SL to trail price]
    UpdateSL --> CheckSL
    UseFixed --> CheckSL

    OnExchange --> SLOrder{SL Order Exists?}
    SLOrder -->|No| CreateSL[Create stoploss order on exchange]
    SLOrder -->|Yes| CheckFill{Order filled?}
    CheckFill -->|Yes| Filled[Trade closed by exchange SL]
    CheckFill -->|No| UpdateCheck{SL needs update?}
    UpdateCheck -->|Yes| CancelRecreate[Cancel old, create new SL order]
    UpdateCheck -->|No| Keep[Keep existing SL order]

    CheckSL -->|Yes| ExitTrade[Execute stoploss exit]
    CheckSL -->|No| Continue[Check next condition: ROI]
```

### Market Data Update Cycle

```mermaid
sequenceDiagram
    participant B as FreqtradeBot
    participant PLM as PairListManager
    participant PL as PairList Handlers
    participant DP as DataProvider
    participant E as Exchange
    participant WS as WebSocket

    B->>PLM: refresh_pairlist()
    PLM->>PL: gen_pairlist() [first handler]
    PL-->>PLM: initial pairs
    loop For each filter handler
        PLM->>PL: filter_pairlist(pairs)
        PL-->>PLM: filtered pairs
    end
    PLM-->>B: whitelist

    B->>DP: refresh(pairs, informative_pairs)

    alt WebSocket available
        DP->>WS: subscribe(pairs, timeframe)
        WS->>DP: streaming OHLCV updates
    else REST fallback
        loop For each pair + timeframe
            DP->>E: fetch_ohlcv(pair, timeframe)
            E-->>DP: candle data
        end
    end

    DP->>DP: Cache dataframes with timestamps
    Note over DP: Data available via dp.get_analyzed_dataframe()
```

### Strategy Signal Processing

```mermaid
sequenceDiagram
    participant B as FreqtradeBot
    participant S as Strategy
    participant DP as DataProvider
    participant FreqAI as FreqAI Model

    B->>S: analyze(pairs)

    loop For each pair
        S->>S: Check last_candle_seen
        alt New candle or first run
            S->>DP: get_pair_dataframe(pair, timeframe)

            opt Informative pairs defined
                loop For each informative pair/timeframe
                    S->>DP: get_pair_dataframe(inf_pair, inf_tf)
                    S->>S: Merge informative data
                end
            end

            S->>S: populate_indicators(dataframe)

            opt FreqAI enabled
                S->>S: feature_engineering_expand_all()
                S->>S: feature_engineering_expand_basic()
                S->>S: feature_engineering_standard()
                S->>FreqAI: predict(dataframe)
                FreqAI-->>S: predictions in dataframe columns
            end

            S->>S: populate_entry_trend(dataframe)
            S->>S: populate_exit_trend(dataframe)
            S->>S: Validate result (column checks)
            S->>DP: Store analyzed dataframe in cache
        end
    end
```

### Backtesting Flow

```mermaid
sequenceDiagram
    participant User as User
    participant BT as Backtesting
    participant S as Strategy
    participant DP as DataProvider
    participant History as History Module

    User->>BT: start()
    BT->>History: load_data(pairs, timerange)
    History-->>BT: OHLCV DataFrames

    loop For each strategy
        BT->>S: analyze(pairs)
        S->>S: populate_indicators()
        S->>S: populate_entry_trend()
        S->>S: populate_exit_trend()
        S-->>BT: Analyzed DataFrames with signals

        BT->>BT: backtest(processed_data)

        loop For each candle (chronological)
            BT->>BT: Check open trade exits
            Note over BT: Evaluate: stoploss, ROI, signals, custom_exit
            BT->>BT: Process DCA adjustments
            BT->>BT: Check entry signals
            alt Entry signal + free slots
                BT->>S: confirm_trade_entry()
                BT->>BT: Create LocalTrade
            end
            alt Exit condition met
                BT->>S: confirm_trade_exit()
                BT->>BT: Close LocalTrade
            end
        end

        BT->>BT: generate_backtest_stats()
        BT->>BT: show_backtest_results()
        BT->>BT: store_backtest_results()
    end
```

### Hyperparameter Optimization Flow

```mermaid
sequenceDiagram
    participant User as User
    participant HO as Hyperopt
    participant Opt as HyperOptimizer
    participant BT as Backtesting
    participant S as Strategy
    participant Optuna as Optuna

    User->>HO: start()
    HO->>Opt: Initialize search spaces
    Opt->>S: Read parameter definitions
    Note over Opt: IntParameter, DecimalParameter,<br/>CategoricalParameter, BooleanParameter

    HO->>Opt: setup_study()
    Opt->>Optuna: Create study with search spaces

    loop For each epoch (parallel via joblib)
        Optuna->>Opt: suggest parameters (Trial)
        Opt->>S: Apply parameters to strategy
        Opt->>BT: backtest(processed_data)
        BT-->>Opt: Trade results

        Opt->>Opt: Calculate loss (selected loss function)
        Note over Opt: SharpeHyperOptLoss, SortinoHyperOptLoss,<br/>CalmarHyperOptLoss, MaxDrawDownHyperOptLoss, etc.

        Opt-->>Optuna: Report loss value
        Optuna->>Optuna: Update search model

        alt New best result
            HO->>HO: Update current_best_epoch
            HO->>HO: Display result
        end
    end

    HO->>HO: Store results to .fthypt file
    HO->>User: Display best parameters
```

### DCA (Dollar Cost Averaging) Flow

When `position_adjustment_enable = True` in the strategy, Freqtrade checks open trades each iteration for position adjustments.

```mermaid
sequenceDiagram
    participant B as FreqtradeBot
    participant S as Strategy
    participant E as Exchange
    participant W as Wallets
    participant DB as Database

    B->>B: process_open_trade_positions()

    loop For each open trade
        alt Trade has open orders
            Note over B: Skip - wait for orders to fill
        else No open orders
            B->>W: update()
            B->>B: check_and_call_adjust_trade_position(trade)
            B->>E: get_rates(pair)
            B->>B: calc_profit_ratio()
            B->>E: get_min/max_pair_stake_amount()
            B->>W: get_available_stake_amount()

            B->>S: adjust_trade_position(trade, current_rate, current_profit, min_stake, max_stake)

            alt stake_amount > 0 (increase position)
                Note over S: Strategy wants to buy more (DCA down/up)
                alt max_entry_position_adjustment not exceeded
                    B->>B: execute_entry(pair, stake_amount, trade=trade, mode=pos_adjust)
                    B->>E: create_order(pair, buy, amount, price)
                    B->>DB: Create new Order linked to existing Trade
                    Note over DB: Trade.recalc_trade_from_orders()<br/>Updates average entry price
                end

            else stake_amount < 0 (decrease position)
                Note over S: Strategy wants to sell part of position
                B->>B: execute_trade_exit(trade, rate, PARTIAL_EXIT, sub_trade_amt)
                B->>E: create_order(pair, sell, amount, price)
                B->>DB: Create exit Order linked to Trade
            end
        end
    end
```

## API/RPC Event Handling

The RPC system processes events asynchronously through a message queue. The `FreqtradeBot` enqueues messages during the trading loop, and they are dispatched to all registered handlers at the end of each iteration.

```mermaid
flowchart TB
    subgraph Events["Trade Events"]
        Entry[Entry Order Placed]
        EntryFill[Entry Order Filled]
        EntryCancel[Entry Cancelled]
        Exit[Exit Order Placed]
        ExitFill[Exit Order Filled]
        ExitCancel[Exit Cancelled]
        Protection[Protection Triggered]
        Status[Status Change]
    end

    subgraph Processing
        Entry --> Queue[Message Queue]
        EntryFill --> Queue
        EntryCancel --> Queue
        Exit --> Queue
        ExitFill --> Queue
        ExitCancel --> Queue
        Protection --> Queue
        Status --> Queue
    end

    Queue --> RPCManager

    subgraph Handlers["RPC Handlers"]
        RPCManager --> TG[Telegram]
        RPCManager --> API[REST API]
        RPCManager --> WS[WebSocket]
        RPCManager --> WH[Webhook]
        RPCManager --> DC[Discord]
    end

    subgraph TelegramCmds["Telegram Commands (Inbound)"]
        Start[/start] --> RPCCore[RPC Core]
        Stop[/stop] --> RPCCore
        StatusCmd[/status] --> RPCCore
        Profit[/profit] --> RPCCore
        ForceSell[/forcesell] --> RPCCore
        ForceBuy[/forcebuy] --> RPCCore
        RPCCore --> |state changes| Bot[FreqtradeBot]
    end

    subgraph APICmds["API Endpoints (Inbound)"]
        GET[GET /api/v1/status] --> RPCCore
        POST[POST /api/v1/forcebuy] --> RPCCore
        DELETE[DELETE /api/v1/trades/{id}] --> RPCCore
    end
```

### WebSocket Real-Time Updates

The API server supports WebSocket connections for real-time streaming of:

- Analyzed DataFrames (for live chart updates in FreqUI)
- Whitelist changes
- New candle notifications

Clients subscribe to specific message types via `RPCRequestType.SUBSCRIBE` and receive pushed updates as they occur.

---
## See Also
- [README](README.md) — Project overview and quick start
- [Architecture](architecture.md) — System design and components
- [State Management](state-management.md) — State lifecycle and data models
- [Development](development.md) — Development guide and best practices
