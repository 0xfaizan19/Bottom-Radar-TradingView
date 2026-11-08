# Bottom Radar — backtest report (Nov 7, 2026)

Weekly charts, full available history per chart. Every signal is scored 26 weeks after it fires.

**How each signal is scored**
- **Fell >20% after**: the price fell more than 20% below the entry within 26 weeks. This is the falling-knife rate.
- **Distance from low**: median gap between the entry and the lowest low from 8 weeks before to 26 weeks after the entry.
- **Further drop**: median deepest drop below the entry within 26 weeks.
- **Naive**: buying every week the stock was already in a dip just as deep. For BOTTOM that is a drop of at least max(20%, 1σ) from the 52-week high; for DIP, at least max(10%, 0.5σ). Beating this baseline means the timing adds value, not just the dip.

## Chart sets

| Set | Charts | Used for |
|---|---|---|
| Test 50 | 40 stocks + 10 ETFs (QQQ, IWM, DIA, XLF, XLE, XLK, XLV, XLI, XLY, SMH; MSFT … UAL) | Finding v1.2's errors and designing v2 |
| Holdout 20 | IBM, MCD, MMM, T, C, GE, ORCL, AMGN, MRK, LOW, TGT, QCOM, TXN, EBAY, LUV, CLF, SNAP, ZM, UPST, BBY | Checking that v2 generalizes (never tuned on) |
| New 100 | 85 stocks + 15 ETFs, none used before (AVGO … GME; XLP … KWEB) | Out-of-sample test of v2, source of the v2.1 learnings, final re-run on TradingView |

The test harness is a JavaScript copy of the Pine logic running on TradingView's own bar data. It matched the live indicator on 1,006 of 1,006 signals (v1.2) and 1,814 of 1,852 (v2). The few mismatches are all 1990s signals on OXY and HUM, where TradingView has older history than the chart loads.

## Errors found in v1.2

1. **Single-stock BOTTOMs were no better than blind dip-buying.** On the 20 unseen stocks, 38% fell >20% after the signal, vs 37% for naive.
2. **Failures cluster in market-wide bear markets.** BOTTOM signals that fell >20% after: 2008 51/68, 2022 24/43, 2001–02 20/49. Most other years were under 15%.
3. **Stocks in long declines kept falling.** Price far below the 40-week average, a falling 40-week average, or a stock deep below its all-time high: 45–54% fell >20% after, vs 21–25% in healthy trends.
4. **RSI divergence pointed to bad bottoms.** It shows up on the 3rd or 4th leg of long bear markets: 48% fell >20% after, vs 24% without.
5. **Slow, grinding setups failed.** Signals more than 7 weeks after the zone opened: 54% fell >20% after.
6. **DIPs fired at the start of bear markets.** DIPs far below the highs or the 40-week average: 42% fell >20% after.
7. **The 2020 COVID low was missed on SPY.** The "max chase" limit was too tight for V-shaped bottoms; fixed in v1.1.

## What changed

| Version | Change | Based on |
|---|---|---|
| v1.1 | Chase limit 2.5 → 4 ATR; strict re-entry only after a failed BOTTOM; DIP uptrend definition tightened | SPY 2018 and 2020 misses |
| v2.0 | Divergence removed from the score and the re-entry rule | Error 4 |
| v2.0 | DIP must be within 35% of the all-time high and no more than 0.3σ below the 40-week average | Error 6 |
| v2.0 | **Grading.** Green when the market is OK (SPY not below a falling 40-week average, or SPY just confirmed its own bottom), the stock's 52-week high is ≥75% of its all-time high, and the signal comes ≤10 weeks after the zone opened. Otherwise grey "?" | Errors 1, 2, 3, 5 |
| v2.1 | Green BOTTOM also needs the stock lagging the market by no more than 15% over 13 weeks, and price within 20% of its 40-week average. Green DIP also needs the relative-strength check | New learnings from the 100 charts |

## Results (test harness, same data for every version)

### Green BOTTOM: share that fell >20% after

| Set | v1.2 | v2.1 green | v2.1 grey | Naive |
|---|---|---|---|---|
| New 85 stocks | 37.3% | **19.8%** | 41.0% | 40.7% |
| New 15 ETFs | 22.2% | **9.0%** | 33.8% | 28.0% |
| Test 40 stocks | 36.2% | **21.8%** | 38.4% | 42.9% |
| Holdout 20 stocks | 38.0% | **21.1%** | 42.9% | 37.0% |
| Test 10 ETFs | 23.6% | **11.0%** | 40.0% | 33.9% |

