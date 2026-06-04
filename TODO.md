# TODO

Project backlog for equity-analyst. Items are grouped by implementation phase.
Move items to Done when complete.

---

## Signal & Pattern Phases

### Phase 1 — Momentum & volatility
*Pure pandas calculations on existing OHLCV data. Estimated effort: 1–2 sessions.*

**Indicators**
- [x] MACD + histogram — trend-following momentum, signal line crossover detection
- [x] Bollinger Bands + %B — volatility envelope, squeeze detection
- [x] Volume vs MA20 — flag when volume >1.5× 20-day average (confirms breakouts)
- [x] ATR (14) — Average True Range, raw volatility measure, required for Phase 4 sizing
- [x] Stochastics — fast %K / slow %D oscillator, overbought/oversold in ranging markets
- [x] StochRSI — RSI of RSI, catches momentum shifts earlier than RSI alone
- [x] Support & resistance levels — swing high/low detection from recent price history

**Single candlestick patterns** (`src/signals/candlesticks.py`)
- [x] Doji / dragonfly doji / gravestone doji — indecision and reversal at extremes
- [x] Hammer / hanging man — bullish and bearish reversal, long lower wick
- [x] Inverted hammer / shooting star — reversal with long upper wick
- [x] Marubozu — full-body candle, no wicks, strong momentum signal
- [x] Marubozu — added to engine BULLISH/BEARISH signals + analyst observations

**Two-candle patterns**
- [x] Bullish / bearish engulfing — most reliable two-candle reversal

---

### Phase 2 — Breadth & reversal
*Requires swing point detection utility from Phase 1. Estimated effort: 2–3 sessions.*

**Indicators**
- [x] ADX + DI+/DI- — trend strength and direction, ADX >25 = trending market
- [x] OBV — On-Balance Volume, confirms price trends with cumulative volume flow
- [x] ROC (14) — Rate of Change, momentum percentage as second confirmation
- [x] Pivot points (daily/weekly) — support/resistance from prior bar H/L/C
- [x] Parabolic SAR — stop-and-reverse trailing stop, trend exhaustion signal
- [x] Linear regression bands — statistical trend channel for mean-reversion setups

**Two-candle patterns**
- [x] Bullish / bearish harami — inside bar reversal, more selective than engulfing
- [x] Piercing line / dark cloud cover — partial engulfing reversals
- [x] Tweezer top / bottom — matching highs or lows at support/resistance

**Three-candle patterns**
- [x] Morning star / evening star — three-candle gap reversal, highly reliable
- [x] Three white soldiers / three black crows — strong continuation confirmation
- [x] Three inside up / down — harami followed by confirmation candle

**Chart patterns**
- [x] Double top / double bottom — two equal peaks/troughs, most common reversal
- [x] Bull flag / bear flag — consolidation after sharp move, clear entry rules

---

### Phase 3 — Advanced patterns
*Build after Phase 2 signals validated against real price history. Estimated effort: 3–4 sessions.*

**Indicators**
- [x] Aroon oscillator — time since last high/low, identifies trend age
- [x] Williams %R — inverted stochastics, confirmation alongside RSI
- [x] CCI (20) — Commodity Channel Index, effective for ASX mining stocks
- [x] Bollinger Bandwidth — squeeze detection, low bandwidth precedes big moves
- [x] Guppy CBL — Count Back Line, Australian stop technique for ASX

**Three-candle patterns**
- [x] Morning / evening doji star — star variant with doji, stronger signal
- [x] Abandoned baby — doji with full gap both sides, rare but high-conviction
- [x] Spinning top sequences — accumulation/distribution signal in context

**Chart patterns**
- [x] Head and shoulders / inverse — three-peak reversal, highest empirical reliability
- [x] Ascending / descending triangle — directional breakout with measurable targets
- [x] Symmetrical triangle — neutral coil, breakout direction determines signal
- [x] Cup and handle — rounded base continuation, common on ASX mid-caps
- [x] Rounding bottom (saucer) — gradual curved reversal, useful on weekly charts
- [x] Triple top / bottom — stronger variant of double top/bottom
- [x] Rectangle / trading range — flat consolidation with clear breakout levels
- [x] Bull / bear pennant — triangle variant of flag, tighter consolidation

---

### Phase 4 — Skip / not applicable
- [ ] ~~Put/Call ratio~~ — ASX options data unavailable on yfinance · **low priority**
- [ ] ~~Arms index / TRIN~~ — requires market-wide advance/decline data · **low priority**
- [ ] ~~McClellan / Net AD Line~~ — index-level breadth, not per-stock · **low priority**
- [ ] ~~Trendlines / channels~~ — unreliable algorithmically · **low priority**
- [ ] ~~Gann / Pitchfork / Wolfe Waves~~ — manual tools, not automatable · **low priority**
- [x] Diffusion / Bullish Percent index — watchlist now 23 stocks, sufficient coverage

