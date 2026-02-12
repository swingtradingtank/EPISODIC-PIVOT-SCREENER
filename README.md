# PEADS Screener - Post-Earnings Announcement Drift

Automated screener to identify stocks with high probability of Post-Earnings Announcement Drift (PEADS) - stocks that continue to move in the direction of their earnings gap for several days.

## 🎯 Strategy Overview

**Execution Time**: 9:25 AM ET (5 minutes before market open)

**Goal**: Find 2-3 high-quality stocks that:
- Gapped up on earnings
- Have strong fundamentals
- Show institutional interest (high volume)
- Are volatile enough to sustain momentum

## 📊 Filters Applied (8 Total)

| # | Filter | Threshold | Purpose |
|---|--------|-----------|---------|
| 1 | **Float** | < 50M shares | Low float = higher volatility |
| 2 | **Market Cap** | $500M - $20B | Mid-cap sweet spot |
| 3 | **IPO Age** | Last 15 years | Growth companies, not mature |
| 4 | **Revenue Growth** | Q1≥80% OR (Q1≥25% AND Q2≥25%) | Strong fundamental growth |
| 5 | **Gap** | > 4% | Live pre-market price vs yesterday's close |
| 6 | **Pre-market Volume** | ≥ 100k shares | Absolute minimum liquidity |
| 7 | **Price** | > $5 | Avoid penny stocks |
| 8 | **Relative Volume** | ≥ 1.2x | Today's pre-market volume vs 20-day average |

## 🚀 Performance Optimization

Filters are applied in order of speed to minimize API calls:

```
80 earnings tickers
  ↓ Float filter (0.5s × 80 = 40s)
40 survivors
  ↓ Market Cap/IPO (0.5s × 40 = 20s)
20 survivors
  ↓ Revenue Growth (0.5s × 20 = 10s)
10 survivors
  ↓ Gap/Volume (3s × 10 = 30s)
5 survivors
  ↓ Relative Volume (15s × 5 = 75s) ← Only calculated for final candidates
2-3 final stocks

Total Time: ~3-4 minutes (vs 20+ minutes unoptimized)
```

## 📋 Requirements

### API Keys (Required)
1. **Interactive Brokers**: TWS or IB Gateway running locally
2. **Finnhub**: Free tier (60 calls/min) - [Get API Key](https://finnhub.io/)
3. **Twelve Data**: Free tier (800 calls/day) - [Get API Key](https://twelvedata.com/)

### Python Libraries
```bash
pip install ib_insync pandas requests twelvedata nest_asyncio
```

## 🔧 Setup

### 1. Install Interactive Brokers
- Download [TWS](https://www.interactivebrokers.com/en/trading/tws.php) or [IB Gateway](https://www.interactivebrokers.com/en/trading/ibgateway-stable.php)
- Enable API connections in settings
- Start TWS/Gateway before running script

### 2. Configure API Keys
Edit the notebook and insert your keys:
```python
# CHUNK 1
td = TDClient(apikey="YOUR_TWELVEDATA_KEY")
FINNHUB_API_KEY = "YOUR_FINNHUB_KEY"
```

### 3. Adjust IB Connection (if needed)
```python
# CHUNK 2
ib.connect('127.0.0.1', 7497, clientId=6)
# 7497 = TWS live
# 7496 = IB Gateway paper trading
```

## ▶️ Usage

### Run at 9:25 AM ET
1. Start Interactive Brokers TWS/Gateway
2. Open Jupyter Notebook
3. Run all cells sequentially
4. Wait 3-4 minutes
5. Check output: `peads_filtered_stocks_optimized.csv`

### Output CSV Columns
- `ticker`: Stock symbol
- `price`: Current pre-market price
- `gap_pct`: Gap percentage (live vs yesterday's close)
- `premarket_volume`: Volume from 4:00-9:25 AM ET
- `relative_volume`: Today's pre-market volume / 20-day average
- `float_shares_M`: Float in millions
- `short_interest_pct`: Short interest as % of float
- `market_cap`: Market capitalization
- `ipo_year`: Year of IPO
- `revenue_growth_Q1`: Q1 revenue growth %
- `revenue_growth_Q2`: Q2 revenue growth %

## 📈 Example Output

```
ticker  price  gap_pct  premarket_volume  relative_volume  float_shares_M  short_interest_pct
SHOP    67.50    8.5%          250,000            2.1x          18.5              12.3%
CRWD    285.20   6.2%          180,000            1.8x          25.8              8.5%
```

## ⚠️ Important Notes

### Gap Calculation
- **Correct**: Live pre-market price (9:25 AM) vs yesterday's close
- **Ignores**: After-hours movement (4:00-8:00 PM previous day)
- **Why**: After-hours is often low liquidity noise

### Relative Volume
- Only calculated for stocks passing all other filters (5 stocks typically)
- Uses 20-day historical average of pre-market volume
- Identifies anomalous institutional interest

### Float Data
- Sourced from Finnhub (may not always be available)
- Float < 50M filters out large caps
- Lower float = higher price impact from same volume

## 🐛 Troubleshooting

### "Connection error" in Chunk 2
- Ensure TWS/IB Gateway is running
- Check port number (7497 for TWS, 7496 for Gateway)
- Enable API in TWS: File > Global Configuration > API > Settings

### "No earnings calendar found"
- Check Finnhub API key is valid
- Verify date/time (needs to be run on a trading day)
- Some days have few/no earnings

### "No float data for [ticker]"
- Finnhub doesn't have data for all stocks
- Stock will be skipped automatically

### Slow execution
- Normal: 3-4 minutes for 80 tickers
- Check internet connection
- Verify IB Gateway is responsive

## 📊 Backtesting Recommendations

Before live trading, backtest this strategy:
1. Collect 6 months of historical data
2. Calculate win rate, average return, max drawdown
3. Test different holding periods (1-5 days)
4. Implement stop-loss and take-profit rules

## ⚖️ Disclaimer

This screener is for educational purposes only. Past performance does not guarantee future results. Always:
- Use proper position sizing
- Implement stop losses
- Never risk more than you can afford to lose
- Do your own due diligence

## 📄 License

MIT License - Free to use and modify

## 🤝 Contributing

Issues and pull requests welcome!

## 📧 Questions?

Open an issue on GitHub or contact via tancredirosch@gmail.com

---

**Last Updated**: February 2025
**Version**: 1.0 (Optimized)
