
# Pairs Trading Cointegration Analyzer

This Jupyter Notebook performs statistical analysis to identify potentially cointegrated pairs of financial assets for pairs trading strategies. It downloads historical price data using `yfinance`, checks for stationarity using the Augmented Dickey-Fuller (ADF) test, calculates correlation, and performs the Engle-Granger cointegration test. If cointegration is found, it also plots the spread and its Z-score.

## Features

*   Downloads historical price data for specified asset pairs.
*   Handles different asset types: stocks ('generic'), forex ('forex'), and futures ('futures').
*   Performs Augmented Dickey-Fuller (ADF) tests for stationarity on individual series and the resulting spread.
*   Calculates and visualizes the correlation matrix.
*   Conducts the Engle-Granger two-step cointegration test.
*   Estimates the hedge ratio for cointegrated pairs.
*   Plots:
    *   Normalized prices of the two assets.
    *   The spread (residuals) of the cointegrating relationship.
    *   The Z-score of the spread with +/-1 and +/-2 standard deviation thresholds.
*   Outputs a summary table of analyzed pairs, their correlation, cointegration p-value, and whether they are cointegrated.

## Requirements

*   Python 3.x
*   Jupyter Notebook or JupyterLab
*   The following Python libraries:
    *   `yfinance`
    *   `pandas`
    *   `numpy`
    *   `statsmodels`
    *   `matplotlib`
    *   `seaborn`

You can install these libraries using pip:

```bash
pip install yfinance pandas numpy statsmodels matplotlib seaborn
```

## How to Use

1.  **Clone or download the repository/notebook.**
2.  **Open the Jupyter Notebook** (e.g., `pairs_analyzer.ipynb`).
3.  **Modify Configuration Settings:** The primary section for user input is near the end of the notebook. See the "Configuration" section below for details.
4.  **Run the cells** in the notebook sequentially.

## Configuration - Where to Change Values

Locate the following sections in the Python script (within the Jupyter Notebook) to customize the analysis:

### 1. Significance Level (`alpha`)

This determines the threshold for statistical tests (ADF and Cointegration). A common value is 0.05.

```python
# --- Configuration ---
alpha = 0.05 # Significance level for tests
```

### 2. Candidate Pairs (`candidate_pairs`)

This is a list of asset pairs you want to analyze. Each pair is a tuple containing:
*   `symbol1`: The ticker symbol for the first asset.
*   `symbol2`: The ticker symbol for the second asset.
*   `asset_type`: A string indicating the type of asset. Use:
    *   `'futures'` for futures contracts (e.g., "ES", "CL"). The script will append "=F".
    *   `'forex'` for forex pairs (e.g., "EURUSD", "AUDJPY"). The script will append "=X".
    *   `'stocks'` or `'generic'` for stocks (e.g., "AAPL", "MSFT"). No suffix is appended by default for these.

**Example (inside the notebook):**

```python
candidate_pairs = [
    ("ES", "NQ", 'futures'),  # E-mini S&P 500 vs E-mini NASDAQ 100
    ("GC", "SI", 'futures'),  # Gold vs Silver
    ("AAPL", "MSFT", 'stocks'), # Apple vs Microsoft
    ("EURUSD", "GBPUSD", 'forex'), # EUR/USD vs GBP/USD
    # Add more pairs here
]
```
**Modify this list to include the pairs you are interested in.** Ensure the symbols are compatible with Yahoo Finance. For futures, use the root symbol (e.g., "ES", not "ESZ23").

### 3. Timeframe and Period (`current_interval`, `current_start`, `current_end`)

This section defines the data frequency (interval) and the historical period for data download.