---

## Data

- [x] Earnings calendar — earnings_date added to 22 stocks in watchlist.json (Aug 2026 season)
      inject into prompt so Claude flags "earnings in X days" as a risk factor
- [ ] Intraday prices — optional yfinance 1h bars · **defer, low priority**
- [x] Iron ore / commodity prices — added VALE, COPX, GLD, USO, LIT as benchmarks
- [x] Exchange rate (AUD/USD) — audusd + audusd_mom in macro features (Sprint 5)
- [x] Dividend calendar — yfinance ex-div date estimation, renders in report

## Analysis & Prompting

- [ ] Prompt tuning — review watch_level accuracy after 4 weeks · **defer to end of June**
      adjust system prompt thresholds based on which signals proved predictive
- [x] Sector context — heatmap uses stock averages (ASX ETFs not on yfinance)
      so Claude can say "CBA underperforming its sector"
- [x] Earnings summary — inject EPS/guidance via personal.earnings_result field + upcoming date warnings + yfinance surprise%
- [x] Confidence score — Claude returns 1-5, shown as stars, HIGH CONVICTION badge at ≥4+ML>0.60
      watch_level to filter low-confidence recommendations

## Tracking & Learning

- [x] Paper trade tracker — log manual trades with entry price, thesis, and outcome
- [x] Backtesting engine — signal accuracy scan and strategy simulation
- [x] Signal accuracy report — weekly script checks acting/interesting call accuracy vs actual returns
      resulted in price moving in the predicted direction over the next 5 days
- [x] Watchlist review — monthly script flags remove/watch/strong candidates, --remove to deactivate
      and whether they should be removed

## Backtesting — Next Steps

**Manual strategy validation**
- [x] Run refined strategy: 3-of-4 combo = 73.3% win rate, +0.505R avg (15 trades)
      (min 3 of 4 signals) — suggested by signal accuracy report
- [x] Morning_star + RSI_oversold combo added to refined_strategy.py backtest
- [ ] Test RSI oversold recovery on BHP.AX — needs more data · **defer**
- [x] Position sizing in strategy backtest — 1% risk ($500/trade on $50k) shows dollar P&L
- [x] Buy-and-hold ^AXJO benchmark added to strategy backtest report
- [x] Short-side backtest — MACD bear cross, death cross+RSI OB, BB upper+stoch OB

**Walk-forward validation (anti-overfitting)**
- [x] Run walk-forward test — train on 2023-2024, validate on 2025-2026 to check overfitting
- [x] Correlation analysis — co-occurrence matrix + predictive pairs by edge over baseline

**Reporting**
- [x] Build HTML backtest report — equity curve chart, drawdown chart, trade log
- [x] Extend watchlist to 10-20 stocks for more signal occurrences and portfolio-level testing

## Next Session — Priority Order

1. **Feature matrix rebuild (interaction fix)** — cartesian product bug fixed
   `python -m src.ml.feature_builder 2>&1 | tee logs/feature_builder.log`
   Should be ~15,630 rows (not 269k), 155 columns (132 + 15 macro + 8 interactions)

2. **Retrain all models** — after feature matrix rebuild completes
   `python -m src.ml.model && python -m src.ml.lgbm_model && python -m src.ml.panel`
   Watch: does ix_yield_change_x_beta stay in top 10 features at correct scale?

3. **Walk-forward report review** — `walkforward_2026-05-28.html` already exists
   Read which signals survived OOS rolling windows

4. **Sprint 6: Model ensemble** — RF + LGBM + Panel weighted by rolling OOS AUC

5. **Buy-and-hold comparison** — add ^AXJO benchmark to backtest report

---

## ML Strategy Discovery

Rather than manually picking signal combinations, use a Random Forest classifier to
discover which signal combinations have genuine predictive edge.

**How it works:**
    Input features:  all 40+ signal values at time T (rsi14, macd_histogram, stoch_k, etc.)
    Target label:    did price rise >2% within 10 days? (1=yes, 0=no)
    Output:          daily probability score per stock + feature importance ranking

**Why Random Forest (not deep learning):**
- Interpretable — feature importance tells you which signals matter most
- Resistant to overfitting vs neural networks on small datasets (750 bars)
- Fast to train and retrain weekly
- Probability output usable as a confidence filter on top of existing signals

**Implementation steps:**
- [x] Feature engineering — compute all Phase 1/2/3 signals for every historical bar
      and store as a flat feature matrix (src/ml/feature_builder.py)
- [x] Label generation — for each bar, calculate forward 5/10/20-day returns
      and create binary labels (src/ml/labels.py)
- [x] Train Random Forest model — scikit-learn, walk-forward cross-validation
      (src/ml/model.py)
