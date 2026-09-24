# Optimizing Stock Trading Strategy with K-Means Clustering

Big Data Analytics mini project — Department of Computer Engineering,
Atharva College of Engineering, University of Mumbai.

Companies are grouped by the **pattern of their daily price movement** rather
than by sector label. The algorithm is given nothing but the difference between
each day's closing and opening price, yet it reconstructs recognisable economic
sectors on its own.

## The idea

For every company and every trading day:

```
movement = Close − Open
```

This strips out the absolute share price — a stock at $900 and a stock at $12
become directly comparable. Each company becomes a 755-dimensional vector, one
value per trading day.

Each vector is then scaled to unit length with `Normalizer`. Because unit
vectors are compared by the angle between them, K-Means is effectively
clustering by **cosine similarity**: companies land together when their prices
rise and fall on the same days, no matter how large those moves are.

## Results

Clustering the full 755-dimensional vectors, K = 10, inertia 9.3806:

| Cluster | Companies | Reading |
|---|---|---|
| 3 | Northrop Grumman, Boeing, Lockheed Martin | Aerospace and defence |
| 2 | Chevron, Exxon | Integrated oil |
| 9 | Pepsi, Coca Cola, Johnson & Johnson | Consumer staples and healthcare |
| 1 | Apple, Intel, Texas Instruments | Technology hardware and semiconductors |
| 6 | Microsoft, MasterCard, Gen Digital, Amazon | Growth technology and payments |
| 8 | IBM, Paccar | Legacy industrials |
| 4 | GE, Sony, Mitsubishi UFJ, Honda, Toyota, American Express, Bank of America, Ford | Cyclicals, automobiles, financials |
| 0, 5, 7 | CVS Health · McDonalds · Valero Energy | No close counterpart in this universe |

The aerospace group and the oil majors are recovered **exactly**, with no sector
information supplied.

The practical consequence: holding Boeing, Lockheed Martin and Northrop Grumman
together is not diversification. They are one position wearing three names, and
the price data alone makes that visible.

A second pipeline adds PCA with two components. Those two components retain only
about 20% of the variance, so some groups break apart — Boeing separates from
the other two defence contractors. PCA earns its place for the cluster map, not
for the partition itself.

## Dataset

| | |
|---|---|
| Source | Yahoo Finance (`yfinance`) |
| Companies | 28 large-cap US-listed |
| Period | 1 Jan 2015 – 31 Dec 2017 |
| Trading days | 755 |
| Raw columns | 168 (28 tickers × 6 attributes) |
| Missing values | 0 |

A snapshot is committed at `data/stock_data_2015_2017.csv`, so the notebook runs
offline and reproduces the numbers above exactly. If the file is deleted, the
notebook re-downloads it.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook Stock_Optimization_KMeans_Clustering.ipynb
```

`random_state=42` throughout, so cluster ids are stable between runs.

## Repository layout

```
Stock_Optimization_KMeans_Clustering.ipynb   main notebook, outputs included
data/stock_data_2015_2017.csv                committed data snapshot
results.json                                 numbers quoted in the report
requirements.txt
```

## A note on the data source

The original version of this project used `pandas_datareader` with
`data_source='yahoo'`. Yahoo retired that endpoint and pandas-datareader has
since removed the reader, so the call now raises
`NotImplementedError: data_source='yahoo' is not implemented`. This version uses
`yfinance` with a committed CSV snapshot instead.

Five tickers from the original company list no longer resolve. Three are simple
renames whose full history is intact; two companies have left the public markets
and were replaced with the closest available substitute.

| Original | Now | Note |
|---|---|---|
| SNE | SONY | Renamed 2021 |
| SYMC | GEN | Symantec → NortonLifeLock → Gen Digital |
| MSBHY | MUFG | ADR line retired; MSBHF exists but traded on only 21 of 755 days |
| NAV | PCAR | Navistar acquired by Traton, delisted 2021 |
| WBA | CVS | Walgreens taken private 2025 |

## Method

1. Fetch daily OHLCV for 28 companies
2. `movement = Close − Open` → matrix of shape (28, 755)
3. `Normalizer` → each company's vector scaled to unit length
4. `PCA(n_components=2)` for the cluster map
5. `KMeans(n_clusters=10, max_iter=1000)` via `make_pipeline`
6. Interpret cluster membership

## References

1. Van Hieu, D., & Meesad, P. (2015). Fast K-Means Clustering for Very Large Datasets Based on MapReduce Combined with a New Cutting Method. *Knowledge and Systems Engineering*, AISC vol. 326, Springer.
2. Desokey, E. N., Badr, A., & Hegazy, A. F. (2017). Enhancing Stock Prediction Clustering Using K-Means with Genetic Algorithm. *ICENCO 2017*, pp. 256–261.
3. Shin, H. W., & Sohn, S. Y. (2004). Segmentation of Stock Trading Customers According to Potential Value. *Expert Systems with Applications*, 27(1), 27–33.
4. Arthur, D., & Vassilvitskii, S. (2007). k-means++: The Advantages of Careful Seeding. *ACM-SIAM SODA*, pp. 1027–1035.
