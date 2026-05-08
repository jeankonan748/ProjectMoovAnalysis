# Moov Africa — Back Office Customer Care Analysis

> Optimisation du traitement des requêtes clients par la data science

---

## Présentation

Ce projet analyse les données du **Back Office Customer Care de Moov Africa** afin d'identifier les goulots d'étranglement, mesurer la performance des agents, évaluer le respect des SLA et produire des indicateurs actionnables pour le management.

| Champ | Détail |
|---|---|
| **Entreprise** | Moov Africa |
| **Domaine** | Customer Care / Back Office |
| **Auteur** | Jean KONAN |
| **Version** | 2.0 |
| **Outils** | Python · Pandas · Seaborn · Plotly · Scikit-learn |
| **Livrable BI** | Dashboard Power BI multi-vues |

---

## Objectifs

1. **Comprendre** la nature et le volume des requêtes clients traitées
2. **Identifier** les catégories les plus chronophages et les goulots d'étranglement
3. **Mesurer** la performance des agents et le respect des SLA
4. **Modéliser** les délais de traitement pour anticiper les charges
5. **Produire** des données structurées pour le Dashboard Power BI

---

## Structure du projet

```
ProjectMoovAnalysis/
│
├── notebooks/                          # Analyses Jupyter
│   ├── 01_exploratory_analysis.ipynb   # Exploration initiale des données
│   └── 02_final_analysis.ipynb         # Analyse complète + ML + exports
│
├── outputs/                            # Visualisations générées
│   ├── output_clustering.png
│   ├── output_delais_traitement.png
│   ├── output_distribution_categories.png
│   ├── output_feature_importance.png
│   ├── output_heatmap_activite.png
│   └── output_performance_equipes.png
│
├── reports/                            # Livrables et documentation
│   ├── Moov_Africa_BackOffice_Analyse_FINAL.html
│   ├── Moov_Africa_BackOffice_Analyse_FINAL.pdf
│   └── Guide_PowerBI_MoovAfrica.md
│
├── powerbi/                            # Exports CSV pour Power BI (non versionnés)
├── data/                               # Données brutes (non versionnées)
│
├── .gitignore
├── requirements.txt
└── README.md
```

> **Sécurité** : Les dossiers `data/` et `powerbi/` contiennent des données confidentielles et sont exclus du contrôle de version (`.gitignore`).

---

## Pipeline d'analyse

```
Données brutes (XLSX)
        │
        ▼
01 · Exploration & qualité des données
        │
        ▼
02 · Nettoyage · Feature Engineering
        │
        ▼
03 · Analyse exploratoire (EDA)
    ├── Distribution des tickets par catégorie
    ├── Heatmap d'activité temporelle
    └── Performance des agents
        │
        ▼
04 · KPIs métier & SLA
        │
        ▼
05 · Modélisation ML
    ├── Clustering agents (KMeans)
    └── Prédiction délais (Random Forest · Gradient Boosting)
        │
        ▼
06 · Exports Power BI (CSV structurés)
```

---

## Résultats clés

| Indicateur | Résultat |
|---|---|
| Modèle de prédiction | Random Forest — meilleur R² |
| Segmentation agents | 3 clusters identifiés |
| Variables les plus impactantes | Catégorie · Priorité · Heure de soumission |
| Livrable BI | 6 tables CSV prêtes pour Power BI |

---

## Installation

```bash
# Cloner le dépôt
git clone https://github.com/jeankonan748/ProjectMoovAnalysis.git
cd ProjectMoovAnalysis

# Créer un environnement virtuel
python -m venv .venv
source .venv/bin/activate   # Windows : .venv\Scripts\activate

# Installer les dépendances
pip install -r requirements.txt
```

---

## Utilisation

```bash
# Lancer Jupyter
jupyter notebook

# Ordre d'exécution recommandé
# 1. notebooks/01_exploratory_analysis.ipynb
# 2. notebooks/02_final_analysis.ipynb
```

> Placer le fichier de données brutes dans le dossier `data/` avant d'exécuter les notebooks.

---

## Dépendances

| Librairie | Usage |
|---|---|
| `pandas` | Manipulation des données |
| `numpy` | Calcul numérique |
| `matplotlib` / `seaborn` | Visualisation statique |
| `plotly` | Visualisation interactive |
| `scikit-learn` | Machine Learning |
| `scipy` | Tests statistiques |

---

## Licence

Projet réalisé dans le cadre d'une mission d'analyse de données pour **Moov Africa**.  
Toute reproduction ou redistribution des données est strictement interdite.

---

*Jean KONAN — Data Analyst / Data Scientist*
