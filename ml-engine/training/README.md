# Training

Fine-tuning pipeline pour sentence-transformers.

## Structure
```
training/
├── fine_tune.py            # Script fine-tuning
├── data_preparation.py     # Préparation données
├── evaluate.py             # Évaluation modèle
└── active_learning.py      # Active learning
```

## Fine-Tune Model
```bash
python training/fine_tune.py \
  --data data/training/ \
  --epochs 3 \
  --batch-size 16 \
  --output models/fine-tuned-v1
```

## Evaluate
```bash
python training/evaluate.py \
  --model models/fine-tuned-v1 \
  --test-data data/validation/
```

## Active Learning

Sélectionne exemples incertains pour labelling prioritaire.
```bash
python training/active_learning.py \
  --n-samples 50
```