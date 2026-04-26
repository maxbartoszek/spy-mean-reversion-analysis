# S&P 500 Mean Reversion Strategy on Large Stock Movements

## Background

I've been investing and trading for a while now, and something I started noticing this year was that there seemed to be a lot more significant single-day moves of 10%+ than I'd ever seen before. The markets feel *different*. It's like they're more reactive with stocks swinging wildly on news that in previous years might have caused a 4-5% move at most.

What really caught my attention, though, was what happened *after* these big drops. It seemed like stocks that fell 10%+ in a single day had a habit of bouncing back over the following week. In the past, if a stock grew/fell 10% in a day, it would have no effect on its future growth. The probability of it going up/down would be 50% with some added noise. I couldn't explain exactly why, but it really felt like some pattern was emerging.

So I decided to actually test it.

---

## The Hypothesis

Historically, a 10%+ single-day move in a stock should carry no meaningful predictive signal for the following week. The efficient market hypothesis would suggest it's a coin flip either way. 50% chance of going up, 50% chance of going down. My hypothesis was that this was no longer holding true in the current market environment. I believed this was driven by elevated volatility and a surge in retail investors, causing emotional overreactions that then get corrected very quickly.

---

## What I Did

I pulled all the stocks in the S&P 500 and downloaded one year of daily price data using `yfinance`. For every stock that moved 10% or more in a single day, I recorded the return over the next day and the following 5 trading days (one week). I then separated these events into gains and drops and analyzed them independently.

**Results:**

| Event | Next Day (avg) | Next Day (up %) | Next Week (avg) | Next Week (up %) |
|---|---|---|---|---|
| 10%+ Gain | +0.58% | 52.2% | +1.46% | 54.9% |
| 10%+ Drop | +0.83% | 56.8% | +1.99% | 58.5% |

After a 10%+ drop, stocks went up **58.5% of the time** over the following week, with a mean return of 1.99%. That fact was my core finding.

---

## Ruling Out Sectors

My first instinct was that this was being driven by a specific sector. I tested Financials, Energy, Industrials, Consumer Staples, Healthcare, and Technology, looking for the group that was pulling the aggregate number so close to 60%. I even tested other indexes like QQQ and VTI. I couldn't isolate a single clear culprit. The effect appears to be broadly based across the market rather than concentrated in one sector, which actually strengthens the case that it's a market-wide behavioural phenomenon rather than a sector-specific issue.

On a side note, if anyone wants something cool to look into, of the Industrial stocks across the S&P 500, 400, and 600, there were 238 large gains in the past year. Of these, 57.1% went up over the next week with a **4.28%** average return. This momentum effect is likely what I'll be researching next.

---

## Statistical Validation

Before treating this as a tradeable signal, I needed to confirm it wasn't just luck. I ran a binomial test against a null hypothesis of 50%:

```
n = 231 drop events
Up = 136 (58.9%)
p-value = 0.0084  
95% CI = [0.522, 0.653]
```

The p-value of 0.0084 is well below the standard 0.05 significance threshold commonly used by scientists, meaning there's only a 0.84% chance this result occurred by random chance. The entire confidence interval sits above 50%, confirming the edge is real even in a conservative case.

I also controlled for broad market movements by subtracting SPY's return over the same 5-day window from each trade's return, to check whether the signal was simply a result of beta exposure to general market recoveries:

```
Mean raw return: 2.00%
Mean SPY (same window): 0.54%
Mean excess return: 1.46%
```

The excess return surviving SPY adjustment suggests there's genuine alpha beyond just riding broad market bounces.

---

## Robustness Validation over Multiple Periods

To validate that the signal wasn't specific to one lucky window, I tested it across four different lookback periods:

| Period | Events | Win Rate | EV/Trade | Excess vs SPY | p-value (binomial) | p-value (t-test) |
|---|---|---|---|---|---|---|
| 3 months | 88 | 63.6% | +3.42% | +3.22% | 0.0138 | 0.0001 |
| 6 months | 135 | 58.5% | +2.51% | +2.34% | 0.0579 | 0.0009 |
| 1 year | 231 | 58.9% | +1.98% | +1.41% | 0.0084 | 0.0001 |
| 1000 days | 657 | 56.9% | +1.53% | +1.11% | 0.0004 | <0.0001 |

