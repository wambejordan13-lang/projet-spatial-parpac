#  Analyse Spatiale du Programme PARPAC sur les 58 Départements du Cameroun

**Projet académique  ISSEA · 2025**
**Auteur : WAMBE Jordan Valère**

> **Note de confidentialité** : Ce dépôt ne contient pas le code source ni les données brutes. Il présente l'architecture analytique, les méthodes géostatistiques appliquées, les cartes produites (schématisées) et les conclusions territoriales.

---

## Contexte et Problématique

Le **Programme d'Appui à la Résilience des Populations Agricoles du Cameroun (PARPAC)**, lancé par le Ministère de l'Agriculture et du Développement Rural (MINADER) en **décembre 2023**, couvre l'ensemble des **58 départements du Cameroun**. Il vise à renforcer la résilience des populations agricoles face aux chocs climatiques et économiques, à travers des investissements ciblés en intrants, infrastructures et formation.

**Problématique analytique** : comment visualiser et quantifier les **disparités spatiales** entre départements dans les indicateurs clés du programme ? Quels départements présentent des profils similaires ? Où les interventions sont-elles les plus urgentes ?

**Objectif** : produire une analyse spatiale complète permettant de cartographier les intensités des indicateurs par département, d'identifier des groupes territoriaux homogènes et de visualiser les convergences et divergences entre zones.

---

## Pipeline d'Analyse Spatiale

```
┌─────────────────────────────────────────────────────────────────┐
│                  PIPELINE STATISTIQUE SPATIALE                  │
└─────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────┐
  │     DONNÉES D'ENTRÉE         │
  │  Indicateurs PARPAC par      │
  │  département (58 unités)     │
  │  + Fonds de carte Cameroun   │
  └──────────────┬───────────────┘
                 │
                 ▼
  ┌──────────────────────────────┐
  │  PRÉTRAITEMENT & JOINTURE    │
  │  Nettoyage des données       │
  │  Jointure attributaire       │
  │  (données × géométries)      │
  │  Vérification des NA         │
  └──────────────┬───────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
┌──────────────┐   ┌──────────────────────────┐
│   ANALYSE    │   │   ANALYSE SPATIALE        │
│  STATISTIQUE │   │   Autocorrélation spatiale│
│  CLASSIQUE   │   │   (Moran's I global +     │
│  (stats des- │   │    local LISA)            │
│  criptives   │   │   Interpolation spatiale  │
│  par dép.)   │   │   Cartes de densité       │
└──────┬───────┘   └──────────────┬────────────┘
       │                          │
       └──────────┬───────────────┘
                  │
                  ▼
  ┌──────────────────────────────┐
  │  CLASSIFICATION SPATIALE     │
  │  Clustering des départements │
  │  par profil d'indicateurs    │
  │  (K-means spatial / CAH)     │
  └──────────────┬───────────────┘
                 │
                 ▼
  ┌──────────────────────────────┐
  │  CARTOGRAPHIE THÉMATIQUE     │
  │  Cartes choroplèthes         │
  │  Cartes de points de         │
  │  convergence / divergence    │
  │  Cartes des clusters         │
  └──────────────────────────────┘
```

---

## Spécifications Méthodologiques

### 1. Statistiques Descriptives Spatiales

Pour chaque indicateur $X$ mesuré sur les $n = 58$ départements :

$$\bar{X} = \frac{1}{n}\sum_{i=1}^{n} X_i \qquad \sigma_X = \sqrt{\frac{1}{n}\sum_{i=1}^{n}(X_i - \bar{X})^2}$$

**Coefficient de variation** pour mesurer la dispersion relative entre départements :

$$CV = \frac{\sigma_X}{\bar{X}} \times 100$$

Un $CV$ élevé signale une forte hétérogénéité territoriale justifiant une intervention ciblée.

---

### 2. Analyse de l'Autocorrélation Spatiale

#### I de Moran (global)

L'indice de Moran mesure si les départements proches tendent à avoir des valeurs similaires (autocorrélation positive) ou opposées (autocorrélation négative) :

$$I = \frac{n}{\sum_{i}\sum_{j} w_{ij}} \cdot \frac{\sum_{i}\sum_{j} w_{ij}(X_i - \bar{X})(X_j - \bar{X})}{\sum_{i}(X_i - \bar{X})^2}$$

où $w_{ij}$ est la matrice de poids spatial (contiguïté de premier ordre : $w_{ij} = 1$ si les départements $i$ et $j$ sont voisins, $0$ sinon).

- $I > 0$ : les indicateurs forts se concentrent spatialement (clusters)
- $I < 0$ : les valeurs alternent (damier)
- $I \approx 0$ : distribution aléatoire

**Test de significativité** par permutation (999 permutations) :

$$
p = \frac{\left| \left( I_{\text{perm}} \geq I_{\text{obs}} \right) \right|}{999}
$$


---

#### LISA — Indicateurs Locaux d'Association Spatiale

Pour identifier les clusters locaux (hot spots / cold spots) :

$$I_i = \frac{(X_i - \bar{X})}{\sigma_X^2} \sum_{j} w_{ij}(X_j - \bar{X})$$

Classification des départements en 4 quadrants du diagramme de Moran :

| Quadrant | Interprétation |
|----------|----------------|
| HH (High-High) | Département avec valeur élevée entouré de voisins élevés — cluster positif |
| LL (Low-Low) | Département avec valeur faible entouré de voisins faibles — cluster négatif |
| HL (High-Low) | Département avec valeur élevée entouré de voisins faibles — anomalie positive |
| LH (Low-High) | Département avec valeur faible entouré de voisins élevés — anomalie négative |

