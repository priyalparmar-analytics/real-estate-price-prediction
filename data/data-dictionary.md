# Data

## Source

The **Ames Housing** dataset — residential property sales in Ames, Iowa between
2006 and 2010, compiled by Dean De Cock and distributed through the Kaggle
*House Prices: Advanced Regression Techniques* competition.

`data_description.txt` is the original column reference shipped with the
dataset: every column, every categorical level, in the authors' own words.

| File | Rows | Columns | Notes |
| --- | ---: | ---: | --- |
| `train_data.csv` | 1,460 | 81 | Includes `SalePrice` — everything in this project is built on it |
| `test_data.csv` | 1,459 | 80 | No `SalePrice`; the Kaggle holdout. Loaded in the notebook but not used for modelling |
| `data_description.txt` | — | — | Official column reference |

## The target — `SalePrice`

| Statistic | Value |
| --- | ---: |
| Minimum | $34,900 |
| Median | $163,000 |
| Mean | $180,921 |
| Maximum | $755,000 |
| Sales above $500,000 | **9** of 1,460 |

That last row matters more than it looks. Nine examples above $500K is not
enough for a tree-based model to learn the top of the market, and it turns out
to be the single thing that separates the two models' behaviour — see the
README.

The distribution is strongly right-skewed, so the notebook models
`log1p(SalePrice)` and converts errors back to dollars with `expm1` at the end.

## Coverage

| Property | Value |
| --- | --- |
| Period | Sales from 2006 to 2010 |
| Neighborhoods | 25 |
| Raw columns | 80 predictors + `SalePrice` |
| Columns with missing values | 19 |
| Features after cleaning and encoding | **207** |

## Missing values, and why they are not all the same

Nineteen columns have gaps, but they mean different things, so they are handled
in four groups rather than blanket-filled:

| Columns | Missing | Treatment | Reason |
| --- | ---: | --- | --- |
| `PoolQC` | 1,453 | fill `"None"` | The house has no pool — absence, not an unknown |
| `MiscFeature` | 1,406 | fill `"None"` | No miscellaneous feature |
| `Alley` | 1,369 | fill `"None"` | No alley access |
| `Fence` | 1,179 | fill `"None"` | No fence |
| `FireplaceQu` | 690 | fill `"None"` | No fireplace |
| `LotFrontage` | 259 | **median of its neighborhood** | Genuinely unknown, and street frontage is a neighborhood characteristic — a global median would flatten real variation |
| Garage and basement numerics | 81 / 37 | fill `0` | No garage means zero cars and zero area, not an average-sized one |
| `Electrical`, `MSZoning`, `KitchenQual`, … | 1–4 | fill mode | Too few to be worth more |

`Utilities` was dropped outright: 1,459 of 1,460 rows carry the same value, so
it holds no information.

## Encoding

| Kind | Columns | Method |
| --- | --- | --- |
| **Ordinal** | `ExterQual`, `BsmtQual`, `KitchenQual`, `FireplaceQu`, `GarageQual`, … | Mapped to a numeric scale: `Po 1 · Fa 2 · TA 3 · Gd 4 · Ex 5`, with `None 0` |
| **Ordinal, own scales** | `BsmtExposure`, `BsmtFinType1/2`, `GarageFinish`, `Fence`, `Functional`, `LandSlope`, `PavedDrive` | Each mapped on its own documented order |
| **Nominal** | `Neighborhood`, `RoofStyle`, `SaleCondition`, … | One-hot encoded |

Quality ratings carry a real order — Excellent *is* better than Good — so
collapsing them to integers keeps information that one-hot encoding would
discard. Nominal columns have no such order, and giving them one would invent a
relationship the data does not contain.

## Engineered features

| Feature | Built from | Rationale |
| --- | --- | --- |
| `TotalSF` | `TotalBsmtSF + 1stFlrSF + 2ndFlrSF` | Total liveable area — ended up the **2nd most important feature** |
| `HouseAge` | `YrSold − YearBuilt` | Age at sale is more directly meaningful than a build year |
| `RemodAge` | `YrSold − YearRemodAdd` | Time since renovation |
| `TotalBath` | Full and half baths, above and below grade | One weighted count (half-baths at 0.5) instead of four weakly informative columns |

## Generated outputs

Written by the notebook into `../outputs/`, and the source for the Power BI dashboard:

| File | Contents |
| --- | --- |
| `model_results_for_powerbi.csv` | 292 test-set rows × 213 columns — all features plus `SalePrice_actual`, `SalePrice_predicted_RF`, `SalePrice_predicted_LR`, `RF_error`, `LR_error` and a readable `Neighborhood` |
| `feature_importances.csv` | All 207 features ranked by Random Forest importance |

## Licence and citation

> De Cock, D. (2011). *Ames, Iowa: Alternative to the Boston Housing Data as an
> End of Semester Regression Project.* Journal of Statistics Education, 19(3).

The dataset is public and widely redistributed for teaching and competition use.
It contains property attributes only — no names, addresses or personal details.
