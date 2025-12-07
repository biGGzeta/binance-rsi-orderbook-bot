# Binance RSI + Orderbook Scalping Bot

High-frequency scalping bot for Binance USDC Futures using RSI, orderbook analysis, and volatility detection.

## 🎯 Strategy Overview

This bot implements a sophisticated scalping strategy that combines:
- **RSI extremes** (< 30 for longs, > 70 for shorts)
- **24h price range positioning** for dynamic position sizing
- **Volatility spike decay detection** to catch momentum exhaustion
- **Orderbook imbalance analysis** as final entry trigger
- **Smart rebalancing system** (not martingale - maintains constant exposure)

## ⚡ Key Features

- **0% Maker Fees**: Optimized for Binance USDC Futures pairs with 0% maker fees
- **Micro TPs**: 0.03% to 0.18% take profits for high win rate
- **Risk Controlled**: Fixed 0.60% stop loss with rebalancing mechanism
- **Real-time Orderbook**: Websocket-based orderbook analysis for low latency
- **Volatility-Aware**: Only enters after momentum exhaustion
- **Full Automation**: Manages entries, exits, and rebalances automatically

## 📊 Strategy Details

### Entry Logic
1. **RSI Filter**: Oversold (< 30) or Overbought (> 70)
2. **Position Sizing**: Based on 24h price range
   - Near 24h low with RSI < 30: 100% size (10 USDC margin)
   - Near 24h high with RSI < 30: 60% size (6 USDC margin)
3. **Volatility Check**: Detect spike followed by decay
4. **Orderbook Trigger**: Bid/ask imbalance > 1.5:1 ratio
5. **Confidence Score**: Must be ≥ 70 points

### Position Management
- **Leverage**: 10x
- **Initial TP**: 0.03% (limit order - maker)
- **Stop Loss**: 0.60% (market order)
- **Rebalancing**: Close 20% every -0.09%, re-enter at -0.12%
- **Max Rebalances**: 5 (TP increases to max 0.18%)

### Rebalancing Logic (NOT Martingale)
The bot does NOT increase exposure. It maintains constant notional value while improving average entry price:
1. Price moves -0.09% against position → Close 20% (limit order)
2. Price moves -0.12% total → Re-enter with freed margin minus realized loss
3. Result: Same ~100 USDC exposure, better average entry price
4. TP increases by 0.03% per rebalance to compensate

## 🚀 Quick Start

### 1. Installation

```bash
# Clone repository
git clone https://github.com/biGGzeta/binance-rsi-orderbook-bot.git
cd binance-rsi-orderbook-bot

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your credentials
nano .env
```

**Required settings:**
- `BINANCE_API_KEY`: Your Binance API key
- `BINANCE_SECRET`: Your Binance API secret
- `BINANCE_TESTNET`: Set to `true` for testing

### 3. Run in Testnet

```bash
# Make sure BINANCE_TESTNET=true in .env
python src/main.py
```

### 4. Backtesting

```bash
python src/backtest/backtester.py --start 2024-01-01 --end 2024-12-01 --pairs BTCUSDC,ETHUSDC
```

## 📁 Project Structure

```
binance-rsi-orderbook-bot/
├── src/
│   ├── main.py                    # Entry point
│   ├── config/                    # Configuration
│   ├── strategies/                # Signal generation
│   ├── market_data/               # Orderbook & volatility analysis
│   ├── execution/                 # Order & position management
│   ├── utils/                     # Logging, DB, notifications
│   └── backtest/                  # Backtesting engine
├── tests/                         # Unit tests
├── data/                          # SQLite database
├── logs/                          # Log files
└── NOTES_FOR_AI_AGENT.md         # Critical notes for code review
```

## ⚠️ Important Warnings

### Risk Disclosure
- **High-frequency trading carries significant risk**
- Maximum loss per trade: ~1% of position (0.60% SL + fees)
- Rebalancing can accumulate losses in strong trends
- Requires stable, low-latency connection to Binance servers

