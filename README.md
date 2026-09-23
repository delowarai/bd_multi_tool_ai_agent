<div align="center">

# 🇧🇩 BD Agent
### Multi-Tool AI Agent for Bangladesh

**An LLM-powered agent that answers real questions about Bangladeshi hospitals, institutions, and restaurants - grounded in real datasets, not hallucinations.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/LangChain-Agent%20Framework-1C3C3C?style=flat)](https://www.langchain.com/)
[![SQLite](https://img.shields.io/badge/SQLite-Database-003B57?style=flat&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![HuggingFace](https://img.shields.io/badge/🤗%20Datasets-HuggingFace-FFD21E?style=flat)](https://huggingface.co/datasets)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#license)

[Overview](#-overview) • [Demo](#-example-queries) • [Architecture](#-architecture) • [Setup](#-getting-started) • [Skills Demonstrated](#-skills-demonstrated)

</div>

---

## 📌 Overview

**BD Agent** is a tool-using AI agent built with **LangChain** that answers questions about Bangladesh by querying **real structured data** instead of relying purely on an LLM's memory. It combines:

- 🗄️ **Three real-world datasets** (hospitals, educational institutions, restaurants) ingested from HuggingFace into local SQLite databases
- 🧠 **An LLM-based agent** that reasons about which tool best answers a given question
- 🌐 **A live web-search tool** for general knowledge questions outside the datasets

The result: a domain-specific assistant that gives **accurate, data-backed answers** - a practical demonstration of **retrieval-augmented, tool-using AI systems**, one of the most in-demand patterns in applied AI/ML engineering today.

## Example Queries

```
> How many hospitals are in Dhaka?
> List hospitals with ICU facilities in Chittagong.
> What universities are listed in the institutions database?
> Show popular restaurants serving Bengali cuisine in Sylhet.
> What is the role of DGHS in Bangladesh?
```

The agent automatically routes each question to the correct tool - SQL query against the right database, or a live web search - without the user needing to specify which.

---

## Architecture

```
                         ┌──────────────────────┐
                         │     User Question     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                       ┌────────────────────────┐
                       │   agent.py (LangChain)  │
                       │  LLM decides best tool  │
                       └────────────┬───────────┘
                                    │
          ┌─────────────┬──────────┼──────────┬─────────────┐
          ▼              ▼          ▼          ▼             
   ┌─────────────┐┌─────────────┐┌─────────────┐┌──────────────┐
   │ Hospitals   ││ Institutions││ Restaurants ││  Web Search  │
   │  SQL Tool   ││   SQL Tool  ││   SQL Tool  ││  (SerpAPI)   │
   └──────┬──────┘└──────┬──────┘└──────┬──────┘└──────────────┘
          ▼              ▼              ▼
   dbs/hospitals.db dbs/institutions.db dbs/restaurants.db
          │              │              │
          └──────────────┴──────────────┘
                          ▼
                  ┌───────────────┐
                  │ Final Answer  │
                  └───────────────┘
```

**Data pipeline:** `ingest.py` pulls each dataset from HuggingFace → converts to a pandas DataFrame → normalizes column names → writes to a local SQLite `.db` file via `to_sql()`.

---

## Project Structure

```
bd_multi_tool_ai_agent/
├── agent.py                  # Entry point — builds & runs the LangChain agent
├── ingest.py                 # Downloads datasets & builds SQLite databases
├── tools/
│   ├── db_tools.py           # SQL tools: hospitals, institutions, restaurants
│   └── web_search_tool.py    # SerpAPI web-search tool
├── requirements.txt          # Python dependencies
├── .env.example               # API key template
└── .gitignore                 # Excludes secrets, DBs, venv
```

### Datasets

| Source (HuggingFace) | Local Table |
|---|---|
| `Mahadih534/Institutional-Information-of-Bangladesh` | `institutions` |
| `Mahadih534/all-bangladeshi-hospitals` | `hospitals` |
| `Mahadih534/Bangladeshi-Restaurant-Data` | `restaurants` |

Adding a new dataset is a one-line change - add a `save_dataset_to_db(dataset_name, table_name, db_path)` call in `ingest.py`.

---

## Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/delowarhossaincse63/bd_multi_tool_ai_agent.git
cd bd_multi_tool_ai_agent
```

### 2. Create a virtual environment
```bash
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS / Linux
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
```bash
copy .env.example .env        # Windows
cp .env.example .env          # macOS / Linux
```
Then add your keys to `.env`:
```
OPENAI_API_KEY=your_openai_key_here
SERPAPI_API_KEY=your_serpapi_key_here
```

### 5. Build the databases
```bash
python ingest.py
```
Creates `dbs/hospitals.db`, `dbs/institutions.db`, `dbs/restaurants.db`.

### 6. Run the agent
```bash
python agent.py                                   # interactive mode
python agent.py "How many hospitals are in Dhaka?" # direct query
```

---

## Skills Demonstrated

This project showcases practical, job-relevant experience with:

- **LLM Agent Design** — building tool-using agents with LangChain that reason about task routing
- **Data Engineering** — ETL pipeline: dataset ingestion, cleaning, normalization, and loading into SQLite
- **API Integration** — OpenAI API and SerpAPI, with secure key management via `.env`
- **Database Design** — schema normalization, SQL query tools, de-duplication logic
- **Software Engineering Practices** — modular code structure, `.gitignore` hygiene, secret management, reproducible setup
- **Applied AI for Real-World Data** — grounding LLM output in verified datasets to reduce hallucination

---

## Verifying the Databases

```bash
sqlite3 dbs/hospitals.db "PRAGMA table_info('hospitals');"
sqlite3 dbs/hospitals.db "SELECT COUNT(*) FROM hospitals;"
```

Or in Python:
```python
import sqlite3
conn = sqlite3.connect('dbs/hospitals.db')
cur = conn.cursor()
print(cur.execute("SELECT name FROM sqlite_master WHERE type='table';").fetchall())
print(cur.execute("SELECT count(*) FROM hospitals;").fetchone())
conn.close()
```

---

## Security Notes

The following are excluded from version control via `.gitignore`:
- `.env` — API keys
- `dbs/*.db` — generated database files
- `venv/` / `.venv/` — virtual environment
- `__pycache__/`, `*.pyc`

If a secret is ever committed accidentally:
```bash
git rm --cached .env
git commit -m "Remove .env from tracking"
git commit --amend --no-edit
git push -f origin main
```

---

## Troubleshooting

| Issue | Solution |
|---|---|
| `OPENAI_API_KEY is required` | Confirm `.env` exists and contains the key; restart the terminal. |
| GitHub blocks push (secret detected) | Remove the secret, amend the commit, force-push. |
| Agent gives unexpected answers | Check console logs for tracebacks; verify all keys in `.env`. |
| Web search not working | Confirm `SERPAPI_API_KEY` is set — without it, only SQL tools work. |

---

## Roadmap

- [ ] Add more Bangladesh-specific datasets (transport, weather, government services)
- [ ] Build a lightweight web UI (Streamlit/FastAPI)
- [ ] Add caching for repeated queries
- [ ] Deploy as a public demo

---

## Contributing

Contributions, issues, and feature requests are welcome. Feel free to check the [issues page](https://github.com/delowarhossaincse63/bd_multi_tool_ai_agent/issues).

## License

This project is licensed under the MIT License.

## Contact

**Delowar Hossain**
📧 delowar.swe@gmail.com
🔗 [GitHub](https://github.com/delowarhossaincse63)

---

<div align="center">

 If you find this project useful, consider giving it a star!

</div>
