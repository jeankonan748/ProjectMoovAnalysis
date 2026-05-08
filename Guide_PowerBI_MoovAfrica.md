# 📊 Guide Dashboard Power BI
## Optimisation Back Office Customer Care — Moov Africa

---

## 1. 🏗️ Architecture du modèle de données (Star Schema)

```
                    ┌─────────────────────┐
                    │   dim_categories    │
                    │─────────────────────│
                    │ categorie (PK)      │
                    │ volume              │
                    │ delai_moyen_h       │
                    │ taux_sla            │
                    └──────────┬──────────┘
                               │
┌──────────────┐    ┌──────────▼──────────┐    ┌──────────────────────┐
│  dim_agents  │    │   fait_tickets      │    │   predictions_ml     │
│──────────────│    │─────────────────────│    │──────────────────────│
│ agent_id (PK)│◄───│ ticket_id (PK)      │───►│ ticket_id (FK)       │
│ nom_agent    │    │ agent_id (FK)       │    │ delai_reel_h         │
│ equipe       │    │ categorie (FK)      │    │ delai_predit_h       │
│ taux_sla     │    │ date_creation       │    │ cluster_label        │
│ anciennete   │    │ delai_traitement_h  │    └──────────────────────┘
└──────────────┘    │ respecte_sla        │
                    │ satisfaction_score  │    ┌──────────────────────┐
                    │ montant_litige      │    │   kpis_mensuels      │
                    │ cluster_label       │    │──────────────────────│
                    │ annee / mois        │───►│ annee + mois (PK)    │
                    └─────────────────────┘    │ volume               │
                                               │ taux_sla_pct         │
                                               │ satisfaction_moy     │
                                               └──────────────────────┘
```

---

## 2. 📥 Importation des données

**Étapes dans Power BI Desktop :**

1. `Accueil` → `Obtenir des données` → `Texte/CSV`
2. Importer dans cet ordre :
   - `fait_tickets.csv` (séparateur `;`, encodage UTF-8)
   - `dim_agents.csv`
   - `dim_categories.csv`
   - `kpis_mensuels.csv`
   - `predictions_ml.csv`
3. Dans **Éditeur Power Query** :
   - Vérifier que `date_creation` est bien en type **Date/Heure**
   - Vérifier que les colonnes numériques sont en type **Nombre décimal**
   - Cocher `Utiliser la première ligne comme en-tête`

---

## 3. 🔗 Relations entre tables

Dans `Modélisation` → `Gérer les relations` :

| Table source        | Colonne       | Table cible      | Colonne   | Cardinalité |
|---------------------|---------------|------------------|-----------|-------------|
| fait_tickets        | agent_id      | dim_agents       | agent_id  | N:1         |
| fait_tickets        | categorie     | dim_categories   | categorie | N:1         |
| fait_tickets        | ticket_id     | predictions_ml   | ticket_id | 1:1         |
| fait_tickets        | annee+mois    | kpis_mensuels    | annee+mois| N:1         |

---

## 4. 📐 Mesures DAX essentielles

Créer une table `[_Mesures]` vide pour organiser toutes les mesures.

### 4.1 Mesures de Volume

```dax
// Total tickets
Total Tickets = COUNTROWS(fait_tickets)

// Tickets résolus
Tickets Résolus =
CALCULATE(
    COUNTROWS(fait_tickets),
    fait_tickets[statut] IN {"Résolu", "Fermé"}
)

// Tickets en cours
Tickets En Cours =
CALCULATE(
    COUNTROWS(fait_tickets),
    fait_tickets[statut] = "En cours"
)

// Tickets escaladés
Tickets Escaladés =
CALCULATE(
    COUNTROWS(fait_tickets),
    fait_tickets[statut] = "Escaladé"
)
```

### 4.2 Mesures SLA

```dax
// Taux de conformité SLA
Taux SLA % =
DIVIDE(
    CALCULATE(COUNTROWS(fait_tickets), fait_tickets[respecte_sla] = TRUE()),
    [Total Tickets],
    0
) * 100

// Tickets hors SLA
Tickets Hors SLA =
CALCULATE(
    COUNTROWS(fait_tickets),
    fait_tickets[respecte_sla] = FALSE()
)

// Dépassement moyen SLA (heures)
Dépassement Moyen SLA (h) =
CALCULATE(
    AVERAGE(fait_tickets[depassement_sla_h]),
    fait_tickets[respecte_sla] = FALSE()
)

// Statut SLA dynamique (pour card colorée)
Statut SLA =
SWITCH(
    TRUE(),
    [Taux SLA %] >= 90, "✅ Excellent",
    [Taux SLA %] >= 80, "🟡 Correct",
    [Taux SLA %] >= 70, "🟠 Attention",
    "🔴 Critique"
)
```

