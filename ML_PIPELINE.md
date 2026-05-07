# Pipeline Machine Learning - URL Safety Checker
## 1. Dataset
### 1.1. Origine du dataset

Lien du dataset sur kaggle: https://www.kaggle.com/datasets/moutasmtamimi/malicious-url-detection-dataset-enhanced-2026/data

### 1.2. Description du dataset

Cet ensemble de données est une vaste collection d'URL riche en informations, conçue pour la recherche et le développement dans le domaine de la détection des URL malveillantes. Il contient au total 651 000 échantillons, chacun étant enrichi d'attributs détaillés d'ordre lexical, structurel, statistique et liés au phishing. L'objectif de cet ensemble de données est de faciliter le développement de modèles d'apprentissage automatique capables d'identifier les URL malveillantes avant qu'elles ne provoquent des incidents de sécurité tels que des attaques de phishing, des vols de données et des infections par des logiciels malveillants.


### 1.3. Contenu du Dataset
On peut retrouver 651 000 URLS différentes avec :
	- 428 103 URL inoffensives
	- 96 457 URL piratées
	- 93 920 phishing URLs
	- 32 520 malware URLs

### 1.4. Features du Dataset
On y trouve 65 features qu'on peut diviser en catégorie:
	- Features Générales: url, type, label, web_is_live, Date_inspection
	- Features liées à l'état du site sur internet web_ (état du site): web_security_score, web_forms_count, web_password_fields, web_has_login, web_ssl_valid
	- Features liées à l'analyse brutes de l'url (caractères): url_len, @, ?, -, =, ., #, %, +, $, !, *, ,, //, digits, letters, domain
	- Features URL générales: abnormal_url, https, Shortining_Service, having_ip_address
	- Features defac_ (defacement): defac_has_hacked_terms, defac_has_suspicious_ext, defac_path_depth, defac_is_deep_path, defac_path_underscores, defac_is_gov_edu, defac_has_index_php, defac_has_option_param
	- Features phish_ (phishing): phish_has_brand, phish_brand_in_subdomain, phish_brand_in_path, phish_hyphen_count, phish_digit_count, phish_long_domain, phish_many_subdomains, phish_suspicious_tld, phish_keyword_count, phish_has_redirect, phish_param_count, phish_encoded_chars
	-  enh_ (enrichies): enh_urgency_count, enh_security_count, enh_brand_count, enh_brand_hijack, enh_subdomain_count, enh_long_path, enh_many_params, enh_suspicious_tld
	- adv_ (avancées): adv_domain_ngram_entropy, adv_path_entropy, adv_consonant_ratio, adv_vowel_ratio, adv_digit_ratio, adv_subdomain_count, adv_avg_subdomain_len, adv_token_count, adv_avg_token_length


### 1.5. Pré sélection des features

**Récap global de notre pré sélection de features avant feature importance:**

| Groupe | Décision |
|---|---|
| Générales | ✅ |
| URL brutes | ✅ |
| URL générales | ✅ |
| `web_` | 🔄 Feature importance |
| `defac_` | ✅ sauf `defac_is_deep_path` |
| `phish_` | ✅ sauf les 3 brand |
| `enh_` | ✅ |
| `adv_` | ✅ |

## 2. Modèle de Machine learning
### 2.1. Stratégie de sélection des modèles

Pour un projet portfolio, entraîner **plusieurs modèles** et comparer leurs performances démontre une vraie maturité ML.

1. **LightGBM** — le choix principal: Standard en industrie, présent sur ML .NET et standard en industrie, souvent **plus rapide** sur des datasets larges comme le nôtre (651k URLs)
2. **FastForest** — Random Forest comme baseline solide
3. **SdcaMaximumEntropy** — modèle linéaire comme baseline simple pour montrer l'intérêt d'un modèle plus complexe tout en proposant un modèle plus simple pour de l'explicabilité

### 2.2. Stratégie ML Complète


| Élément | Décision |
|---|---|
| Dataset | dataset_with_all_features v2 (651k URLs) |
| Classes | 4 — benign, defacement, phishing, malware |
| Features | ~50 features sélectionnées |
| Algorithmes | LightGBM (principal), FastForest, SdcaMaximumEntropy (baselines) |
| Métriques | Precision, Recall, F1, AUC-PR, Brier Score |
| Class imbalance | SMOTE |

### 2.3. Récap du pipeline complet

```
[URL brute]
     ↓
[Feature Extraction] ← ton algorithme C#
     ↓
[Vecteur de ~50 features]
     ↓
[Suppression colonnes inutiles]
     ↓
[Gestion valeurs manquantes]
     ↓
[MapValueToKey sur le label]
     ↓
[LightGBM / FastForest / SdcaMaximumEntropy]
     ↓
[Modèle sérialisé .zip]
```

## 3. Feature Engineering et Validation
### 3.1 Stratégie de validation des features
Après le premier entraînement, on réalisera une feature importance pour déterminer quelles sont les features qui ont le plus d'impact sur la classification pour pouvoir diminuer le nombre de features tout en gardant la fiabilité de l'algorithme.

