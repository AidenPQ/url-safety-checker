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


## 5. Flux de données

### Chargement initial
1. L'utilisateur arrive sur la page de Web.
2. Web demande à l'Api un historique des 5 dernières URL qui ont été entrées sur l'application.
3. L'Api récupère depuis Redis les 5 dernières URL stockés en mémoire et leurs résultats
4. Web affiche le titre de la page, une boite de dialogue pour entrer l'URL, un bouton de validation et l'historique des 5 dernières URLs entrées et leurs résultats.

### Soumission d'une URL
1. L'utilisateur saisit une URL dans la boite de dialogue de Web et valide à l'aide du bouton où de la touche Entrée.
2. Web envoie à l'Api l'URL saisi.
3. L'Api vérifie dans Redis si l'URL saisi est présente et avait déjà été analysé.
   - Cache hit : l'URL est présente, et l'Api récupère les résultats de l'analyse de cet URL.
   - Cache miss : l'Api fournit cette URL au modèle ML et retourne les résultats de celui-ci.
     a. Le modèle ML analyse l'URL fourni et donne en sortie un label de dangerosité et un score de confiance et la contribution de chaque features pour l'explicabilité.
     b. L'Api stocke dans SQLite l'URL, le label, le score, FeatureContributions, et date d'analyse.
     c. L'Api stocke dans Redis l'URL, le label, le score, FeatureContributions
4. L'Api retourne à Web le label, le score, FeatureContributions
5. Web affiche le label et le score dans un code couleur défini ainsi qu'un graphique à barres horizontales pour présenter les FeatureContributions.

