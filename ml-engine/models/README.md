# Models

Modèles ML pour matching sémantique.

## Structure
```
models/
├── sentence_encoder.py         # Wrapper sentence-transformers
├── semantic_matcher.py         # Calcul similarité
├── confidence_scorer.py        # Scoring confiance
└── embedding_cache.py          # Cache gestion
```

## sentence_encoder.py

Encode textes en embeddings 384-dim.
```python
encoder = SentenceEncoder()
embeddings = encoder.encode(["VIR SALAIRE", "Paiement salaire"])
# Shape: (2, 384)
```

## semantic_matcher.py

Calcule similarité cosinus entre embeddings.
```python
matcher = SemanticMatcher(encoder)
matches = matcher.match(
    source_texts=["VIR SALAIRE"],
    target_texts=["Paiement salaire", "Achat pain"],
    threshold=0.7
)
# Returns: [(0, 0, 0.92)]  # (source_idx, target_idx, similarity)
```

## confidence_scorer.py

Score confiance match basé sur similarité.
```python
scorer = ConfidenceScorer()
confidence = scorer.score(similarity=0.92)
# Returns: "high"
```

## embedding_cache.py

Cache embeddings pour éviter recalculs.
```python
cache = EmbeddingCache()
embedding = cache.get_or_compute("VIR SALAIRE", encoder)
```