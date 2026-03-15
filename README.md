# FridgeRAG

Multimodal recipe recommender: snap a fridge photo → detect ingredients
via YOLOv8 + DETR + CLIP ensemble → retrieve and rank recipes via RAG.

## Quick Start

# 1. Install dependencies
pip install -r requirements.txt

# 2. Add your OpenAI key
cp .env.example .env
# Edit .env and paste your key

# 3. Download the dataset (requires kaggle.json in ~/.kaggle/)
python scripts/download_data.py

# 4. Build the vector database (run once, ~5 min for 230k recipes)
python scripts/build_vectordb.py

# 5. Start the API
uvicorn api.main:app --reload --port 8000

# 6. Launch the Streamlit dashboard (new terminal)
streamlit run dashboard/app.py

# 7. Run tests
pytest tests/ -v

## Dataset
Food.com Recipes and Reviews
https://www.kaggle.com/datasets/irkaal/foodcom-recipes-and-reviews/data

## Tech Stack
- Vision:     YOLOv8, DETR, CLIP (ViT-B-32)
- Embeddings: all-MiniLM-L6-v2
- Vector DB:  ChromaDB
- LLM:        GPT-4o-mini
- API:        FastAPI
- Dashboard:  Streamlit