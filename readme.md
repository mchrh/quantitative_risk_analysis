# stETH/ETH Historical Basis & VaR Analysis

This project retrieves historical price data for ETH and stETH from Dune to calculate and visualize the 14-day 99% Value-at-Risk (VaR) of the stETH/ETH peg ratio.

## 1. Data Source
The data is fetched using the following Dune SQL query using the `prices.day` table.

**Dune Query Link:** [https://dune.com/queries/6308595](https://dune.com/queries/6308595)

```sql
SELECT
    date_trunc('day', timestamp) AS time,
    AVG(CASE WHEN contract_address = 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2 
        THEN price END) AS eth_price,
    AVG(CASE WHEN contract_address = 0xae7ab96520de3a18e5e111b5eaab095312d7fe84 
        THEN price END) AS steth_price,
    AVG(CASE WHEN contract_address = 0xae7ab96520de3a18e5e111b5eaab095312d7fe84 
        THEN price END) / 
    NULLIF(AVG(CASE WHEN contract_address = 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2 
        THEN price END), 0) AS steth_eth_ratio
FROM prices.day
WHERE blockchain = 'ethereum'
  AND contract_address IN (
      0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2, -- WETH
      0xae7ab96520de3a18e5e111b5eaab095312d7fe84  -- stETH
  )
  AND timestamp >= CAST('2021-01-01' AS TIMESTAMP)
GROUP BY 1
ORDER BY 1 DESC
```

## 2. Setup & Execution (using uv)

This project uses uv as a package manager.

### Prerequisites
* Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
* A Dune  API Key.

### Installation
1. Create a virtual environment:
   ```bash
   uv venv
   ```

2. Activate the environment:
   * **macOS/Linux:** `source .venv/bin/activate`
   * **Windows:** `.venv\Scripts\activate`

3. Install dependencies:
   ```bash
   uv pip install -r requirements.txt
   ```

### Running the Analysis
1. Set your API key as an environment variable (optional, or paste into notebook):
   ```bash
   export DUNE_API_KEY="your_api_key_here"
   ```
2. Launch the notebook:
   ```bash
   uv run jupyter notebook
   ```
3. Run the code in the notebook. 

## 3. Analysis Commentary

The graph in the notebook highlights the sticky nature of historical vol, where the 720d lookback window forces the risk model to "remember" the 2022 depegging event long after the market stabilized. 
This creates a visible lag between market recovery and model recovery, resulting in a step-function drop in risk around mid-2024 only once those volatile data points are finally out of the rolling window. As a consequence, capital efficiency was artificially suppressed for nearly two years, as the model demanded high collateral haircuts despite the stETH/ETH ratio returning to parity much earlier. Currently, the model reflects a more stable regime with minimal implied risk, suggesting LPs and MMs can now operate with significantly tighter spreads and lower capital buffers.