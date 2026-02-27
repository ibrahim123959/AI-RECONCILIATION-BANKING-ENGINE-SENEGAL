# Parser Configuration

Configurations mappings colonnes et patterns par banque.

## Files
```
config/
├── column_mappings.json    # Mappings colonnes
└── format_patterns.json    # Patterns regex
```

## column_mappings.json
```json
{
  "ecobank": {
    "date_column": ["Date", "Date Opération"],
    "description_column": ["Libellé", "Description"],
    "debit_column": ["Débit", "Sortie"],
    "credit_column": ["Crédit", "Entrée"],
    "date_format": "DD/MM/YYYY",
    "decimal_separator": ","
  }
}
```

## format_patterns.json
```json
{
  "date_patterns": [
    "\\d{2}/\\d{2}/\\d{4}",
    "\\d{4}-\\d{2}-\\d{2}"
  ],
  "amount_patterns": [
    "\\d+[,\\.]\\d{2}",
    "\\d{1,3}([ ,]\\d{3})*[,\\.]\\d{2}"
  ]
}
```