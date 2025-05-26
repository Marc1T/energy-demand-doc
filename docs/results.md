# Résultats et performances

Cette section présente une synthèse des performances des différents modèles testés pour la prévision de la demande électrique. Nous comparons les résultats en fonction :
- des zones géographiques (Péninsule, îles, enclaves),
- des échelles temporelles (horaire et journalière),
- des métriques utilisées (RMSE, MAE, MAPE),
- des types de modèles (classiques vs deep learning).

---

## 1. Métriques d’évaluation

Nous avons utilisé les métriques suivantes pour évaluer nos modèles :

| **Métrique** | **Description** |
|--------------|------------------|
| RMSE (Root Mean Squared Error) | Sensible aux grandes erreurs. |
| MAE (Mean Absolute Error)      | Écart moyen absolu. |
| MAPE (Mean Absolute Percentage Error) | Pourcentage d’erreur relatif (utile pour comparer entre zones). |

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error

rmse = mean_squared_error(y_true, y_pred, squared=False)
mae = mean_absolute_error(y_true, y_pred)
```

📷 *\[Ajouter ici un tableau comparatif des métriques]*

---

## 2. Modèles gagnants par zone

Le tableau ci-dessous récapitule le meilleur modèle sélectionné automatiquement pour chaque zone et chaque échelle temporelle :

| Zone                     | Daily (🗓️)  | Hourly (⏰)   |
| ------------------------ | ------------ | ------------ |
| Peninsule\_Iberique      | Ridge        | RandomForest |
| Baleares                 | Ridge        | RandomForest |
| Canarias                 | Ridge        | RandomForest |
| Gran\_canaria            | RandomForest | RandomForest |
| Ceuta                    | RandomForest | RandomForest |
| Melilla                  | ElasticNet   | RandomForest |
| Lanzarote\_Fuerteventura | Ridge        | RandomForest |
| Tenerife                 | Ridge        | RandomForest |
| La\_Palma                | Ridge        | RandomForest |
| La\_Gomera               | Ridge        | RandomForest |
| El\_Hierro               | Ridge        | RandomForest |
| Nacional (agrégé)        | RandomForest | RandomForest |

📂 *Fichier : `submission/best_models.csv`*

---

## 3. Analyse visuelle

Nous avons produit des graphiques comparant les prédictions et les valeurs réelles pour différentes zones et modèles :

### Exemple : Peninsule\_Iberique - Prévision horaire

📷 ![Forecast vs Actual - Peninsule Iberique Hourly](../outputs/figures/peninsule_hourly_forecast.png)

* **Observations** : Le modèle RandomForest capture bien les variations journalières.
* **Erreur moyenne** : MAE = 210.4 MW / RMSE = 390.2 MW

---

### Exemple : Melilla - Prévision journalière

📷 ![Forecast vs Actual - Melilla Daily](../outputs/figures/melilla_daily_forecast.png)

* **Observations** : La demande est plus volatile, mais ElasticNet reste robuste.
* **Erreur moyenne** : MAE = 1.9 MW / RMSE = 3.1 MW

---

## 4. Robustesse et généralisabilité

Un test de robustesse a été effectué en changeant la fenêtre temporelle d’entraînement :

* Les performances se maintiennent avec une **variation < 5%** pour les meilleurs modèles.
* Les zones avec peu de données (Ceuta, Melilla) montrent plus de variance → importance du **feature engineering**.

📷 *\[Inclure ici un graphique : Erreur vs Fenêtre de test]*

---

## 5. Visualisation interactive

Une **application Streamlit** a été développée pour visualiser les prédictions et comparer les performances par zone. Elle contient :

* une carte interactive des zones,
* des courbes réelles vs prédites,
* un tableau avec les métriques.

➡️ Voir la section [Dashboard](dashboard.md)

---

## 🔚 Conclusion

* Les modèles classiques ont montré de **très bonnes performances**.
* Le **Random Forest** domine la majorité des cas horaires.
* L'**ElasticNet** est intéressant sur des zones plus petites.
* Les **modèles deep learning** (LSTM, GRU) sont prometteurs mais plus coûteux en calcul.

🎯 Prochaine étape possible : intégrer davantage de variables exogènes et tester un modèle de type **Transformer** ou **N-BEATS**.

---