---

### 3. Cartographie Thématique

#### Cartes choroplèthes

Les indicateurs PARPAC sont représentés par des **cartes choroplèthes** avec discrétisation par la méthode de **Jenks (Natural Breaks)** qui minimise la variance intra-classe :

$$\min \sum_{k=1}^{K} \sum_{i \in C_k} (X_i - \bar{X}_{C_k})^2$$

#### Visualisation des convergences et divergences

Pour chaque paire d'indicateurs $(X, Y)$, un indice de divergence locale est calculé :

$$D_i = |r_{X_i} - r_{Y_i}|$$

où $r_{X_i}$ est le rang du département $i$ pour l'indicateur $X$.

Les départements avec $D_i$ élevé sont ceux où les deux indicateurs divergent fortement, signalant des situations complexes nécessitant une analyse approfondie.

---

### 4. Classification des Départements par Profil

#### Classification Ascendante Hiérarchique (CAH)

Distance euclidienne entre profils d'indicateurs standardisés (score Z) :

$$d(i,j) = \sqrt{\sum_{k=1}^{p} \left(\frac{X_{ik} - \bar{X}_k}{\sigma_k} - \frac{X_{jk} - \bar{X}_k}{\sigma_k}\right)^2}$$

Agrégation par le critère de **Ward** (minimisation de l'inertie intra-classe) :

$$\Delta(A, B) = \frac{n_A \cdot n_B}{n_A + n_B} \|m_A - m_B\|^2$$

Le dendrogramme produit a permis de déterminer le nombre optimal de groupes par coupure à la plus grande différence de hauteur.

#### Validation par K-means

Les groupes identifiés par CAH ont été validés par K-means avec le même nombre de clusters, en comparant les affectations par l'**indice de Rand ajusté** :

$$\text{ARI} = \frac{\text{RI} - \mathbb{E}[\text{RI}]}{\max(\text{RI}) - \mathbb{E}[\text{RI}]}$$

---

## Cartes Produites

```
┌──────────────────────────────────────────────────────┐
│  CARTE 1 : Intensité de l'indicateur X               │
│  par département (choroplèthe — 5 classes Jenks)     │
│                                                      │
│  [Extrême-Nord]  ██████ Classe 5 (très élevé)       │
│  [Nord]          ████░░ Classe 4                     │
│  [Adamaoua]      ███░░░ Classe 3                     │
│  [Centre]        ██░░░░ Classe 2                     │
│  [Sud / Est]     █░░░░░ Classe 1 (très faible)       │
│                                                      │
│  I de Moran = 0.XX  (p < 0.05)                      │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  CARTE 2 : Clusters LISA (hot spots / cold spots)    │
│                                                      │
│  ■ HH (cluster positif)                              │
│  □ LL (cluster négatif)                              │
│  ▨ HL (anomalie positive)                            │
│  ▧ LH (anomalie négative)                            │
│  ░ Non significatif (p > 0.05)                       │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  CARTE 3 : Groupes de départements                   │
│            (profils similaires — CAH + K-means)      │
│                                                      │
│  Groupe A : départements à fort potentiel agricole   │
│             mais faible résilience climatique        │
│  Groupe B : départements à résilience élevée         │
│             mais faible accès aux intrants           │
│  Groupe C : départements en situation équilibrée     │
│  ...                                                 │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  CARTE 4 : Points de convergence et de divergence    │
│            entre indicateurs X et Y                  │
│                                                      │
│  ● Convergence forte (départements où X et Y        │
│    évoluent dans le même sens)                       │
│  ○ Divergence forte (départements où X et Y         │
│    s'opposent — situations à analyser en priorité)  │
└──────────────────────────────────────────────────────┘
```

---

## Résultats Clés

| Indicateur | Résultat |
|------------|---------|
| Nombre de clusters de départements identifiés | *confidentiel* |
| I de Moran global (indicateur principal) | *confidentiel* |
| Nombre de hot spots significatifs (p < 0.05) | *confidentiel* |
| Nombre de cold spots significatifs (p < 0.05) | *confidentiel* |
| ARI entre CAH et K-means | *confidentiel* |

> Les valeurs exactes et les cartes finales sont disponibles sur demande dans le cadre d'un entretien professionnel.

---

## Conclusion et Apports

Cette analyse a permis de produire une lecture géographique structurée et quantifiée du programme PARPAC sur l'ensemble du territoire camerounais. Les principaux apports sont :

- **Pour le MINADER** : identifier les départements prioritaires d'intervention selon leur profil de vulnérabilité, au-delà d'une lecture administrative classique.
- **Pour les bailleurs de fonds** : fournir des cartes thématiques exploitables dans les rapports de suivi et les présentations aux partenaires.
- **Pour la recherche** : démontrer la valeur ajoutée de l'autocorrélation spatiale (Moran's I + LISA) pour détecter des clusters territoriaux invisibles à l'analyse statistique classique.

**Perspective** : intégrer des données satellitaires (NDVI, précipitations, températures) pour enrichir l'analyse avec des indicateurs climatiques en temps quasi-réel et renforcer la dimension de résilience agricole.

---

*Projet réalisé dans le cadre du cours de Statistique Spatiale  ISSEA · 2025*
*Auteur : WAMBE Jordan Valère  wambejordan13@gmail.com*