- [x] Feature importance report — which signals the model weighted most heavily;
      this becomes your data-driven strategy definition
- [x] Daily inference — run model on today's signals to generate a probability
      score (0-1) per stock; inject into Claude prompt as "ML confidence: 0.73"
- [x] Backtest ML strategy — ML only: 54.7% win, +0.640R (3.4x better than manual)
      vs manual strategy performance
- [x] Retrain schedule — retrain model monthly as new data accumulates

## Infrastructure

- [x] GitHub Actions — run `pytest tests/` on every push (CI)
- [x] Cron job verification — daily health check that alerts if main.py didn't run
- [x] DB maintenance — weekly job to vacuum SQLite and prune news older than 30 days
- [x] Config validation — validate watchlist.json schema on startup

## Known issues / tech debt
- [x] NewsAPI rate limit — batch mode in news_fetcher.py (5 requests vs 47)
- [x] New stocks default alerts — rsi_overbought:70, rsi_oversold:30 on all 47 stocks

## Multi-agent enhancements (future sprints)
- [x] MLflow experiment tracking — RF, LGBM, Panel training runs + walk-forward logged automatically
- [x] FastAPI /predict endpoint — GET /predict/{ticker}, POST /predict, GET /watchlist, GET /signals/{ticker}, GET /portfolio
- [x] MLflow automated monitoring — Level 1 weekly summary, Level 2 drift detection,
      Level 3 auto-retrain trigger, Level 4 model registry promotion
- [x] Docker containerisation — Dockerfile + docker-compose (API + MLflow), model files mounted as volumes
- [x] Portfolio manager agent — Claude Sonnet ranks daily actions (ENTER/CLOSE/TRAIL/WATCH/SKIP) with CGT awareness, injected at top of email and report
- [ ] Bull/Bear debate agent — two Claude instances argue opposite sides of each trade, third adjudicates — reduces false positives on acting calls
- [ ] Tool-use analyst agent — Claude autonomously decides when to fetch more data, run backtest, check dividend calendar before producing analysis (replaces pre-fetched context)
- [ ] Researcher + Analyst separation — one agent summarises news/macro, another analyses technicals, third synthesises into trade recommendation with specific levels

## ML model improvements (before going live)
- [ ] Fix cross-sectional feature leakage — fit cs_rank features on train only, transform test separately
- [ ] Add 10-day embargo gap between train and test splits (López de Prado methodology)
- [ ] Re-run walk-forward after fixes — expect modest AUC improvement
- [ ] Validate 20+ paper trades show 60%+ win rate before live trading
- [ ] 4 consecutive walk-forward Fridays with stable robust tickers
- [ ] Signal accuracy report: acting calls > 55% correct

## Automation (future)

- [ ] Broker API research — evaluate IBKR or Stake APIs for ASX and US execution
- [ ] Order sizing — position sizing module based on ATR and portfolio allocation rules
- [ ] Dry-run mode — paper orders through broker API without real money
- [ ] Automation guardrails — human approval required before any real order is placed

## Trading plan (added 2026-05-28)

- [x] Watchlist expanded to 50 stocks (27 new: gold miners, REITs, US tech, industrials)
- [x] Partial combo alert — email fires when 3/4 on robust ticker
- [x] Weekly performance email — Fridays, shows R, win rate, combo fires, monthly target progress
- [x] Three-strategy framework — swing (73% win rate), short (death cross+double top), position (CGT discount)
- [x] Tax-adjusted R in tracker — Net R column with CGT✓ badge at 12m+, 47% short-term / 23.5% long-term
- [x] Monthly P&L analysis — $50k → $10k/month net needs ~$200k account + 18 trades/month
- [x] All strategies backtested — refined 3/4, marubozu, morning star, short-side, trend combos
- [x] Correlation analysis — death_cross+double_top 81.8% win rate, golden_cross combos 60-68%
- [x] README — full rewrite covering all 6 sprints, ML results, backtest table, trading plan
- [x] MODELS.md — ML methodology, time-series validation, known limitations, pre-live checklist

---

## Done

