---
title: "Addendum PRD : application de course à pied personnelle"
created: 2026-11-20
updated: 2026-11-20
---

# Addendum au PRD

Ce document contient les détails techniques utiles à l'architecture et à l'UX, qui n'ont pas leur place dans le [PRD](prd.md). Sources : le brief et son addendum, la revue party mode du 20/11, la recherche du 20/11 ([research-digest.md](research-digest.md), avec les URL) et l'extraction [extract-inputs.md](extract-inputs.md).

## 1. Mécanismes intervals.icu
- **Authentification** : clé API personnelle en Basic auth, toujours côté serveur. L'identifiant d'athlète `0` désigne l'athlète authentifié.
- **Publication** : `POST /api/v1/athlete/{id}/events` ; `events/bulk?upsert=true` avec un `external_id` pour republier sans doublon (confirmé par la recherche).
- **Format des Séances** : syntaxe texte du champ `description`. Éviter `workout_doc` (non documenté ; des fichiers FIT vides ont été signalés fin 2026).
- **Lecture des Courses** : `GET /api/v1/activity/{id}?intervals=true`, puis `icu_intervals`. Un lap par Bloc, associés dans l'ordre, un à un.
- **Quotas** (confirmés par la recherche) : 5 000 requêtes par jour, 2 500 par 15 min et environ 10 par seconde ; en cas de dépassement, réponse 429 avec Retry-After.
- Les activités issues de Strava ne sont pas exposées par l'API.

## 2. Pipelines et garde-fous (« le LLM raconte, le code compte »)
- **Plan** : le LLM produit un JSON structuré (semaines, Séances, Blocs, cibles). Un validateur vérifie les garde-fous de FR-5, puis une fonction déterministe le traduit vers la syntaxe `description`. Le LLM n'écrit jamais cette syntaxe, qui comporte des pièges : `m` y signifie minutes, les mètres s'écrivent `mtr`, et les répétitions `4x` ne s'imbriquent pas.
- **Feedback** : les calculs sont déterministes (allure par Bloc, écart à la cible, Conformité, dérive cardiaque) et sont passés au LLM comme des faits. Un validateur rejette tout nombre absent des entrées et déclenche une régénération.
- **Imprévus** : un arbre de décision codé ; le LLM ne sert qu'à régénérer un Plan.
- **Garde-fou de charge** : selon Frandsen (BJSM 2025), le risque de blessure augmente quand une séance dépasse de plus de 10 % la plus longue sortie des 30 derniers jours. L'ACWR (ratio de charge aiguë/chronique) est contesté. Le digest propose 10 garde-fous codés hors du LLM.
- **Bandes de répartition des Méthodes** (garde-fou de FR-5), à valider en architecture :
  - *80/20* : au moins 75 % du temps en Z1-Z2 (la cible est de 80 %).
  - *Polarisé* : au moins 75 % du temps en Z1, au plus 10 % en Z3 et au moins 10 % en Z4-Z5.
  - *Daniels* (allures E = endurance, T = seuil, I = intervalles VO2max, R = répétitions) : allure E pour la majorité du volume ; allure T au plus 10 %, I au plus 8 % et R au plus 5 % du volume hebdomadaire.
- **Choix de la Méthode** : le LLM choisit dans la liste fermée et justifie son choix en une ligne ; la bande ci-dessus le contrôle.
- **Chrono irréaliste (FR-4)** : on calcule une équivalence de type VDOT à partir des records des 6 derniers mois ; le seuil d'écart reste à fixer.

## 3. Hébergement et transport vers la montre
- Développement et exécution d'abord en local sur le Mac, puis auto-hébergement. La PWA est servie derrière Tailscale : rien n'est exposé sur Internet et aucune authentification n'est à écrire en v1.
- **HealthFit** importe les Séances planifiées d'intervals.icu depuis octobre 2025. Depuis mars 2026, il faut l'achat intégré « Fanatical Supporter » (9,99 $), que Badr possède.
- Le transport vers la montre et la remontée bloc par bloc ont été vérifiés en conditions réelles (revue party mode) : ce ne sont plus des risques à prototyper.
- Plan B pour le transport : Watchletic.
- Pistes écartées : WorkoutKit/HealthKit (compte développeur payant, sideload limité à 7 jours) et Intervals Companion (cibles d'allure douteuses).

## 4. Portes de qualité (NFR-8)
- Aucune story n'est terminée sans ses tests.
- Aucun appel à intervals.icu sans test de contrat ; un test aller-retour envoie le Plan, le relit et le compare au JSON source.
- Aucun chiffre du Feedback sans un test qui le recalcule.

## 5. Vérifications en attente
- **Une cible par étape sur la montre** : WorkoutKit n'accepte probablement qu'une seule cible par étape (allure ou FC). La règle de FR-9 (une cible par Bloc) s'applique quoi qu'il arrive ; si deux cibles s'avéraient possibles, on pourrait l'assouplir en v1.1.

## 6. Protocole du test de qualité (SM-5)
Le plan de référence sera la préparation marathon (avril 2027, moins de 3 h 30) construite avec l'ancienne méthode. Les deux plans sont anonymisés et présentés dans un ordre aléatoire avant la comparaison.

## 7. Paysage
intervals.icu Supporter (4 $/mois) propose déjà un connecteur MCP pour brancher son propre LLM, et des coachs LLM tiers existent autour d'intervals.icu. Ce n'est pas une menace pour un usage personnel, mais c'est à garder en tête si l'application est un jour partagée.
