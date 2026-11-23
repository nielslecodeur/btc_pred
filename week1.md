# Forecasting Next-Day Bitcoin Transaction Fees

**Niels Hallgren & Anselme Audinos**
*Machine Learning – Project Proposal, Week 1*

## 1. Context, Research Question and Objectives

**A concrete use case.** Sending the same amount of bitcoin can cost more on some days than on others. When many users compete for limited space in a block, those who want a faster confirmation may offer higher fees. A Bitcoin wallet could use a fee forecast to help users decide whether to send a payment today or wait until tomorrow.

We approach this as a regression problem. At the end of each day, we use recent transaction counts and fees to predict the next day’s average fee per transaction across the Bitcoin network, measured in satoshis. Our research question is whether these data can provide a more accurate forecast than simply using today’s average fee.

**Research question.** *Can recent transaction fees and network activity predict tomorrow’s average Bitcoin transaction fee more accurately than using today’s average fee?*

**Problem type.** This is a regression problem. Each evening, we want to predict the average fee paid per transaction across the whole Bitcoin network on the following day, expressed in satoshis (1 BTC = 10<sup>8</sup> satoshis). The target is a continuous, non-negative number, not a class label.

### Objectives

- Build a clean daily series of the network-wide average fee per transaction, in satoshis, from 2020 to 2024.
- Implement a simple baseline (Week 2) and a linear regression that uses the last 7 days of fees and transaction counts.
- Evaluate both on data the model has never seen, using a chronological split, and decide whether the extra complexity is justified.
- Pay particular attention to sudden fee spikes, which are exactly the moments when a warning would matter most to a wallet user.

**How we will know the model is useful.** The model is useful only if it makes a smaller average error than the “same as today” baseline on the final 2024 test period. If it does not, that is also a valid finding: it would show that the previous day’s fee is already hard to beat.

## 2. Dataset and Exploratory Analysis

**Source.** We use public daily data on Bitcoin from [Coin Metrics (Community API)](https://community-api.coinmetrics.io/v4/timeseries/asset-metrics?assets=btc&metrics=FeeTotNtv,TxCnt&start_time=2020-01-01&end_time=2024-12-31&frequency=1d&page_size=2000&format=csv), covering 1 January 2020 to 31 December 2024: **1,827 daily observations**. Two columns are needed:

- **FeeTotNtv**: total fees paid by all transactions of the day, in BTC.
- **TxCnt**: number of transactions of the day.

One bitcoin equals 100 million satoshis.

The average fee per transaction, in satoshis, is computed as:

$$
\text{fee}_t = \frac{10^8 \times \text{FeeTotNtv}_t}{\text{TxCnt}_t} \quad [\text{satoshis/tx}]
$$

*Figure 1. Average fee per transaction (satoshis), daily, 2020–2022.* (See the Week 1 PDF; the plot will be reproduced in `notebooks/01_prepare_data.ipynb`.)

### Observations

- Over 2020–2022, the median daily average fee is about **7,971 satoshis per transaction**, which gives the typical level of the series.
- The series is far from stable: it peaks at **112,514 satoshis on 21 April 2021**, roughly 14 times the median. A few extreme days dominate the picture, which will make large errors on spikes the main difficulty.
- This raises our guiding question: *how large is the error of a forecast based on the previous day when fees suddenly jump?*

**Data checks planned.** Verify that all 1,827 dates are present and unique, look for missing values or days with `TxCnt = 0` (which would make the ratio undefined), and keep every quantity in satoshis per transaction so that the plots, formulas and text use one unit.

## 3. Baseline, Model and Evaluation Strategy

**Baseline.** *Our baseline predicts that tomorrow’s average fee will equal today’s average fee.* For example, if the average fee is 8,000 satoshis today, the baseline predicts 8,000 satoshis for tomorrow. It needs no training and sets the bar to beat. It will be implemented in Week 2.

**Proposed model.** A linear regression using the average fee and the number of transactions of the last 7 days (14 input variables), all known at the time of the forecast:

$$
\widehat{\text{fee}}_{t+1} = \alpha + \sum_{k=0}^{6} \left( \beta_k \, \text{fee}_{t-k} + \gamma_k \, \text{TxCnt}_{t-k} \right)
$$

**Chronological split**, by the date being predicted: 2020–2022 to train, 2023 to choose settings, 2024 for the final test, used only once. Data is never shuffled, so the model never uses information from the future.

**Metric.** Methods are compared with the mean absolute error, in satoshis per transaction (lower is better):

$$
\text{MAE} = \frac{1}{n} \sum_t \left| \text{fee}_{t+1} - \widehat{\text{fee}}_{t+1} \right|
$$

### How we will use the Python functions

**Read and check the data.** We will open the Coin Metrics file with `read_csv()` and put the rows in date order with `sort_values()`. We will check that each row is one day after the previous one. A missing day would make the previous row a poor stand-in for yesterday. Once we have calculated the average fee per transaction, `plot()` will show us when fees rose sharply.

**Build a week of history.** Suppose we make a forecast on Sunday evening. In Sunday’s row, `shift(1)` gives us Saturday’s values and `shift(6)` gives us Monday’s. We will keep the fee and transaction count for each day from Monday to Sunday, giving us 14 inputs. Applying `shift(-1)` to the fee column puts the following Monday’s actual fee on that same row as the value we want to predict.

**Make a forecast.** With scikit-learn’s `LinearRegression`, `fit()` will use the training years to learn a weight for each input. Then `predict()` will combine the latest seven days of data with those weights to estimate tomorrow’s fee. We can compare that estimate with simply carrying today’s fee forward.

**Measure the error.** We will use `mean_absolute_error()` to compare each method’s predictions with the actual fees on the same dates. If the real fee is 7,500 and we predict 8,000, the error for that day is 500 satoshis per transaction. Predicting 7,000 gives the same error. Averaging these distances keeps high and low forecasts from cancelling each other out.

### Initial work plan

Niels will check the daily data and prepare the lagged variables. Anselme will implement the baseline and the linear regression. We will compare both methods and analyse errors on fee-spike days together.
