# Tests ML

Tests et benchmarks pour ML engine.

## Structure
```
tests/
├── test_encoder.py         # Tests encoding
├── test_matcher.py         # Tests matching
└── benchmark.py            # Performance benchmarks
```

## Run Tests
```bash
pytest tests/ -v
```

## Benchmarks
```bash
python tests/benchmark.py

# Output:
# Encoding 1000 texts: 2.3s (435 texts/sec)
# Matching 100x100: 0.8s
# Cache hit rate: 87%
```
