# Trained models

This folder is where the notebook writes its two fitted models:

| File | Model | Approx. size |
| --- | --- | --- |
| `linear_regression_model.pkl` | `LinearRegression` baseline | 12 KB |
| `random_forest_model.pkl` | `RandomForestRegressor(n_estimators=200)` | 20 MB |

**Both are git-ignored and not committed.** Run the notebook end to end and the
final cell recreates them:

```python
joblib.dump(rf, '../models/random_forest_model.pkl')
joblib.dump(lr, '../models/linear_regression_model.pkl')
```

Two reasons they are not in the repository: the forest is 20 MB of binary that
changes wholesale on every retrain, and a pickle only loads reliably under the
same scikit-learn version that wrote it — so a committed one goes stale quietly.
`random_state=42` is set on the split and on the forest, so a rerun reproduces
the same models and the same numbers.
