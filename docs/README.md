# Documentation

Documentation complète du projet Reconciliation Engine.

## Structure
```
docs/
├── technical/              # Documentation technique
│   ├── api-specification.md
│   ├── data-model.md
│   ├── matching-algorithm.md
│   ├── ml-pipeline.md
│   ├── performance-benchmarks.md
│   └── deployment.md
│
├── business/               # Documentation métier
│   ├── reconciliation-workflow.md
│   ├── bank-formats.md
│   ├── mobile-money-specifics.md
│   ├── tolerance-rules.md
│   └── audit-requirements.md
│
├── user/                   # Documentation utilisateur
│   ├── quick-start.md
│   ├── validation-guide.md
│   ├── reports-export.md
│   └── faq.md
│
└── governance/             # Documentation conformité
    ├── data-privacy.md
    ├── security-policy.md
    ├── ai-ethics.md
    ├── regulatory-compliance.md
    └── change-log.md
```

## Documentation par Audience

### Développeurs → `technical/`
- Architecture système
- API REST specification
- Modèle de données
- Pipeline ML
- Procédures déploiement

**Commencer par :** `api-specification.md`

### Experts Comptables → `business/`
- Workflow réconciliation
- Règles métier matching
- Formats relevés bancaires
- Exigences traçabilité
- Conformité BCEAO

**Commencer par :** `reconciliation-workflow.md`

### Utilisateurs Finaux → `user/`
- Guide démarrage rapide
- Validation manuelle
- Export rapports
- FAQ

**Commencer par :** `quick-start.md`

### Auditeurs/Compliance → `governance/`
- Politique protection données
- Mesures sécurité
- Principes éthiques IA
- Conformité réglementaire

**Commencer par :** `regulatory-compliance.md`

## Quick Links

### Démarrage Rapide
1. [Installation](../README.md#démarrage-rapide)
2. [Configuration](technical/deployment.md)
3. [Premier matching](user/quick-start.md)

### Référence API
- [Endpoints complets](technical/api-specification.md)
- [Schemas Pydantic](../backend/app/schemas/)
- [Examples requêtes](technical/api-specification.md#examples)

### Troubleshooting
- [FAQ Utilisateurs](user/faq.md)
- [Issues GitHub](https://github.com/your-repo/issues)

## Contribution Documentation

Documentation maintenue en Markdown.

### Ajouter Documentation
```bash
# 1. Créer fichier dans dossier approprié
touch docs/technical/new-feature.md

# 2. Suivre template
cat > docs/technical/new-feature.md << 'EOF'
# Feature Name

## Overview
[Brief description]

## Technical Details
[Implementation details]

## Examples
[Code examples]

## Related
- [Link to related doc]
EOF

# 3. Commit et PR
git add docs/technical/new-feature.md
git commit -m "docs: add new-feature documentation"
```

### Standards
- **Format :** Markdown (.md)
- **Langue :** Français (documentation métier), Anglais acceptable (technique)
- **Structure :** Titres clairs, exemples code, liens internes
- **Images :** Stocker dans `docs/assets/images/`

## Génération Documentation API

Documentation API auto-générée depuis code :
```bash
# Lancer backend
uvicorn app.main:app --reload

# Accéder docs
open http://localhost:8000/docs  # Swagger UI
open http://localhost:8000/redoc # ReDoc
```

## Mise à Jour

Vérifier documentation à jour lors de :
- Nouveau endpoint API
- Changement règles métier
- Modification workflow
- Nouvelle feature

**Reviewer checklist :**
- [ ] Code changes → Docs updated
- [ ] New API endpoint → api-specification.md updated
- [ ] Business rules change → business/ updated