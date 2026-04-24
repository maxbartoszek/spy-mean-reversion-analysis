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
| 10%+ Gain | +0.58% | 52.2% | +1.47% | 55.1% |
| 10%+ Drop | +0.81% | 56.7% | +2.00% | 58.9% |

After a 10%+ drop, stocks went up **58.9% of the time** over the following week, with a mean return of 2.00%. That fact was my core finding.

---

## Ruling Out Sectors

My first instinct was that this was being driven by a specific sector. I tested Financials, Energy, Industrials, Consumer Staples, Healthcare, and Technology, looking for the group that was pulling the aggregate number so close to 60%. I even tested other indexes like QQQ and VTI. I couldn't isolate a single clear culprit. The effect appears to be broadly based across the market rather than concentrated in one sector, which actually strengthens the case that it's a market-wide behavioural phenomenon rather than a sector-specific issue.

By the way, if anyone wants something cool to look into, I found that of the Industrial stocks in the S&P500, S&P600 small caps, and S&P400 mid caps, there were 238 large gains in the past year. Of these, 57.1% went up over the next week with a **4.28% average return**. This momentum is likely what I'll be looking into next, honestly.

---

## Statistical Validation

Before treating this as a tradeable signal, I needed to confirm it wasn't just luck. I ran a binomial test against a null hypothesis of 50%:

```
n = 231 drop events
Up = 136 (58.9%)
p-value = 0.0084  
95% CI = [0.522, 0.653]
```

The p-value of 0.0084 is well below the standard 0.05 significance threshold used by actual scientists, meaning there's only a 0.84% chance this result occurred by random chance. The entire confidence interval sits above 50%, confirming the edge is real even in a conservative case.

I also controlled for broad market movements by subtracting SPY's return over the same 5-day window from each trade's return, to check whether the signal was simply a result of beta exposure to general market recoveries:

```
Mean raw return: 2.00%
Mean SPY (same window): 0.54%
Mean excess return: 1.46%
```

The excess return surviving SPY adjustment confirms there's genuine alpha beyond just riding broad market bounces.

---

## Strategy & Sizing

Using the statistical results, I modelled a simple rules-based strategy:

- **Entry:** Buy at the close on any day a S&P 500 stock drops 10%+
- **Exit:** Sell at the close 5 trading days later, no exceptions
- **Sizing:** Half Kelly Criterion based on observed win/loss profile

**Trade metrics:**

```
Win rate: 58.9%
Avg win: +6.77%
Avg loss: -4.81%
Largest gain: +39.41%
Largest loss: -19.55%
Expected value / trade: 2.00%
Half Kelly sizing (recommended): 3.1% of portfolio per trade
Expected Yearly Gain on Strategy: 15.3%
```

At a 3.1% position size, you can hold up to 33 concurrent positions, which means you should be able to take every position the signal provides, even with multiple 10% drops within a single week. The projected 15.3% annual return assumes every qualifying trade is taken and does not account for transaction costs, slippage, or bid-ask spread, so real-world returns would be modestly lower, likely closer to 12-13%, but a little higher when using zero-commission brokers like Wealthsimple.

---

## Further Reflection

How much you make from using this signal really depends on your position sizing. This last year has been crazy, and the market's moved similarly to a rocketship (+30%). Apart from the drop in March, it has pretty much only gone up. As a result, if your portfolio is held in cash most of the time (like with the half Kelly sizing), your return will not be able to match the S&P. In a different backtest, I modelled the strategy with a 20% position sizing for each trade. It returned 75.7% with a 2.17 Sharpe. 

The reason I used (and recommend using) the half Kelly sizing was to ward against sudden major geopolitical events like the 2020 COVID crash that could wipe out the portfolio. If one were to seek out the gains provided from using a 20% position sizing, I would highly recommend not putting more than 60-70% of your portfolio in the market at any given time. I'd also use a sector cap, so that if 3 different banks fell 10% in one day, I'd only buy one of them.  

If you used the full Kelly sizing (6.2%), which tends to determine the best investment size to maximize long-term growth, you would have returned 32.1% and a 1.94 Sharpe, barely beating out the S&P 500. Your returns entirely depend on how much risk you'd like to take on and how much longer you think the current investing environment will remain. 

---

## Caveats

- This is based on one year of data during an unusually volatile investing environment. This signal may (and likely will) weaken if volatility normalizes, so watch VIX before trading (maybe only enter when VIX > 16?)
- Some portion of the edge is market beta
- No transaction costs are modelled
- Past performance of a 1-year backtest is not a reliable predictor of future returns (this is a signal based entirely on the current trading environment we are in, which may not be well represented in the future)
- Returns will heavily depend on position sizing 

---

## Stack

- Python, Jupyter Notebook
- `yfinance` — price data
- `pandas`, `numpy` — data processing
- `scipy.stats` — statistical testing
- `matplotlib` — visualizations
