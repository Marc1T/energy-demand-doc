# Données utilisées & Préparation

![Flux des données](images/data_flow.png)

---

## 1. Sources & indicateurs

| Source            | Indicateur                        | Période      | Zones couvertes                             |
|-------------------|-----------------------------------|--------------|----------------------------------------------|
| **ESIOS / REE**   | Consommation électrique (ID=2037) | 2015-2025    | National (`nacional`), 11 régions            |
| **ESIOS / REE**   | PVPC (ID=10229)                   | 2015-2021-06 | National                                    |
| **ESIOS / REE**   | PVPC (ID=10391)                   | 2021-06-2025 | Régional (geo_ids: 8741–8745)               |
| **Open-Meteo**    | Météo horaire (températures, vent, précipitations, etc.) | 2015-2025 | 11 zones                                    |
| **holidays**      | Jours fériés officiels            | 2015-2025    | National → dupliqué par région avant 2021-06 |

> **Remarque** : les `geo_ids` et `indicator_id` sont paramétrés dans `config.yaml`.

---

## 2. Récupération des données brutes

### 2.1 API ESIOS — fenêtre mensuelle
Pour éviter les **timeouts** et les erreurs 504, nous découpons la période en **fenêtres d’un mois** :

```python
from src.data.external_data import generate_periodic_windows, fetch_indicator_hourly

# Exemple de génération de fenêtres
windows = generate_periodic_windows("2015-05-01", "2025-05-01", months=1)
# Puis pour chaque fenêtre, on appelle fetch_indicator_hourly()
df_parts = []
for start, end in windows:
    df_month = fetch_indicator_hourly(
        indicator_id=2037,      # demande national
        token=ESIOS_TOKEN,
        start_date=to_esios_date(start),
        end_date=to_esios_date(end),
        base_url=API_URL
    )
    df_parts.append(df_month)
# Concaténation finale
df_hourly = pd.concat(df_parts).sort_index().drop_duplicates()
```

Chaque `df_hourly` est alors sauvegardé :

```python
df_hourly.to_csv("data/raw/nacional_hourly.csv", index_label="datetime")
df_hourly.resample("D").sum().to_csv("data/raw/nacional_daily.csv", index_label="date")
```

![Extrait CSV raw](images/csv_example.png)

---

## 3. Données exogènes

### 3.1 Météo via Open‑Meteo

Nous récupérons les variables suivantes **horaire** pour chaque zone :

* `temperature_2m`
* `relative_humidity_2m`
* `wind_speed_10m`
* `shortwave_radiation`
* `precipitation`

```python
from src.data.external_data import fetch_open_meteo

df_weather = fetch_open_meteo(
    lat=40.4168, lon=-3.7038,
    start="2015-05-01", end="2025-05-01",
    vars=["temperature_2m","relative_humidity_2m","wind_speed_10m","shortwave_radiation","precipitation"]
)
df_weather.to_csv("data/external/Madrid_weather_hourly.csv", index_label="datetime")
df_weather.resample("D").mean().to_csv("data/external/Madrid_weather_daily.csv", index_label="date")
```

![Exemple météo](images/weather_example.png)

### 3.2 Jours fériés

À l’aide de la bibliothèque `holidays`, on génère tous les jours fériés nationaux :

```python
import holidays

es_hols = holidays.Spain(years=range(2015, 2026))
records = [{"date": d.strftime("%Y-%m-%d"), "holiday": name} for d, name in es_hols.items()]
df_hols = pd.DataFrame(records).drop_duplicates().sort_values("date")
df_hols.to_csv("data/external/spain_holidays.csv", index=False)
```

Avant 2021‑06, ces jours fériés sont **dupliqués** pour chaque région.

---

## 4. Fusion & prétraitement initial (`interim`)

Dans `docs/intermediate_flow.png` (cf. image ci‑dessus), on illustre :

1. **Lecture** de tous les CSV horaires bruts et exogènes
2. **Merge** sur la colonne `datetime` (UTC)
3. **Ajout** de variables temporelles :

   * `hour`, `dayofweek`, `is_weekend`, `is_holiday`
   * `month`, `dayofyear`
4. **Imputation** des valeurs manquantes : interpolation linéaire + forward/backward fill
5. **Création** de lags & rolling windows (intervalles 1, 3, 6, 12, 24, 72 heures)

```python
from src.data.preprocessing import preprocess_zone

df_int = preprocess_zone("Peninsule_Iberique")
df_int.to_pickle("data/interim/Peninsule_Iberique_interim_hourly.pkl")
df_int.resample("D").agg({
    "demand":"sum","pvpc":"mean",
    "temperature_2m":"mean", ...
}).to_pickle("data/interim/Peninsule_Iberique_interim_daily.pkl")
```

