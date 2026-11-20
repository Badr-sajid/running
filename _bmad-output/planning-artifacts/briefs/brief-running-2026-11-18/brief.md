---
title: "Product Brief : application de course à pied personnelle"
status: final
created: 2026-11-18
updated: 2026-11-18
---

# Product Brief : application de course à pied personnelle

## Résumé

Je construis, pour mon seul usage, une application de course à pied qui remplace une routine d'environ 1 h 30 par plan : conversation avec une IA, traduction, saisie manuelle dans intervals.icu. Elle garde mon profil de coureur en mémoire, génère et explique le plan à partir de mes données réelles et le publie dans intervals.icu, d'où HealthFit l'envoie sur mon Apple Watch. Après chaque course, elle me donne un feedback concret qui sépare son avis de mes données. Elle ne touche au plan qu'en cas de grand imprévu, avec ma confirmation.

Le prix de Runna a déclenché le projet ; la continuité en est le vrai moteur. La v1 ferme la boucle (générer, publier, courir, analyser) avec une barre claire : un plan au moins aussi bon qu'aujourd'hui, pour 10 minutes de manipulation au lieu de 90.

## Le problème

Je cours avec une Apple Watch Series 10, HealthFit et intervals.icu, et je construis mes plans d'entraînement en conversant avec une IA. Je relis chaque plan : il est presque toujours cohérent. Mais chacun me coûte **environ 1 h 30** :

- **Environ 1 h de conversation.** Un prompt dédié, puis je repartage tout : niveau, historique, objectifs, contraintes. L'IA ne retient rien d'un plan à l'autre.
- **Environ 30 min de traduction et de saisie.** Un second prompt convertit le plan au format des séances structurées d'intervals.icu, puis je saisis chaque séance à la main.

Je paie ce coût à chaque nouveau plan, et à chaque grand imprévu (maladie, blessure) qui m'oblige à le refaire. Ensuite, Strava ne me dit ni si j'ai respecté la séance ni ce qu'elle révèle de ma forme.

**Le problème n'est donc pas la qualité des plans, mais le fait que chacun repart de zéro et passe par une saisie manuelle.** Par ordre de gêne :

1. **La friction** : 1 h 30 de travail répétitif par plan.
2. **La continuité** : aucun plan ne tient compte du précédent ni de mon historique réel, sauf si je le réexplique.
3. **Le feedback** : aucun verdict structuré après une course.

## La solution

L'application se décrit en trois moments reliés par un même fil : **un profil vivant qui s'enrichit à chaque étape**.

**1. Créer un plan**
- **La première fois**, l'application importe mon historique (intervals.icu, HealthFit) et préremplit le profil : records, charge, forme (CTL), blessures, disponibilités, jour de la sortie longue. Je le valide ou le corrige ; on vise un profil complet, pas un démarrage rapide.
- **Ensuite**, je fixe un objectif parmi six types (un principal et des secondaires). L'application génère le plan et affiche en une ligne la méthode choisie (80/20, Daniels, polarisé…) et sa raison.
- **Je relis le plan** : je l'accepte en entier, ou je le refuse avec une consigne en langage naturel (« moins de volume en semaine 3 ») et l'application régénère. Je ne modifie jamais une séance à la main.
- **Une fois accepté**, le plan est écrit dans le calendrier d'intervals.icu en séances structurées, puis atteint la montre via HealthFit, sans aucune saisie de ma part.

**2. Après chaque course**
- À l'ouverture qui suit une nouvelle course, l'application me pose une seule question de ressenti (😌 / 😐 / 🥵) avant d'afficher le moindre chiffre.
- Le feedback compare le prévu et le réalisé bloc par bloc et situe la séance dans le plan et dans l'objectif. « Mon avis (IA) » reste visuellement séparé de « Tes données » (3 à 5 indicateurs) ; le détail complet reste dans intervals.icu (spécification dans l'addendum).
- La matrice respect × ressenti lit ma forme du jour. **Elle ne touche pas au plan**, sauf si un seuil de signaux concordants est franchi : elle déclenche alors le moment 3.

**3. Gérer un imprévu**
- **Le déclencheur** : le bouton « Empêchement » (dernière minute, malade, blessé) ou un seuil de surmenage atteint.
- **La proposition** : l'application évalue s'il suffit d'annuler ou de déplacer une séance, ou s'il faut régénérer le plan, puis propose et attend ma confirmation.
- **La republication** : la modification confirmée est republiée dans intervals.icu. Ce sont les seules synchronisations après la création du plan : **rares et volontaires**.

## Principes directeurs

Ces principes tranchent les arbitrages du PRD et de l'architecture.

- **Pilote automatique transparent.** L'application décide (méthode, plan, propositions d'ajustement) et explique toujours pourquoi en une ligne. J'ai un droit de veto, mais je ne fais pas le travail à sa place.
- **Aucune modification sans confirmation.** Le plan ne change jamais en silence.
- **La confiance avant les fonctionnalités.** Une synchro fiable et des recommandations crédibles comptent plus qu'une fonctionnalité de plus.
- **Un feedback concret, au ton motivant et réaliste.** Chaque affirmation cite une donnée mesurée ou ressentie ; aucun compliment sans chiffre pour le justifier. L'avis de l'IA reste séparé des données brutes, pour que je garde mon propre jugement.
- **Le stress et la récupération se dosent.** La prévention des blessures est une contrainte de base, et chaque séance part de l'état réel du jour.
- **Les données servent uniquement à la course.**
- **Simple et efficace d'abord.** Le projet sert aussi à apprendre des technologies modernes, mais quand une approche simple qui marche s'y oppose, l'application gagne.

## Pourquoi ne pas utiliser Runna

