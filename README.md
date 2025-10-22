# 💹 RebalanceAI — AI-Powered Investment Portfolio Analyzer

**One-sentence summary:**  
RebalanceAI is a LangChain-powered portfolio co-pilot that fuses **quantitative finance** (yfinance) with **social/news sentiment** (Reddit) and **LLM rationale** to propose **transparent, risk-aware** allocations and **explainable** trade-offs.  

> 🧩 *Educational use only — not financial advice.*

---

## 🚀 Live Demo

**Hosted Streamlit (ngrok):**  
👉 [https://stethoscoped-nondiocesan-cohen.ngrok-free.dev/](https://stethoscoped-nondiocesan-cohen.ngrok-free.dev/)  

> ⚠️ The ngrok link may change or expire.  
> If unavailable, follow the **“Run in Google Colab”** section below to recreate the app in minutes.

---

## ✨ Features

- **Single LangChain Agent** coordinating tools for data ingest, analytics, sentiment fusion, and allocation.
- **Quant + Sentiment Fusion**: annual return, volatility, Sharpe ratio + Reddit sentiment (+ optional LLM sentiment).
- **Risk-aware allocation** with weight caps, explainable scoring, and full portfolio statistics.
- **Two-Tab Streamlit UI:**
  - 🧮 **Analyzer** — structured input (tickers, risk, cap, amount) → weights · shares · stats · chart vs SPY  
  - 💬 **Chat** — natural language requests (e.g., “$10 k, moderate risk, tech emphasis, show sentiment impact”)
- **Caching** for yfinance + local CSV/JSON sentiment files to handle API limits.
- **Runs in Colab or locally** with a single cell launch.

---

## 🧱 Architecture

```
yfinance → prices → returns → [mean, vol, Sharpe]
                                   │
Reddit CSV → baseline sentiment ───┤ → Fusion Score (return↑, risk↓, sentiment↑) + cap → weights
LLM JSON (optional) ───────────────┘
weights + prices + returns → stats + shares + charts
```

**Layers**

| Layer | Description |
|-------|--------------|
| **Data** | yfinance prices, Reddit posts (+ cached CSV), optional LLM sentiment JSON |
| **Analytics** | pandas / NumPy / scikit-learn for returns · vol · Sharpe · fusion scoring |
| **Agent (optional)** | LangChain tool orchestration for metrics + sentiment + allocation |
| **UI** | Streamlit (Analyzer & Chat tabs) |

---

## ▶️ Run in Google Colab

1️⃣ **Install dependencies**

```python
!pip -q uninstall -y langchain langchain-core langchain-community langchain-openai langgraph langgraph-prebuilt langchain-classic || true
!pip -q install "numpy==2.0.2" "pandas==2.2.2" "pyarrow==17.0.0" "requests==2.32.4"
!pip -q install yfinance praw python-dotenv scikit-learn matplotlib plotly streamlit pyngrok
```

2️⃣ **Ensure app file exists**

Your notebook already writes `/content/app.py`.  
If not, copy the latest version into a cell and execute:
```python
from pathlib import Path
Path("/content/app.py").write_text(app_code)
```

3️⃣ **Create sample demo data**

```python
from pathlib import Path; import pandas as pd, json
DATA = Path("/content/data"); DATA.mkdir(parents=True, exist_ok=True)
pd.DataFrame({"Symbol":["AAPL","MSFT","NVDA","AMZN","GOOGL","META","TSLA"]}).to_csv(DATA/"sp500.csv",index=False)
(Path(DATA/"llm_sentiment.json")).write_text(json.dumps({"NVDA":{"sentiment":0.6},"AAPL":{"sentiment":0.2}},indent=2))
```

4️⃣ **Launch Streamlit via ngrok**

```python
from pyngrok import ngrok; import os, subprocess, time
os.environ["NGROK_AUTHTOKEN"]="YOUR_NGROK_TOKEN"
ngrok.kill(); ngrok.set_auth_token(os.environ["NGROK_AUTHTOKEN"])
try: subprocess.run(["pkill","-f","streamlit"], check=False)
except: pass
subprocess.Popen(["streamlit","run","/content/app.py","--server.port","8501","--server.headless","true"])
time.sleep(3)
print("Public URL:", ngrok.connect(8501, bind_tls=True).public_url)
```

Open the printed public URL in a new tab ✅

---

## 💻 Run Locally (VS Code / Terminal)

```bash
python -m venv .venv && source .venv/bin/activate
pip install "numpy==2.0.2" "pandas==2.2.2" "pyarrow==17.0.0" "requests==2.32.4"             yfinance praw python-dotenv scikit-learn matplotlib plotly streamlit
export REDDIT_CLIENT_ID=... REDDIT_CLIENT_SECRET=... REDDIT_USER_AGENT="rebalanceai/0.1"  # optional
streamlit run app.py
```

---

## 🧮 Methodology

| Component | Logic |
|------------|--------|
| **Metrics** | Daily returns → Annual Return = mean × 252; Vol = std × √252; Sharpe = (mean – RF)/vol (RF = 0.02) |
| **Sentiment** | Reddit post scores → rescaled to [–1, 1]; optional LLM cache adds rationale |
| **Fusion Scoring** | Normalize (ret_n, risk_n [inv vol], senti_n) → weighted sum (profile-dependent) |
| **Profiles** | Conservative (0.3, 0.5, 0.2) · Moderate (0.5, 0.3, 0.2) · Aggressive (0.6, 0.2, 0.2) |
| **Allocation** | Clip weights ≤ max_w (25%) → renormalize → portfolio stats + chart vs SPY |

---

## 💬 Example Interactions

- “Analyze a **$10,000** portfolio for **moderate risk** and **tech emphasis**. Show **sentiment impact**.”  
- “Compare **TSLA vs NVDA** on **volatility**, **Sharpe**, and recent **news sentiment**.”  
- “Propose a **diversified S&P 500 mini-basket** for a **conservative** profile. Explain trade-offs.”  
- “I have **$5 k** and want **20% annual returns** — which 5 stocks fit best?”

---

## 🧪 Sample Data  (`/content/data/`)

| File | Purpose |
|------|----------|
| `sp500.csv` | Subset of valid S&P 500 symbols |
| `reddit_<TICKER>.csv` | Cached Reddit posts + scores |
| `llm_sentiment.json` | Optional LLM sentiment overrides |

---

## 📊 Evaluation / Rubric Mapping

| Rubric Criterion | Evidence |
|------------------|-----------|
| **Problem & Vision** | README intro · Slides 2-3 |
| **LangChain Agent Design** | Analyzer + Chat flows · app.py functions |
| **Multi-LLM Evaluation** | Cached sentiment JSON · slides on evaluation |
| **Data Use & Pipeline** | yfinance · Reddit · fusion logic |
| **Presentation & Demo** | Live Streamlit UI + 6 min deck |
| **Innovation & Creativity** | Chat parser · fusion scoring · risk explanations |

---

## ⚠️ Limitations

- Historical metrics ≠ future performance  
- Reddit sentiment can be noisy/manipulated  
- yfinance adjusted data may shift slightly  
- ≥ 20 % targets imply higher risk/concentration

---

## 🧰 Troubleshooting

| Issue | Fix |
|-------|-----|
| **ngrok 4018 / auth error** | Set `NGROK_AUTHTOKEN` in Colab cell |
| **Stats = 0 or NaN** | Use valid tickers (from `sp500.csv`) |
| **Rate limits / timeouts** | Cached CSV/JSON auto-loaded |
| **Dependency conflicts** | Use the pinned versions above |

---

## 🏁 Demo Script (6 Minutes)

1. **Intro & Vision** (1 min)  
2. **Architecture & Agent Design** (1 min)  
3. **Analyzer Tab Walkthrough** (2 min)  
4. **Chat Tab Prompts × 3** (1.5 min)  
5. **Limitations + Q&A** (0.5 min)

---

## 👥 Team Contributions

| Member | Responsibility |
|---------|----------------|
| Sankalp | Agent & prompt design & Sentiment ingestion |
| Rohan | Quant metrics & fusion & cache |
| Sachin | Streamlit UI & demo |

---

## 🔐 Responsible AI

- Transparent reasoning and risk disclosure  
- No personal data collection  
- Optional LLM sentiment module  
- Educational only — not investment advice

---

## 🧾 License

MIT © 2025 RebalanceAI Team
