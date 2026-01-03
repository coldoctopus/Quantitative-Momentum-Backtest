# Backtest Metrics Explained

This document explains the key performance metrics computed in `backtest.ipynb` and `backtest(no shift).ipynb`.

> **Conventions used below**
>
> - Let \(P_t\) be the closing price on day \(t\).
> - Let \(r_t\) be the daily simple return on day \(t\).
> - Let \(w_t\) be the strategy exposure (also called weight/position):
>   - \(w_t = 1\): fully invested (long) - buy in
>   - \(w_t = 0\): in cash - sell
> - In the notebook, `Signal` is computed from today’s close, but **`Position` is shifted by 1 day** to avoid look-ahead bias.

---

## 1) Market Return (`Market_Return`)

### Purpose
This is the baseline **buy & hold daily return** of the asset. It’s used to:
- measure what the market did each day
- build the buy & hold equity curve
- serve as the raw return stream that the strategy “earns” while it is invested

### Implementation
```python
df['Market_Return'] = df['Close'].pct_change()
```

### Formula
\[
 r_t = \frac{P_t}{P_{t-1}} - 1
\]

### Meaning
- If `Market_Return = 0.02`, the asset gained **+2%** that day.
- If `Market_Return = -0.01`, the asset lost **-1%** that day.
- These values are **decimal returns** and constrained to \([0, 1]\).

---

## 2) Trading Signal (`Signal`) and Position (`Position`)

### Purpose
These columns define **when you are in the market**.

- `Signal` is the “decision rule” output: invest when the fast moving average (SMA20) is above the slow moving average (SMA50).
- `Position` is the **tradable exposure** used for returns. It is shifted one day so you only take the trade **after** the signal exists.

### Implementation
```python
# Signal: 1 when SMA_20 > SMA_50 else 0
df['Signal'] = np.where(df['SMA_20'] > df['SMA_50'], 1.0, 0.0)

# Position: shift by 1 day to avoid look-ahead bias
df['Position'] = df['Signal'].shift(1)
```

**Why shift?**
If you compute the signal from the close of day \(t\), you can’t realistically enter at that same close without assuming perfect execution at the close. Shifting approximates: *signal at \(t\) => trade at \(t+1\).* This reduces **look-ahead bias**.

---

## 3) Strategy Return (`Strategy_Return`)

### Purpose
This is the daily return stream of the strategy. It’s used to:
- build the strategy equity curve
- compute Sharpe Ratio
- compute drawdowns (via the equity curve)

### Implementation
```python
df['Strategy_Return'] = df['Position'] * df['Market_Return']
```

### Formula
\[
 r^{\text{strat}}_t = w_t \cdot r_t
\]

### Meaning
This is a **gated return**:
- If you are invested (\(w_t=1\)), you earn the market return that day (\(r^{\text{strat}}_t = r_t\)).
- If you are in cash (\(w_t=0\)), you earn 0 that day.

#### Common confusion: “Don’t we also profit when we sell higher?”
The profit from “selling higher” is *already captured* by summing/compounding the positive daily returns **while you were holding**.

Example: price 100 → 110 → 120, then you exit.
- Daily returns are +10% and +9.09%
- Compounded: \((1+0.10)(1+0.0909) = 1.20\) → +20%, which matches 100 → 120.

There is no extra “sell bonus”; the gain is the accumulated holding-period returns.

---

## 4) Cumulative Returns / Equity Curve

The notebook builds two equity curves:

### 4.1 Buy & Hold cumulative return (`Cumulative_Market_Return`)

#### What it is used for
This is the **benchmark**. It answers: *“If I just bought on day 1 and held, how would my wealth grow?”*

#### Notebook implementation
```python
df['Cumulative_Market_Return'] = (1 + df['Market_Return']).cumprod()
```

#### Formula
\[
 \text{Equity}^{\text{mkt}}_t = \prod_{i=1}^{t}(1 + r_i)
\]

#### Meaning
- Starts near 1.0.
- If it ends at 2.50, that means roughly **2.5×** growth in capital over the period.

### 4.2 Strategy cumulative return (`Cumulative_Strategy_Return`)

#### What it is used for
This is the strategy’s **equity curve** (wealth index). It is used for:
- visually comparing strategy vs benchmark
- drawdown calculations

#### Notebook implementation
```python
df['Cumulative_Strategy_Return'] = (1 + df['Strategy_Return']).cumprod()
```

#### Formula
\[
 \text{Equity}^{\text{strat}}_t = \prod_{i=1}^{t}(1 + r^{\text{strat}}_i)
\]

---

## 5) Total Return (as printed)

### Meaning
The value is a **wealth multiple**, not a percent.
- `1.00x` means you broke even.
- `1.50x` means +50% total gain.
- `0.80x` means -20% total loss.

---

## 6) Sharpe Ratio (`sharpe_ratio`)

### Purpose
The Sharpe Ratio measures **risk-adjusted return**: how much average return you get per unit of volatility.

It’s commonly used to:
- compare strategies with different risk levels
- evaluate whether higher returns are “worth” higher volatility

### Implementation
```python
sharpe_ratio = (df['Strategy_Return'].mean() / df['Strategy_Return'].std()) * np.sqrt(252)
```

### Formula
This notebook uses an annualized Sharpe with **0 risk-free rate**:
\[
\text{Sharpe} = \frac{\mu}{\sigma} \sqrt{252}
\]
where:
- \(\mu\) is the mean of daily strategy returns
- \(\sigma\) is the standard deviation of daily strategy returns
- 252 is the typical number of trading days per year

### Meaning
- Higher is better (in general).
- Rough mental guide (very context-dependent):
  - \(< 0\): strategy loses on average
  - \(0 \text{ to } 1\): weak / noisy edge
  - \(> 1\): decent
  - \(> 2\): very strong (often hard to sustain in reality)

---

## 7) Drawdown and Maximum Drawdown (`max_drawdown`)

### Purpose
Drawdown quantifies **how bad the declines are** from peaks (pain level).

Maximum drawdown is mainly used to:
- understand worst-case historical loss
- compare downside risk between strategies
- sanity-check whether a strategy is investable for a given risk tolerance

### Implementation
```python
running_max = df['Cumulative_Strategy_Return'].cummax()
drawdown = (df['Cumulative_Strategy_Return'] / running_max) - 1
max_drawdown = drawdown.min()
```

### Formula
Let \(E_t\) be the strategy equity curve.
\[
\text{Peak}_t = \max_{i \le t} E_i
\]
\[
\text{Drawdown}_t = \frac{E_t}{\text{Peak}_t} - 1
\]
\[
\text{Max Drawdown} = \min_t (\text{Drawdown}_t)
\]

### Meaning
- Drawdown is **0** at new highs.
- Drawdown is negative during declines.
- If `max_drawdown = -0.35`, the worst peak-to-trough loss was **-35%**.

---


