---
title: "Addendum PRD : application de course à pied personnelle"
created: 2026-11-20
updated: 2026-11-25
---

# Addendum au PRD

Ce document contient les détails techniques utiles à l'architecture et à l'UX, qui n'ont pas leur place dans le [PRD](prd.md). Les références entre crochets renvoient à la [recherche domaine du 25/11](../../research/domain-donnees-apple-watch-entrainement-course-2026-11-25/research.md).

Sources :
- le brief et son addendum, la revue party mode du 20/11 ;
- la recherche du 20/11 ([research-digest.md](research-digest.md), avec les URL) et l'extraction [extract-inputs.md](extract-inputs.md) ;
- la recherche domaine du 25/11, réconciliée avec le PRD dans [reconcile-research-2026-11-25.md](reconcile-research-2026-11-25.md).

## 1. Mécanismes intervals.icu

### 1.1 Confirmés
- **Authentification** : clé API personnelle en Basic auth, toujours côté serveur. L'identifiant d'athlète `0` désigne l'athlète authentifié.
- **Publication** : `POST /api/v1/athlete/{id}/events` ; `events/bulk?upsert=true` avec un `external_id` pour republier sans doublon.
- **Format des Séances** : syntaxe texte du champ `description`. Éviter `workout_doc` (non documenté ; des fichiers FIT vides ont été signalés fin 2026).
- **Lecture des Courses** : `GET /api/v1/activity/{id}?intervals=true`, puis `icu_intervals`. Chaque lap correspond à un Bloc ; ils sont associés un à un, dans l'ordre.
- **Quotas** : 5 000 requêtes par jour, 2 500 par tranche de 15 min et environ 10 par seconde ; en cas de dépassement, réponse 429 avec Retry-After.
- **Strava** : les activités issues de Strava ne sont pas exposées par l'API.

