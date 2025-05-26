# Dashboard de visualisation interactive

Dans le cadre de ce projet de prévision de la demande électrique, nous avons développé un **dashboard interactif** permettant de :

- Visualiser la demande réelle vs prédite pour chaque zone.
- Comparer les performances des modèles.
- Explorer dynamiquement les séries temporelles selon les filtres (zone, date, fréquence…).
- Faciliter la prise de décision en rendant les résultats accessibles à des non-experts.

---

## 🛠️ Outils utilisés

Le dashboard a été développé avec :

| Outil         | Rôle                                               |
|---------------|----------------------------------------------------|
| **Streamlit** | Développement rapide d’interface web en Python     |
| **Plotly**    | Visualisation interactive (zoom, survol, etc.)     |
| **Pandas**    | Chargement et traitement des données               |
| **PyYAML**    | Chargement des paramètres globaux (`config.yaml`)  |

📂 Fichiers :

```

📁 dashboard/
├── app.py           # Script principal
└── components.py    # Fonctions de visualisation personnalisées

```

---

## 🧩 Fonctionnalités du dashboard

### 1. Sélection de la zone

L'utilisateur peut sélectionner une **zone géographique** dans une liste déroulante. Les fichiers de données associés sont automatiquement chargés.

📷 *[Insérer ici une capture : menu déroulant des zones]*

---

### 2. Visualisation des courbes

Pour chaque zone, une **courbe interactive** montre la demande réelle et la prédiction du modèle choisi :

```python
import plotly.graph_objects as go

def plot_forecast(df, title="Prévision vs Réel"):
    fig = go.Figure()
    fig.add_trace(go.Scatter(x=df.index, y=df["real"], name="Réel"))
    fig.add_trace(go.Scatter(x=df.index, y=df["forecast"], name="Prévision"))
    fig.update_layout(title=title, xaxis_title="Date", yaxis_title="Demande (MW)")
    return fig
```

📷 *\[Insérer ici une capture d’écran avec la courbe interactive]*

---

### 3. Comparaison des modèles

Une section permet de visualiser les performances (RMSE, MAE, MAPE) selon le modèle et la fréquence :

* Graphique en barres comparant les scores
* Tableaux dynamiques par zone

📷 *\[Inclure ici un exemple de graphe : "Comparaison RMSE par modèle"]*

---

### 4. Carte interactive des zones

Une carte (utilisant `folium` ou `plotly.express`) indique la localisation des zones couvertes par la prévision :

* Clic sur une zone = accès direct à la courbe
* Code couleur selon le score (vert = bon, rouge = mauvais)

📷 *\[Insérer une image : carte des zones avec scores]*

---

## ⚙️ Paramètres configurables

Le fichier `config.yaml` permet de configurer dynamiquement :

* les chemins des données (`data_dir`)
* les colonnes utilisées (`features`)
* les modèles chargés (`models_path`)

---

## ▶️ Lancer l'application

### En local :

```bash
cd dashboard/
streamlit run app.py
```

### En ligne :

L’application peut être déployée sur [Streamlit Cloud](https://streamlit.io/cloud), ou hébergée sur un serveur institutionnel.

---

## 📈 Exemple d’utilisation

* **Contexte** : L’analyste veut vérifier si le modèle utilisé sur "Tenerife" est fiable.
* **Actions** :

  * Sélectionne "Tenerife"
  * Visualise la courbe réelle vs prévision
  * Consulte le score RMSE = 108.4 MW
* **Conclusion** : Le modèle est globalement cohérent, mais peut être affiné.

---

## 🚀 Perspectives

À l’avenir, le dashboard pourra intégrer :

* La possibilité de modifier les paramètres de modèle et re-prédire.
* Un système d’alerte en cas d’écart inhabituel.
* Des intégrations avec des APIs météo en temps réel.
