# LLM Summarization & Thematic Filtering Pipeline

Notebook-based implementation for **long-document** company intelligence: ingest ultra-long, unstructured text (e.g., Wikipedia), extract structured signals with a MapReduce-style workflow, and support downstream thematic research (search, filtering, portfolio construction, backtests).

## What This Repo Contains

This repository is intentionally lightweight and currently **centered around notebooks**:

- `MASTER_DOC_20251202_Data_Warehousing.ipynb`: end-to-end working notebook focused on building a modular, resumable data warehouse (MongoDB as pipeline state), plus downstream experiments (e.g., embeddings/search/backtesting).
- `Documents.ipynb`: design/spec notebook describing the full multi-part pipeline (data warehouse, long-doc summarization, semantic search, thematic filtering, and deliverables).

## Key Ideas

- **Long-doc constraint is the default**: single documents can be 10k-20k+ words, so the system is designed around chunking + synthesis.
- **MapReduce-style summarization**:
  - Map: process chunks independently to extract material points
  - Reduce: synthesize chunk-level signals into a concise, structured brief
- **Reliability over “pretty text”**: treat LLMs as fallible components; keep artifacts, validate, and make stages resumable.
- **Thematic filter taxonomy**: classify per theme into `Core` / `Secondary` / `Not Relevant` to prevent “semantic match” from polluting investable baskets.

## Repository Layout

```text
.
├── MASTER_DOC_20251202_Data_Warehousing.ipynb
├── Documents.ipynb
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.10+ (tested with Python 3.14)
- Jupyter (Lab or Notebook)
- MongoDB (local or hosted) if you want the resumable “pipeline state” workflow

### Setup (Recommended)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

### Run

```bash
jupyter lab
```

Open `MASTER_DOC_20251202_Data_Warehousing.ipynb` and follow the sections in order.

## Configuration

The main notebook expects a MongoDB connection string (see `MONGO_URI` in `MASTER_DOC_20251202_Data_Warehousing.ipynb`).

Suggested approach:
- Put secrets in an env var (recommended) and load it in the notebook
- Do not hardcode credentials into the notebook or commit them

## Data Model (MongoDB as Pipeline State)

The “self-healing” idea is: each stage only processes docs that are missing a given field, and writes progress back to Mongo so reruns resume safely.

Typical fields (illustrative):
- `ticker`, `company_name`
- `wiki_content`: long-form text
- `wiki_vcard`: parsed infobox/vcard metadata (when available)
- `wiki_resolver`: which resolver produced the page (e.g., `wikipedia`, `bing`, `yfinance`)
- `..._status`: stage markers to support retries/repair

## Pipeline Stages (Conceptual)

1. **Ingest / Resolve** (implemented in `MASTER_DOC_20251202_Data_Warehousing.ipynb`)
   - Multi-pass resolution strategy (primary + fallbacks)
   - Data quality checks and “unset to retry” healing pattern
2. **Chunking** (documented; partial experiments)
   - Baseline word-count chunking
   - Upgrades: sentence-aware / structural / semantic chunking
3. **Summarization (MapReduce)** (documented; can be wired into the warehouse)
   - Map: extract 1-3 material points per chunk
   - Reduce: synthesize a structured brief suitable for embedding/search
4. **Embeddings / Semantic Search / Theme Matching** (experiments in notebook)
5. **LLM-as-a-Filter** (documented)
   - `Core` vs `Secondary` vs `Not Relevant` classification per theme
6. **Portfolio Construction + Backtesting** (experiments in notebook)

## Project Status

- This is a **collaborative university finance group project** (team-written).
- The pipeline is **runnable and working end-to-end** in `MASTER_DOC_20251202_Data_Warehousing.ipynb` (warehouse + downstream experiments).
- The roadmap items below are optional improvements to modularize the notebook into scripts/packages over time (not required to run the current pipeline).

## Roadmap (If You Want This as a “Real” Python Package)

- Extract reusable modules from notebook:
  - `ingest/`, `resolvers/`, `dq/`, `chunking/`, `summarize/`, `embed/`, `filter/`
- Add typed schemas + validation for all intermediate artifacts
- Add CLI entrypoints (e.g., `python -m pipeline ingest`, `... summarize`)
- Add small golden datasets + tests for chunking and schema validation

## Notes / Caveats

- Many steps depend on external websites/APIs and can be rate-limited.
- Wikipedia/company names are ambiguous; fallbacks and DQ checks are not optional if you want stability.
- If you integrate LLM calls, prefer structured JSON outputs + schema validation.
