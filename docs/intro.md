# Introduction

## Contexte

La consommation d’électricité varie fortement suivant l’heure de la journée, le jour de la semaine, la saison et la région géographique.  
Anticiper la demande avec précision est crucial pour :

- **Optimiser** la production et la distribution d’énergie  
- **Réduire** les coûts opérationnels  
- **Garantir** la stabilité du réseau électrique  
- **Accompagner** les énergies renouvelables intermittentes  

---

## Objectifs du projet

Ce projet vise à développer un pipeline complet de prévision de la demande électrique en Espagne, à deux granularités :

1. **Horaire** (prévision des 24 heures suivantes)  
2. **Journalière** (prévision des 7 jours suivants)

Le pipeline comprend :

- **Collecte** des données via l’API REE/ESIOS et sources externes (PVPC, météo, jours fériés).  
- **Prétraitement** : nettoyage, rééchantillonnage, fusion, création de features temporelles.  
- **Feature engineering** : lags, moyennes mobiles, degree days, indicateurs calendaires, exogènes.  
- **Modélisation** :  
  - Modèles classiques (ElasticNet, Ridge, RandomForest, LightGBM, ARIMA)  
  - Deep Learning (LSTM, GRU, 1D CNN, CNN–LSTM)  
- **Évaluation** : comparaison des RMSE pour chaque zone, sélection des meilleurs modèles.  
- **Déploiement** : dashboard Streamlit pour visualiser les prévisions et les KPI.

---

## Structure du projet

Voici l’arborescence **avant** l’ajout de la documentation MkDocs :

```
energy_demand_forecasting/
│
├── 📁 data/
│ ├── raw/ # Données brutes (API REE)
│ ├── interim/ # Données nettoyées et enrichies
│ ├── processed/ # Données prêtes pour l’entraînement
│ ├── external/ # Données exogènes (météo, jours fériés, PVPC…)
│ └── submission/ # Prédictions à soumettre, features sélectionnées
│
├── 📁 notebooks/ # Notebooks 01 à 08 : retrieval → dashboard
│
├── 📁 src/ # Code source modulaire
│ ├── data/
│ ├── features/
│ ├── models/
│ └── utils/
│
├── 📁 models/ # Modèles entraînés (.pkl, .h5)
├── 📁 outputs/ # Figures & rapports
├── 📁 dashboard/ # Code de l’app Streamlit/Dash
├── requirements.txt # Dépendances
├── README.md # Présentation générale
└── config.yaml # Paramètres & chemins
```

> **Remarque** : la documentation MkDocs a été ajoutée dans un dépôt/ dossier séparé `energy-demand-doc`  
> (contenant `mkdocs.yml`, `docs/` et `readthedocs.yml`), pour générer ce site statique hébergé sur Read the Docs.

---

Après lecture de cette introduction, rendez‑vous dans la section [**Données**](data.md) pour découvrir en détail les sources, formats et l’architecture de collecte.  