### 4.3 Mesures de Délai

```dax
// Délai moyen de traitement
Délai Moyen (h) = AVERAGE(fait_tickets[delai_traitement_h])

// Délai médian
Délai Médian (h) = MEDIAN(fait_tickets[delai_traitement_h])

// Délai P90 (90ème percentile)
Délai P90 (h) = PERCENTILE.INC(fait_tickets[delai_traitement_h], 0.9)

// Délai moyen par catégorie (pour comparaison)
Délai Moyen Catégorie (h) =
AVERAGEX(
    VALUES(fait_tickets[categorie]),
    CALCULATE(AVERAGE(fait_tickets[delai_traitement_h]))
)
```

### 4.4 Mesures Qualité

```dax
// Taux de première résolution
Taux 1ère Résolution % =
DIVIDE(
    CALCULATE(COUNTROWS(fait_tickets), fait_tickets[premiere_resolution] = TRUE()),
    [Total Tickets],
    0
) * 100

// Score satisfaction moyen
Satisfaction Moyenne = AVERAGE(fait_tickets[satisfaction_score])

// Taux d'escalade
Taux Escalade % =
DIVIDE([Tickets Escaladés], [Total Tickets], 0) * 100

// Nombre moyen de relances
Relances Moyennes = AVERAGE(fait_tickets[nb_relances])
```

### 4.5 Mesures Agents

```dax
// Volume moyen par agent
Tickets / Agent =
DIVIDE([Total Tickets], DISTINCTCOUNT(fait_tickets[agent_id]), 0)

// Top agent (délai minimal)
Meilleur Agent (Délai) =
CALCULATE(
    FIRSTNONBLANKVALUE(dim_agents[nom_agent],
        CALCULATE(AVERAGE(fait_tickets[delai_traitement_h]))),
    TOPN(1, dim_agents, CALCULATE(AVERAGE(fait_tickets[delai_traitement_h])))
)

// Productivité agent (tickets / objectif)
Productivité % =
DIVIDE(
    AVERAGE(dim_agents[nb_tickets]),
    AVERAGE(dim_agents[objectif_tickets_j]) * 22,
    0
) * 100
```

### 4.6 Mesures Financières

```dax
// Montant total des litiges
Montant Litiges Total (FCFA) =
SUM(fait_tickets[montant_litige])

// Montant moyen par litige
Montant Moyen Litige (FCFA) =
CALCULATE(
    AVERAGE(fait_tickets[montant_litige]),
    fait_tickets[montant_litige] > 0
)
```

---

## 5. 📄 Structure des 4 pages du Dashboard

---

### PAGE 1 — Vue Opérationnelle Globale

**Objectif :** Vision d'ensemble en temps réel pour les managers

**Visuels à créer :**

| Visuel | Type | Champs | Position |
|--------|------|--------|----------|
| Total Tickets | Carte | [Total Tickets] | Haut gauche |
| Taux SLA | Jauge | [Taux SLA %] (max=100) | Haut centre |
| Satisfaction | Carte | [Satisfaction Moyenne] | Haut droite |
| Évolution mensuelle | Courbe | Axe: mois_nom, Valeurs: [Total Tickets] | Centre |
| Répartition statuts | Anneau | Légende: statut, Valeurs: [Total Tickets] | Centre droite |
| Top catégories | Barres horizontales | Axe: categorie, Valeurs: [Total Tickets] | Bas gauche |
| Heatmap activité | Matrice | Lignes: jour_semaine, Colonnes: heure | Bas centre |

**Filtres (Slicers) :**
- Période (date_creation) : slicer date entre-entre
- Région : slicer liste déroulante
- Canal : slicer boutons
- Priorité : slicer cases à cocher

**Formatage :**
- Couleur principale : `#003399` (bleu Moov)
- Couleur secondaire : `#FF6600` (orange Moov)
- Police : Segoe UI, 12pt
- Fond : blanc `#FFFFFF`

---

### PAGE 2 — Analyse des Délais & SLA

**Objectif :** Suivi détaillé de la performance opérationnelle

**Visuels à créer :**

| Visuel | Type | Champs |
|--------|------|--------|
| Taux SLA % | Jauge colorée | [Taux SLA %], objectif=85 |
| Délai Moyen | Carte | [Délai Moyen (h)] |
| Délai P90 | Carte | [Délai P90 (h)] |
| Délai par catégorie | Barres groupées | Axe: categorie, Valeurs: [Délai Moyen (h)], [Délai P90 (h)] |
| Évolution SLA | Courbe | Axe: mois_nom, Valeurs: [Taux SLA %] |
| Distribution délais | Histogramme | Bins par tranche_horaire |
| SLA par priorité | Matrice | Lignes: priorite, Colonnes: respecte_sla, Valeurs: [Total Tickets] |
| Tickets hors SLA | Tableau | Colonnes: ticket_id, categorie, priorite, delai, agent_id |

