# Quantitative Momentum Backtest

A demonstrative repository implementing a **Dual Moving Average Crossover** strategy for trend-following analysis. This project serves as a "Hello World" introduction to quantitative trading and algorithmic backtesting.

## Purpose

This project provides a **vectorized backtesting framework** to:
- Download historical stock data (NVDA, FPT, etc.) using `yfinance`
- Implement a classic **Moving Average Crossover** trading strategy
- Calculate risk-adjusted returns using the **Sharpe Ratio**
- Visualize performance metrics including:
  - Cumulative returns vs. Buy & Hold benchmark
  - Maximum Drawdown analysis
- Demonstrate core quantitative finance concepts in a beginner-friendly manner

## Getting Started

### Prerequisites

- Python 3.14 or higher
- Git (to clone the repository)

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/Quantitative-Momentum-Backtest.git
   cd Quantitative-Momentum-Backtest
   ```

2. **Create a virtual environment**

   **On Windows:**
   ```bash
   python -m venv venv
   venv\Scripts\activate
   ```

   **On macOS/Linux:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. **Install required libraries**
   ```bash
   pip install -r requirements.txt
   ```

## Required Libraries

The project uses the following Python libraries:
- `pandas` - Data manipulation and analysis
- `numpy` - Numerical computing
- `matplotlib` - Data visualization
- `yfinance` - Download historical stock data from Yahoo Finance

All dependencies are listed in [requirements.txt](requirements.txt).

## Usage

(Instructions will be added once the backtester script is implemented)

```bash
python backtest.py
```

## Features

- **Vectorized Backtesting**: Efficient computation using pandas vectorization
- **Dual Moving Average Crossover**: Configurable short-term and long-term moving averages
- **Performance Metrics**:
  - Total Returns
  - Sharpe Ratio (risk-adjusted returns)
  - Maximum Drawdown
  - Win Rate
- **Visualization**: Publication-quality charts comparing strategy vs. benchmark

## Strategy Overview

The **Moving Average Crossover** strategy generates trading signals based on:
- **Buy Signal**: When short-term MA crosses above long-term MA (Golden Cross)
- **Sell Signal**: When short-term MA crosses below long-term MA (Death Cross)

This is compared against a simple **Buy & Hold** benchmark to evaluate strategy effectiveness.

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the issues page.

## 📝 License

This project is for educational purposes.