The signal held across all four windows. Win rate stayed in the 57-64% range regardless of the period. The 6-month p-value of 0.0579 slightly exceeded the 0.05 threshold due to the smaller sample size, not necessarily because the underlying edge weakened. The p-value of the 3-month backtest sat below 0.02 and had an even smaller sample size, supporting the idea that the edge is growing. 

The t-test confirms that the magnitude of returns is statistically significant across every single period tested, including the 6-month window where the binomial test slightly missed the 0.05 threshold. Together, the two tests validate both aspects of the signal independently. Stocks go up more often than chance after a 10%+ drop, and the average return itself is meaningfully above zero. Both findings hold across all four time periods.

Moreover, the excess return over SPY has been increasing recently, adding to the idea that the signal is strengthening rather than fading. The 3.22% excess return over SPY for the nearest term period is particularly significant. In the current environment, the return is nearly independent of beta. SPY only returned 0.20% over those same windows while this strategy averaged 3.42%. The signal is genuinely separating from broad market returns right now.

---

## Strategy & Sizing

Using the statistical results, I modelled a simple rules-based strategy:

- **Entry:** Buy at the close on any day a S&P 500 stock drops 10%+
- **Exit:** Sell at the close 5 trading days later, no exceptions
- **Sizing:** Half Kelly Criterion based on the observed win/loss profile

**Trade metrics:**

```
Win rate: 58.5%
Avg win: +6.81%
Avg loss: -4.81%
Largest gain: +39.41%
Largest loss: -19.55%
Expected value / trade: 1.99%
Half Kelly sizing (recommended): 3.0% of portfolio per trade
Expected Yearly Gain on Strategy: 14.8%
```

At a 3.0% position size, you can hold up to 33 concurrent positions, which means you should be able to take every position the signal provides, even with many 10% drops within a single week. The projected 14.8% annual return assumes every qualifying trade is taken and does not account for transaction costs, slippage, or bid-ask spread, so real-world returns would be modestly lower, likely closer to 12-13%, but a little higher when using zero-commission brokers.

---

## Further Reflection

How much you make from using this signal really depends on your position sizing. This last year has been crazy, and the market's moved similarly to a rocketship (+30%). Apart from the drop in March, it has only gone up. As a result, if your portfolio is held in cash most of the time (like with the half Kelly sizing), your return will not be able to match the S&P. In a quick backtest generated by Claude (provided), I modelled the strategy with an arbitrarily chosen 20% position sizing for each trade. It returned 65.3% with a 1.93 Sharpe. 

The reason I used (and recommend using) the half Kelly sizing was to ward against sudden major geopolitical events like the 2020 COVID crash that could wipe out the portfolio. If one were to seek out the gains provided from using a 20% position sizing, I would highly recommend keeping 30-40% of your portfolio in cash at all times and using a sector cap.  

Using the full Kelly sizing (6.0%), which tends to determine the best investment size to maximize long-term growth, the Claude-generated backtest returned 31.5% with a 1.91 Sharpe and beat the S&P 500. 

Your returns entirely depend on how much risk you're willing to take on and how much longer you think the current investing environment will remain. 

---

## Caveats

- This is based on data derived from a period enduring an unusually volatile investing environment.
- This signal appears tied to the current retail-driven market environment. Be sure to monitor VIX and reassess if the current market structure changes significantly.
- Some portion of the edge is market beta.
- No transaction costs are modelled.
- Past performance of a 1-year backtest is not a reliable predictor of future returns (this is a signal based entirely on the current trading environment we are in, which may not be well represented in the future).
- Returns will heavily depend on position sizing.

---

## Stack

- Python, Jupyter Notebook
- `yfinance` - price data
- `pandas`, `numpy` - data processing
- `scipy.stats` - statistical testing
- `matplotlib` - visualizations
- Claude - providing a quick backtest
