# Parser Tests

Tests pour tous les parsers.

## Structure
```
tests/
├── test_pdf_parser.py
├── test_csv_parser.py
├── test_bank_parsers.py
└── fixtures/               # Sample files
```

## Run Tests
```bash
pytest parsers/tests/ -v
```

## Fixtures

Sample files dans `fixtures/` :
- `sample_ecobank.pdf`
- `sample_boa.csv`
- `sample_wave.csv`