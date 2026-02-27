# Scripts

Utilitaires pour setup, testing, maintenance.

## Structure
```
scripts/
├── setup_database.py       # Init BDD + tables
├── load_test_data.py       # Load sample data
├── backup_database.py      # Backup automatisé
├── generate_report.py      # Test report generation
└── deploy.sh               # Deployment script
```

## Usage

### Setup Database
```bash
python scripts/setup_database.py
```

### Load Test Data
```bash
python scripts/load_test_data.py --transactions 1000
```

### Backup
```bash
python scripts/backup_database.py --output backups/
```

### Deploy
```bash
./scripts/deploy.sh production
```