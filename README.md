# FinanceImpact 📈

> A full-stack financial intelligence platform powered by a 7-stage ML pipeline, corporate ripple propagation, and real-time market analysis.

![Version](https://img.shields.io/badge/version-v7.0-blue)
![Python](https://img.shields.io/badge/python-3.13-blue)
![Streamlit](https://img.shields.io/badge/streamlit-1.36+-red)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What is FinanceImpact?

FinanceImpact lets you submit any financial news headline and receive a multi-dimensional ML-driven analysis in seconds. It tells you not just *how* a headline affects a company, but *which subsidiaries and partners* absorb the ripple — modelling corporate impact the way it actually works in markets.

**Example:** Paste *"Tesla faces SEC probe over Autopilot data"* → get sentiment polarity, event classification, rumour probability, word-level attribution, historical analogues with 3-day price predictions, VIX-adjusted macro context, and a full corporate ripple tree showing impact on Tesla Energy, Autopilot AI LLC, Dojo Compute, and more.

---

## Features

- **7-Stage ML Pipeline** — NER → Event Classification → FinBERT Sentiment → Rumour Detection → SHAP Attribution → Historical Similarity → Macro Context (VIX)
- **Corporate Ripple Engine** — Directed graph propagation through subsidiary trees using ownership %, relationship type, and depth decay
- **Market Analysis** — OHLCV charts, 15+ technical indicators (MACD, RSI, Bollinger Bands, VWAP, ATR), multi-ticker comparison, live global index tracking
- **Live News Feed** — 10 RSS sources with quick sentiment scoring and one-click pipeline routing
- **Watchlist & History** — Portfolio tracker with live prices, full analysis history with re-analyze and CSV export
- **Secure Auth** — bcrypt password hashing (12 rounds), session-scoped guards on every page

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Web Framework | Streamlit ≥ 1.36 | Multi-page app, session state, sidebar navigation |
| ML — Sentiment | ProsusAI/FinBERT | Financial sentiment classification (pos/neg/neutral) |
| ML — Embeddings | all-MiniLM-L6-v2 | 384-dim semantic similarity for historical search |
| ML — Fallback | scikit-learn TF-IDF | Similarity search when transformer unavailable |
| Market Data | yFinance ≥ 0.2.36 | OHLCV, live prices, VIX, global indices |
| Graph Engine | NetworkX ≥ 3.2 | Directed corporate hierarchy + ripple propagation |
| Charts | Plotly ≥ 5.18 | Interactive candlestick, line, bar, heatmap charts |
| Database | SQLite 3 (WAL mode) | Users, history, watchlist, settings |
| Auth | bcrypt ≥ 4.1 | Salted password hashing |
| RSS | feedparser ≥ 6.0 | Live headline ingestion from 10 financial sources |

---

## The 7-Stage ML Pipeline

Every headline passes sequentially through seven analytical stages:

```
User Input (headline + ticker)
    │
    ▼
Stage 1 ── NER                  Keyword dict → detected tickers
    │
    ▼
Stage 2 ── Event Classification  Regex (9 categories) → multiplier [×0.70 – ×1.35]
    │
    ▼
Stage 3 ── FinBERT Sentiment     HuggingFace transformer → polarity [-1, 1]
    │
    ▼
Stage 4 ── Rumour Detection      25 signal phrases → credibility score
    │
    ▼
Stage 5 ── SHAP Attribution      Financial lexicon → top-10 word contributions
    │
    ▼
Stage 6 ── Historical Similarity Sentence Transformer + cosine → top-5 analogues + T+3 prediction
    │
    ▼
Stage 7 ── Macro Context         Live VIX fetch → asymmetric amplification factor
    │
    ▼
    └──► Ripple Engine → Result Dict → DB → UI
```

### Polarity Formula

```
1. raw_pol, conf  = finbert_score(headline)
2. evt_mult       = EVENT_MULTIPLIER[event_type]        # ×0.70 – ×1.35
3. polarity       = clamp(raw_pol × evt_mult, -1, 1)
4. if is_rumour:  polarity = polarity × credibility
5. macro_f        = vix_amplifier(vix, polarity)        # asymmetric
6. final_polarity = clamp(polarity × macro_f, -1, 1)
```

---

## Corporate Ripple Engine

When a parent company is impacted, the signal propagates through its subsidiary tree using three decay factors:

```
impact(child) = impact(parent) × ownership_factor × relationship_decay × depth_decay
```

| Relationship | Decay | Example |
|---|---|---|
| wholly_owned | 1.00 | Tesla → Tesla Energy |
| majority_owned | 0.90 | >50% stake |
| division | 0.95 | Integrated business unit |
| joint_venture | 0.70 | Shared ownership |
| strategic_investment | 0.50 | Minority stake |
| investment | 0.40 | Passive investment |

| Depth | Decay |
|---|---|
| 1 (direct child) | 1.00 |
| 2 (grandchild) | 0.65 |
| 3 (great-grandchild) | 0.30 |
| 4+ | 0.15 |

**Example** — Tesla polarity = −0.72:
- Tesla Energy (wholly owned, depth 1) → impact **−0.720**
- Autopilot AI LLC (division, depth 1) → impact **−0.684**
- Dojo Compute (wholly owned, depth 2) → impact **−0.468**

---

## Project Structure

```
miniproj/
├── app.py                        # Entry point — auth gate, st.navigation(), sidebar
├── requirements.txt
├── .streamlit/
│   └── config.toml               # Dark theme tokens, server settings
├── pages/
│   ├── 0_Home.py                 # Landing page — feature cards, stats, quick access
│   ├── 1_Dashboard.py            # Headline Analyzer — 6-tab result view
│   ├── 2_News.py                 # Live RSS feed with sentiment scoring
│   ├── 3_Watchlist.py            # Portfolio tracker with live prices
│   ├── 4_History.py              # Analysis history — filter, re-analyze, export
│   └── 5_Settings.py             # User preferences
├── core/                         # Pure Python — no Streamlit imports
│   ├── engine.py                 # 7-stage ML pipeline
│   ├── graph.py                  # Corporate hierarchy + ripple propagation
│   ├── stocks.py                 # yFinance wrapper + technical indicators
│   ├── feeds.py                  # RSS ingestion + quick sentiment scorer
│   └── seeder.py                 # 200+ historical headline dataset generator
├── db/
│   ├── schema.py                 # SQLite DDL, connection factory, seed users
│   └── ops.py                    # All CRUD — users, history, watchlist, settings
├── ui/
│   ├── auth.py                   # Login/logout, session state, require_login()
│   ├── theme.py                  # Global CSS injection, Plotly theme, SVG icons
│   └── components.py             # Reusable HTML components (cards, badges, bars)
└── data/
    ├── corporate_hierarchy.json  # 11-company graph — subsidiaries & decay config
    ├── news.csv                  # Auto-generated historical headline dataset
    └── finance_impact.db         # SQLite database (created at runtime)
```

---

## Getting Started

### Prerequisites

- Python 3.13+
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/miniproj.git
cd miniproj

# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the application
streamlit run app.py
```

App will be available at **http://localhost:8501**

### Default Accounts

| Username | Password | Role |
|---|---|---|
| admin | admin123 | Admin |
| demo | demo1234 | User |
| guest | guest123 | User |

> ⚠️ Change these credentials before any production deployment.

---

## Configuration

### `.streamlit/config.toml`

```toml
[theme]
base                   = "dark"
backgroundColor        = "#07090D"
secondaryBackgroundColor = "#0B0F16"
textColor              = "#DDE6F0"
primaryColor           = "#00C8F0"

[server]
headless = true
enableCORS = false
```

### Environment Variables

No API keys are required for current functionality — yFinance is unauthenticated. For future integrations, add secrets via Streamlit Cloud's Secrets Manager (never hardcode):

```toml
# .streamlit/secrets.toml (local only — add to .gitignore)
OPENAI_API_KEY = "sk-..."
ALPHA_VANTAGE_KEY = "..."
```

---

## Technical Indicators

The market analysis module computes 15+ indicators without TA-Lib — pure pandas/numpy:

| Indicator | Column | Method |
|---|---|---|
| MA 20 / 50 / 200 | MA20, MA50, MA200 | Rolling mean |
| EMA 12 / 26 | EMA12, EMA26 | Exponential weighted mean |
| MACD + Signal + Histogram | MACD, MACD_Signal, MACD_Hist | EMA difference |
| Bollinger Bands | BB_Upper, BB_Lower, BB_Mid | MA20 ± 2σ |
| RSI (14) | RSI | Wilder smoothing |
| ATR (14) | ATR | True range rolling mean |
| VWAP | VWAP | Cumulative volume-weighted price |

---

## Database Schema

Four tables in SQLite (WAL mode):

- **users** — id, username, password_hash, email, role, created_at, last_login
- **analysis_history** — full pipeline result per analysis, serialised as JSON
- **watchlist** — user_id + ticker with UNIQUE constraint
- **user_settings** — per-user preferences, upserted via INSERT OR REPLACE

---

## Deployment

### Streamlit Cloud (Current)

1. Push to GitHub (`main` branch)
2. Connect repo at [share.streamlit.io](https://share.streamlit.io)
3. Set entry point to `app.py`
4. Add any secrets via the Secrets Manager

> **Note:** SQLite resets on each deploy on Streamlit Cloud. For persistent storage, migrate to PostgreSQL (Supabase free tier recommended).

### Production Upgrade Path

| Component | Current | Recommended |
|---|---|---|
| Database | SQLite | PostgreSQL / Supabase |
| Hosting | Streamlit Cloud | Railway / Render / AWS |
| Models | Loaded per instance | Redis cache / model server |
| Rate limiting | None | FastAPI middleware |
| Billing | None | Stripe Checkout |

---

## RSS News Sources

| Source | Coverage |
|---|---|
| Reuters Business & Tech | Global business, technology |
| Economic Times Markets | Indian markets |
| Moneycontrol | Indian markets |
| Yahoo Finance | Global finance |
| Bloomberg Markets | Global markets |
| CNBC Business | US business |
| Livemint | Indian finance |
| Seeking Alpha | Equities analysis |

---

## Supported Tickers (NER)

11 companies with full corporate hierarchy trees:

`TSLA` · `AAPL` · `NVDA` · `RELIANCE` · `HDFCBANK` · and 6 more defined in `corporate_hierarchy.json`

New companies can be added to `data/corporate_hierarchy.json` without any code changes.

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Keep all business logic in `core/` — no Streamlit imports there
4. Keep pages thin — they import from `core/`, `db/`, and `ui/` only
5. Submit a pull request with a clear description

---

## Roadmap

- [ ] PostgreSQL migration for persistent production storage
- [ ] REST API layer (FastAPI) over `core/` for programmatic access
- [ ] Stripe billing integration with tiered feature gating
- [ ] Expand NER dictionary beyond 11 tickers
- [ ] Webhook alerts for watchlist price movements
- [ ] Multi-language headline support

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Finance Impact v7.0 · Built with Streamlit, FinBERT, NetworkX, and yFinance*
