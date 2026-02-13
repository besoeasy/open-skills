---
name: trading-indicators-from-price-data
description: Compute common trading indicators from OHLCV price data for analysis and strategy development.
---

# Trading Indicators from Price Data (20 common indicators)

Calculate 20 widely used trading indicators from OHLCV candles (open, high, low, close, volume) using Python.

This skill is useful for:
- signal generation
- strategy backtesting
- feature engineering for ML models
- market condition dashboards

## Requirements

Install dependencies:

```bash
pip install pandas pandas-ta
```

Input data must include these columns:
- `open`
- `high`
- `low`
- `close`
- `volume`

## 20 indicators included

1. RSI (14)
2. MACD line (12,26)
3. MACD signal (9)
4. MACD histogram
5. SMA (20)
6. SMA (50)
7. EMA (20)
8. EMA (50)
9. WMA (20)
10. Bollinger upper band (20,2)
11. Bollinger middle band (20,2)
12. Bollinger lower band (20,2)
13. Stochastic %K (14,3,3)
14. Stochastic %D (14,3,3)
15. ATR (14)
16. ADX (14)
17. CCI (20)
18. OBV
19. MFI (14)
20. ROC (12)

## Python skill: compute all 20 indicators

```python
import pandas as pd
import pandas_ta as ta

def calculate_20_indicators(df: pd.DataFrame) -> pd.DataFrame:
    required = {'open', 'high', 'low', 'close', 'volume'}
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"Missing required columns: {sorted(missing)}")

    out = df.copy()

    # 1) RSI
    out['rsi_14'] = ta.rsi(out['close'], length=14)

    # 2-4) MACD
    macd = ta.macd(out['close'], fast=12, slow=26, signal=9)
    out['macd_line'] = macd['MACD_12_26_9']
    out['macd_signal'] = macd['MACDs_12_26_9']
    out['macd_hist'] = macd['MACDh_12_26_9']

    # 5-9) Moving averages
    out['sma_20'] = ta.sma(out['close'], length=20)
    out['sma_50'] = ta.sma(out['close'], length=50)
    out['ema_20'] = ta.ema(out['close'], length=20)
    out['ema_50'] = ta.ema(out['close'], length=50)
    out['wma_20'] = ta.wma(out['close'], length=20)

    # 10-12) Bollinger bands
    bb = ta.bbands(out['close'], length=20, std=2)
    out['bb_upper'] = bb['BBU_20_2.0']
    out['bb_middle'] = bb['BBM_20_2.0']
    out['bb_lower'] = bb['BBL_20_2.0']

    # 13-14) Stochastic
    stoch = ta.stoch(out['high'], out['low'], out['close'], k=14, d=3, smooth_k=3)
    out['stoch_k'] = stoch['STOCHk_14_3_3']
    out['stoch_d'] = stoch['STOCHd_14_3_3']

    # 15) ATR
    out['atr_14'] = ta.atr(out['high'], out['low'], out['close'], length=14)

    # 16) ADX
    adx = ta.adx(out['high'], out['low'], out['close'], length=14)
    out['adx_14'] = adx['ADX_14']

    # 17) CCI
    out['cci_20'] = ta.cci(out['high'], out['low'], out['close'], length=20)

    # 18) OBV
    out['obv'] = ta.obv(out['close'], out['volume'])

    # 19) MFI
    out['mfi_14'] = ta.mfi(out['high'], out['low'], out['close'], out['volume'], length=14)

    # 20) ROC
    out['roc_12'] = ta.roc(out['close'], length=12)

    return out


# Example usage with CSV candles:
# CSV columns: timestamp,open,high,low,close,volume
if __name__ == '__main__':
    candles = pd.read_csv('candles.csv')
    result = calculate_20_indicators(candles)

    selected_cols = [
        'close', 'rsi_14', 'macd_line', 'macd_signal', 'macd_hist',
        'sma_20', 'sma_50', 'ema_20', 'ema_50', 'wma_20',
        'bb_upper', 'bb_middle', 'bb_lower', 'stoch_k', 'stoch_d',
        'atr_14', 'adx_14', 'cci_20', 'obv', 'mfi_14', 'roc_12'
    ]

    print(result[selected_cols].tail(5))
```

## Notes

- Indicators need warmup candles (first rows can be `NaN`).
- For stable output, use at least 200 candles.
- If you run this on minute candles, indicators are intraday; on daily candles, they are swing/position oriented.

## Agent prompt

```text
You have a trading-indicators skill.

When given OHLCV price data, calculate the following 20 indicators:
RSI(14), MACD line/signal/histogram (12,26,9), SMA(20), SMA(50), EMA(20), EMA(50), WMA(20),
Bollinger upper/middle/lower (20,2), Stoch %K/%D (14,3,3), ATR(14), ADX(14), CCI(20), OBV, MFI(14), ROC(12).

Return a table with the latest value of each indicator and include the last 50 rows when requested.
If data is insufficient, ask for more candles.
```
