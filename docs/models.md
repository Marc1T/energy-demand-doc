# Modèles de prévision

Dans ce projet, nous avons testé plusieurs modèles de machine learning et de deep learning pour prédire la demande électrique en Espagne à différentes échelles temporelles (horaire et journalière). Chaque modèle a été évalué sur ses performances de prédiction, sa robustesse et sa capacité à généraliser selon les zones géographiques.

---

## 1. Modèles classiques de Machine Learning

### 🎯 Régression Ridge

- **Principe** : Variante de la régression linéaire avec régularisation L2 pour éviter le surapprentissage.
- **Pourquoi l'utiliser ?** : Simple, rapide et efficace avec des séries temporelles bien prétraitées.
- **Utilisation dans le projet** :
  - Testé comme baseline.
  - Bonnes performances sur certaines zones, notamment "Baleares" et "Peninsule_Iberique".
- **Avantages** :
  - Interprétable.
  - Faible coût computationnel.
- **Inconvénients** :
  - Faible capacité à modéliser la non-linéarité.

📷 *[Inclure ici un screenshot de la prédiction Ridge vs réalité]*

---

### 🎯 ElasticNet

- **Principe** : Combine régularisation L1 (Lasso) et L2 (Ridge).
- **Pourquoi l'utiliser ?** : Utile quand plusieurs features sont corrélées.
- **Utilisation dans le projet** : Meilleur modèle sur certaines zones comme "Melilla".
- **Avantages** :
  - Sélectionne automatiquement les variables pertinentes.
- **Limites** :
  - Moins stable sur les petites séries.

---

### 🌳 Random Forest

- **Principe** : Ensemble de plusieurs arbres de décision entraînés sur des sous-échantillons.
- **Pourquoi l'utiliser ?** : Très robuste, performant sur des données non linéaires.
- **Utilisation dans le projet** :
  - Meilleur modèle horaire dans la majorité des zones.
  - Également utilisé en journalier.
- **Avantages** :
  - Prédictions stables.
  - Gère les interactions implicites entre variables.
- **Inconvénients** :
  - Moins interprétable.
  - Plus lent à entraîner.

📷 *[Inclure un graphe RF : importance des variables]*

---

### ⚡ LightGBM

- **Principe** : Boosting d'arbres (Gradient Boosting) très rapide.
- **Pourquoi l'utiliser ?** : Excellente performance pour les problèmes structurés.
- **Utilisation dans le projet** : Testé mais parfois battu par Random Forest.
- **Avantages** :
  - Efficace en entraînement.
  - Gère bien les séries avec bruit.
- **Limites** :
  - Sensible à l'overfitting sans réglage.

---

### ⏳ ARIMA (AutoRegressive Integrated Moving Average)

- **Principe** : Modélisation basée uniquement sur la série passée.
- **Pourquoi l'utiliser ?** : Baseline statistique fiable.
- **Utilisation dans le projet** :
  - Modèle de référence sur séries stationnaires.
  - Moins compétitif que les modèles ML en cas de forte variabilité.
- **Avantages** :
  - Interprétable et théoriquement solide.
- **Limites** :
  - Ne prend pas en compte les variables exogènes.
  - Moins performant sur des séries complexes.

---

## 2. Modèles Deep Learning

### 🧠 LSTM (Long Short-Term Memory)

- **Principe** : Réseau de neurones récurrents adapté à la mémoire à long terme.
- **Pourquoi l'utiliser ?** : Excellent pour capturer des dépendances temporelles lointaines.
- **Utilisation dans le projet** :
  - Prévision de la demande horaire et journalière.
  - Exige un preprocessing soigné.
- **Avantages** :
  - Prend en compte le contexte historique.
- **Inconvénients** :
  - Long à entraîner.
  - Sensible aux hyperparamètres.