### 1.2 À vérifier (sources périmées ou forum ; liste de contrôle au §5)
- **Charge et Form** : charge allure ≈ h × IF² × 100 (IF : facteur d'intensité), HRSS (score de stress cardiaque) pour la FC [39][40] ; la Form du jour inclut la séance du jour [41] ; zones de Form autour de −30 [64]. Ces indicateurs sont descriptifs, jamais des garde-fous.
- **Conformité native** : charge réalisée / prévue, vert entre 80 et 120 % ; appariement refusé si la charge réalisée est sous 50 % de la prévue [97][98]. À comparer au rattachement de FR-11.
- **Seuil et CS** : intervals.icu calcule la CS (vitesse critique) et la D′ (réserve anaérobie) sur la courbe d'allures [107], mais ne mettrait pas à jour automatiquement l'allure seuil en course (forum, sept. 2025 [108]). Ce point conditionne FR-23 et la question Q10 (PRD §12) ; le calcul propre à l'application est décrit au §2.2.
- **GAP (allure ajustée à la pente) et température** : vérifier leur présence dans les streams de l'activité (FR-13).

## 2. Pipelines et paramètres d'entraînement

### 2.1 Pipelines (« le LLM raconte, le code compte »)
- **Plan** : le LLM produit un JSON structuré (semaines, Séances, Blocs, cibles). Un validateur vérifie les garde-fous de FR-5, puis une fonction déterministe traduit ce JSON en syntaxe `description`. Le LLM n'écrit jamais cette syntaxe, qui comporte des pièges : `m` y signifie minutes, les mètres s'écrivent `mtr`, et les répétitions `4x` ne s'imbriquent pas.
- **Feedback** : les calculs sont déterministes (allure par Bloc, écart à la cible, Conformité, dérive cardiaque) et sont transmis au LLM comme des faits établis. Un validateur rejette toute réponse contenant un nombre absent des entrées et déclenche une régénération.
- **Imprévus** : un arbre de décision codé ; le LLM ne sert qu'à régénérer un Plan.

### 2.2 Paramètres d'entraînement
Chaque entrée suit le même schéma : règle ou valeur, sources, puis statut.

- **Chrono irréaliste (FR-4)** : équivalence de type VDOT calculée sur les records des 6 derniers mois. Les équivalences de type Riegel sont trop optimistes d'au moins 10 min au marathon pour la moitié des amateurs ; le kilométrage hebdomadaire améliore la prédiction [111], la CS sur 16 semaines aussi [102][105]. *Statut* : seuil d'écart à fixer en architecture ; correction par kilométrage différée.
- **Garde-fou de charge (FR-5)** : le risque de blessure augmente quand une sortie dépasse de plus de 10 % la plus longue des 30 derniers jours (Frandsen, BJSM 2025 [65]). L'ACWR (ratio de charge aiguë / chronique) est contesté [42][65]. La règle hebdomadaire des 10 % n'est qu'un garde-fou secondaire (PRD §15.1). *Statut* : arbitré.
- **Choix de la Méthode (FR-5)** : règle déterministe, Daniels (pyramidale) par défaut, polarisé au-delà d'un certain volume hebdomadaire. Aucune étude ne fonde un choix de Méthode sur le profil [46][47] ; la répartition réalisée dérive vers le pyramidal quelle que soit la prescription [46]. MAF et méthode norvégienne restent exclues [53][54]. *Statut* : volume de bascule à fixer en architecture.
- **Bandes de répartition des Méthodes (FR-5)** :
  - *Daniels* (allures E = endurance, T = seuil, I = intervalles VO2max, R = répétitions) : allure E pour la majorité du volume ; allure T au plus 10 %, I au plus 8 % et R au plus 5 % du volume hebdomadaire.
  - *Polarisé* : au moins 75 % du temps en Z1, au plus 10 % en Z3 et au moins 10 % en Z4-Z5. L'indice de polarisation suit la formule de Treff (2019) [36][43].
  - *80/20 (hors v1)* : au moins 75 % du temps en Z1-Z2 (la cible est de 80 %).
  - *Statut* : bandes à valider en architecture ; formule de Treff à vérifier avant usage.
- **Affûtage (FR-5)** : 2 semaines (3 pour un marathon), volume réduit de 41 à 60 %, intensité et fréquence maintenues [44][56]. *Statut* : paramètre par distance, sans branche de code dédiée.
- **Conformité et GAP (FR-13)** : la GAP suit le polynôme de Minetti ; le modèle de Strava n'est pas publié [104]. *Statut* : présence dans les streams à vérifier (§1.2).
- **Indicateurs de tendance (FR-14)** : FC à allure comparable [70] et découplage Pa:HR sur les sorties longues [29][101][102]. La récupération de FC (HRRc) n'est calculée que si la FC a dépassé la FC seuil (LTHR), et ne fonde jamais seule un verdict. *Statut* : le facteur d'efficience (EF) n'est pas retenu en v1.
- **Seuil de surmenage (FR-16)** : Ligne de base ≥ 28 j [114] ; l'écart entre la moyenne glissante de 7 j et la Ligne de base est comparé à un seuil de 0,5 à 1 × le coefficient de variation individuel [70] ; la FC au repos varie d'environ 10 % d'un jour à l'autre, d'où la moyenne [70]. *Statut* : calibrage par rétro-analyse de l'historique avant la mise en usage (PRD §11).
- **Seuil d'allure (FR-23)** : vitesse critique par régression d = CS·t + D′ sur les meilleurs efforts (par exemple 400, 800 et 5 000 m) des 16 dernières semaines, en GAP [23][105]. Une correction des efforts non maximaux relèverait la CS d'environ 5 % [105] : elle donnerait des allures plus rapides et n'est donc **pas** appliquée, par prudence. FC max : la valeur observée prime sur la formule de Tanaka (208 − 0,7 × âge) [26] ; ne pas mélanger les protocoles de FC seuil (LTHR) de Friel [35][109]. *Statut* : comportement d'intervals.icu au §1.2.
- **Mode Reprise (FR-24)** : facteurs VDOT de Daniels après arrêt (≈ × 0,973 à 14 j, × 0,931 à 28 j, jusqu'à × 0,80 au-delà de 72 j) [95]. Ils s'appliquent aux allures seulement, jamais au volume ni au plafond par sortie. *Statut* : paramètres ajustables.

## 3. Données, hébergement et transport vers la montre

### 3.1 Sources de données et fiabilité (NFR-10)
- **Apple Watch** :
  - fiables : allure, FC en effort stable, FC au repos (−0,08 bpm face à une ceinture [72]) et durée de sommeil [6][21] ;
  - en tendance seulement : VO2max (−6,3 ml/kg/min, 13 % d'erreur sur Series 10 [10]) ; VFC stockée en SDNN, pas en RMSSD (deux modes de calcul de la variabilité), sous-estimée de 8,3 ms au repos, sans validation nocturne [22][28][72][73] ;
  - sans validation publiée : puissance et dynamique de foulée [14] ;
  - sommeil : stades à kappa 0,68 [6][21] ; score non comparable entre versions de watchOS (26.2) [7] ; Vitals (VFC) sur Series 12 seulement [11].
- **HealthFit** :
  - importe les Séances planifiées d'intervals.icu depuis octobre 2025 ; depuis mars 2026, cette fonction requiert l'achat intégré « Fanatical Supporter » (9,99 $), que Badr possède ;
  - il faut activer la synchronisation automatique ; le contenu du FIT exporté (fréquence de la FC, laps, puissance) n'est pas documenté [15][19] ; la VFC envoyée serait une moyenne journalière [15] ;
  - alternative pour le wellness : IntervalsWellnessSync [20] (source faible).

### 3.2 Hébergement et transport
- Développement et exécution d'abord en local sur le Mac, puis auto-hébergement. La PWA est servie derrière Tailscale : rien n'est exposé sur Internet et aucune authentification n'est à développer en v1.
- Le transport vers la montre et la remontée bloc par bloc ont été vérifiés en conditions réelles (revue party mode) : ce ne sont plus des risques à prototyper.
- Plan B pour le transport : Watchletic.
- Pistes écartées : WorkoutKit/HealthKit (compte développeur payant, sideload limité à 7 jours) et Intervals Companion (cibles d'allure douteuses).

## 4. Portes de qualité (NFR-8)
- Aucune story n'est terminée sans ses tests.
- Aucun appel à intervals.icu sans test de contrat ; un test aller-retour envoie le Plan, le relit et le compare au JSON source.
- Aucun chiffre du Feedback sans un test qui le recalcule.

## 5. Vérifications en attente
- **Une cible par étape sur la montre** : WorkoutKit n'accepte probablement qu'une seule cible par étape (allure ou FC). La règle de FR-9 (une cible par Bloc) s'applique quoi qu'il arrive ; si deux cibles s'avéraient possibles, on pourrait l'assouplir en v1.1.
- **Contenu du FIT HealthFit** (PRD §12 Q9) : exporter une vraie Course et lire le fichier (FC, laps, puissance) avant les stories Feedback.
- **Comportements d'intervals.icu** (PRD §12 Q4, Q10) : chaque point du §1.2, plus la source de charge d'une Course (allure, FC ou puissance).
- **Durée de sommeil** (PRD §12 Q11) : vérifier qu'elle arrive dans le wellness d'intervals.icu via HealthFit.
- **Hors liste** : la valeur de VFC envoyée par HealthFit n'est pas à vérifier tant que la VFC reste exclue (NFR-10).

## 6. Protocole du test de qualité (SM-5)
Le plan de référence sera la préparation marathon (avril 2027, objectif sous 3 h 30) construite avec l'ancienne méthode ; il sera comparé au plan produit par l'application. Les deux plans sont anonymisés et présentés dans un ordre aléatoire avant la comparaison.

## 7. Paysage
intervals.icu Supporter (4 $/mois) propose déjà un connecteur MCP pour brancher son propre LLM, et des coachs LLM tiers existent autour d'intervals.icu. Ces offres ne menacent pas un usage personnel, mais c'est à garder en tête si l'application est un jour partagée.