- [x] Project scaffold + WatchlistConfig
- [x] Price fetcher — yfinance OHLCV to SQLite
- [x] News fetcher — NewsAPI headlines to SQLite
- [x] Daily runner — manual + cron + schedule modes
- [x] Signal calculator — MA20, MA50, RSI14, relative performance
- [x] Prompt builder — structured context assembly per stock
- [x] Claude analyst — API calls, retry logic, structured JSON output
- [x] Report store — daily JSON persistence
- [x] HTML dashboard — self-contained report with sparklines and watch-level cards
- [x] Phase 1 signals — MACD, Bollinger, Volume, ATR, Stochastics, StochRSI, S&R, candlestick patterns
- [x] Phase 2 signals — ADX, OBV, ROC, Pivot Points, PSAR, LR bands, harami/piercing/tweezer/star/soldier/crow patterns, double top/bottom, bull/bear flags
- [x] Phase 3 signals — Aroon, Williams %R, CCI, BB Bandwidth, CBL, doji stars, abandoned baby, spinning tops, H&S, triangles, triple top/bottom, rectangle, cup&handle, rounding bottom, pennants
- [x] HTML backtest report — equity curve, drawdown chart, signal heatmap, trade log
- [x] Extended watchlist to 15 stocks across 7 ASX sectors + AAPL
- [x] GitHub Actions CI — pytest runs on every push
- [x] Health check — daily cron verifies runner executed and data is fresh
- [x] DB maintenance — weekly vacuum + news pruning
- [x] Config validator — validates watchlist.json on startup
- [x] ML feature builder — 65 features × 10,175 rows across 15 tickers
- [x] Random Forest model — per-ticker training, 5 tickers show AUC > 0.55
- [x] ML scores wired into Claude prompt and daily HTML report
- [x] Walk-forward validator — train/validate split with HTML report
- [x] Monthly retrain script — src/utils/retrain.py with drift logging
- [x] ML + walk-forward test coverage — tests/test_ml.py
- [x] Phase 4 signals — Reflex + SuperSmoother (Ehlers zero-lag)
- [x] Phase 4 signals — TrendFlex (trend-retaining oscillator)
- [x] Phase 4 signals — Sentiment: CMF, Elder Force Index, Ease of Movement, VIX
- [x] Phase 4 signals — MA200, Golden Cross, Death Cross, MA spread
- [x] Phase 4 signals — RSMK (Katsanos relative strength vs benchmark)
- [x] Phase 4 signals — Kaufman 1st & 2nd Cross
- [x] Reflex/TrendFlex/RSI divergence detection in prompt
- [x] Paper trading tracker HTML report (src/tracking/tracker_report.py)
- [x] Paper trading section in daily HTML report
- [x] Benchmark price fetch (^AXJO, ^GSPC) — enables real beta computation
- [x] Sprint 1: continuous return labels + OOS R² evaluation
- [x] Sprint 2: cross-sectional features (sector, beta, cap tier, CS momentum ranks)
- [x] Sprint 3: LightGBM + expanding-window CV
- [x] Sprint 4: panel model with stock fixed effects
- [x] main.py — full pipeline (fetch → analyse → report)
- [x] Cron working directory fix
- [x] ASX 100 screener — daily candidates section in report (src/screener/)
- [x] News-based macro + company action suggestions (src/news/macro_monitor.py)
- [x] Paper trading tracker HTML report (src/tracking/tracker_report.py)
- [x] Paper trading section in daily HTML report
- [x] Benchmark price fetch (^AXJO, ^GSPC) — real beta computation
- [x] Beta computation fixed — real values (BHP 1.35, TLS 0.15, WDS -0.23)
- [x] Watchlist expanded to 23 stocks — 8 ASX 100 screener additions
- [x] Sprint 5: macro features (RBA, yield curve, AUD/USD, VIX, market regime)
- [x] ML strategy backtest — Manual vs ML vs Combo, fair OOS comparison
- [x] ML backtest result: ML 3.4x better avg R (+0.64R vs +0.19R), Sharpe 0.43 vs 0.13
- [x] Sector rotation heatmap in daily report
- [x] 52-week high proximity scoring in screener
- [x] Trade journal section in tracker report
- [x] Test coverage: test_position_sizer, test_macro_monitor, test_macro_features, test_screener, test_sector_heatmap (22 tests, all passing)
- [x] Sector rotation heatmap with commodity proxy context (VALE, COPX, GLD, USO, LIT)
- [x] Commodity proxies in analyst prompt (iron ore/copper for Materials, oil for Energy)
- [x] Refined strategy backtest — 3-of-4 combo: 73% win rate vs 49% base (Item 19)
- [x] Refined combo badge in daily report cards (🎯 3/4 or ◑ partial 2/4)
- [x] HIGH CONVICTION badge — confidence≥4 AND ML>0.60 (Item 16)
- [x] Daily email notification (Gmail SMTP, working)
- [x] Walk-forward report runs automatically on Fridays
- [x] Refined combo signal section in analyst prompt (73% win rate backtest context)
- [x] Paper trading: 5 open positions, 3 closed, +1.99R cumulative
- [x] Earnings calendar (ASX API, news DB fallback)
- [x] Position sizing calculator with portfolio heat
- [x] Heat warning on trade open CLI
- [x] Paper trade tracker — open/close trades, P&L, R-multiple, accuracy report CLI
- [x] Backtesting engine — signal accuracy scan and strategy simulation over 3 years
