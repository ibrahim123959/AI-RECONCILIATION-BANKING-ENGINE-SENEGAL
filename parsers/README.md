# Parsers

Extraction de transactions depuis documents (PDF, CSV, Excel).

## Structure
```
parsers/
├── base_parser.py          # Interface abstraite
├── pdf_parser.py           # Parser PDF
├── csv_parser.py           # Parser CSV
├── excel_parser.py         # Parser Excel
├── bank_specific/          # Parsers banques
├── mobile_money/           # Parsers mobile money
├── config/                 # Configurations
└── tests/                  # Tests
```

## Usage
```python
from parsers.pdf_parser import PDFParser

parser = PDFParser()
transactions = parser.parse("releve_novembre.pdf")

# Returns: List[Dict]
# [
#   {"date": "2024-11-15", "description": "VIR SALAIRE", "amount": 250000},
#   ...
# ]
```

## Supported Formats

- **PDF:** Via Camelot + pdfplumber + OCR fallback
- **CSV:** Multi-delimiter, multi-encoding
- **Excel:** .xlsx, .xls

## Bank-Specific

Parsers optimisés par banque :
- Ecobank Sénégal
- BOA
- SGBS
- Generic fallback

## Tests
```bash
pytest parsers/tests/ -v
```