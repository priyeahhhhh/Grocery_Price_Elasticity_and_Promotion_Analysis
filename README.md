# Grocery Price Elasticity & Promotion Analysis

How sensitive are grocery shoppers to price, and does it differ enough across categories, price tiers and promotions to justify a segmented pricing strategy instead of one rule for everything?

This project estimates price elasticity from the full Dominick's Finer Foods scanner dataset (three categories, 93 stores, 17.1M raw rows, 10.1M after cleaning) and uses the results to test pricing and promotion scenarios.

## Data

Dominick's Finer Foods store-level scanner data, provided by the James M. Kilts Center for Marketing, University of Chicago Booth School of Business. The data is for academic research use and must be acknowledged in any publication that uses it.

- Download: https://www.chicagobooth.edu/research/kilts/datasets/dominicks
- Manual: https://chicagobooth.edu/-/media/enterprise/centers/kilts/datasets/dominicks-dataset/dominicks-manual-and-codebook_kiltscenter
- Files used: `wana.csv` (analgesics), `wcer.csv` (cereals), `wfrj.csv` (frozen juices). The data files are not included in this repo.

## How to run

1. Download the three movement files above and put them in the same folder as the notebook.
2. Install `pandas` and `numpy`.
3. Run all cells in `Grocery_Price_Elasticity_Analysis_v3_full_data.ipynb`. The files are large, so use a machine with 8GB+ of memory or Google Colab.

## Method

- **Cleaning:** keep rows flagged valid (`OK = 1`) with positive price, units and bundle size.
- **Unit price:** `PRICE / QTY`, because the manual defines `PRICE` as the price of a bundle of `QTY` items.
- **Margin:** `PROFIT` is gross margin in percent (per the manual), not cents per unit.
- **Elasticity:** log(units) on log(price) with store-SKU and week fixed effects, using regular-price weeks only, with standard errors clustered by SKU.
- **Price tiers:** each SKU is assigned to one tier by its own median regular price.
- **Robustness check:** the manual warns that the sale flag is not always set, so I re-estimated using only weeks priced within 5% of each store-SKU's usual price. This drops some real price cuts too, so it likely understates sensitivity and is shown as a sensitivity check, not a replacement.
- **Promotions:** each promoted week is compared with that store-SKU's own average regular-price week.

## Results

**Elasticity by category** (95% CI in brackets, strict version in the last column)

| Category | Elasticity | Strict regular weeks |
|---|---|---|
| Analgesics | -0.75 (-0.87, -0.63) | -0.65 |
| Cereals | -1.38 (-1.57, -1.20) | -0.86 |
| Frozen Juices | -1.74 (-1.97, -1.51) | -1.40 |

Analgesics are the least price-sensitive and frozen juices the most under both definitions, which supports segmenting by category.

**Cereal elasticity by price tier**

| Tier | Elasticity | Strict regular weeks |
|---|---|---|
| Budget (<$2.50) | -1.03 (-1.40, -0.67) | -0.87 |
| Mid ($2.50-$3.50) | -1.67 (-1.88, -1.46) | -0.94 |
| Premium ($3.50-$5) | -1.26 (-1.69, -0.83) | -0.96 |
| Super Premium (>$5, 6 SKUs only) | -2.36 | -1.57 |

Differences between tiers depend on how promotions are defined, so I would not claim one tier is clearly the most sensitive. Mid-tier elasticity is roughly -1 to -1.7. Store pricing zones showed no meaningful difference (about -1.34 to -1.42).

**Pricing scenario, mid-tier cereals** (a model estimate, not realized profit). Baseline: $3.12 average price, 17.2% gross margin, about $18.3M annual revenue and $3.06M annual gross profit across the 93 stores in the data. A 5% price increase is modeled to cut volume about 8% and revenue about $0.59M while raising gross profit about $0.58M a year (about 19%). The gain is positive across the confidence range ($0.54M to $0.62M) because margins are thin. Scenarios are limited to moves of 10% or less, since elasticity is estimated from small price changes.

**Promotions** (versus each store-SKU's own regular weeks)

| Code (per manual) | Price vs regular | Units vs regular | Gross profit vs regular |
|---|---|---|---|
| B, bonus buy | -11.5% | +148% | +17% |
| S, simple price reduction | -25.0% | +444% | +113% |
| G, not defined in manual | -32.4% | +675% | loses money (-2.4% margin) |
| All promotions | -15.9% | +254% | +35% |

Code C (coupon) shows very large lifts but has only 3,289 rows with a small price cut, so I do not rely on it.

## Limitations

- Elasticities come from historical price variation that is mostly small, so results should not be extrapolated to large price moves.
- Retailers may set prices in response to demand, which can bias elasticity estimates toward zero.
- Promotion results do not capture stockpiling, cannibalization of other products or store traffic effects.
- Gross margin is measured against average acquisition cost (not replacement cost, per the manual) and excludes operating costs.
- Dominick's was a 1990s Chicago chain, so results are illustrative and should be checked on current data before use.

## Changes from the first version

An earlier version of this project used partial data files, treated `PROFIT` as cents per unit, ignored bundle sizes, and used weekly averages that mixed promotions into the price effect. Those problems inflated some results (for example, a positive elasticity for premium cereals). This version fixes all of them, so its numbers differ from the earlier ones.