```python
# --- CHOOSE YOUR TIMEFRAME AND PERIODS ---

# For 1-MINUTE analysis:
# yfinance provides 1m data for the last 7 calendar days.
# The script includes logic to automatically determine current_start and current_end
# for the most recent full trading day for 1m data.
# current_interval = '1m'
# current_start = current_start_dt_obj.strftime('%Y-%m-%d') # Uses dynamically calculated date
# current_end = current_end_dt_obj.strftime('%Y-%m-%d')   # Uses dynamically calculated date

# To override with specific dates for 1m (or other intervals):
# current_start = 'YYYY-MM-DD'
# current_end = 'YYYY-MM-DD' # For yfinance, end_date is exclusive for intraday.

# Example for DAILY analysis:
# current_interval = '1d'
# current_start = '2020-01-01'
# current_end = '2023-12-31'

# Example for HOURLY analysis:
# current_interval = '1h' # or '60m'
# current_start = (datetime.now() - timedelta(days=720)).strftime('%Y-%m-%d') # Approx. 2 years
# current_end = datetime.now().strftime('%Y-%m-%d')
```

*   **`current_interval`**: Data frequency. Common values:
    *   `'1m'` for 1-minute data.
    *   `'5m'`, `'15m'`, `'30m'`, `'60m'`, `'90m'`, `'1h'` for intraday data.
    *   `'1d'` for daily data.
    *   `'1wk'` for weekly data.
    *   `'1mo'` for monthly data.
*   **`current_start`**: The start date for data download in 'YYYY-MM-DD' format.
*   **`current_end`**: The end date for data download in 'YYYY-MM-DD' format.
    *   For daily, weekly, monthly data, `end_date` is inclusive.
    *   For intraday data (like '1m', '1h'), `end_date` is exclusive. So, to get all data for '2023-05-10', set `start='2023-05-10'` and `end='2023-05-11'`.

**Important Note on Your Provided Code's Date Setting for 1-Minute Data:**
The script you provided has the following lines for setting `current_start` and `current_end` for 1-minute data analysis:

```python
# ... dynamic date calculation for target_day_for_data ...

current_start_dt_obj = target_day_for_data
current_end_dt_obj = target_day_for_data + timedelta(days=1)

current_start = current_start_dt_obj.strftime('2025-05-05') # <-- HARDCODED!
current_end = current_end_dt_obj.strftime('2025-05-13')   # <-- HARDCODED!

current_interval = '1m'
print(f"USING 1-MINUTE DATA: Attempting to fetch for the full day of {current_start} (data from {current_start} up to, but not including, {current_end})")
```
**You MUST change the hardcoded dates `'2025-05-05'` and `'2025-05-13'`**.
To use the dynamically calculated recent trading day, change them to:
```python
current_start = current_start_dt_obj.strftime('%Y-%m-%d')
current_end = current_end_dt_obj.strftime('%Y-%m-%d')
```
Or, set your desired specific fixed dates.

### 4. Plot Verbosity (`verbose_plots` in the main loop)

Inside the `if __name__ == "__main__":` block, when `analyze_asset_relationship` is called, you can control whether plots are displayed for each pair.

```python
if __name__ == "__main__":
    for s1, s2, asset_cat in candidate_pairs:
        # ...
        corr, p_val, is_coint = analyze_asset_relationship(
            # ... other arguments ...
            verbose_plots=True # Set to False to quickly screen many pairs without plots
        )
        # ... rest of the loop
```
Set `verbose_plots=True` to see all charts, or `verbose_plots=False` to suppress them (useful when screening many pairs and only interested in the summary table).

## Interpreting Results

*   **Stationarity (ADF Test):**
    *   A low p-value (typically <= `alpha`) suggests the series is stationary.
    *   For cointegration, we generally look for two non-stationary (I(1)) series whose linear combination (the spread) is stationary (I(0)).
*   **Correlation:** Measures the linear relationship between the two price series. High correlation is often a prerequisite for cointegration but does not guarantee it.
*   **Cointegration (Engle-Granger Test):**
    *   A low p-value (typically <= `alpha`) suggests the series are cointegrated.
    *   If cointegrated, the script checks if the spread (residuals from regressing one series on the other) is stationary. A stationary spread is key for a mean-reverting pairs trading strategy.
*   **Z-Score Plot:** Visualizes how many standard deviations the current spread is from its historical mean. Trading signals are often generated when the Z-score crosses certain thresholds (e.g., +/- 2.0).

