# AutoMeca Systems — Maintenance prédictive IoT

AutoMeca Systems conçoit des équipements de freinage pour l'industrie
automobile. Ce projet met en place une plateforme de données pour la
maintenance prédictive de son parc de machines de production : capteurs
IoT en atelier, modélisation et stockage des données, pipelines
d'ingestion, et déploiement d'un modèle prédictif de panne.

Le projet est organisé en dépôts indépendants, un par domaine :

| Dépôt | Contenu |
|---|---|
| [Bloc2-AutoMeca-Maintenance-Predictive-IoT-Architecture-Data](https://github.com/<user>/Bloc2-AutoMeca-Maintenance-Predictive-IoT-Architecture-Data) | Architecture de données : diagramme Edge/Cloud, modèle conceptuel, star schema, dictionnaire de données |
| [Bloc3-AutoMeca-Maintenance-Predictive-IoT-Pipeline-Data](https://github.com/<user>/Bloc3-AutoMeca-Maintenance-Predictive-IoT-Pipeline-Data) | Pipelines d'ingestion et de transformation des données (ELT) |
| [Bloc4-AutoMeca-Maintenance-Predictive-IoT-Solution-IA](https://github.com/<user>/Bloc4-AutoMeca-Maintenance-Predictive-IoT-Solution-IA) | Modèles de maintenance prédictive (entraînement) |
| [Bloc4-AutoMeca-Maintenance-Predictive-IoT-CICD](https://github.com/<user>/Bloc4-AutoMeca-Maintenance-Predictive-IoT-CICD) | Intégration et déploiement continus |

---

## Ce dépôt : Bloc4-AutoMeca-Maintenance-Predictive-IoT-Solution-IA

Deux modèles, entraînés sur le vrai dataset Kaggle *Microsoft Azure
Predictive Maintenance* (déjà utilisé aux Blocs 2 et 3) :

- **Détection d'anomalies** (Isolation Forest) — repère une dérive de
  comportement machine (vibration, pression, tension, rotation) avant
  qu'elle ne se traduise en panne.
- **Prédiction de durée de vie résiduelle — RUL** (forêt de survie) —
  estime le temps restant avant la prochaine panne d'une machine,
  utilisé pour calculer le niveau de criticité d'un ticket GMAO.

## Structure

```
Bloc4-AutoMeca-Maintenance-Predictive-IoT-Solution-IA/
├── data/                              # CSV Kaggle (non versionnes, voir Prerequis)
├── notebooks/
│   ├── 01_detection_anomalies.ipynb     # 5 etapes completes, code execute
│   └── 02_prediction_rul.ipynb           # 5 etapes completes, code execute (analyse de survie)
├── .gitignore
├── requirements.txt
└── README.md
```

> [!NOTE]
> `data/` n'est jamais versionné (voir `.gitignore`) — cohérent avec le reste du projet. **Prérequis** : télécharger le dataset [arnabbiswas1/microsoft-azure-predictive-maintenance](https://www.kaggle.com/datasets/arnabbiswas1/microsoft-azure-predictive-maintenance) (5 fichiers CSV) et le placer dans `data/`.

## Stack technique

- 🐍 **Python / pandas / numpy** — préparation des données
- 🌲 **scikit-learn** (Isolation Forest) — détection d'anomalies
- ⏱️ **scikit-survival** (Random Survival Forest) — prédiction RUL
- 📓 **Jupyter** — notebooks d'entraînement, exécutés de bout en bout

## Contenu

- **`notebooks/01_detection_anomalies.ipynb`** — Isolation Forest, entraîné sur du comportement normal (fenêtres glissantes 24h). AUC 0,906 (test) / 0,931 (optimisé) / 0,932 ± 0,003 (5 folds temporels). Explicabilité SHAP (`rotate`/`volt` glissants en tête).
- **`notebooks/02_prediction_rul.ipynb`** — Random Survival Forest, sur des épisodes de vie de composant (entre panne/maintenance, 68 % censurés — d'où le choix d'un modèle de survie plutôt qu'une régression classique). C-index 0,791 (test) / 0,795 (optimisé) / 0,784 ± 0,019 (5 folds temporels). Explicabilité SHAP (`age`, nombre d'erreurs récentes en tête).

Les deux notebooks sont exécutés de bout en bout (pas de cellule vide ni de sortie fabriquée) et documentent chacun un vrai bug trouvé et corrigé pendant leur construction (erreur de seuils incohérents pour le premier, erreur de tri `merge_asof` et de fenêtre temporelle pour le second).
