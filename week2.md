# Week 2 — Bitcoin Transaction Fee Forecasting

Our goal this week is to prepare the daily Bitcoin dataset and answer one question: **if we use yesterday’s average fee to predict today’s fee, how far off are we on average?** This baseline will give us a score to beat in Week 3.

[Download the daily Bitcoin data from Coin Metrics (2020–2024)](https://community-api.coinmetrics.io/v4/timeseries/asset-metrics?assets=btc&metrics=FeeTotNtv,TxCnt&start_time=2020-01-01&end_time=2024-12-31&frequency=1d&page_size=2000&format=csv)

## Niels — Data preparation and exploration

- [ ] Load the CSV and sort it by date. Check for missing or duplicate dates, missing fees and days with zero transactions. The raw file should contain 1,827 daily observations.
- [ ] Calculate the average fee in satoshis per transaction using `100_000_000 * FeeTotNtv / TxCnt`. Flag any day for which this calculation is impossible.
- [ ] Build one row per `target_date`. Save that day’s observed fee as `actual_fee_sat`. Use the fees and transaction counts from the previous seven calendar days as the 14 inputs. Name the previous day’s fee `fee_lag_1_sat`.
- [ ] Assign each row to a split using its target date. Exclude rows without seven complete days of history and record how many were removed.
- [ ] On the training period, plot the daily average fee, report its median and maximum, and describe how transaction counts varied around one fee spike.

**Files to commit**
- `notebooks/01_prepare_data.ipynb`
- `data/processed/bitcoin_daily.csv`

## Anselme — Baseline and first results

- [ ] Implement the baseline using `prediction_sat = fee_lag_1_sat`. It requires no training.
- [ ] Check the calculation by hand. If Sunday’s average fee was 8,000 satoshis, Monday’s prediction should be 8,000. If Monday’s actual fee was 7,500, the absolute error should be 500 satoshis per transaction.
- [ ] Evaluate the baseline on the 2023 validation dates using `mean_absolute_error(actual_fee_sat, prediction_sat)`. Report the number of days evaluated and the MAE in satoshis per transaction.
- [ ] Plot the predicted and observed fees on the same chart. Identify the date with the largest absolute error, give both values and explain what changed from the previous day.

**File to commit**
- `notebooks/02_baseline.ipynb`, including the score, plot and short interpretation.

## Chronological split

| Split | Target dates | Purpose |
| --- | --- | --- |
| Training | 2020–2022 | Explore the data and prepare inputs for the future model |
| Validation | 2023 | Measure baseline performance in Week 2 |
| Final test | 2024 | Reserve for the final model comparison |

For example, the forecast for **1 January 2023** belongs to validation. Its seven input days run from **25 to 31 December 2022**, and its baseline prediction is the observed fee on **31 December**.

## Final check — both teammates

- [ ] Run both notebooks in order and confirm that they reproduce the clean CSV and the same validation MAE.
- [ ] Check that every forecast uses only information available before its target date.
- [ ] Confirm that the clean dataset, baseline code and first validation results are all visible in the repository.
