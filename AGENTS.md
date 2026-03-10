# Jesse Repository Guide for AI Agents

## Overview
The jesse repository is the **core open-source framework** of the Jesse trading system. It contains the main Python codebase for backtesting trading strategies, importing historical data from crypto exchanges, running optimizations, and providing the API backend for the dashboard. It glues together the other repositories and makes them work together.

## Key Characteristics

### Central Framework
- **jesse-live depends on this** - Changes here affect live trading
- **jesse-rust integrates here** - Rust functions are called from this codebase
- **dashboard consumes this API** - Frontend uses the FastAPI routes and controller files. 

### Technology Stack
- **Python** - Primary language
- **FastAPI** - API framework for all routes
- **NumPy** - Array operations and calculations
- **keewee** - ORM for the database

## Development Workflow

### Making Changes
When implementing features or fixing bugs:

1. **Understand the scope** - Determine if other repositories such as the dashboard need updates
2. **Implement the code** in the appropriate module
3. **Write/update tests** - Maintain test coverage
4. **Run tests** to verify changes:
   ```bash
   cd-jesse && pytest
   ```
5. **Consider jesse-live** - Does this affect live trading?
6. **Update API routes** if needed - Follow FastAPI patterns
7. **Don't restart server** unless specifically asked

### Python Environment
Use the Jesse Python interpreter:
```
/Users/salehmir/miniconda3/envs/jesse3.12/bin/python
```

### Running Jesse Backend
The API server provides routes for the dashboard:
```bash
# Stop any running process
pkill -f "jesse run"

# Start Jesse from bot directory (not jesse/)
cd /Users/salehmir/codes/jesse/dev-jesse/bot
/Users/salehmir/miniconda3/envs/jesse3.12/bin/jesse run > /tmp/jesse-output.log 2>&1 &

# Server runs at http://localhost:9001

# Check logs
tail -f /tmp/jesse-output.log
```

**Important**: Don't restart Jesse after code changes unless explicitly requested.

### Running Tests
Run the test suite after changes if asked.
```bash
cd-jesse && pytest
```

If you've updated jesse-rust, run tests after building:
```bash
cd /Users/salehmir/Codes/jesse/dev-jesse/jesse-rust
./build-local.sh

cd /Users/salehmir/Codes/jesse/dev-jesse/jesse
pytest
```

## Important Notes

### Debugging
- **Use `jh.debug()` for all debugging output** - Never use plain `print()`
- **Log format**: `[2024-12-06 18:23:12] ==> Your message here`
- Logs include timestamps and `==>` prefix
- Essential for debugging backtests and live trading sessions

### API Routes
- **Default to POST endpoints** unless specifically asked for GET
- Use FastAPI decorators and patterns
- Follow the structure of existing routes in `jesse/routes/`
- Return proper HTTP status codes and JSON responses
- Handle errors gracefully

### Code Style
- Don't write comments for functions unless asked
- Never try to install new packages - assume they're already installed. if need to install new packages, ask me first.
- Follow existing patterns and conventions
- Maintain consistency with the current codebase
- Try to import only at the top of the file.

### Jesse-Rust Integration
- When using Rust functions, **assume they exist** - don't add existence checks
- Update Python code to call new Rust implementations
- Build jesse-rust locally and run tests to verify integration
- Performance-critical code should be delegated to jesse-rust when possible

## File Structure
- `jesse/` - Main source code
  - `indicators/` - Technical indicators
  - `modes/` - Backtest, optimize, import modes, monte carlo, etc
  - `routes/` - FastAPI route handlers
  - `services/` - data services, etc
  - `strategies/` - Base strategy classes
  - `store/` - State management
- `tests/` - Test suite
- `storage/` - Logs and temporary files
- `requirements.txt` - Python dependencies
- `setup.py` - Package configuration

## Testing Strategy

