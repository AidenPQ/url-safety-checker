# Cahier des charges — URL Safety Checker
## 1. Objectifs

Une application qui permet à son utilisateur:
	- d'évaluer le niveau de dangerosité d'une URL en passant par score de confiance entre 0 et 1 et un label (safe, suspicious, malicious)
	- d'expliquer ce label et ce score par des features claires (longueur de l'URL, présence de certains caractères...) et l'influence de chacun sur la prédiction


## 2. Périmètre fonctionnel
### 2.1 Fonctionnalités incluses

| Fonctionnalité | Détail |
|---|---|
| Saisie d'URL | Champ texte + bouton + touche Entrée |
| Résultat d'analyse | Label + score de confiance + code couleur |
| Visualisation features | Graphique en barres horizontal (nom + importance relative) |
| Historique | 5 entrées max, affiché depuis Redis |
| Persistance long terme | SQLite — toutes les URLs analysées + label + score |



### 2.2 Fonctionnalités exclues

	- Pas d'analyse en temps réel via API externe
	- Pas d'entrainement de modèles ML en temps réel
	- Pas d'authentification de l'utilisateur


### 2.3 Évolutions futures envisagées

	- Déploiement Cloud
	- Identifications du niveau de dangerosité d'autres éléments (numéro de téléphone, mail de phishing, etc.)


## 3. Contraintes techniques


Le tableau des technologies est le suivant :

| Technologie | Rôle |
|---|---|
| C# / .NET 10 | Langage unique |
| ML.NET | Entraînement et inférence du modèle |
| ASP.NET Core | API REST |
| Blazor WebAssembly | Frontend |
| Redis | Cache + historique session |
| SQLite + EF Core | Persistance long terme |
| Docker + Docker Compose | Conteneurisation |


**Récap complet des contraintes techniques :**

| Contrainte | Détail |
|---|---|
| Exécution | 100% local, aucun déploiement cloud |
| Langage unique | C# / .NET 10 |
| Communication | Blazor → API REST → ML.NET, jamais en direct |
| Conteneurisation | Tout dockerisé via docker-compose |
| Pas d'APIs externes | Pas de VirusTotal, pas de services tiers |
| Pas d'authentification | Hors périmètre |


## 4. Critères de succès


| Critère | Mesure |
|---|---|
| Qualité ML classification | Accuracy, Precision, Recall — seuils à définir phase ML |
| Qualité ML score | Brier Score — seuil à définir phase ML |
| Performance cache hit | < 0.5s |
| Performance cache miss | < 5s |
| UX | Code couleur lisible d'un seul regard |
| Stack moderne | .NET 10, Docker, ML.NET |
| Bonnes pratiques | Git conventionnel, architecture multi-projets, CDC |
| ML en contexte réel | Modèle ML au cœur du produit via ML.NET |
