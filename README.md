# Conversion Rate Challenge 🏆

Prédire si un visiteur du site [datascienceweekly.org](https://www.datascienceweekly.org) va s'abonner à la newsletter, à partir de quelques variables comportementales.

---

## Structure du projet

```
Conversion_Rate/
├── data/
│   ├── conversion_data_train.csv          # Données labellisées (284 580 lignes)
│   └── conversion_data_test.csv           # Données sans labels (31 620 lignes)
├── 02-Conversion_rate_challenge.ipynb     # Description complète du projet
├── 02-Conversion_rate_challenge_template.ipynb  # Notebook principal (EDA + modèle + soumission)
├── conversion_data_test_predictions_IBRM.csv    # Fichier de soumission Kaggle
└── README.md
```

---

## Installation

```bash
conda activate conversion_rate
```

Les dépendances utilisées sont :

```
pandas
numpy
scikit-learn
matplotlib
seaborn
plotly
```

Pour les installer si besoin :

```bash
conda activate conversion_rate
pip install pandas numpy scikit-learn matplotlib seaborn plotly
```

---

## Données

| Fichier | Rôle | Lignes | Colonnes |
|--------|------|--------|----------|
| `data/conversion_data_train.csv` | Entraînement + évaluation (labellisé) | 284 580 | 6 |
| `data/conversion_data_test.csv` | Soumission finale (non labellisé) | 31 620 | 5 |

**Variables disponibles :**

| Variable | Type | Description |
|----------|------|-------------|
| `country` | catégorielle | Pays du visiteur (US, China, UK, Germany) |
| `age` | numérique | Âge du visiteur |
| `new_user` | binaire | 1 = nouveau visiteur, 0 = visiteur de retour |
| `source` | catégorielle | Canal d'acquisition (Ads, Seo, Direct) |
| `total_pages_visited` | numérique | Nombre de pages consultées lors de la session |
| `converted` | binaire | **Cible** — 1 = abonné, 0 = non abonné *(absent dans le test set)* |

---

## Méthodologie

### EDA
- Déséquilibre des classes : seulement **3.23%** de conversions → nécessite `class_weight='balanced'` et optimisation du F1
- `total_pages_visited` est le prédicteur le plus fort
- La Chine représente 24% du trafic mais seulement 0.13% de conversions
- Les anciens utilisateurs (`new_user=0`) convertissent 2× plus que les nouveaux

### Préprocessing
- Clip `age` à 80 ans (2 outliers à 123 ans)
- Feature engineering :
  - `pages_per_age` = `total_pages_visited / (age + 1)`
  - `is_returning_engaged` = 1 si `new_user=0` ET `total_pages_visited > 5`
- `ColumnTransformer` : `StandardScaler` sur les variables numériques, `OneHotEncoder` sur les catégorielles

### Modèles

| Modèle | F1 (test) | AUC-ROC |
|--------|-----------|---------|
| Baseline — LogisticRegression (1 variable) | 0.4954 | 0.9847 |
| Baseline — LogisticRegression (toutes features) | 0.5081 | — |
| **Random Forest (best)** | **0.5637** | **0.9521** |

**Hyperparamètres du meilleur modèle :**
```python
RandomForestClassifier(
    n_estimators=300,
    max_depth=20,
    min_samples_leaf=1,
    max_features='sqrt',
    class_weight='balanced',
    random_state=42,
    n_jobs=-1
)
```

---

## Lancer le notebook

```bash
conda activate conversion_rate
jupyter notebook 02-Conversion_rate_challenge_template.ipynb
```

Le notebook s'exécute de bout en bout et génère automatiquement le fichier de soumission `conversion_data_test_predictions_IBRM.csv`.

---

## Soumission

Le fichier de soumission respecte le format Kaggle :
- Une seule colonne `converted` (0 ou 1)
- Pas d'index
- Nom : `conversion_data_test_predictions_IBRM.csv`

---

## Recommandations métier (Part 4)

| Priorité | Levier | Impact |
|----------|--------|--------|
| 🥇 | Augmenter les pages visitées par session (navigation interne, CTAs persistants) | Élevé |
| 🥈 | Retargeting des visiteurs de retour non convertis | Élevé |
| 🥉 | Déprioritiser le trafic China (0.13% de conversion) | Moyen |
| 4 | Contenu orienté tranche 17-25 ans | Moyen |
| 5 | Landing pages dédiées par source d'acquisition | Moyen |