### Unit Tests
- Run `pytest` after every change if asked in the conversation.
- Maintain or improve test coverage
- Add tests for new features if asked in the conversation.
- Fix failing tests immediately

## Related Repositories
This repository is the foundation of the Jesse ecosystem:
- **jesse-live** - Depends heavily on jesse for live trading
- **jesse-rust** - Performance layer integrated into jesse
- **dashboard-v1** - Frontend that consumes jesse's API
- **bot** - Jesse project instance that runs the framework
- **laravel-jesse-trade** - Laravel project that contains the api1 backend of the jesse-trade website.
- **go-jesse-trade/backend** - Go project that contains the api2 backend of the jesse-trade website.
- **go-jesse-trade/frontend** - NuxtJS project that contains the frontend of the jesse-trade website.
- **strategy-executor** - Go project that contains the strategy executor microservice used to execute strategies submitted by the users of the
- 
## Google Antigravity Skill Configuration

This section enables Google AI Studio (Antigravity/Gemini) to recognize and use this repository as a **Skill**.

### Skill Identity

- **Skill Name**: Jesse Crypto Trading Bot
- **Version**: 1.13.7
- **Language**: Python
- **Type**: Algorithmic Trading Framework
- **Category**: Crypto / Finance / Backtesting

### What This Skill Can Do

This skill allows an AI agent to:

1. **Backtest trading strategies** - Run historical simulations on OHLCV candle data
2. **Import candle data** - Fetch historical price data from crypto exchanges (Binance, Bybit, etc.)
3. **Optimize strategies** - Run hyperparameter optimization using genetic algorithms or grid search
4. **Live trade** - Execute real-time trades via exchange APIs
5. **Analyze indicators** - Access 100+ technical indicators (RSI, MACD, Bollinger Bands, EMA, etc.)
6. **Run Monte Carlo simulations** - Stress-test strategies with randomized scenarios
7. **Manage routes** - Define symbol/timeframe/strategy combinations
8. **Generate performance reports** - Sharpe ratio, max drawdown, win rate, PnL, etc.

### Entry Points for the Agent

| Command | Description |
|---|---|
| `jesse backtest` | Run a backtest on a strategy |
| `jesse import-candles` | Import OHLCV data from an exchange |
| `jesse optimize` | Run strategy optimization |
| `jesse run` | Start the live trading API server |

### Key Modules the Agent Should Know

- `jesse/strategies/` - Base class `Strategy` that all custom strategies extend
- `jesse/indicators/` - All technical indicators available as functions
- `jesse/modes/` - Backtest, import, optimize, live modes
- `jesse/routes/` - FastAPI REST endpoints (server runs on port 9001)
- `jesse/store/` - Global state management during sessions
- `jesse/services/` - Utilities like logger, file IO, exchange connectors

### How to Write a Strategy

All strategies inherit from `jesse.strategies.Strategy` and must implement:

```python
from jesse.strategies import Strategy
import jesse.indicators as ta

class MyStrategy(Strategy):
    def should_long(self) -> bool:
        return ta.ema(self.candles, 9) > ta.ema(self.candles, 21)

    def should_short(self) -> bool:
        return ta.ema(self.candles, 9) < ta.ema(self.candles, 21)

    def go_long(self):
        self.buy = 1, self.price

    def go_short(self):
        self.sell = 1, self.price

    def should_cancel_entry(self) -> bool:
        return False
```

### Supported Exchanges

- Binance Spot & Futures
- Bybit Spot & Perpetual
- Bitfinex
- FTX (historical data only)
- Coinbase

### Agent Interaction Guidelines

- Always use `jh.debug()` for logging, never `print()`
- Never restart the server unless explicitly asked
- Strategy files live in the `strategies/` folder of the **bot** project (not this repo)
- This repo is the **framework** — the bot project imports and uses it
- API server runs at `http://localhost:9001`
- Database ORM is `keewee` (not SQLAlchemy)website.