### Requirements
- Minimum capital: 500-1000 USDC recommended
- VPS with <50ms latency to Binance servers
- 24/7 uptime and monitoring
- Understanding of futures trading and risks

### Best Practices
- ✅ **Always start in testnet**
- ✅ Test for at least 1 week before live trading
- ✅ Start with 2-5 USDC per position
- ✅ Monitor first 50-100 trades manually
- ✅ Use circuit breakers (max daily loss)
- ❌ Never risk more than you can afford to lose

## 📈 Expected Performance

**Realistic Targets** (conditions dependent):
- Win Rate: 65-75%
- Average Win: 0.08-0.12%
- Average Loss: -0.60%
- Trades per day per pair: 5-15
- Sharpe Ratio Target: > 1.5

**Best Market Conditions:**
- Sideways/ranging markets
- Moderate volatility (0.5-2% per hour)
- High liquidity pairs (>$50M daily volume)

**Worst Market Conditions:**
- Strong trending markets
- Extreme volatility (>5% moves)
- Low liquidity pairs

## 🔧 Configuration

Key parameters in `.env`:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `BASE_MARGIN_USDC` | 10 | Base margin per position |
| `LEVERAGE` | 10 | Leverage multiplier |
| `RSI_PERIOD` | 14 | RSI calculation period |
| `RSI_OVERSOLD` | 30 | Long signal threshold |
| `RSI_OVERBOUGHT` | 70 | Short signal threshold |
| `INITIAL_TP_PCT` | 0.0003 | Initial take profit (0.03%) |
| `STOP_LOSS_PCT` | 0.0060 | Stop loss (0.60%) |
| `MAX_REBALANCES` | 5 | Maximum rebalances per trade |
| `MIN_CONFIDENCE_SCORE` | 70 | Minimum score to enter |
| `ORDERBOOK_DEPTH` | 20 | Levels to analyze |
| `IMBALANCE_THRESHOLD` | 1.5 | Bid/ask ratio for signal |

## 📊 Monitoring

### View Live Trades
```bash
# Check database
sqlite3 data/trades.db
SELECT * FROM trades ORDER BY timestamp DESC LIMIT 10;
```

### Check Logs
```bash
# Real-time logs
tail -f logs/bot.log

# Error logs
tail -f logs/error.log
```

## 🧪 Testing

```bash
# Run all tests
pytest tests/

# Run specific test
pytest tests/test_orderbook_analyzer.py -v

# Run with coverage
pytest --cov=src tests/
```

## 📚 Documentation

- **NOTES_FOR_AI_AGENT.md**: Critical implementation notes for code review
- **Strategy Deep Dive**: See wiki for detailed strategy explanation
- **API Reference**: See docstrings in source code

## 🤝 Contributing

This is a personal trading bot. Fork for your own modifications.

## 📄 License

MIT License - Use at your own risk

## 🆘 Troubleshooting

### Common Issues

**"API key invalid"**
- Check `.env` file has correct keys
- Verify testnet keys for testnet mode

**"Insufficient margin"**
- Check account balance
- Reduce `BASE_MARGIN_USDC` setting

**"Websocket disconnected"**
- Check internet connection
- Bot auto-reconnects, wait 30 seconds

**"No signals generated"**
- Market may not meet entry criteria
- Check `MIN_CONFIDENCE_SCORE` setting
- Review logs for filtering reasons

### Support

- Check logs in `logs/` directory
- Review `NOTES_FOR_AI_AGENT.md` for known issues
- Open an issue for bugs

## 📝 Changelog

### v1.0.0 (Initial Release)
- Complete bot implementation
- RSI + Orderbook + Volatility strategy
- Smart rebalancing system
- Backtesting engine
- Risk management
- SQLite tracking

---

**⚠️ DISCLAIMER: This software is for educational purposes. Trading cryptocurrencies carries risk. Past performance does not guarantee future results. Use at your own risk.**