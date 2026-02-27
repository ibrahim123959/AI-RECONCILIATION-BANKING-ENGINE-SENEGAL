# Tests Backend

Suite de tests pour le backend (unitaires + intégration).

## Structure
```
tests/
├── conftest.py             # Fixtures pytest partagées
│
├── unit/                   # Tests unitaires
│   ├── test_rule_matcher.py
│   ├── test_semantic_matcher.py
│   ├── test_date_parser.py
│   ├── test_amount_parser.py
│   └── test_normalizers.py
│
├── integration/            # Tests intégration
│   ├── test_matching_flow.py
│   ├── test_learning_flow.py
│   └── test_validation_flow.py
│
└── fixtures/               # Données test
    ├── sample_bank_statements/
    ├── sample_ledgers/
    └── expected_matches.json
```

## Lancer Tests

### Tous les tests
```bash
pytest tests/ -v
```

### Tests unitaires seulement
```bash
pytest tests/unit/ -v
```

### Tests intégration seulement
```bash
pytest tests/integration/ -v
```

### Test spécifique
```bash
pytest tests/unit/test_rule_matcher.py::test_exact_amount_match -v
```

### Avec coverage
```bash
pytest tests/ -v --cov=app --cov-report=html

# Ouvrir rapport HTML
open htmlcov/index.html
```

### Markers
```bash
# Tests rapides seulement
pytest tests/ -v -m "not slow"

# Tests nécessitant DB
pytest tests/ -v -m "database"
```

## Fixtures (conftest.py)

### Database Test
```python
@pytest.fixture
def test_db():
    """Database de test isolée."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    TestingSessionLocal = sessionmaker(bind=engine)
    
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(engine)
```

### Sample Transactions
```python
@pytest.fixture
def sample_bank_transaction():
    return BankTransaction(
        id=uuid4(),
        transaction_date=date(2024, 11, 15),
        description="VIR SALAIRE NOVEMBRE",
        debit_amount=None,
        credit_amount=Decimal("250000.00"),
        currency="XOF"
    )

@pytest.fixture
def sample_ledger_transaction():
    return LedgerTransaction(
        id=uuid4(),
        transaction_date=date(2024, 11, 15),
        description="Paiement salaire novembre 2024",
        debit_amount=Decimal("250000.00"),
        credit_amount=None,
        currency="XOF"
    )
```

## Tests Unitaires

### Example: test_rule_matcher.py
```python
def test_exact_amount_match(sample_bank_transaction, sample_ledger_transaction):
    """Test matching montant exact."""
    matcher = RuleMatcher()
    
    match = matcher.match(sample_bank_transaction, sample_ledger_transaction)
    
    assert match is not None
    assert match.score_amount == 40
    assert match.confidence_level == "high"

def test_close_amount_1pct():
    """Test tolérance 1% montant."""
    bank_txn = BankTransaction(amount=250000)
    ledger_txn = LedgerTransaction(amount=252000)  # +0.8%
    
    matcher = RuleMatcher()
    score = matcher.compute_amount_score(bank_txn, ledger_txn)
    
    assert score == 30

def test_different_amount_no_match():
    """Test montant trop différent."""
    bank_txn = BankTransaction(amount=250000)
    ledger_txn = LedgerTransaction(amount=500000)  # +100%
    
    matcher = RuleMatcher()
    match = matcher.match(bank_txn, ledger_txn)
    
    assert match is None
```

### Example: test_date_parser.py
```python
@pytest.mark.parametrize("date_string,expected", [
    ("15/11/2024", date(2024, 11, 15)),
    ("2024-11-15", date(2024, 11, 15)),
    ("15-Nov-2024", date(2024, 11, 15)),
    ("15.11.2024", date(2024, 11, 15)),
])
def test_parse_date_formats(date_string, expected):
    """Test parsing multiple date formats."""
    result = parse_date(date_string)
    assert result == expected

def test_parse_invalid_date():
    """Test date invalide retourne None."""
    result = parse_date("invalid-date")
    assert result is None
```

## Tests Intégration

### Example: test_matching_flow.py
```python
@pytest.mark.integration
def test_full_matching_flow(test_db):
    """Test flux complet upload → matching → validation."""
    
    # 1. Upload bank statement
    bank_file = upload_file(test_db, "sample_bank.csv", "bank_statement")
    assert bank_file.status == "completed"
    assert bank_file.total_transactions == 125
    
    # 2. Upload ledger
    ledger_file = upload_file(test_db, "sample_ledger.csv", "internal_ledger")
    assert ledger_file.status == "completed"
    assert ledger_file.total_transactions == 120
    
    # 3. Run matching
    orchestrator = MatchingOrchestrator(test_db)
    result = orchestrator.match(bank_file.id, ledger_file.id)
    
    # 4. Verify results
    assert result.total_matched >= 100
    assert result.high_confidence >= 80
    assert result.processing_time < 30  # seconds
    
    # 5. Verify matches in DB
    matches = test_db.query(Match).filter(Match.status == "pending").all()
    assert len(matches) > 0
```

## Coverage Target

**Minimum requis :** 80%

### Par module
- `core/` : 90%+ (logique critique)
- `api/` : 80%+
- `utils/` : 85%+
- `services/` : 75%+

### Exclure du coverage
```ini
# pytest.ini
[tool:pytest]
omit = 
    */migrations/*
    */tests/*
    */conftest.py
```

## Best Practices

### ✅ DO
- 1 test = 1 comportement
- Noms tests descriptifs (`test_exact_amount_gives_40_points`)
- Utiliser fixtures pour setup
- Tester cas limite (edge cases)
- Mocker services externes (ML engine, Redis)

### ❌ DON'T
- Tests dépendants ordre exécution
- Tests partageant état mutable
- Tests ignorant erreurs silencieusement
- Tests trop longs (>2s unitaire, >30s intégration)

## Continuous Integration

Tests lancés automatiquement sur :
- Push vers `develop` ou `main`
- Pull Request ouverte/mise à jour

Workflow `.github/workflows/backend-ci.yml` :
```yaml
- name: Run tests
  run: pytest tests/ -v --cov=app --cov-report=xml

- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    file: ./coverage.xml
```