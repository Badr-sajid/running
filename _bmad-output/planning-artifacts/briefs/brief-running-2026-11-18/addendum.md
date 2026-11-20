---
title: "Addendum : application de course à pied personnelle"
created: 2026-11-18
updated: 2026-11-18
---

# Addendum : détails pour le PRD et l'architecture

## Création d'objectif

Il existe six types d'objectifs : distance, durée, chrono sur une distance, récupération après une course, condition physique générale, reprise après blessure. Chaque plan a un objectif principal et des objectifs secondaires, par exemple « 10 km en 45 min » et « ne pas se blesser ».

L'objectif s'appuie sur ces entrées : historique, records, forme actuelle, blessures actuelles et passées, jours disponibles, jour de la sortie longue, périodes d'indisponibilité.

## Feedback après chaque course (spécification validée lors du brainstorming)

- **La question de ressenti** : une seule question, trois emojis (😌 facile / 😐 normal / 🥵 dur). Elle est posée à l'ouverture de l'application qui suit la détection d'une nouvelle course, pendant le chargement du feedback et avant tout chiffre.
- **Le respect de la séance, bloc par bloc** : échauffement, chaque fractionné, récupération. Chaque bloc reçoit un verdict d'une ligne, et la séance une note de conformité globale.
- **Trois niveaux** : la séance, l'objectif de la séance (son rôle dans le plan), l'objectif global.
- **La structure** : un point fort, un point à améliorer, une conséquence pour la suite. Si la séance n'est pas respectée, le feedback dit ce qui a manqué et comment faire mieux la prochaine fois.
- **Deux couches visuellement séparées** : « Mon avis (IA) » et « Tes données » (3 à 5 indicateurs clés).

## Bouton « Empêchement » : règles

- **Dernière minute** : annuler ou déplacer la séance.
- **Malade** : règle « au-dessus ou en dessous du cou ». Un rhume léger permet un footing, la fièvre impose le repos.
- **Blessé** : annuler le plan, ou le suspendre jusqu'au rétablissement.
- **Dans tous les cas**, l'application évalue s'il faut régénérer le plan ou seulement annuler une ou plusieurs séances.

## Matrice respect × ressenti et signaux de surmenage

En v1, la matrice sert à lire la forme du jour : elle alimente le feedback et la détection du surmenage. Au-delà d'un seuil de signaux concordants, elle propose une modification du plan, traitée comme un grand imprévu. Seuil candidat : trois séances 🥵 d'affilée et une FC au repos en hausse. Le seuil exact est à calibrer dans le PRD.

Signaux de surmenage à capter pour calibrer la matrice :

- douleurs ;
- récupération plus longue ;
- effort perçu en hausse à charge égale (écart entre le ressenti et la FC ou l'allure) ;
- jambes lourdes ;
- baisse de motivation.

La plupart de ces signaux sont subjectifs, d'où le poids de l'emoji.

## Décisions de cadrage prises pendant la rédaction du brief

- **La synchro n'est pas un critère de réussite.** Sa fiabilité ne figure pas parmi les critères de la v1, mais l'écran « état de la synchro » reste dans le MUST.
- **HealthFit doit être ouvert.** Le principe « usage quotidien quasi nul » admet cette exception : il faut ouvrir HealthFit pour recevoir les séances.
- **Deux rythmes de synchro.** La synchro lourde est rare : création du plan et grands imprévus. Il n'y a pas de synchro légère après chaque course.

## Alternatives écartées

- **App iOS native** : écartée, faute d'abonnement Apple Developer.
- **Ajustement léger après chaque course** : écarté. Un feedback ne doit pas systématiquement modifier les séances à venir.
- **Éditeur de séance manuel** : écarté. La relecture se fait en acceptant ou en refusant le plan, avec une consigne en cas de refus.

## Contraintes techniques issues de la recherche (2026-11-18, sources surtout forums, à vérifier)

- **API intervals.icu** : `POST /api/v1/athlete/{id}/events` et la variante `events/bulk?upsert=true` pour envoyer un plan complet. Authentification par clé API personnelle (Basic auth). Quotas : environ 5 000 requêtes/jour, 2 500 par tranche de 15 min, 10 requêtes/s. En cas de dépassement, réponse 429 avec Retry-After.
- **Format des séances** : la syntaxe texte du champ `description` est le format le plus sûr. Le champ `workout_doc` n'est pas documenté, et un FIT vide a été signalé fin 2026.
- **Vers l'Apple Watch** : il n'existe pas de synchro officielle. Il faut passer par HealthFit (import des séances planifiées, en bêta) ou Intervals Companion (cibles d'allure douteuses). En pratique, il faut ouvrir l'app iPhone, et le délai va de quelques minutes à quelques heures.
- **Remontée des courses** : `GET /api/v1/activity/{id}?intervals=true` renvoie `icu_intervals`. L'API n'expose pas les activités venues de Strava. HealthFit doit donc être la seule source, sans connecter Strava à intervals.icu (risque de doublons).
- **WorkoutKit / HealthKit** : ils permettraient d'envoyer les séances directement sur la montre. L'option est écartée car elle exige un compte Apple Developer payant ; avec un compte gratuit, l'installation expire au bout de 7 jours.
