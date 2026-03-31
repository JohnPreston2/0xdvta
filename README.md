# 0xDVTA — Autonomous Forensic Trading Agent for Solana

<p align="center">
  <img src="https://img.shields.io/badge/Chain-Solana-9945FF?style=for-the-badge&logo=solana&logoColor=white" />
  <img src="https://img.shields.io/badge/Ecosystem-Bags.fm-FF6B35?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Engine-Forensic%20v6-00D4AA?style=for-the-badge" />
  <img src="https://img.shields.io/badge/AI-Gemini%20%2B%20Venice-4285F4?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Metrics-65%2B-FFD700?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Live%20on%20GCP-34A853?style=for-the-badge" />
</p>

> **0xDVTA** is a fully autonomous forensic trading agent running 24/7 on Solana. It monitors Bags-native tokens via GeckoTerminal, computes 65+ on-chain forensic metrics, detects quantitative trading signals, and executes positions through Jupiter — all without human intervention.

> Fork of [0xDELTA](https://github.com/JohnPreston2/0xdelta-hub) (Base chain, ERC-8004 Agent #32715), adapted and calibrated for Solana's liquidity structure and the Bags ecosystem.

---

## Why 0xDVTA for Bags?

The Bags ecosystem launches tokens via Meteora DAMM v2 — fast, liquid, but hard to evaluate in real-time. 0xDVTA solves this by providing **deep forensic intelligence** on every Bags-native token, detecting accumulation patterns, whale movements, and pump signals **before** they become visible on charts.

**For the Bags ecosystem, 0xDVTA is:**
- A **trading tool** that enhances how users interact with Bags-launched tokens
- A **forensic scanner** that brings institutional-grade analysis to retail traders
- An **autonomous agent** that demonstrates what's possible with Bags APIs + AI

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    0xDVTA Pipeline                       │
│                   (cron every 2h)                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐    ┌───────────────────┐              │
│  │ GeckoTerminal │───▶│ collect_tokens    │──▶ data.json │
│  │ Bags API      │    │ _solana.py        │              │
│  └──────────────┘    └───────────────────┘              │
│                              │                          │
│                              ▼                          │
│                 ┌────────────────────────┐               │
│                 │ forensic_engine_v6.py  │               │
│                 │ 65+ metrics computed   │               │
│                 └────────────────────────┘               │
│                              │                          │
│                   forensic_YYYYMMDD.json                │
│                              │                          │
│                              ▼                          │
│                 ┌────────────────────────┐               │
│                 │ enrich_forensic.py     │               │
│                 │ velocities + z-scores  │               │
│                 │ signature detection    │               │
│                 └────────────────────────┘               │
│                              │                          │
│                              ▼                          │
│          ┌───────────────────────────────────┐          │
│          │  request_analysis_sol.py          │          │
│          │  Venice AI (private) + Gemini     │          │
│          │  (anonymized) hybrid analysis     │          │
│          └───────────────────────────────────┘          │
│                              │                          │
│                              ▼                          │
│          ┌───────────────────────────────────┐          │
│          │  enter_new_position.py            │          │
│          │  RULE_BOOK filters → Jupiter swap │          │
│          └───────────────────────────────────┘          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Forensic Engine v6 — 65+ Metrics

The core forensic engine computes metrics across 7 analytical dimensions on every pipeline cycle:

### Liquidity Intelligence
| Metric | Description |
|--------|-------------|
| **ICR** | Immediate Crash Risk — price impact of a $10K/$50K/$100K sell |
| **LCR** | Liquidity Concentration Ratio — pool fragility assessment |
| **LVR** | Liquidity-to-Volume Ratio — sustainability indicator |
| **IPS** | Impact Price Sensitivity at multiple thresholds |

### Flow Analysis
| Metric | Description |
|--------|-------------|
| **NBP** | Net Buy Pressure — buy vs sell flow differential |
| **VWAD** | Volume-Weighted Accumulation/Distribution |
| **EV** | Expected Value — risk-adjusted return potential |
| **AC** | Accumulation Coefficient |

### Forensic Concentration
| Metric | Description |
|--------|-------------|
| **WCC** | Whale Concentration Change — smart money tracking |
| **TCI** | Top Concentration Index — holder centralization risk |
| **FCI** | Forensic Cluster Index — coordinated wallet detection |
| **SCR** | Supply Concentration Ratio |
| **Top5/10/20%** | Holder distribution percentiles |

### Bull Flag Detection
| Metric | Description |
|--------|-------------|
| **BPI** | Bull Power Index — momentum scoring |
| **FQS** | Flag Quality Score — pattern reliability |
| **Fibonacci targets** | 1.618 extension projected prices |
| **Squeeze Factor** | Consolidation compression measurement |

### Technical Signals
| Metric | Description |
|--------|-------------|
| **RSI** | Relative Strength Index (1h + 15m) |
| **SI** | Sentiment Index — multi-timeframe |
| **BER** | Bull/Bear Energy Ratio |
| **RMD** | Relative Momentum Divergence |

### Convergence
| Metric | Description |
|--------|-------------|
| **FHS** | Forensic Health Score (0-10) — composite quality rating |
| **CP** | Convergence Percentage — signal alignment |
| **Phase** | Market phase classification |

### Dynamic Metrics (Enrichment Layer)
| Metric | Description |
|--------|-------------|
| **lcr_vel** | Liquidity velocity — Δ(LCR) between snapshots |
| **nbp_zscore** | Net Buy Pressure z-score — rolling statistical position |
| **wcc_vel** | Whale movement velocity — smart money acceleration |
| **bpi_zscore** | Bull Power momentum z-score |
| **LMI** | Liquidity Momentum Index (composite) |
| **SMP** | Smart Money Pressure (composite) |

---

## Quantitative Backtest Results

Calibrated on 40 days of on-chain data (7,146 snapshots, 17 tokens) on the parent chain, then cross-validated on Solana:

### Optimal Signal
```
lcr_vel > 0.05 AND nbp_zscore > 0 AND fhs ≥ 6 AND wcc_vel > 0
→ Hit Rate: 66.7% | Avg Return T+3: +19.5%
```

### Top 5 Predictive Features (SHAP Analysis)
```
1. lcr_vel        0.00686  (liquidity acceleration)
2. nbp_zscore     0.00510  (buy pressure momentum)
3. bpi_zscore     0.00418  (bull power momentum)
4. volume_24h_z   0.00404  (volume anomaly)
5. lcr_zscore     0.00350  (liquidity statistical position)
```

### Signature Detection
| Signature | Hit Rate | Avg Return T+3 | Sample Size |
|-----------|----------|-----------------|-------------|
| **PUMP_IMMINENT** | 75% | +5.97% | n=24 |
| SILENT_ACCUMULATION | 62% | +3.2% | n=18 |
| DISTRIBUTION_STEALTH | 81% accuracy | — | n=15 |

### Solana Calibration
Structural ratios between Solana and Base chain were computed to adjust thresholds:

| Parameter | Base | Solana | Ratio |
|-----------|------|--------|-------|
| ICR (avg) | 3.8 | 17.2 | 4.5x |
| FHS (avg) | 6.1 | 5.22 | 0.86x |
| BPI (avg) | 0.8 | 0.36 | 0.45x |

Calibrated thresholds: `MAX_ICR=30`, `MIN_FHS=5.5`, `MIN_BPI=0.5`

---

## Monitored Tokens

0xDVTA tracks 13 Solana tokens including Bags-native launches:

### Bags Ecosystem (Meteora DAMM v2)
- **LORIA** — Bags-native token
- **GSD** — Bags-native token  
- **CMEM** — Bags-native token

### Solana Majors (Benchmark)
FARTCOIN, AI16Z, GRIFFAIN, ZEREBRO, GOAT, ARC, SWARMS, PIPPIN, NEUR, ELIZA

---

## Hybrid Privacy Model

0xDVTA uses a dual-AI architecture for privacy-preserving analysis:

- **Venice AI** (Llama 3.3 70B) — Signal tracking and sensitive analysis. Runs through Venice's private inference (staked VVV tokens). No data logging.
- **Gemini Flash** — Synthesis and reporting. Receives only anonymized, aggregated data. No wallet addresses or position data exposed.

This ensures forensic intelligence stays private while leveraging frontier AI for analysis quality.

---

## RULE_BOOK — Autonomous Trading Filters

The agent follows a strict rule book before entering any position:

```python
# Absolute blockers (Solana-calibrated)
MAX_ICR  = 30     # Crash risk ceiling
MIN_FHS  = 5.5    # Minimum forensic health
MIN_BPI  = 0.5    # Minimum bull power

# Dynamic filters (from enrichment layer)
lcr_vel  >= 0.05  # Liquidity must be accelerating
nbp_zscore >= 0   # Buy pressure above historical mean
wcc_vel  >= 0     # Whales not exiting

# Safety
BLOCKED_PHASES = ["DISTRIBUTION"]
MAX_POSITION_SIZE = dynamic (based on liquidity depth)
```

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| **Runtime** | Python 3.11, GCP VPS (Ubuntu) |
| **Data** | GeckoTerminal API, Bags API |
| **Forensics** | Custom engine v6 (65+ metrics) |
| **Enrichment** | Rolling velocity/z-score computation |
| **AI — Private** | Venice AI (Llama 3.3 70B via VVV stake) |
| **AI — Public** | Gemini Flash (anonymized synthesis) |
| **Execution** | Jupiter Aggregator (Solana swaps) |
| **Wallet** | Solana native (`BRKz...kyk`) |
| **Scheduling** | Cron (2h cycles, odd hours UTC) |
| **Quant Stack** | SHAP, Lasso, rolling statistics, signature matching |
| **Frontend** | GitHub Pages (Solana violet theme) |

---

## Ecosystem Partners Used

<p align="center">
  <img src="https://img.shields.io/badge/Solana-9945FF?style=flat-square&logo=solana&logoColor=white" />
  <img src="https://img.shields.io/badge/Meteora-FF6B35?style=flat-square" />
  <img src="https://img.shields.io/badge/Jupiter-4FC08D?style=flat-square" />
  <img src="https://img.shields.io/badge/GeckoTerminal-48DB8B?style=flat-square" />
  <img src="https://img.shields.io/badge/Venice%20AI-000000?style=flat-square" />
  <img src="https://img.shields.io/badge/Gemini-4285F4?style=flat-square&logo=google&logoColor=white" />
  <img src="https://img.shields.io/badge/Bags.fm-FF6B35?style=flat-square" />
</p>

---

## Lineage

0xDVTA is a Solana fork of **0xDELTA**, an autonomous forensic trading agent registered as **ERC-8004 Agent #32715** on Base chain. The parent agent has been running autonomously since February 2026 with a full pipeline including on-chain report sealing ($0.05 USDC per forensic scan to a dedicated Forensic Wallet) and x402 paywalled access.

The Solana fork adapts the forensic engine for Solana's liquidity structure (higher ICR, lower FHS/BPI baselines) and integrates Bags-native token monitoring via Meteora DAMM v2 pools.

---

## Running

```bash
# Full pipeline (runs via cron every 2h)
./run_pipeline.sh

# Individual steps
python3 collect_tokens_solana.py          # Fetch token data
python3 forensic_engine_v6.py data.json   # Compute forensics
python3 enrich_forensic.py                # Add velocities & signals
python3 request_analysis_sol.py           # AI analysis
python3 enter_new_position.py             # Execute trades (when enabled)
```

---

## License

MIT

---

<p align="center">
  <b>Built for the Bags Hackathon 2026</b><br/>
  <i>Autonomous forensic intelligence for the Bags ecosystem</i>
</p>
