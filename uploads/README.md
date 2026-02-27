# Uploads

Stockage temporaire fichiers uploadés.

**⚠️ Ce dossier est gitignored** (fichiers utilisateurs).

## Structure
```
uploads/
├── bank_statements/        # Relevés bancaires
├── ledgers/                # Comptabilité interne
└── temp/                   # Fichiers temporaires
```

## Lifecycle

1. User upload → `uploads/temp/`
2. Parser extraction → Move to `uploads/bank_statements/` or `uploads/ledgers/`
3. Après matching → Archive ou suppression (selon config)

## Cleanup
```bash
# Supprimer fichiers >30 jours
find uploads/ -mtime +30 -delete
```

## Storage

- **Dev:** Local filesystem
- **Staging:** Render disk
- **Production:** Cloud Storage (GCS/S3)