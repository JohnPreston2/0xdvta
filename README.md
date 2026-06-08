# 0xDVTA — Autonomous Quantitative Trading Agent

<p align="center">
  <img src="https://img.shields.io/badge/Exchange-Hyperliquid-00D4FF?style=for-the-badge" />
  <img src="https://img.shields.io/badge/AI-DeepSeek%20v4%20Pro%2FFlash-FF6B35?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Forensics-Engine%20V5-FFD700?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Signals-44%20OOS%20Rules-9945FF?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Live%20on%20GCP-34A853?style=for-the-badge" />
</p>

> **0xDVTA** is a fully autonomous crypto trading agent running 24/7 on Hyperliquid. It executes a quantitative pipeline every 2 hours: forensic data collection, 44 statistically validated signal rules, LLM-driven BIAS filtering, and automated position management with per-token TP/SL and horizon-based timeouts.

> Originally built as [0xDELTA](https://github.com/JohnPreston2/0xdelta-hub) (Solana forensic agent, Bags Hackathon 2026), the project pivoted to Hyperliquid perpetuals with institutional-grade signal evaluation and a two-model LLM intelligence layer.

---

## Orchestration Map

![OpenClaw Trading Agent Orchestration Map](docs/openclaw_orchestration_map.png)

---

## Pipeline Architecture

The trading pipeline runs every 2 hours via `run_pipeline.sh` — 13 sequential steps, 35 scripts, 17,730 lines of Python.

### Phase A — Data Collection (0 LLM calls)

| Step | Script | Source | Output |
|------|--------|--------|--------|
| 0 | `check_position_hl.py` | Hyperliquid API | Exit if 2 positions open |
| 0b | `check_balance.py` | Venice billing API | Log balance |
| 1 | `collector.py` (118L) | DexScreener + GeckoTerminal | `data/raw/` (1,306 files) |
| 2 | `report_builder.py` (119L) | ForensicEngine V5 | `data/processed/` (~54KB/cycle) |
| 2c | `velocity_engine.py` (441L) | Rolling computations | `data/dynamics/` (742 files) |
| bg | `binance_collector.py` (362L) | Binance + HL APIs (*/30min) | `binance_context.json` |

#### Forensic Engine V5 — 30+ Metrics

Computes metrics across 6 analytical dimensions every cycle:

- **Liquidity** — ICR (crash risk), LCR (concentration), IPS (impact sensitivity)
- **Flows** — NBP (net buy pressure), EV (expected value), VWAD
- **Concentration** — WCC (whale change), TCI (top concentration), FCI (cluster index)
- **Bull Flag** — BPI (bull power), FQS (flag quality), Fibonacci targets
- **Technical** — RSI (1h/15m), SI (sentiment), BER (bull/bear energy)
- **Convergence** — FHS (forensic health 0-10), phase classification

The **velocity engine** adds a dynamic layer: velocity, acceleration, z-scores, and 5 composite indicators (LMI, SDA, NBR, SMP, RSI_ICR) on rolling windows.

### Phase B — Signal Processing

| Step | Script | Lines | Description | Output |
|------|--------|-------|-------------|--------|
| 2e | `signal_evaluator.py` | 679 | V1 legacy rules | `evaluated_signals.json` |
| 2e-bis | `signal_evaluator_v2.py` | 560 | **44 OOS-validated rules** | `evaluated_signals_v2.json` |
| 2e-ter | `signal_evaluator_composite.py` | — | Composite signals | `evaluated_signals_composite.json` |
| 2f | `signal_tracker.py` | 786 | BIAS filter via `regime_card.json` | `signals.json` |

#### Signal Evaluator V2 — The Edge

44 institutional-grade rules surviving rigorous out-of-sample validation:
- **Statistical validation**: Fisher exact test + Benjamini-Hochberg FDR correction
- **OOS requirement**: WR >= 50% on both training and test periods
- **Coverage**: 6 tokens (AERO, AIXBT, BRETT, MORPHO, VIRTUAL, VVV)
- **Average R:R**: ~2.5 across all rules

The signal tracker applies the **BIAS filter** (LONG/SHORT/NEUTRAL from the morning LLM briefing) to block signals that contradict the current market regime.

### Phase C — LLM Intelligence (Goldman v4)

`financial_agent.py` (3,038 lines) implements **11 operational modes** using a two-model architecture: a pro strategist for high-stakes decisions, a flash sentinel for continuous monitoring.

#### Pro Model (deepseek-v4-pro) — ~3-7 calls/day

| Mode | Schedule | Output | Role |
|------|----------|--------|------|
| `--morning` | 07:00 UTC | `regime_card.json` | BIAS + thesis + token ranking + conviction |
| `--monitor` | */1h (if positions) | `monitor_verdicts.json` | CUT/TIGHTEN/HOLD + BIAS_UPDATE |
| `--reflect` | on position close | `trade_reflections.json` | Compare thesis vs outcome |
| `--intelligence-memo` | 00:15 UTC | `intelligence_memo.json` | Daily strategic synthesis |

#### Flash Model (deepseek-v4-flash) — ~40-60 calls/day

| Mode | Schedule | Output | Role |
|------|----------|--------|------|
| `--quick-monitor` | */30min | `quick_monitor.json` | CUT if danger detected |
| `--pre-morning` | 06:55 UTC | `pre_morning_brief.json` | Overnight brief for pro |
| `--brief-compiler` | 06:58 UTC | `morning_brief_compiled.json` | Pre-digest for morning |
| `--pre-monitor` | */1h :03 | `pre_monitor_data.json` | Data collector for monitor |
| `--alert` | */2h | BIAS_UPDATE | BTC momentum check |
| `--gate` | pre-trade | `gate_result.json` | GO/WAIT trade decision |
| `--monitor` | */1h (if 0 pos) | verdicts | Lightweight market check |

#### Intelligence Flywheel

```
trades ──► reflector (0 API) ──► lessons.json (WR/token, WR/side, patterns)
                                      │
trades ──► reflect (pro) ──────► trade_reflections.json
                                      │
                                      ▼
intelligence-memo (pro, daily) ──► intelligence_memo.json
                                      │
                                      ▼
pre-morning ──► brief-compiler ──► morning (pro) ──► regime_card.json
                                                          │
                                                          ▼
                                               Better trade decisions
```

### Phase D — Execution

| Component | Description |
|-----------|-------------|
| `enter_position_hl.py` (1,257L) | Gate LLM → Execute on Hyperliquid → TP/SL trigger orders on-chain |
| `manage_positions` | TP/SL monitoring, timeout management, CUT retry (3 attempts: 5%/10%/15% slippage) |
| `reflector.py` (490L) | Daily trade analysis at midnight, 0 API calls |

---

## Trading Parameters

| Parameter | Value |
|-----------|-------|
| Capital | ~$100 USDC |
| Notional per trade | $100 (5x leverage = ~$20 margin) |
| Max positions | 2 simultaneous |
| Min confidence | >= 55 |
| Gate decision | GO/WAIT (no forced trading) |

### Per-Token TP/SL (Backtest-Optimized)

| Token | TP | SL | Horizon |
|-------|----|----|---------|
| AERO | +3.5% | -1.4% | T6 (timeout 14h) |
| AIXBT | +8.5% | -2.6% | T12 (timeout 28h) |
| BRETT | +3.3% | -2.3% | T24 (timeout 52h) |
| MORPHO | +8.4% | -2.6% | — |
| VIRTUAL | +5.3% | -2.9% | — |
| VVV | +6.3% | -2.5% | — |

---

## External Connections

| Service | Role | Protocol |
|---------|------|----------|
| **Venice API** | LLM inference (DeepSeek v4 pro + flash) | HTTPS |
| **Hyperliquid** | MAINNET perps execution + trigger orders | SDK |
| **Hermes** (separate project) | Market intelligence terminal via VPC | HTTP :5002 |
| **DexScreener** | OHLCV price data | API |
| **GeckoTerminal** | Trending + OHLCV | API |
| **Binance** | Funding rates + candles | API |
| **Telegram** | Alerts + briefs | Bot API |
| **GitHub** | Auto-push every 30min | Git |

> **Hermes** is a separate intelligence agent running on a dedicated VPS. It provides live market data (32 tokens, funding rates, signals, news, composite intelligence scores) to 0xDVTA via GCP VPC internal network. See [hermes-wiki](https://github.com/JohnPreston2/hermes-wiki) for details.

---

## Key Metrics (June 8, 2026)

| Metric | Value |
|--------|-------|
| Scripts | 35 |
| Lines of code | 17,730 |
| Data files | 21 JSON + 742 dynamics |
| Total data | 1.1 GB |
| Cron entries | 25 |
| LLM calls/day | ~50-70 (pro + flash) |
| Uptime | 20 days |

### Trading Performance (last 14 days)
- **216 roundtrips** total
- **WR 14d**: 40.5%
- **Avg duration**: 344 min
- LONG WR: 34% | SHORT WR: higher
- Current BIAS: SHORT | Conviction: 7/10

---

## Evolution Timeline

| Session | Date | Key Changes |
|---------|------|-------------|
| Origin | Feb 2026 | 0xDELTA — Solana forensic agent (Bags Hackathon) |
| s3 | May 21 | Goldman v4 — two-model architecture (pro + flash) |
| s7 | May 24 | CUT retry 3x, gate WAIT enabled, Venice flash fallback |
| s9 | May 28 | **Signal recalibration** — 44 institutional rules (Fisher exact + BH-FDR) |
| s9b | May 28 | Per-token TP/SL, horizon timeouts, intelligence memo, reflector v2 |
| s10 | May 29 | Composite signal evaluator, binance_collector |
| s12b | Jun 5 | Blacklist format fix, gate WR filter, bias conviction guard |
| s13 | Jun 7 | Intelligence context builder, morning_bias, BTC surge alert, alert re-enabled |

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| **Runtime** | Python 3.11, GCP e2-medium |
| **Trading** | Hyperliquid SDK (MAINNET perps) |
| **Forensics** | Custom ForensicEngine V5 (30+ metrics, systemd) |
| **Signal Eval** | 44 OOS rules + composite evaluator |
| **LLM** | Venice API (DeepSeek v4 pro + flash) |
| **Scheduling** | Cron (25 entries, */2h pipeline) |
| **Monitoring** | Telegram alerts + auto-push GitHub |
| **Orchestration** | Claude Code (Opus 4.6) — session-based patching |

---

## License

MIT

---

<p align="center">
  <b>0xDVTA — Autonomous Quantitative Trading Agent</b><br/>
  <i>Forensic analysis + statistical signals + LLM intelligence → Hyperliquid perps</i>
</p>
