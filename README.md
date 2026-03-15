# FRIDGE-RAG

Turn a fridge snapshot into practical, constraint-aware meal options using computer vision + retrieval-augmented generation.

---

## Why this project exists

Most "what can I cook?" demos stop at ingredient detection. In real kitchens, users also care about:

- **What they can cook right now** with available items.
- **How confident the system is** about detected ingredients.
- **Why a recipe was recommended** (traceable retrieval evidence).
- **How to run the stack reliably** in a production-like environment.

FRIDGE-RAG is structured as a real-world prototype that separates ingestion, retrieval, ranking, and serving concerns so the system can evolve without a full rewrite.

---

## System architecture (real-world oriented)

### 1) Runtime request flow

```text
[Client App / Streamlit UI]
            |
            v
      [FastAPI Gateway]
            |
            v
   [Pipeline Orchestrator]
     |               |
     |               +--------------------------+
     v                                          v
[Vision Ensemble]                          [RAG Service]
(YOLO + DETR + CLIP)        query ---> [Embedding + Chroma Retrieval]
     |                                          |
     +------ detected ingredients --------------+
                         |
                         v
               [LLM Re-ranker / Explainer]
                         |
                         v
         [Ranked recipe candidates + rationale]
```

### 2) Offline data/index flow

```text
[Kaggle recipe dataset]
          |
          v
 [scripts/build_vectordb.py]
  - cleaning
  - chunking/formatting
  - embedding generation
          |
          v
   [Local Chroma recipe index]
```

### 3) Production responsibilities by layer

- **API layer (`api/`)**
  - Input validation and response contracts.
  - Health endpoints and request-level error handling.
- **Orchestration layer (`src/pipeline.py`)**
  - Coordinates vision inference, retrieval, and reranking.
  - Keeps business flow centralized and testable.
- **Vision layer (`src/vision/`)**
  - Multi-model ingredient detection to reduce single-model blind spots.
- **RAG layer (`src/rag/`)**
  - Builds and queries recipe embeddings.
  - Supports semantic matching over free-form ingredient context.
- **UI layer (`dashboard/`)**
  - Thin client for demos and operator validation.

---

## Repository layout

```bash
FRIDGE-RAG/
├── api/                  # FastAPI app + schemas
├── dashboard/            # Streamlit front-end
├── scripts/              # Offline jobs (index build)
├── src/
│   ├── pipeline.py       # End-to-end orchestration
│   ├── config.py         # Runtime configuration
│   ├── rag/              # Ingestion, retrieval, reranking
│   └── vision/           # YOLO/DETR/CLIP + ensemble logic
├── tests/                # Unit/integration-oriented tests
├── requirements.txt
└── README.md
```

---

## Technology choices

- **Backend API:** FastAPI + Uvicorn
- **UI:** Streamlit
- **Vision models:** YOLOv8, DETR, CLIP (ensemble strategy)
- **Embeddings:** sentence-transformers (`all-MiniLM-L6-v2`)
- **Vector store:** ChromaDB
- **LLM layer:** OpenAI models for reranking/explanations
- **Dataset:** Food.com Recipes and Reviews (Kaggle)

Dataset: https://www.kaggle.com/datasets/irkaal/foodcom-recipes-and-reviews/data

---

## Quick start

### Prerequisites

- Python 3.10+
- `pip` and virtual environment tooling
- Kaggle API credentials
- OpenAI API key

### Install

```bash
git clone <your-repo-url>
cd FRIDGE-RAG
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Configure env

```bash
cp .env.example .env
```

Set:

```env
OPENAI_API_KEY=your_openai_key
```

### Configure Kaggle credentials

```bash
mkdir -p ~/.kaggle
echo '{"username":"YOUR_KAGGLE_USERNAME","key":"YOUR_KAGGLE_KEY"}' > ~/.kaggle/kaggle.json
chmod 600 ~/.kaggle/kaggle.json
```

### Build vector index

```bash
python scripts/build_vectordb.py
```

---

## Run locally

### API

```bash
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```

### Dashboard

```bash
streamlit run dashboard/app.py
```

---

## Testing

```bash
pytest tests/ -v
```

Optional:

```bash
pytest tests/test_pipeline.py -v
pytest tests/test_vision.py -v
pytest tests/test_rag.py -v
```

---

## Operational notes

- `data/` artifacts are intentionally local and typically git-ignored.
- Rebuild the vector index whenever recipe corpus or embedding model changes.
- For deployment hardening, add:
  - containerized services,
  - centralized logging/tracing,
  - secret management,
  - model/version pinning,
  - request throttling and caching.

---

## Roadmap

- Add dietary and allergen-aware filters.
- Add retrieval quality evaluation (Precision@K / Recall@K).
- Add Dockerized local stack and CI checks.
- Add feedback loop for improving ranking quality.
