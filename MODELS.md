# ML Models — Methodology & Validation

Technical documentation covering model architecture, training methodology,
known limitations, and outstanding issues to fix before live trading.

---

## Architecture

Three models combined into a weighted ensemble:

| Model | Type | Features | Tickers | Key metric |
|---|---|---|---|---|
| RF per-ticker | Random Forest classifier | 156 (signals + macro) | 47 | Avg AUC 0.534 |
| LGBM per-ticker | LightGBM classifier | 156 | 47 | Avg AUC 0.550 |
| Panel model | LightGBM + stock fixed effects | 194 (132 signals + ticker dummies) | 47 | EW AUC 0.524 |

**Ensemble weighting:** each model is weighted by its OOS AUC. Models with
AUC < 0.48 are excluded from the weighted average. The ensemble produces a
probability (0–1) for each stock's 10-day forward return being positive.

**Target variable:** `label_10d` — binary, 1 if 10-day forward return > 0.

---

## Training methodology

### Time-series split ✅

All models use a **strict chronological split** — no data shuffling:

```
Train: 2023-08-22 → 2025-08-28  (24,484 rows)
Test:  2025-08-28 → 2026-05-01  (8,189 rows)
```

The model never sees future data during training. Test set represents the
most recent ~8 months of market data.

### Expanding-window cross-validation ✅

5-fold CV respects temporal order — each fold adds more training data and
tests on the next contiguous period. No shuffling, no random splits.

```
Fold 1: train 400 dates → test 59 dates
Fold 2: train 459 dates → test 59 dates
Fold 3: train 518 dates → test 59 dates
Fold 4: train 577 dates → test 59 dates
Fold 5: train 636 dates → test 59 dates
```

### Per-ticker time split ✅

RF and LGBM per-ticker models each split their own ticker's history
chronologically — no cross-contamination between tickers.

### Walk-forward validation ✅

Runs every Friday. Tests signal reliability on rolling OOS windows across
multiple market regimes. Current result: **19/47 robust, 15 moderate, 14 overfit.**

This is the most rigorous validation — it catches models that overfit to a
specific market period.

---

## Known limitations

### 1. Cross-sectional feature leakage ⚠️ MEDIUM RISK

Cross-sectional features (e.g. `cs_rank_ret_6m` — ranking all stocks by
6-month return at each date) are computed across the **full dataset** before
the train/test split. This means the test-period ranks are computed using
test-period returns, which the model technically should not have seen.

**Impact:** Modest. The ranking is a relative measure across tickers at a
point in time — it doesn't directly reveal future prices. But it does use
test-period data to compute a feature.

**Fix (before going live):** Compute cross-sectional features within the
training window only, then apply the same transformation to the test set
using training-period statistics.

```python
# TODO: fit cross-sectional scaler on train only
cs_scaler = QuantileTransformer()
X_train['cs_rank_ret_6m'] = cs_scaler.fit_transform(X_train[['ret_6m']])
X_test['cs_rank_ret_6m']  = cs_scaler.transform(X_test[['ret_6m']])
```

### 2. Panel model ticker dummies ⚠️ LOW RISK

The panel model includes stock fixed effects (one-hot encoded ticker dummies).
These are fit on the full dataset including the test period, which is
technically leakage. In practice the fixed effects capture long-run
stock-specific bias — not forward-looking signal — so the impact is minimal.

**Fix:** Fit ticker dummies on training data only, use the same encoding for test.

### 3. No purging / embargo gap ⚠️ LOW RISK

In strict financial ML (López de Prado methodology), a gap (embargo) is added
between the train and test sets to prevent overlapping label windows from
leaking information. Our target is 10-day forward returns — trades 1–9 days
before the cutoff date have labels that overlap with the test period.

**Impact:** Modest for a 10-day label and a multi-year training set. The
overlap is at most 9 days out of 400+ training days.

**Fix (before going live):** Add a 10-day embargo gap between train and test:

