# Bank-Specific Parsers

Parsers optimisés pour formats spécifiques banques.

## Structure
```
bank_specific/
├── ecobank_parser.py       # Ecobank Sénégal
├── boa_parser.py           # BOA
├── sgbs_parser.py          # SGBS
└── generic_parser.py       # Fallback générique
```

## Ecobank

**Format:** PDF structuré  
**Colonnes:** Date | Libellé | Débit | Crédit | Solde  
**Date format:** DD/MM/YYYY  
**Decimal:** Virgule (,)

## BOA

**Format:** CSV  
**Colonnes:** Date Opération | Description | Montant | Sens  
**Date format:** YYYY-MM-DD  
**Decimal:** Point (.)

## SGBS

**Format:** Excel (.xlsx)  
**Colonnes:** Date | Réf | Libellé | Débit | Crédit  
**Date format:** DD/MM/YYYY  
**Decimal:** Virgule (,)