**DAX spéciale pour formatage conditionnel du tableau SLA :**

```dax
// Couleur du délai (formatage conditionnel)
Couleur Délai =
SWITCH(
    TRUE(),
    SELECTEDVALUE(fait_tickets[delai_traitement_h]) <= 12, "#28A745",
    SELECTEDVALUE(fait_tickets[delai_traitement_h]) <= 24, "#FFC107",
    "#DC3545"
)
```

---

### PAGE 3 — Performance Agents & Équipes

**Objectif :** Suivi RH et pilotage des équipes Back Office

**Visuels à créer :**

| Visuel | Type | Champs |
|--------|------|--------|
| Classement agents | Tableau trié | nom_agent, equipe, nb_tickets, taux_sla, satisfaction |
| Charge vs Délai | Nuage de points | X: nb_tickets, Y: delai_moyen_h, Taille: satisfaction, Couleur: equipe |
| KPIs par équipe | Histogramme groupé | Axe: equipe, Valeurs: [Délai Moyen], [Taux SLA %] |
| Taux résolution | Barres 100% | Axe: equipe, Valeurs: taux_1ere_resolution |
| Ancienneté vs perf | Dispersion | X: anciennete_mois, Y: taux_resolution |
| Top 5 agents | Tableau | Filtre: TOPN(5, dim_agents, taux_sla) |
| Spécialité agents | Treemap | Catégorie: specialite, Taille: nb_tickets |

**Formatage conditionnel pour le tableau agents :**
- `taux_sla` ≥ 85% → vert
- `taux_sla` entre 70-85% → orange
- `taux_sla` < 70% → rouge

---

### PAGE 4 — Prédictions & Alertes ML

**Objectif :** Exploitation des résultats du modèle ML

**Visuels à créer :**

| Visuel | Type | Champs |
|--------|------|--------|
| Précision modèle | Carte | Écart moyen prédit/réel |
| Segmentation clusters | Anneau | Légende: cluster_label, Valeurs: count |
| Réel vs Prédit | Dispersion | X: delai_reel_h, Y: delai_predit_h |
| Profil clusters | Radar/Araignée | Axes: features ML, par cluster |
| Tickets à risque | Tableau | Filtré: delai_predit_h > 24h et statut=En cours |
| Prédictions par cat | Barres | categorie vs delai_predit_h moyen |
| Timeline alertes | Tableau | Tickets proches SLA avec alerte |

**DAX pour tableau des alertes :**

```dax
// Tickets à risque de dépasser SLA
Alerte Délai =
CALCULATE(
    COUNTROWS(predictions_ml),
    predictions_ml[delai_predit_h] > 20,
    predictions_ml[delai_predit_h] <= 24
)

// Tickets déjà en dépassement prédit
Dépassement Prédit =
CALCULATE(
    COUNTROWS(predictions_ml),
    predictions_ml[delai_predit_h] > 24
)
```

---

## 6. 🎨 Thème Power BI personnalisé

Crée un fichier `theme_moovAfrica.json` et importe-le via `Affichage` → `Thèmes` :

```json
{
  "name": "Moov Africa",
  "dataColors": [
    "#003399", "#FF6600", "#0055CC", "#FF8833",
    "#0077FF", "#FFAA55", "#003380", "#CC5500"
  ],
  "background": "#FFFFFF",
  "foreground": "#003399",
  "tableAccent": "#FF6600",
  "visualStyles": {
    "*": {
      "*": {
        "fontFamily": [{"value": "Segoe UI"}],
        "fontSize": [{"value": 11}]
      }
    }
  }
}
```

---

## 7. 🔄 Rafraîchissement automatique

Pour un rafraîchissement automatique des données :

1. Publier le rapport sur **Power BI Service**
2. `Jeux de données` → `Planifier l'actualisation`
3. Configurer : Quotidien à 7h00 et 12h00
4. Connecter via **Power BI Gateway** si les CSV sont en local

---

## 8. ✅ Checklist avant publication

- [ ] Toutes les relations entre tables sont vérifiées
- [ ] Les filtres croisés fonctionnent sur toutes les pages
- [ ] Les mesures DAX renvoient les bonnes valeurs
- [ ] Le thème Moov Africa est appliqué
- [ ] Les slicers sont synchronisés entre les pages (`Affichage` → `Synchroniser les slicers`)
- [ ] Les tooltips sont configurés sur les visuels principaux
- [ ] Le rapport est optimisé (éviter les visuels > 10 000 points)
- [ ] La sécurité au niveau des lignes (RLS) est configurée si nécessaire

---

*Guide produit dans le cadre du projet Optimisation Back Office — Moov Africa*