---

## 5. Données finales (`processed`)

Après sélection des variables pertinentes (ElasticNet, importance RF…), on construit le **jeu final** :

```python
from src.features.feature_engineering import select_features, transform_for_model

df_final = transform_for_model(
    df_int, features=selected_features_list
)
df_final.to_parquet("data/processed/Peninsule_Iberique_processed_hourly.parquet")
```

* **Sélection** : `features_selected_hourly.csv` et `features_selected_daily.csv`
* **Partition** 80 % train / 20 % test
* **Format** Parquet pour un chargement rapide

---

## 6. Feature Engineering Avancé

Pour améliorer la performance des modèles, nous avons mis en place une **pipeline de transformation automatique**, en appliquant les étapes suivantes à chaque zone :

#### ⚙️ Étapes appliquées pour chaque série :

```python
df = pd.read_pickle(inter_dir/f"{zone}_interim_hourly.pkl")

df = add_degree_days(df)             # Ajout Heating Degree Days & Cooling Degree Days
df = add_comfort_indices(df)         # Humidex, Apparent Temperature...
df = add_rolling_features(df)        # Moyennes / écarts-types glissants sur 1h, 3h, 6h...
df = add_cyclic_time_features(df)    # sin/cos de l'heure et du jour de l'année
df = add_interactions(df)            # Termes d’interaction entre variables météo, temporelles et passées
df = df.ffill().bfill()              # Imputation finale
```

Exemple de colonnes générées :

| Variable                      | Description                    |
| ----------------------------- | ------------------------------ |
| `lag_1h`, `lag_3h`, `lag_24h` | Valeurs de demande passées     |
| `mean_6h_temperature_2m`      | Moyenne glissante              |
| `sin_hour`, `cos_hour`        | Représentation cyclique        |
| `temperature_2m * lag_3h`     | Interaction météo / historique |
| `humidex * pvpc`              | Interaction exogène            |

---

## 7. Sélection des Variables Pertinentes

Plutôt que d’entraîner les modèles sur toutes les variables, nous avons automatisé une **sélection intelligente** de features grâce à une méthode basée sur la **mutual information** :

```python
features_to_keep = ["demand", "lag_1h", ..., "lag_24h"]
features_to_keep += select_features(df, top_k=12).index.tolist()
features_to_keep = list(set(features_to_keep))
```
> voici la fonction `select_features`
```python
    import pandas as pd
    from pathlib import Path
    from sklearn.feature_selection import mutual_info_regression
    # mutual information et importance simple
    def select_features(df, target="demand", top_k=20, drop_na_thresh=0.2, random_state=0):
        # Retirer les colonnes avec trop de NaN
        na_frac = df.isna().mean()
        cols_keep = na_frac[na_frac < drop_na_thresh].index.tolist()
        if target not in cols_keep:
            cols_keep.append(target)
        df = df[cols_keep].copy()
        
        # Remplir les NaN restants
        X = df.drop(columns=[target]).fillna(df.median(numeric_only=True))
        y = df[target].values

        # Mutual information
        mi = mutual_info_regression(X, y, discrete_features=False, random_state=random_state)
        mi_s = pd.Series(mi, index=X.columns).sort_values(ascending=False)
        return mi_s.head(top_k)
```
Le fichier suivant a été généré et utilisé pour la suite :

📄 `submission/features_selected_hourly.csv`

```csv
zone,features
Peninsule_Iberique,"['demand', 'lag_1h', ..., 'temperature_2m * lag_3h']"
...
```

> 🔍 **Pourquoi c’est important ?**
> En réduisant le nombre de variables à celles qui sont **réellement explicatives**, on :
>
> * Évite l’overfitting
> * Réduit le temps d’entraînement
> * Améliore la robustesse du modèle

---

## 8. Datasets finaux produits

Pour chaque zone :

| Dataset                                    | Format  | Description                                  |
| ------------------------------------------ | ------- | -------------------------------------------- |
| `*_processed_hourly.parquet`               | Parquet | Données d’entraînement (jusqu’au 01/01/2024) |
| `*_processed_hourly.parquet` (submission/) | Parquet | Données de test pour soumission              |
| `features_selected_hourly.csv`             | CSV     | Variables retenues pour chaque zone          |

---

> ⚠️ **Note** : Le même processus est appliqué pour les séries journalières (`daily`) avec des variables agrégées.

---

>[ **À suivre :**](models.md)
> Rendez‑vous dans la section [**Modèles**](models.md) pour découvrir en détail les approches de machine learning et deep learning utilisées.

---

<!-- les images mentionnées (`data_flow.png`, `csv_example.png`, `weather_example.png`, `intermediate_flow.png`)   -->