📷 *[Inclure courbe d'apprentissage + prédiction LSTM vs réelle]*

---

### 🔁 GRU (Gated Recurrent Unit)

- **Principe** : Variante simplifiée de LSTM.
- **Pourquoi l'utiliser ?** : Moins coûteux que LSTM, performances souvent comparables.
- **Utilisation dans le projet** :
  - Utilisé comme alternative à LSTM.
- **Avantages** :
  - Moins de paramètres.
- **Limites** :
  - Légèrement moins performant pour de longues dépendances.

---

### 🧩 CNN-1D

- **Principe** : Applique des filtres convolutifs sur les séries pour détecter des patterns locaux.
- **Pourquoi l'utiliser ?** : Très bon pour détecter des motifs cycliques.
- **Utilisation dans le projet** :
  - Prévision sur des séquences fixes (input sliding window).
- **Avantages** :
  - Rapide et efficace.
- **Limites** :
  - Contexte temporel plus local que LSTM/GRU.

---

### 🔀 CNN-LSTM

- **Principe** : Combine un bloc CNN (extraction de motifs) + LSTM (séquence).
- **Pourquoi l'utiliser ?** : Combine les points forts des deux approches.
- **Utilisation dans le projet** :
  - Prévision avancée avec séquence d’entrée multi-feature.
- **Avantages** :
  - Très performant si bien calibré.
- **Inconvénients** :
  - Coût d'entraînement élevé.

---

## 🧪 Sélection des meilleurs modèles

Après entraînement, chaque modèle a été évalué à l’aide de :
- RMSE (Root Mean Squared Error)
- MAE (Mean Absolute Error)
- MAPE (Mean Absolute Percentage Error)

Les meilleurs modèles par zone ont été automatiquement sélectionnés puis sauvegardés :

```text
📁 submission/
├── Peninsule_Iberique_processed_hourly.parquet
├── Melilla_processed_daily.parquet
├── features_selected_hourly.csv
├── best_models.csv
```
metrics pas tres bons pour les modeles deep
```csv
zone,horizon,model,MAE,RMSE,MAPE
Ceuta,daily,LSTM,294.184478086403,297.5592351506879,79.46752716653282
Ceuta,daily,GRU,290.5378141736367,293.860330991986,79.22204350660417
Ceuta,daily,CNN,44.61273922839268,80.9607488539366,50.97870938287471
Ceuta,daily,CNN-LSTM,291.22003079047994,294.55189605442234,79.26797527424722
Ceuta,hourly,LSTM,0.6781376151244601,1.239811621017631,inf
Ceuta,hourly,GRU,0.6156814975927049,1.2077079105214479,inf
Ceuta,hourly,CNN,0.7745091407810173,1.5464381211602554,inf
Ceuta,hourly,CNN-LSTM,0.5962147199231576,1.1674468191333844,inf
```
pourtant on a de bon resultats avec les modeles classiques.
```csv
zone,daily_ElasticNet_MAE,daily_ElasticNet_RMSE,daily_ElasticNet_MAPE,daily_Ridge_MAE,daily_Ridge_RMSE,daily_Ridge_MAPE,daily_RandomForest_MAE,daily_RandomForest_RMSE,daily_RandomForest_MAPE,daily_auto_arima_MAE,daily_auto_arima_RMSE,daily_auto_arima_MAPE,hourly_ElasticNet_MAE,hourly_ElasticNet_RMSE,hourly_ElasticNet_MAPE,hourly_Ridge_MAE,hourly_Ridge_RMSE,hourly_Ridge_MAPE,hourly_RandomForest_MAE,hourly_RandomForest_RMSE,hourly_RandomForest_MAPE,hourly_auto_arima_MAE,hourly_auto_arima_RMSE,hourly_auto_arima_MAPE
Ceuta,0.014531369384538452,0.019753042463590328,0.011075109418962639,0.00020269046454933297,0.0002574187522392442,0.00013044864687825013,19.26567012477717,31.865167651828283,5.206201858597312,49.81000526183854,100.7861014693329,69.04778687335315,0.8051295554100869,1.4303024831726077,inf,0.7740636064212212,1.4207898605458968,inf,0.49929184159544143,1.0516865447893236,inf,inf,inf,inf
```
---

## ✅ Résumé

Chaque modèle a été testé de manière rigoureuse, avec :

* Prétraitement spécifique par zone.
* Sélection de features contextuelles.
* Validation croisée (CV).
* Sélection automatique du meilleur algorithme.

➡️ Voir la section [Résultats](results.md) pour les scores détaillés et la robustesse des modèles.


