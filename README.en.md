# skill-factor-decay

[简体中文](./README.md) | [English](./README.en.md)

Multi-horizon Rank IC decay curves → exponential/power-law/bi-exponential fitting → Bootstrap half-life confidence intervals → turnover decay + Q5-Q1 group-return decay → optimal rebalancing frequency recommendation.

`role: skill` `output: DecayReport + rebalance rec` `paradigm: daily cross-sectional IC`


---

`skill-factor-decay` is a **factor decay analysis Skill** provided by PandaAI Quant Skills. It answers the core quant research question: **how long does this factor work, and how often should I rebalance?**

## 🎯 What This Skill Solves

A factor with IC = 0.05 — how does its predictive power vary across 1-day, 5-day, and 20-day horizons?

- If IC(1d) = 0.05, IC(20d) ≈ 0 → fast decay, needs daily rebalancing
- If IC(1d) = 0.05, IC(20d) = 0.04 → persistent signal, monthly rebalancing works
- If IC(1d) = -0.02, IC(20d) = +0.05 → **sign reversal**, short-term reversal + long-term momentum

**Without decay analysis, you don't know the optimal holding period — either overtrade or miss alpha.**

## ⚡ 7-Step Analysis

```
1. Validate inputs: signal [date×symbol] + forward returns [date×symbol×horizon]
2. Compute daily Rank IC (Spearman) for each horizon
3. Fit IC decay curve (exponential / power-law / bi-exponential)
4. Block Bootstrap (1000×) estimate τ₀.₅ 95% CI
5. Turnover decay: daily turnover vs rebalance interval
6. Group-return decay: Q5−Q1 spread vs holding horizon
7. Output DecayReport (JSON + text report)
```

## ⚠️ Sign Reversal

Common in A-share markets: short-horizon IC is negative (mean reversion) while long-horizon IC is positive (momentum continuation). This is real factor structure, not a bug.

| Signal Pattern | Handling |
|----------|----------|
| Unidirectional decay | Standard exponential/power-law fitting |
| Sign reversal | Downgrade to non-parametric; label IC signs per horizon |
| Pure noise | Mark as unusable |

## 🗃️ Input Requirements

- Factor signal: `[date, symbol, factor_value]` parquet files
- Forward returns: 1d/3d/5d/10d/20d horizons (auto-computed from Pandadata OHLCV)

## 📦 Project Script

```bash
# Multi-factor decay analysis (includes equal-weight composite)
python decay_analysis.py
```

Input: `data/factors/F*.parquet` (or `factors_orthogonalized/F*_residual.parquet`)
Output:
- `data/decay_reports/decay_report_*.json` — structured DecayReport
- Console text report (IC curves + half-life CI + rebalance recommendation)

## 🔗 Pipeline Position

```
Factor Mining → Evaluation → Orthogonalize → Decay Analysis (this Skill) → Blending → Backtest
```

Quality gate between orthogonalization and multi-factor blending. Fast-decaying factors are unsuitable for low-frequency blending.

## 📜 License

GPL-3.0. Copyright (C) 2026 QuantSkills.


Pandadata compatibility maintenance: [abgyjaguo](https://github.com/abgyjaguo). Existing authorship is retained.
