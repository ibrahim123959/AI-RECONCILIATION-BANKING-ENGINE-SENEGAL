# Data

Données pour entraînement et évaluation ML.

**⚠️ Ce dossier est gitignored** (contient données sensibles).

## Structure
```
data/
├── training/               # Données entraînement
│   └── positive_pairs.json
│   └── negative_pairs.json
│
├── validation/             # Données validation
│   └── validation_set.json
│
└── embeddings_cache/       # Cache embeddings (gitignored)
    └── *.pkl
```

## Format Données

### Paires Positives (matches corrects)
```json
{
  "positive_pairs": [
    {
      "text1": "VIR SALAIRE NOVEMBRE",
      "text2": "Paiement salaire novembre 2024",
      "label": 1
    }
  ]
}
```

### Paires Négatives (matches incorrects)
```json
{
  "negative_pairs": [
    {
      "text1": "VIR SALAIRE",
      "text2": "Achat épicerie",
      "label": 0
    }
  ]
}
```

## Collecte Données

Données collectées depuis validations manuelles utilisateurs.
```python
# Extraire depuis DB
python scripts/extract_training_data.py --output data/training/
```

## Cache Embeddings

Embeddings calculés stockés pour performance.
```python
# Clear cache
rm -rf data/embeddings_cache/*
```