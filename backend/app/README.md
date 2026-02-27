# App - Application Code

Code principal de l'application backend.

## Structure
```
app/
├── main.py                 # Entry point FastAPI
├── config.py               # Configuration centralisée
│
├── api/                    # REST API
│   ├── v1/                 # Version 1
│   │   ├── upload.py       # Upload endpoints
│   │   ├── matching.py     # Matching endpoints
│   │   ├── validation.py   # Validation endpoints
│   │   ├── reports.py      # Reports endpoints
│   │   ├── dashboard.py    # Dashboard endpoints
│   │   └── admin.py        # Admin endpoints
│   └── dependencies.py     # Dependency injection
│
├── core/                   # Business logic
│   ├── matching_orchestrator.py    # Coordination matching
│   ├── rule_matcher.py             # Matching règles
│   ├── semantic_matcher.py         # Matching IA
│   ├── suggested_matcher.py        # Matching suggéré
│   ├── duplicate_detector.py       # Détection doublons
│   ├── anomaly_detector.py         # Détection anomalies
│   ├── learning_controller.py      # Apprentissage supervisé
│   ├── split_transaction_handler.py # Transactions splitées
│   ├── fee_detector.py             # Détection frais
│   └── audit_logger.py             # Logging audit
│
├── models/                 # ORM models
│   ├── uploaded_file.py
│   ├── bank_transaction.py
│   ├── ledger_transaction.py
│   ├── match.py
│   ├── validation_event.py
│   ├── learning_event.py
│   ├── audit_trail.py
│   └── reconciliation_report.py
│
├── schemas/                # Pydantic validation
│   ├── upload_schema.py
│   ├── transaction_schema.py
│   ├── match_schema.py
│   ├── validation_schema.py
│   └── report_schema.py
│
├── services/               # External services
│   ├── parser_service.py   # Interface parsers
│   ├── ml_service.py       # ML engine client
│   ├── storage_service.py  # File storage
│   ├── export_service.py   # Export rapports
│   └── notification_service.py  # Notifications
│
├── database/               # Database
│   ├── session.py          # Session factory
│   ├── migrations/         # Alembic migrations
│   │   └── versions/
│   └── seeds/              # Seed data
│
└── utils/                  # Utilities
    ├── date_parser.py      # Parse dates multi-formats
    ├── amount_parser.py    # Parse montants
    ├── normalizers.py      # Normalisation texte
    ├── validators.py       # Validateurs métier
    ├── logger.py           # Logging config
    └── exceptions.py       # Custom exceptions
```

## Modules Clés

### main.py
Entry point FastAPI. Configure :
- CORS
- Routes API
- Middleware
- Exception handlers
- Startup/shutdown events

### config.py
Configuration centralisée :
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    REDIS_URL: str
    ML_ENGINE_URL: str
    ENVIRONMENT: str = "development"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

### core/matching_orchestrator.py
**Rôle :** Coordonne les 3 couches de matching

**Flow :**
1. Indexe transactions par montant
2. Lance couche 1 (règles exactes)
3. Lance couche 2 (IA sémantique) pour restantes
4. Lance couche 3 (suggéré) pour restantes
5. Détecte cas spéciaux (splits, fees, duplicates)
6. Sauvegarde matches en DB

### core/rule_matcher.py
**Rôle :** Matching basé règles déterministes

**Critères :**
- Montant exact (40 pts)
- Date ±2j (30 pts)
- Libellé >95% similarité (20 pts)
- Seuil : 85+ = haute confiance

### core/semantic_matcher.py
**Rôle :** Matching sémantique via ML

**Process :**
1. Appelle ML engine pour encoder libellés
2. Calcule similarité cosinus
3. Score = montant + date + sémantique
4. Seuil : 75-90 = moyenne confiance

### services/ml_service.py
**Rôle :** Client HTTP vers ML engine

**Methods :**
```python
class MLService:
    async def encode(self, texts: List[str]) -> np.ndarray:
        """Encode textes en embeddings."""
        
    async def match(self, source: List[str], target: List[str]) -> List[Match]:
        """Trouve matches sémantiques."""
        
    async def score(self, text1: str, text2: str) -> float:
        """Score similarité entre deux textes."""
```

## Patterns Utilisés

### Dependency Injection
```python
# app/api/dependencies.py

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Usage dans endpoint
@router.get("/matches")
async def get_matches(db: Session = Depends(get_db)):
    return db.query(Match).all()
```

### Service Layer
```python
# Séparation concerns: Controller → Service → Repository

# Controller (API)
@router.post("/matching/run")
async def run_matching(request: MatchingRequest):
    result = await matching_service.run(request)
    return result

# Service (Business logic)
class MatchingService:
    async def run(self, request):
        # Complex logic here
        pass

# Repository (Data access)
class MatchRepository:
    def save(self, match):
        # DB operations
        pass
```

### Factory Pattern
```python
# app/services/parser_service.py

class ParserFactory:
    @staticmethod
    def get_parser(file_type: str):
        if file_type == "pdf":
            return PDFParser()
        elif file_type == "csv":
            return CSVParser()
        elif file_type == "excel":
            return ExcelParser()
        else:
            raise ValueError(f"Unknown file type: {file_type}")
```