# Modèles

Cette section présente les différentes familles de modèles testées pour la prévision de la demande électrique, ainsi que les choix finaux retenus.

---

## 1. Modèles classiques (Machine Learning & statistiques)

| Modèle        | Description                                                  | Usage               |
|---------------|--------------------------------------------------------------|---------------------|
| **ElasticNet**| Régression linéaire avec régularisation L1 (Lasso) et L2 (Ridge).<br>Permet la sélection de variables. | Baseline, sélection de features |
| **Ridge**     | Régression linéaire avec pénalité L2.                        | Baseline           |
| **RandomForest** | Modèle d’ensemble (bagging) d’arbres de décision.<br>Robuste face aux non-linéarités et peu sensible aux outliers. | Meilleur hourly/journalier sur plusieurs zones |
| **LightGBM**  | Gradient boosting séquentiel optimisé pour la rapidité.<br>Excellent pour les gros jeux de données. | Tuning avancé (optionnel) |
| **ARIMA**     | Modèle statistique pour séries temporelles non stationnaires.<br>Auto-SARIMAX avec pmdarima (`auto_arima`). | Capturer saisonnalité journalière/hebdomadaire |

### Exemple d’entraînement (RandomForest)

```python
from sklearn.ensemble import RandomForestRegressor
from src.models.modeling import train_test_split_zone, evaluate_regression

# Chargement des données processed
df = pd.read_parquet("data/processed/Peninsule_Iberique_processed_daily.parquet")
X_train, X_test, y_train, y_test = train_test_split_zone(df, target="demand")

# Entraînement
rf = RandomForestRegressor(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

# Évaluation
metrics = evaluate_regression(rf, X_test, y_test)
print(metrics)
```

---

## 2. Modèles Deep Learning

Nous avons expérimenté quatre architectures principales :

1. **LSTM** (Long Short-Term Memory)

   * Capture les dépendances longues dans les séries.
   * Architecture simple : couche LSTM → Dropout → Dense.

2. **GRU** (Gated Recurrent Unit)

   * Variante légère de LSTM.
   * Moins de paramètres, convergence plus rapide.

3. **1D CNN**

   * Convolutions sur la dimension temporelle pour extraire des motifs locaux.
   * Architecture : Conv1D → Pooling → Conv1D → Pooling → Flatten → Dense.

4. **CNN–LSTM**

   * Combine CNN pour extraire des caractéristiques locales et LSTM pour la dépendance temporelle.
   * Architecture : TimeDistributed(CNN) → LSTM → Dense.

### Exemple de création d’un LSTM

```python
from src.models.dl_models import build_lstm
from src.models.dl_utils   import create_sequences, scale_data
from src.models.dl_training import train_and_save

# Préparation des séquences
df = pd.read_parquet("data/processed/Peninsule_Iberique_processed_hourly.parquet")
features = [c for c in df.columns if c != "demand"]
X, y = create_sequences(df, "demand", features, lookback=24, horizon=1)

# Split et scaling
split  = int(len(X)*0.8)
X_train, X_val = X[:split], X[split:]
y_train, y_val = y[:split], y[split:]
X_train_s, X_val_s, scaler = scale_data(X_train, X_val)

# Build & train
model = build_lstm(input_shape=(24, len(features)))
hist, filepath = train_and_save(
    model, X_train_s, y_train, X_val_s, y_val,
    models_dir="models/dl", zone="Peninsule_Iberique",
    horizon="hourly", name="LSTM",
    epochs=50, batch_size=32
)
```

---

## 3. Sélection finale des modèles

* À l’issue de la comparaison des **RMSE**, le meilleur modèle par zone et horizon est enregistré dans `data/submission/best_models.csv`.
* Ces modèles sont ensuite chargés automatiquement dans le dashboard Streamlit pour la production.

```yaml
# Extrait de best_models.csv
zone,best_model_daily,best_model_hourly
Peninsule_Iberique,Ridge,RandomForest
Baleares,Ridge,RandomForest
...
```

![Comparaison RMSE](images/rmse_comparison.png)

---

> **À suivre** :
> Passe à la section **Résultats** pour visualiser en détail les performances et anomalies détectées.

---
