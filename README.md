# Projet 13 – Créez votre portfolio de professionnel de la data

## 🎯 Objectif

Améliorer, grâce à l'IA, un pipeline d'analyse de données existant en le dotant d'une **chaîne qualité hybride** : bloquer automatiquement les erreurs connues en amont, et détecter les anomalies inconnues en aval, de façon comparative, justifiée et documentée.

## 📌 Contexte

Projet réalisé dans le cadre de la formation **Data Analyst OpenClassrooms**.

**BottleNeck**, marchand de vin, gère son catalogue avec des outils artisanaux : un ERP, un export du site WooCommerce et une table de liaison qui ne communiquent pas entre eux. Les données comportent des erreurs de saisie et aucun garde-fou automatique n'empêche ces erreurs de polluer le reporting du CODIR.

Ce projet reprend un pipeline d'agrégation validé et l'améliore par l'IA, utilisée comme un **levier testé de manière critique** (essais, variantes, décisions tracés).

## ❓ Problématique

Comment garantir, à chaque export mensuel, que les données de prix, de coût et de marge sont **d'abord conformes aux règles connues**, puis **exemptes d'incohérences imprévues** — y compris celles invisibles à un contrôle sur une seule variable — avant de consolider le reporting ?

## 📂 Données utilisées

Extraction au 31 octobre (ventes du 1ᵉʳ au 31 octobre) :

- **ERP** — 825 produits : référence, prix, prix d'achat, stock, état du stock
- **Web (WooCommerce)** — SKU, quantités vendues, description des produits
- **Table de liaison** — correspondances entre références ERP et base WordPress

### Variables métier dérivées

- Prix HT, **taux de marge**, **ratio prix / prix d'achat**, chiffre d'affaires

## 🛠️ Méthodologie

### 1. Nettoyage & consolidation *(repris du P6)*

- Correction des prix et stocks négatifs, exclusion des pièces jointes WordPress
- Dédoublonnage de la table de liaison
- Jointure ERP ↔ Web via la table de liaison

### 2. Contrats de données automatisés — *bloquer le connu*

- Validation par **Pandera** : clé primaire unique, prix et stocks positifs, types corrects
- Le pipeline **s'arrête** si une règle est violée (au lieu de propager l'erreur)

### 3. Détection d'anomalies multivariée — *découvrir l'inconnu*

- **Isolation Forest** (détecteur principal) + **Local Outlier Factor** (second regard)
- Standardisation des variables, taux d'anomalies (`contamination`) calibré

### 4. Interprétabilité

- **SHAP** : explication globale et produit par produit de chaque alerte

## 🔎 Veille métier & technologique

Solutions comparées selon des critères explicites (qualité, robustesse, reproductibilité, interprétabilité, maintenabilité) :

| Piste | Options étudiées | Retenu |
| --- | --- | --- |
| Détection d'anomalies | IQR (baseline) · **Isolation Forest** · **LOF** · One-Class SVM | Isolation Forest + LOF |
| Contrats de données | **Pandera** · Great Expectations · assertions | Pandera |
| Interprétabilité | Importances brutes · **SHAP** | SHAP |

**Source récente clé** : *Bouman, Bukhsh & Heskes (2024), « Unsupervised Anomaly Detection Algorithms on Real-world Data », Journal of Machine Learning Research* — benchmark de 33 algorithmes sur 52 jeux tabulaires réels. Il confirme la performance de la famille Isolation Forest et, via sa distinction **« local / global »**, justifie de combiner Isolation Forest (global) et LOF (local). Piste d'évolution : l'**Extended Isolation Forest**.

## 📊 Résultats

- Les **5 erreurs structurelles** (3 prix négatifs, 2 stocks négatifs) corrigées à la main au P6 sont désormais **interceptées automatiquement** par les contrats.
- La détection multivariée capte **4/4** des erreurs de marge certaines, contre **1/4** pour la méthode univariée d'origine.
- Un produit vendu **12,65 €** pour un prix d'achat de **77,48 €** (marge de −635 %), invisible à un contrôle sur le seul prix, est correctement signalé.
- Livraison d'une **liste de 41 produits à vérifier**, triée par marge, prête pour le service achats.

## 🧰 Technologies utilisées

- Python
- Pandas · NumPy
- Scikit-Learn *(Isolation Forest, LOF)*
- Pandera *(contrats de données)*
- SHAP *(interprétabilité)*
- Matplotlib · Seaborn
- Jupyter Notebook

## 📁 Structure du projet

```
data/
│
├── erp.xlsx
├── web.xlsx
└── liaison.xlsx

Zaidi_Mohamed_1_notebook_ameliore_072025.ipynb     # Notebook : pipeline + chaîne qualité hybride
Zaidi_Mohamed_2_documentation_demarche_072025.docx # Veille, cahier des charges, tests, décisions
alertes_anomalies.xlsx                             # Livrable métier : 41 produits à vérifier

README.md
```

## ✅ Compétences développées

- Qualité des données & contrats de données (data contracts)
- Détection d'anomalies non supervisée (Isolation Forest, LOF)
- Machine Learning interprétable (SHAP)
- Veille métier et technologique comparative
- Reproductibilité & documentation d'une démarche
- Usage critique et tracé de l'IA

## 💡 Valeur ajoutée métier

Cette solution sécurise le reporting en empêchant les erreurs de prix de le polluer, fournit au service achats une **liste d'alertes actionnable** et expliquée, et livre une base propre et **reproductible** pour le futur projet de data-visualisation.

## 🚀 Limites & prochaines étapes

- Détection non supervisée : les alertes sont validées par le métier (pas de suppression automatique)
- Volume modeste (825 lignes) : le ML apporte surtout de la robustesse méthodologique
- **Prochaines étapes** : tester l'Extended Isolation Forest, recalibrer le seuil sur un historique d'erreurs confirmées, ajouter un monitoring de dérive (Evidently)

## 👨‍💻 Auteur

**Mohamed Zaidi**