Runna a servi de référence : j'aime son accueil clair, sa génération de plan et ses commentaires de séance. Je l'écarte pour une raison franche, son prix (**environ 120 $/an**) : s'il était gratuit, je n'aurais pas lancé ce projet.

Ce qui en fait un projet, c'est ce que Runna ne fait pas : **chaque nouveau plan repart de zéro et ignore l'historique, et on ne peut pas enchaîner les plans**. Je veux que mes plans soient les chapitres d'une même histoire. L'application se distingue par :
- **la continuité** : la mémoire en v1 (le plan suivant part de ma forme réelle), la saison de plans enchaînés en v1.2, avec le bilan de chaque plan ;
- **l'intégration** à intervals.icu et HealthFit, sans nouvelle plateforme ;
- **l'explicabilité**, selon les principes ci-dessus ;
- **le coût** : pas d'abonnement, seulement l'usage de l'IA, plafonné à moins de 10 € par mois.

Elle n'a pas d'avantage technique exclusif. Les graphiques et la nutrition ne justifient pas sa création : ils viendront une fois la boucle fiable.

## Forme et dépendances

- **Une application web personnelle, conçue pour le mobile**, d'abord en local sur mon Mac, puis auto-hébergée et ouverte sur l'iPhone comme une PWA. Pas d'application iOS native, faute d'abonnement Apple Developer. Le choix des technologies revient à l'architecture.
- **intervals.icu est le pivot** : l'application y écrit les séances structurées par l'API (clé personnelle) et y lit les courses réalisées.
- **HealthFit fait le transport** vers l'Apple Watch (fonction en bêta : il faut ouvrir l'app pour recevoir la séance) et il est **la seule source** des courses. Strava n'est pas connecté à intervals.icu, pour éviter les doublons et les activités que l'API ne renvoie pas.
- **Un modèle de langage appelé par API**, payé à l'usage, génère les plans et les feedbacks.
- **Utilisateur unique** : moi. Ni comptes, ni multi-utilisateur, ni conformité au-delà de mes propres données.

## Périmètre de la v1

**Dans la v1** : la boucle fermée, de la génération au feedback.
- Le profil vivant, importé puis validé, avec dès le départ une place réservée à la nutrition et au sommeil, pour les brancher plus tard sans refonte.
- L'objectif (6 types, un principal et des secondaires) et le plan généré avec sa méthode, accepté ou refusé avec une consigne.
- La publication sur intervals.icu à la création et à chaque modification confirmée, avec un écran « état de la synchro » (dernière séance envoyée, dernière course reçue, statut, bouton resynchroniser).
- Le feedback après chaque course (moment 2), sur trois niveaux.
- La matrice respect × ressenti, qui ne propose une modification qu'au-delà d'un seuil, et le bouton « Empêchement ».
- Un accueil minimaliste, sans notification.

**Hors de la v1** (détail dans `roadmap.md` du brainstorming du 2026-11-17) :
- **v1.1, motivation** : jauge de faisabilité, prédiction de chrono, échauffements, vue « escalier », 3 notifications.
- **v1.2, continuité** : saison de plans enchaînés, conversation avec le coach, retour de blessure progressif.
- **Plus tard** : renforcement, chaussures, parcours, graphiques enrichis, puis nutrition et sommeil.
- **Écartés** : application iOS native, éditeur manuel de séances, ajustement après chaque course, choix manuel de la méthode.

## Critères de réussite

La v1 est réussie quand :

1. **La boucle fermée tourne en conditions réelles.** Un plan complet est généré, publié, couru et commenté, sans aucune saisie manuelle dans intervals.icu.
2. **L'ancienne routine disparaît.** Je n'ouvre plus de conversation avec une IA ni Strava pour planifier ou analyser.
3. **Le temps** : un plan suivant demande au plus 10 min de manipulation active, contre 1 h 30 aujourd'hui. Le temps de lecture ou d'analyse que je choisis d'y passer n'est pas compté.
4. **Le coût** : moins de 10 € par mois d'appels à l'IA.
5. **La crédibilité** : aucun chiffre faux dans les feedbacks.

**Test de qualité du plan** : pour un même objectif, je compare à la relecture un plan de l'application et un plan obtenu par l'ancienne méthode. Celui de l'application doit me paraître, selon mon expérience, au moins aussi crédible ; sinon, la v1 n'est pas prête.

## Inconnues et risques

- **Le transport vers la montre** [à prototyper en premier] : l'import des séances planifiées par HealthFit est en bêta. Il faut vérifier que les séances structurées arrivent exploitables sur l'Apple Watch S10 (blocs, cibles d'allure), et dans quel délai (contraintes de l'API dans l'addendum).
- **La remontée des courses** : obtenir des données complètes par intervalle (FC, allure, blocs) et sans doublons, avec HealthFit comme seule source.
- **La crédibilité de l'IA** : des chiffres exacts, sans flatterie.
- **La qualité du plan généré** : le test de comparaison la valide ou l'invalide.
- **Les seuils de la matrice respect × ressenti** : à calibrer pour éviter les propositions trop fréquentes ou les alertes inutiles.
- **Le coût réel de l'IA** : inconnu tant que la génération et le feedback n'ont pas été mesurés.
- **La dépendance à deux outils tiers** (intervals.icu, HealthFit) : je ne les maîtrise pas, et un changement chez l'un d'eux peut casser la boucle.

## Vision

À terme, l'application devient mon coach unique, avec une seule histoire d'entraînement : des saisons de plans enchaînés (préparation, course, récupération, préparation suivante), chacune avec le bilan du plan précédent, et un coach avec qui discuter à partir d'un feedback et qui retient ce qui compte. Le profil vivant accueille ensuite ce qu'il réserve déjà : la nutrition et le sommeil. Le principe reste le même : l'application décide et explique, je confirme.
