# Règles Métier de Réconciliation

## Vue d'Ensemble

Ce document définit les règles métier utilisées par le moteur de matching pour réconcilier transactions bancaires et écritures comptables.

---

## Critères de Matching

### 1. Montant (40 points max)

| Critère | Score | Exemple |
|---------|-------|---------|
| Montant **exact** | 40 | 250,000 = 250,000 |
| Écart **< 1%** | 30 | 250,000 vs 252,000 (+0.8%) |
| Écart **< 5%** | 20 | 250,000 vs 260,000 (+4%) |
| Écart **< 10%** | 10 | 250,000 vs 270,000 (+8%) |
| Écart **> 10%** | 0 | Pas de match |

### 2. Date (30 points max)

| Critère | Score | Exemple |
|---------|-------|---------|
| Date **exacte** | 30 | 15/11 = 15/11 |
| Écart **≤ 2 jours** | 25 | 15/11 vs 17/11 |
| Écart **≤ 5 jours** | 15 | 15/11 vs 20/11 |
| Écart **≤ 10 jours** | 10 | 15/11 vs 25/11 |
| Écart **≤ 15 jours** | 5 | 15/11 vs 30/11 |
| Écart **> 15 jours** | 0 | Pas de match |

### 3. Libellé (30 points max)

**Méthode 1 : Fuzzy Matching** (Couche 1)
| Similarité | Score |
|------------|-------|
| **≥ 95%** | 20 |
| **≥ 85%** | 15 |
| **≥ 70%** | 10 |
| **< 70%** | 0 |

**Méthode 2 : Sémantique IA** (Couche 2)
| Similarité Cosinus | Score |
|--------------------|-------|
| **≥ 0.90** | 30 |
| **≥ 0.80** | 24 |
| **≥ 0.70** | 18 |
| **< 0.70** | 0 |

---

## Seuils de Confiance

### Score Total = Montant + Date + Libellé (max 100)

| Niveau | Seuil | Action |
|--------|-------|--------|
| **Haute confiance** | ≥ 85 | Match automatique |
| **Moyenne confiance** | 75-84 | Match suggéré, validation recommandée |
| **Faible confiance** | 60-74 | Validation manuelle obligatoire |
| **Pas de match** | < 60 | Non matché |

---

## Cas Spéciaux

### 1. Transactions Splitées

**Scénario :** 1 virement bancaire → plusieurs écritures comptables

**Exemple :**
```
Banque:  300,000 FCFA (Virement multiple factures)
Ledger:  - 100,000 FCFA (Facture A)
         - 150,000 FCFA (Facture B)
         - 50,000 FCFA (Facture C)
         = 300,000 FCFA ✓
```

**Règles :**
- Somme ledger = montant banque (exact)
- Dates ≤ 5 jours d'écart
- Max 4 écritures splitées
- Confiance : Moyenne (75)

### 2. Frais Bancaires

**Scénario :** Frais déduits du montant

**Exemple :**
```
Ledger:  100,000 FCFA (Montant attendu)
Banque:  97,500 FCFA (Net reçu)
Frais:   2,500 FCFA (2.5%)
```

**Règles :**
- Différence < 3% du montant
- Écriture "frais bancaires" dans ledger proche date
- Confiance : Haute (90) si frais trouvés

### 3. Doublons

**Scénario :** Transaction apparaît 2x (erreur saisie)

**Critères détection :**
- Montant **identique**
- Date ≤ 1 jour
- Libellé similarité > 90%
- Même compte bancaire/ledger

**Action :** Flag pour validation manuelle

---

## Tolérances par Devise

### XOF (Franc CFA)
- Arrondis acceptés : ±5 FCFA
- Frais typiques : 1,000-5,000 FCFA (virements)

### EUR/USD
- Arrondis acceptés : ±0.10
- Frais typiques : 0.5-3% (virements internationaux)

---

## Normalisation Libellés

### Transformations Appliquées
```python
# Avant matching
original = "VIR. SALAIRE  NOVEMBRE 2024"

# Normalisation
normalized = normalize_description(original)
# → "vir salaire novembre 2024"

# Étapes:
# 1. Lowercase
# 2. Supprimer ponctuation
# 3. Supprimer espaces multiples
# 4. Supprimer accents (é → e)
```

### Synonymes Reconnus

| Terme Bancaire | Équivalents Acceptés |
|----------------|---------------------|
| VIR | Virement, Transfer |
| PRLV | Prélèvement, Debit |
| CHQ | Chèque, Cheque |
| CB | Carte bancaire, Card |
| FRAIS | Fees, Commission |

---

## Règles Validation Manuelle

### Obligatoire Si

- Score < 75
- Montant > 1,000,000 FCFA
- Anomalie détectée
- Transaction splitée
- Frais bancaires détectés

### Workflow
```
Collaborateur → Chef de Mission → Directeur Technique
   (Review)        (Approve)           (Final Approval)
```

Chaque niveau peut :
- ✅ Approuver
- ❌ Rejeter
- ✏️ Corriger et valider

---

## Métriques Performance

### Objectifs Cibles

| Métrique | Target | Pilote |
|----------|--------|--------|
| **Taux matching auto** | > 85% | TBD |
| **Précision haute confiance** | > 95% | TBD |
| **Faux positifs** | < 2% | TBD |
| **Faux négatifs** | < 5% | TBD |

---

## Évolution Règles

Les règles sont **configurables** via `backend/app/config.py` :
```python
class MatchingConfig:
    EXACT_AMOUNT_SCORE = 40
    CLOSE_AMOUNT_1PCT_SCORE = 30
    
    EXACT_DATE_SCORE = 30
    DATE_DIFF_2DAYS_SCORE = 25
    
    HIGH_CONFIDENCE_THRESHOLD = 85
    MEDIUM_CONFIDENCE_THRESHOLD = 70
```

Modifications nécessitent :
1. Update config
2. Tests validation
3. PR review + approval
4. Deploy + monitoring

---

**Version :** 1.0.0  
**Dernière MAJ :** 26 Février 2025