### Green BOTTOM: distance from the true low (median)

| Set | v1.2 | v2.1 green | Naive |
|---|---|---|---|
| New 85 stocks | 26.0% | **17.5%** | 23.2% |
| New 15 ETFs | 17.5% | **13.7%** | 18.0% |
| Test 40 stocks | 23.4% | **16.6%** | 25.2% |
| Holdout 20 stocks | 23.2% | **15.8%** | 21.3% |
| Test 10 ETFs | 18.5% | **16.5%** | 20.8% |

### DIP: share that fell >20% after

| Set | v1.2 | v2.1 green DIP | v2.1 grey DIP? | Naive |
|---|---|---|---|---|
| New 85 stocks | 31.3% | **26.1%** | 39.5% | 34.4% |
| Test 40 stocks | 27.2% | **20.2%** | 26.9% | 32.6% |
| Holdout 20 stocks | 22.3% | **15.6%** | 29.2% | 31.2% |

### The trade-off: major lows flagged

| Set | v1.2 | v2.1 green only | v2.1 green + grey |
|---|---|---|---|
| New 85 stocks | 445/842 (53%) | 111 (13%) | 485 (58%) |
| Test 40 stocks | 224/455 (49%) | 69 (15%) | 253 (56%) |
| New 15 ETFs | 97/143 (68%) | 54 (38%) | 107 (75%) |

A green BOTTOM is now rare but reliable: about 1 in 5 keeps falling, vs 2 in 5 before. Grey labels keep coverage of the remaining major lows, but they carry the old risk.

## Caveats

- 26-week outcomes on weekly charts. These are statistical tendencies, not guarantees. Even a green signal fails roughly 1 time in 5.
- Single stocks remain much harder than ETFs and indices.
- Green signals are concentrated in years when the market was healthy. In a 2008- or 2022-style bear market, most signals will be grey until SPY confirms its own bottom.

## Final re-run of the same 100 charts on TradingView (v2.1)

The saved v2.1 script was run on all 100 charts in TradingView, and its own signals were read back.

- **Learnings implemented correctly.** The live signals match the revised logic on 1,823 of 1,844 signals (98.9%). The differences are the known 1990s history gaps on OXY and HUM.

| TradingView v2.1 signals | Count | Fell >20% after | Distance from low | Median further drop | Higher 26w later | At a real low |
|---|---|---|---|---|---|---|
| 85 stocks, green BOTTOM | 210 | **20.0%** | **17.7%** | −9.4% | 69% | 86% |
| 85 stocks, grey BOTTOM? | 760 | 40.5% | 26.2% | −15.6% | 62% | 84% |
| 85 stocks, green DIP | 603 | 27.0% | 18.2% | −11.7% | 62% | — |
| 85 stocks, naive dip-buy | 9,203 weeks | 40.7% | 23.2% | −15.8% | 61% | — |
| 15 ETFs, green BOTTOM | 89 | **9.0%** | **13.7%** | −6.4% | 81% | 89% |
| 15 ETFs, green DIP | 111 | 10.8% | 10.2% | −7.5% | 65% | — |

### Does it trigger at the bottom? The big market lows across the 100 charts

| Market low | Charts down 15%+ | BOTTOM within 8 weeks of that chart's low | Typical timing |
|---|---|---|---|
| Q4 2018 | 85 | 35 (24 green, 11 grey) | 1 week after the low, 11% above it |
| COVID 2020 | 94 | 67 (32 green, 35 grey) | 2 weeks after, 36% above (V-bounce) |
| Tariff low 2025 | 95 | 44 (24 green, 20 grey) | 2 weeks after, 16% above |
| 2008–09 crash | 73 | 48 (1 green, 47 grey) | 1 week after |
| 2022 bear market | 98 | 34 (0 green, 34 grey) | 1 week after, 14% above |

Green signals catch sharp corrections in otherwise healthy markets. At deep bear-market bottoms (2009, 2022) the signal still fires on time, but it is graded grey, because every stock is far below its averages by then.

**Tested and rejected:** letting a confirmed SPY bottom turn those grey signals green. It flags about 40% more major lows in green, but green failures rise on every set (unseen stocks: 21% → 28%), so it was not adopted. In a deep bear market, treat a grey BOTTOM? that comes right after SPY's own BOTTOM as the higher-quality grey.
