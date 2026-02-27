# Backend - Reconciliation Engine API

FastAPI application gérant le cœur logique du système de réconciliation.

## Structure
```
backend/
├── app/                    # Code application
│   ├── main.py             # Entry point FastAPI
│   ├── config.py           # Configuration
│   ├── api/v1/             # API endpoints
│   ├── core/               # Business logic
│   ├── models/             # Database models (ORM)
│   ├── schemas/            # Pydantic schemas
│   ├── services/           # External services
│   ├── database/           # DB session, migrations
│   └── utils/              # Utilities
│
├── tests/                  # Tests
├── Dockerfile              # Docker image
├── requirements.txt        # Dependencies
└── pyproject.toml          # Project config
```

## Quick Start

### Installation
```bash
# Créer environnement virtuel
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows

# Installer dépendances
pip install -r requirements.txt
```

### Configuration
```bash
# Copier template
cp .env.example .env

# Éditer .env
DATABASE_URL=postgresql://user:pass@localhost:5432/reconciliation
REDIS_URL=redis://localhost:6379
ML_ENGINE_URL=http://localhost:8001
```

### Migrations Database
```bash
# Créer migration
alembic revision --autogenerate -m "description"

# Appliquer migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

### Lancer Serveur
```bash
# Development (avec reload)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Production
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### API Documentation

Une fois lancé, accéder à :
- **Swagger UI :** http://localhost:8000/docs
- **ReDoc :** http://localhost:8000/redoc
- **OpenAPI JSON :** http://localhost:8000/openapi.json

## Tests
```bash
# Tous les tests
pytest tests/ -v

# Avec coverage
pytest tests/ -v --cov=app --cov-report=html

# Tests unitaires seulement
pytest tests/unit/ -v

# Tests intégration
pytest tests/integration/ -v

# Test spécifique
pytest tests/unit/test_rule_matcher.py -v
```

## Linting & Formatting
```bash
# Linting
flake8 app/

# Formatting (check)
black app/ --check

# Formatting (apply)
black app/

# Type checking
mypy app/
```

## Structure Code

### API Endpoints (`app/api/v1/`)
```python
# Example: app/api/v1/matching.py

from fastapi import APIRouter, Depends
from app.schemas.match_schema import MatchingRequest, MatchingResponse
from app.core.matching_orchestrator import MatchingOrchestrator

router = APIRouter()

@router.post("/matching/run", response_model=MatchingResponse)
async def run_matching(
    request: MatchingRequest,
    db: Session = Depends(get_db)
):
    """Lance processus de matching."""
    orchestrator = MatchingOrchestrator(db)
    result = await orchestrator.match(
        bank_file_id=request.bank_file_id,
        ledger_file_id=request.ledger_file_id
    )
    return result
```

### Business Logic (`app/core/`)
```python
# Example: app/core/rule_matcher.py

class RuleMatcher:
    """Matching basé sur règles exactes."""
    
    def match(self, bank_txn: Transaction, ledger_txn: Transaction) -> Optional[Match]:
        """
        Matche deux transactions selon règles.
        
        Returns:
            Match si score >= 85, sinon None
        """
        score = self.compute_score(bank_txn, ledger_txn)
        
        if score >= 85:
            return Match(
                bank_transaction=bank_txn,
                ledger_transaction=ledger_txn,
                score=score,
                confidence="high"
            )
        
        return None
```

### Models (`app/models/`)
```python
# Example: app/models/match.py

from sqlalchemy import Column, String, Numeric, DateTime, UUID
from app.database.session import Base

class Match(Base):
    __tablename__ = "matches"
    
    id = Column(UUID, primary_key=True)
    bank_transaction_id = Column(UUID, ForeignKey("bank_transactions.id"))
    ledger_transaction_id = Column(UUID, ForeignKey("ledger_transactions.id"))
    score_total = Column(Numeric(5, 2))
    confidence_level = Column(String(20))
    status = Column(String(50), default="pending")
```

## Variables Environnement

| Variable | Description | Exemple |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection | `postgresql://...` |
| `REDIS_URL` | Redis connection | `redis://localhost:6379` |
| `ML_ENGINE_URL` | ML service URL | `http://localhost:8001` |
| `ENVIRONMENT` | Environment name | `development` / `production` |
| `LOG_LEVEL` | Logging level | `INFO` / `DEBUG` |

## Troubleshooting

### Database Connection Error
```bash
# Vérifier PostgreSQL running
docker ps | grep postgres

# Tester connexion
psql $DATABASE_URL
```

### Import Error
```bash
# Réinstaller dependencies
pip install -r requirements.txt --force-reinstall
```

### Migration Error
```bash
# Reset migrations (⚠️ DEV ONLY)
alembic downgrade base
alembic upgrade head
```