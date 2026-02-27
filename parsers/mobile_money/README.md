# Mobile Money Parsers

Parsers pour Wave, Orange Money, Free Money.

## Structure
```
mobile_money/
├── wave_parser.py          # Wave
├── orange_money_parser.py  # Orange Money
└── free_money_parser.py    # Free Money
```

## Wave

**Format:** CSV UTF-8  
**Colonnes:** Date | Type | Destinataire/Émetteur | Montant | Frais | Solde  
**Particularités:** Frais séparés

## Orange Money

**Format:** Excel  
**Colonnes:** Date | Service | Numéro | Montant | Solde  
**Particularités:** Référence OM dans description

## Free Money

**Format:** CSV  
**Colonnes:** Date | Opération | Montant | Statut  
**Particularités:** Format simplifié