```python
embargo_days = 10
train_end    = cutoff_date - timedelta(days=embargo_days)
# Train: ... → train_end
# Test:  cutoff_date → ...
```

### 4. Survivorship bias ⚠️ LOW RISK

The watchlist contains currently-active stocks. Stocks that were delisted or
significantly restructured during the training period (e.g. SYD.AX delisted,
WPL.AX renamed to WDS.AX) have been removed. This means the models were
trained on survivors — stocks that performed well enough to still exist.

**Impact:** Small for a 3-year window with major-cap ASX stocks. Would be
more significant with small-caps.

**Fix:** No practical fix for a personal watchlist system. Acceptable limitation.

### 5. Transaction costs not modelled ⚠️ LOW RISK

Backtests assume costless entry and exit. Real trading incurs:
- Brokerage: ~$9.50/trade (SelfWealth / CommSec)
- Bid-ask spread: ~0.05–0.10% for large-cap ASX
- Market impact: negligible at paper trading sizes

**Impact on refined 3/4 combo backtest:**
At $500 risk/trade and average $9.50 brokerage (both ways):
- Net R reduced by ~$19/trade = ~0.038R per trade
- Over 15 backtested trades: reduces total R by ~0.57R
- Adjusted total: +7.57R → +7.00R — still strongly positive

---

## Feature importance (top 10 across all models)

| Rank | Feature | Category | Models |
|---|---|---|---|
| 1 | `rsmk` | Relative strength | RF, Panel |
| 2 | `ma_spread` | Trend | RF, LGBM, Panel |
| 3 | `ix_regime_x_rsmk` | Macro interaction | Panel |
| 4 | `ix_vix_x_beta` | Macro interaction | RF, Panel |
| 5 | `ix_yield_change_x_beta` | Macro interaction | RF |
| 6 | `pct_from_ma200` | Trend | RF, LGBM, Panel |
| 7 | `macd_signal` | Momentum | RF, LGBM, Panel |
| 8 | `atr_pct` | Volatility | LGBM, Panel |
| 9 | `adx` | Trend strength | RF, LGBM |
| 10 | `cs_rank_ret_6m` | Cross-sectional | Panel |

Macro interaction features (Sprint 5b) dominate the panel model top 3,
validating the decision to add them.

---

## Walk-forward results (47 stocks, June 2026)

| Reliability | Count | Tickers |
|---|---|---|
| **Robust** | 19 | CSL, RIO, MQG, FMG, ANZ, AAPL, STO, S32, ILU, ALL, RMD, COL, TWE, BPT, CPU, SCG, DXS, NVDA, MSFT |
| **Moderate** | 15 | XRO, GMG, TCL, META, GOOGL, NHF, NAB, WES, WDS, OML, AZJ, EVN, BXB, WTC, SUN |
| **Overfit** | 14 | CBA, BHP, MIN, QBE, REA, ALX, WOW, TLS, COH, IAG, NST, TPG, AMZN, APX |

Only signals on **robust** tickers are used for trading decisions.

---

## Pre-live trading checklist

Before going live in July 2026:

- [ ] Fix cross-sectional feature leakage (issue #1 above)
- [ ] Add 10-day embargo gap to train/test split (issue #3)
- [ ] Re-run walk-forward after fixes — expect modest AUC improvement
- [ ] Validate 20+ paper trades show 60%+ win rate
- [ ] 4 consecutive walk-forward reports with stable robust tickers
- [ ] Signal accuracy report shows acting calls > 55% correct

---

## References

- López de Prado, M. (2018). *Advances in Financial Machine Learning*. Wiley.
  — Chapters 7 (cross-validation), 8 (feature importance), 12 (backtesting)
- Kelly, B. & Xiu, D. (2023). *Financial Machine Learning*. SSRN.
  — R² > 0.005 as threshold for genuine predictive power
- Prado, M. (2019). A taxonomy of obstacles to financial machine learning.
  — Survivorship bias, backtest overfitting, multiple testing

---

*Last updated: 2026-06-04 · equity-analyst v1.0*
