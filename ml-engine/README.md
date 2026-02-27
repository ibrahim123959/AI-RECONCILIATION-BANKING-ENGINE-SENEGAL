# ML Engine

Service de matching sémantique isolé utilisant sentence-transformers.

## Structure
```
ml-engine/
├── models/                 # Modèles ML
├── training/               # Fine-tuning pipeline
├── data/                   # Données entraînement (gitignored)
├── server/                 # API FastAPI
├── tests/                  # Tests + benchmarks
├── Dockerfile
└── requirements.txt
```

## Quick Start
```bash
# Install dependencies
pip install -r requirements.txt

# Run server
uvicorn server.main:app --host 0.0.0.0 --port 8001 --reload
```

## API Endpoints

### POST /encode
Encode textes en embeddings.
```bash
curl -X POST http://localhost:8001/encode \
  -H "Content-Type: application/json" \
  -d '{
    "texts": ["VIR SALAIRE", "Paiement salaire"]
  }'
```

### POST /match
Trouve matches sémantiques.
```bash
curl -X POST http://localhost:8001/match \
  -H "Content-Type: application/json" \
  -d '{
    "source_texts": ["VIR SALAIRE"],
    "target_texts": ["Paiement salaire", "Achat pain"],
    "threshold": 0.7
  }'
```

### GET /health
Health check.
```bash
curl http://localhost:8001/health
```

## Modèle

**Nom:** `paraphrase-multilingual-MiniLM-L12-v2`

**Caractéristiques:**
- Multilingue (50+ langues)
- 384 dimensions
- ~120MB
- Rapide (1-5ms par phrase sur CPU)

## Fine-Tuning
```bash
# Préparer données
python training/data_preparation.py

# Fine-tune modèle
python training/fine_tune.py --epochs 3 --batch-size 16

# Évaluer
python training/evaluate.py
```

## Tests
```bash
# Tests unitaires
pytest tests/ -v

# Benchmarks performance
python tests/benchmark.py
```

## Docker
```bash
# Build
docker build -t reconciliation-ml:latest .

# Run
docker run -d -p 8001:8001 reconciliation-ml:latest
```