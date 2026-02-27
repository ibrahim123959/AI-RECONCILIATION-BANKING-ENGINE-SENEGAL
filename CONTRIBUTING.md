# Guide de Contribution

## Workflow Git

### Branches
- `main` → Production (protected)
- `develop` → Integration (protected)
- `feature/*` → Nouvelles fonctionnalités
- `fix/*` → Corrections bugs

### Process
```bash
# 1. Créer feature branch depuis develop
git checkout develop
git pull origin develop
git checkout -b feature/your-feature

# 2. Développer et commit
git add .
git commit -m "feat: description"

# 3. Sync avec develop régulièrement
git checkout develop
git pull origin develop
git checkout feature/your-feature
git merge develop

# 4. Push et créer PR
git push origin feature/your-feature
# Créer PR sur GitHub: feature/your-feature → develop
```

## Standards Code

### Python (Backend)
```python
# Utiliser type hints
def match_transactions(bank: List[Transaction], ledger: List[Transaction]) -> List[Match]:
    pass

# Docstrings obligatoires
def compute_score(amount: Decimal, date: datetime) -> float:
    """
    Calcule score matching.
    
    Args:
        amount: Montant transaction
        date: Date transaction
    
    Returns:
        Score 0-100
    """
    pass

# Nommage: snake_case
def calculate_similarity_score():
    pass
```

### JavaScript/React (Frontend)
```javascript
// Nommage: camelCase
const calculateTotal = () => {};

// Components: PascalCase
const MatchingDashboard = () => {};

// Constants: UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
```

## Tests Requis

### Backend
```bash
# Tests unitaires + coverage >80%
pytest backend/tests/ -v --cov=backend/app --cov-report=html

# Linting
flake8 backend/app/
black backend/app/ --check
```

### Frontend
```bash
# Tests unitaires
npm test --prefix frontend

# Linting
npm run lint --prefix frontend
```

## Commit Messages

Format: `type: description`

**Types:**
- `feat`: Nouvelle fonctionnalité
- `fix`: Correction bug
- `docs`: Documentation
- `style`: Formatting (pas de changement code)
- `refactor`: Refactoring code
- `test`: Ajout tests
- `chore`: Maintenance (deps, config)

**Exemples:**
```
feat: add semantic matching layer
fix: correct amount parsing for comma decimal
docs: update API specification
test: add unit tests for rule_matcher
```

## Pull Requests

### Checklist
- [ ] Tests passent (backend + frontend)
- [ ] Coverage >80% pour nouveau code
- [ ] Documentation mise à jour
- [ ] Pas de secrets committés
- [ ] Branch sync avec `develop`
- [ ] PR description claire

### Template
```markdown
## Description
[Brève description changements]

## Type
- [ ] Feature
- [ ] Bug fix
- [ ] Documentation
- [ ] Refactoring

## Tests
- [ ] Tests unitaires ajoutés
- [ ] Tests intégration ajoutés
- [ ] Tests manuels effectués

## Checklist
- [ ] Code reviewed
- [ ] Documentation updated
- [ ] Tests passing
```

## Questions?

- **Issues GitHub:** [github.com/AI-RECONCILIATION-ENGINE/issues](https://github.com/ibrahim123959/AI-RECONCILIATION-BANKING-ENGINE-SENEGAL/issues)
- **Email:** ibrahima4234@gmail.com