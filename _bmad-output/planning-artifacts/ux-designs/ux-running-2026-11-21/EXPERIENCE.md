---
title: running — EXPERIENCE
name: running
status: final
updated: 2026-11-22
sources:
  - _bmad-output/planning-artifacts/prds/prd-running-2026-11-20/prd.md
  - _bmad-output/planning-artifacts/briefs/brief-running-2026-11-18/brief.md
---

# running — Experience Spine

**En cas de conflit, ce document prime sur les maquettes.** Maquettes promues dans [`mockups/`](mockups/) ; liens au fil des sections ci-dessous.

## Foundation

- **Forme** : PWA mobile-first sur iPhone (NFR-1), ajoutée à l'écran d'accueil, servie derrière Tailscale. Aucune app native, aucun écran Apple Watch (la montre affiche les Séances via HealthFit).
- **Utilisateur** : une seule personne, Badr. **Pas de login en v1** (NFR-6) ; l'onboarding demande seulement un prénom.
- **Notifications** : aucune en v1. Tout se passe à l'ouverture et sur rafraîchissement manuel (FR-11).
- **Thème** : mode clair uniquement en v1. **Unités** : km et min/km (allure en `m:ss /km`), FC en bpm, virgule décimale française.
- **Système UI** : aucun ; conventions iOS (barre d'onglets en bas, sheets qui montent du bas, retour en haut à gauche). `DESIGN.md` est la référence d'identité visuelle ; ce document décrit le comportement. Les tokens sont cités sous la forme `{colors.primary}`.
- **Glossaire** : celui du PRD § 2, repris tel quel (Profil coureur, Objectif, Plan, Séance, Bloc, Course, Ressenti, Conformité, Feedback, Imprévu, Ajustement, Bilan de fin de Plan, Consigne, Publication…).

## Information Architecture

Barre de 4 onglets : **Accueil · Plan · Synchro · Profil**. Une seule sheet à la fois ; pile de navigation de 2 niveaux maximum sous un onglet.

**Onglet Plan = hub** de trois entrées : **Plan actuel** · **Historique** · **Profil coureur**. Le **Bilan de fin de Plan** est une sous-page de l'onglet (**Plan › Bilan**), atteinte depuis une carte « Bilan de ton plan » en haut du hub (affichée tant qu'aucun nouveau Plan n'est lancé), et depuis chaque ligne d'Historique. *(v1.2, plus tard : « Plans à venir » — enchaînement de saisons. Non construit en v1.)*

### Inventaire des surfaces

| Surface | Accès | Contenu / rôle | FR / NFR |
|---|---|---|---|
| Bienvenue — prénom | 1re ouverture | Saisie du prénom, rien d'autre | NFR-6 |
| Import de l'historique | Onboarding | « Importer ton historique » depuis intervals.icu, progression visible | FR-1 |
| Profil coureur — complément | Onboarding, puis Plan › Profil coureur | Valeurs importées corrigeables + blessures actuelles et passées, disponibilités par jour, jour de la sortie longue, périodes d'indisponibilité ; « Valider mon profil » | FR-2, FR-3 |
| Nouvel Objectif — [maquette](mockups/key-nouvel-objectif.html) | Onboarding, Accueil sans Plan, Plan actuel, Bilan | Saisie de l'Objectif : distance, chrono, date (obligatoire), disponibilités, Objectifs secondaires (texte libre) ; « Créer mon plan » | FR-4, FR-7 |
| Génération en cours — [maquette](mockups/key-plan-propose.html) | Après Objectif ou Consigne | Plein écran, étapes cochées, attente visible < 2 min | FR-5, NFR-9 |
| Plan proposé — [maquette](mockups/key-plan-propose.html) | Fin de génération | Plan à valider ou refuser avant toute Publication | FR-5, FR-6 |
| Sheet Consigne — [maquette](mockups/key-plan-propose.html) | « Refuser » | Consigne obligatoire puis régénération | FR-6 |
| Échec de génération | Après 3 tentatives | Nomme le garde-fou violé, propose de modifier l'Objectif, rien n'est publié | FR-5 |
| Confirmation de remplacement | Nouvel Objectif avec Plan actif/suspendu | « Ton plan actuel sera annulé » | FR-7 |
| **Accueil** — [maquette](mockups/key-accueil.html) | Onglet | Salutation, carte « Séance du jour », ligne « Dernier feedback », progression de la semaine, compte à rebours de l'Objectif, bouton calendrier | FR-20 |
| **Détail Séance** — [maquette](mockups/key-seance-detail.html) | Carte Séance du jour, ligne de Séance du Plan, ligne « Dernier feedback » | But en une ligne, liste des Blocs ; après la Course : « Mon avis (IA) », « Tes données », Conformité Bloc par Bloc, graphes, « Mes notes » | FR-13, FR-14, FR-15, FR-21 |
| Sheet Ressenti — [maquette](mockups/key-seance-detail.html) | Course détectée et rattachée | 😌 facile · 😐 normal · 🥵 dur | FR-12 |
| Sheet Rattachement | Course la veille/le lendemain d'une Séance non réalisée | « C'est ta séance de … ? » — confirmer d'un tap | FR-11 |
| **Plan › Plan actuel** — [maquette](mockups/key-plan-actuel.html) | Hub Plan, bouton calendrier | Le Plan en cours semaine par semaine, avec ses actions (Empêchement, Modifier, Arrêter, Reprendre) | FR-5, FR-7, FR-17, FR-19 |
| Sheet Empêchement | Plan actuel ; Détail de la Séance du jour (raccourci) | Dernière minute / Malade / Blessé, une question au plus, proposition + confirmation | FR-17, FR-19 |
| Sheet Modifier — [maquette](mockups/key-plan-actuel.html) | Plan actuel | 3 modes : Déplacer · Modifier à la main · Demander à l'IA | FR-22, FR-19 |
| Proposition de surmenage | Ouverture, seuil atteint | Ajustement proposé + pourquoi, confirmer / refuser | FR-16, FR-19 |
| **Plan › Historique** — [maquette](mockups/key-bilan.html) | Hub Plan, Bilan | Anciens Plans (*terminé*, *annulé*), du plus récent au plus ancien ; chaque ligne ouvre son Bilan | FR-8 |
| **Plan › Bilan** (Bilan de fin de Plan) — [maquette](mockups/key-bilan.html) | Carte « Bilan de ton plan » (hub Plan), ligne d'Historique, « Voir ton bilan » (Accueil) | Résumé chiffré du Plan, sans IA (voir Blocs du Bilan) ; « Définir un nouvel objectif » | FR-8 |
| **Plan › Profil coureur** | Hub Plan | Infos, blessures, disponibilités ; performances : records + indicateurs (CTL/forme, FC au repos, tendances) ; édition | FR-2, FR-3 |
| **Synchro** | Onglet | État de la chaîne de synchro, FR-10 seul (pas d'historique ni de journal) ; « Resynchroniser » (voir State Patterns › Synchronisation) | FR-10, FR-1, NFR-7 |
| **Profil** | Onglet | Prénom, réglages, suivi du coût IA du mois | NFR-4 |
| Feedbacks récents | Plan actuel (Séances *réalisées* de la semaine), ligne « Dernier feedback » sous la carte Séance du jour (Accueil) | Tap → Détail Séance | FR-20 |

## Voice and Tone

Français, **tutoiement**, coach chaleureux et bref : motivant et réaliste, jamais flatteur ; chaque affirmation cite un chiffre mesuré ou un Ressenti (PRD § 7). L'app parle à la première personne dans « Mon avis (IA) » uniquement.

| Faire | Ne pas faire |
|---|---|
| « Salut Badr » | « Bon retour parmi nous, champion ! 🎉 » |
| « Pourquoi cette séance — ta base est solide (CTL 42), on ajoute de l'intensité courte. » | « Séance générée par notre algorithme intelligent. » |
| « Comment tu l'as sentie ? » (Ressenti) | « Veuillez évaluer votre séance. » |
| « Analyse en cours… Je regarde tes 800 m un par un. » | « Chargement… » |
| « Point fort : 5 répétitions sur 6 dans la cible (4:03 de moyenne). » | « Super séance, bravo ! » (compliment sans chiffre) |
| « Avis indisponible pour cette séance. Tes données restent justes. » | « L'IA a planté. » |
| « manquée » | « Tu as raté ta séance ! » |
| « Plan impossible : la sortie longue dépasserait de plus de 10 % ta plus longue sortie des 30 derniers jours. » | « Impossible de générer le plan. » |
| Une ligne de raison par décision (NFR-5). | Paragraphes d'explication, jargon non expliqué. |
| « ⚠️ Règle franchie — sortie longue la veille d'un fractionné. Garde un jour facile entre les deux. » | « Action non autorisée. » |

### Microcopies validées (maquettes)

| Où | Texte |
|---|---|
| Séance *en attente de la course* | « Pas encore de course. Ouvre HealthFit pour l'envoyer, puis tire pour rafraîchir. » |
| Sheet Ressenti — titre | « Comment tu l'as sentie ? » |
| Sheet Ressenti — sous-titre | « Ta course est arrivée. Ton ressenti d'abord, les chiffres ensuite. » |
| Sheet Ressenti — rappel | « Un seul choix, il ne pourra plus être modifié. » |
| Mes notes — rappel | « Pour toi seul, jamais envoyé à l'IA. » |
| Sheet Modifier — sous-titre | « Chaque changement est revérifié, puis republié sur ta montre. » |
| Sheet Consigne — titre | « Qu'est-ce qui ne va pas ? » |
| Plan proposé — titre | « Ton plan est prêt » |
| Nouvel Objectif — sous-titre | « Ton plan ira de demain jusqu'au jour de ta course. » |
| Nouvel Objectif — aides | « Soit 4:57 /km de moyenne. » · « 20 semaines de préparation. » · « Repris de ton Profil coureur. » · « Pris en compte pour construire ton plan. » |
| Nouvel Objectif — chrono irréaliste | « ⚠️ Chrono jugé réaliste : 3 h 41 — d'après ton semi en 1 h 44 (mars 2026), ton meilleur record des 6 derniers mois. » + « Garder mon chrono » |
| Nouvel Objectif — date refusée | « Trop proche : il faut au moins 4 semaines de plan. Choisis une date à partir du 13 décembre. » |
| Bilan — titre / pied | « Bilan de ton plan » · « Ton prochain plan partira de ce bilan. » |
| Historique — sous-titre | « Touche un plan pour revoir son bilan. » |

### Libellés de la Matrice (FR-15)

Repris du PRD, affichés en une ligne « État du jour — … » sous l'en-tête du Détail Séance. Formulation **provisoire, ajustable à l'usage**.

| | 😌 | 😐 | 🥵 |
|---|---|---|---|
| **Respectée** | marge, en progrès | dans le Plan | à la limite, à surveiller |
| **Non respectée** | Séance mal calibrée ou trop prudente | exécution à revoir | fatigue probable |

## Component Patterns

Comportement uniquement ; le visuel est décrit dans `DESIGN.md` § Components.

| Composant | Où | Règles de comportement |
|---|---|---|
| Barre d'onglets | Partout sauf onboarding, Génération en cours, Plan proposé et sheets | 4 onglets ; nouveau tap sur l'onglet actif = retour en haut de sa pile. Point d'alerte sur Synchro en cas d'erreur de synchro. |
| Carte Séance du jour | Accueil | Unique (FR-20). Tap n'importe où ou « Voir le détail » → Détail Séance. Contenu selon l'état (voir State Patterns). Jour sans Séance : carte « Repos » + prochaine Séance ; « Voir le détail » ouvre cette prochaine Séance. |
| Bouton icône calendrier | Accueil | → Plan › Plan actuel, semaine courante dépliée. |
| Progression de la semaine | Accueil | Lecture seule ; un segment par Séance, allumé quand elle est *réalisée*. |
| Ligne « pourquoi » | Carte Séance, Plan proposé, propositions d'Ajustement | Toujours une ligne (NFR-5). Jamais tronquée sans moyen de la lire en entier. |
| Carte « Mon avis (IA) » | Détail Séance | Apparaît seulement après le Ressenti (FR-12). Exactement 3 parties : Point fort · À améliorer · Pour la suite (FR-14). Si la Séance est non respectée : « ce qui a manqué » + « comment faire mieux ». |
| « Tes données » | Détail Séance | 1 chiffre principal + 2 secondaires (FR-14), visuellement séparés de l'avis ; masqués tant que le Ressenti n'est pas posé. Lien « Voir dans intervals.icu ». |
| Ligne de Bloc | Détail Séance | Avant la Course : Bloc, durée/distance, cible (FR-21). Après : réalisé + verdict mot + icône (FR-13). Si tours ≠ Blocs ou Course non planifiée : pas de verdict par Bloc, bannière « Analyse globale : les tours de ta montre ne correspondent pas aux blocs prévus ». |
| Carte graphe | Détail Séance, sous « Tes données » | Lecture seule, pas d'interaction en v1 ; texte alternatif résumant la série. |
| Notes perso | Bas du Détail Séance | Texte libre « Mes notes », jamais envoyé à l'IA. Enregistrement automatique à la sortie du champ. |
| Sheet Ressenti | Détail Séance | 3 options, un tap = choix + fermeture → « Analyse en cours… ». Non fermable sans réponse (pas de poignée, ni glisser ni tap sur le voile) ; Ressenti non modifiable ensuite. |
| Sheet Rattachement | Détail Séance / Accueil | « Oui, c'est elle » / « Non, course libre » ; un tap (FR-11). |
| Sheet Empêchement | Plan actuel ; Détail de la Séance du jour (raccourci) | Raccourci visible sur la **Séance du jour** uniquement tant qu'elle est *à venir* — masqué dès qu'elle est *réalisée* et sur toute autre Séance (FR-17 : 2 taps depuis l'Accueil). Tap 1 « Empêchement », tap 2 type. Au plus une question (Malade : « Symptômes au-dessus ou en dessous du cou ? » ; Dernière minute : « Annuler ou déplacer ? » ; Blessé : « Suspendre ou annuler le plan ? »). Proposition + pourquoi, « Confirmer » d'un tap ; « Refuser » = Plan inchangé (FR-19). |
| Sheet Modifier | Plan actuel | Disponible seulement sur une Séance à venir sans Course rattachée ; une Séance passée ou rattachée n'affiche pas « Modifier » (verrouillée, FR-22). Choix du mode, puis : **Déplacer** (Séance → sélecteur de jour ; seuls les jours futurs, libres et antérieurs à l'Objectif sont sélectionnables, les autres sont grisés) ; **Modifier à la main** (distance, allure, type) ; **Demander à l'IA** (FR-22, confirmé comme un Ajustement FR-19 : champ texte libre → proposition + pourquoi → « Confirmer » / « Refuser »). Toute modification enregistrée est republiée (FR-9). Après Déplacer / Modifier à la main, l'app vérifie les garde-fous FR-5 (FR-22) : un garde-fou de **sécurité physique** franchi → modification **refusée** (bannière nommant le garde-fou, « Enregistrer » désactivé, rien n'est publié) ; un écart aux **préférences du Profil** (jour disponible, jour de la sortie longue) → autorisé ponctuellement, avec une ligne d'information « Hors de tes jours habituels · ton profil ne change pas ». |
| Plan proposé | Après génération | Méthode + raison, semaines dépliables (lisibles avant acceptation, FR-5). Barre d'action **fixe à l'écran** pendant le défilement. « Valider le plan » → *actif* + Publication → Accueil. « Refuser » → sheet Consigne. |
| Étapes de génération | Génération en cours | Trois étapes (Profil lu · Méthode choisie · Garde-fous vérifiés) : faite ✓, en cours (pastille pulsante), pas commencée (point gris). VoiceOver annonce « fait » / « en cours » ; zone `status`. Tentatives invisibles. |
| Sheet Consigne | Plan proposé | Champ obligatoire : « Régénérer » désactivé tant qu'il est vide (FR-6). |
| Section de semaine | Plan actuel, Plan proposé | Semaine courante dépliée par défaut ; les autres repliées ; tap = déplier/replier. Tap sur une Séance → Détail Séance. |
| Formulaire Objectif | Nouvel Objectif | Date à moins de 4 semaines refusée dès la saisie, sous le champ, texte d'erreur qui propose la première date possible (FR-4) ; **« Créer mon plan » désactivé tant que la date est refusée** (la date seule bloque). Chrono irréaliste : bannière d'avertissement « Chrono jugé réaliste : 3 h 41 » + raison chiffrée, puis « Garder mon chrono » (non bloquant) ; **la bannière disparaît après « Garder mon chrono »**. Jours disponibles et **jour de la sortie longue préremplis depuis le Profil coureur**. |
| Sélecteur natif | Nouvel Objectif (distance, chrono, date, jours de course par semaine) | Tap sur le champ → sélecteur natif iOS ; la valeur choisie s'affiche dans le champ, avec son aide dessous (allure moyenne, nombre de semaines). |
| Puces à choix unique | Jour de la sortie longue (Nouvel Objectif), nouveau jour (sheet Déplacer) | Un seul jour actif ; tap sur un autre jour = déplacement de la sélection. Pour la sortie longue, seuls les jours disponibles sont sélectionnables ; les autres sont grisés. Rôle radio pour VoiceOver. Les puces bascule (jours disponibles) restent à choix multiple. |
| Formulaire Profil coureur | Onboarding, Plan › Profil coureur | Valeurs importées éditables ; « Valider mon profil » requis une fois avant tout Plan (FR-2). |
| Ligne de liste | Hub Plan, Synchro, Profil | Tap → écran ou détail ; chevron si navigation. |
| Ligne d'Historique | Plan › Historique | 2e ligne : dates + durée, ou « arrêté en semaine 3 » pour un Plan *annulé*. Tap → **son Bilan** (Plan › Bilan). |
| Blocs du Bilan | Plan › Bilan | Lecture seule, calculés sans IA : Volume (réalisé vs prévu, Séances manquées) · Conformité moyenne · Forme et records (CTL début → fin, chiffres seuls ; pas de graphe CTL en v1, peut-être plus tard) · Ressentis (répartition 😌 / 😐 / 🥵). Ligne « Historique » → Plan › Historique. Bouton principal unique « Définir un nouvel objectif ». |
| Bouton « Resynchroniser » | Synchro | Republie le Plan (FR-10) ; état « Publication… » puis ✅ / ⚠️. |
| Bouton « Reprendre » | Plan actuel, Accueil (Plan *suspendu*) | Seul moyen de réactiver un Plan *suspendu* (FR-17) ; passe par la génération (FR-19). |
| « Arrêter le plan » | Plan actuel, **en bas de la liste** (loin des actions courantes) | Confirmation → Plan *annulé*, Séances à venir retirées d'intervals.icu, Bilan calculé (FR-8). |

### « Tes données » par type de Séance (validé)

| Type | Principal | Secondaires | Graphe |
|---|---|---|---|
| Fractionné | Allure moyenne des Blocs rapides vs cible | Distance totale · FC moyenne des Blocs rapides | Allure par répétition + zone cible |
| Tempo / seuil / allure de course | Allure moyenne des Blocs à allure vs cible | Distance totale · FC moyenne de ces Blocs | Allure au fil de la Course + zone cible |
| Footing / récupération | % du temps dans la zone FC cible | Distance · allure moyenne | FC au fil de la Course + zone |
| Sortie longue | Distance | % du temps en zone FC · allure moyenne | FC et allure au fil de la Course |
| Course non planifiée | Distance | Allure moyenne · FC moyenne | Allure au fil de la Course |

## State Patterns

### Séance (carte et détail)

| État | Traitement |
|---|---|
| *à venir* | But, chiffres prévus, pourquoi, « Voir le détail ». Sur le Détail de la Séance du jour : raccourci « Empêchement ». |
| *en attente de la course* | Badge neutre « en attente de la course » (FR-11) + ligne de rappel HealthFit (voir Microcopies validées). |
| Course détectée, Ressenti non posé | Sheet Ressenti ouverte automatiquement **une fois la Course synchronisée et rattachée** (FR-12). Aucun chiffre affiché avant. |
| Analyse en cours… | Carte « Mon avis (IA) » : pastille pulsante + « Analyse en cours… » + skeleton. « Tes données » et Conformité visibles dessous (chiffres déterministes). Cible < 30 s (NFR-9). Pas de message d'attente prolongée. |
| Erreur d'analyse | Message clair + « Relancer » / « Arrêter l'analyse ». 2 relances au plus par Feedback, et seulement tant que le Plan est actif ; ensuite « Relancer » disparaît (FR-14). « Arrêter l'analyse » mène à l'état « Avis indisponible ». |
| *réalisée* · Feedback prêt | Avis, données, Blocs, graphes, notes. Badge « respectée » / « non respectée » + note de Conformité %. Raccourci « Empêchement » masqué. |
| Avis indisponible | Après 3 rejets (FR-14) ou « Arrêter l'analyse » : la carte affiche « avis indisponible » ; « Tes données » et Conformité restent visibles. Bouton « Relancer » qui relance la génération, dans la limite de 2 relances (FR-14). |
| *manquée* | Badge neutre « manquée », le lendemain (FR-18). Aucune alerte, aucun Ajustement. |
| *annulée* | Badge neutre « annulée » + raison en une ligne. |
| Course non planifiée | Analyse globale, sans verdict par Bloc ; « Course libre ». |

### Synchronisation

| État | Où | Traitement |
|---|---|---|
| Synchro en cours (ouverture / tirer pour rafraîchir) | Accueil, Détail Séance | Indicateur discret « Synchro… » en haut ; contenu en cache affiché. |
| Première synchro | Onboarding import | Barre de progression + « Ça peut prendre quelques minutes, ce n'est pas une erreur. » (FR-1). |
| Synchro ✅ | Synchro | Dernière Séance publiée, dernière Course reçue, horodatées. |
| Synchro ⚠️ | Synchro (+ point sur l'onglet) | Cause en clair (quota intervals.icu, clé, réseau) ; « Ton plan est conservé. » ; « Resynchroniser » (FR-10, NFR-7). |
| Hors ligne | Global | Dernière version en cache affichée, bannière « Hors ligne — données de … ». |

### Plan

| État | Traitement |
|---|---|
| Aucun Plan (premier usage ou après *terminé* / *annulé*) | Accueil : « Pas de plan en cours » + « Définir un objectif » (FR-20, FR-7). |
| Profil non validé | « Définir un objectif » mène d'abord au Profil coureur (FR-2). |
| Génération en cours | Plein écran « Je prépare ton plan… » + Étapes de génération (voir Component Patterns) ; < 2 min (NFR-9). |
| Échec de génération | « Plan impossible » + garde-fou violé nommé + « Modifier l'objectif » ; rien n'est publié (FR-5). |
| *proposé* | Écran Plan proposé ; rien n'est publié tant qu'il n'est pas validé. |
| *actif* | Plan actuel normal. Après Publication : bannière d'information « Ouvre HealthFit pour envoyer les séances sur ta montre. » |
| *suspendu* | Accueil et Plan actuel : « Plan en pause » + raison + « Reprendre » (FR-17). Aucune Séance du jour. |
| *terminé* | Le lendemain de la dernière Séance : Bilan en Plan › Bilan ; Accueil propose « Voir ton bilan » et un nouvel Objectif (FR-7, FR-8). |
| Modification refusée (garde-fou de sécurité physique) | Bannière d'avertissement nommant le garde-fou (« Sortie longue à 24 km : +20 % au lieu de 10 % max ») ; « Enregistrer » désactivé ; Badr choisit une autre valeur ou « Annuler » (FR-22). Ne concerne que les hausses de risque : une réduction de charge n'est jamais refusée. Variante surmenage : « Séance allégée il y a 3 jours (surmenage) : pas de hausse avant le 12 déc. » |
| Écart aux préférences du Profil | Ligne d'information « Hors de tes jours habituels · ton profil ne change pas » ; « Enregistrer » actif (FR-22). Vaut aussi pour une demande à l'IA et pour « déplacer » dans Empêchement. |
| Demande à l'IA hors périmètre | Plus d'une semaine, ajout/suppression de Séance, changement d'Objectif ou demande ambiguë : « Je ne peux modifier qu'une semaine à la fois. » ; rien ne change (FR-22). Après 3 échecs : garde-fou nommé + « Modifier à la main ». |
| Régénération après modifications | Le « pourquoi » signale les modifications non conservées (« ton déplacement du 4 déc. n'est pas conservé ») (FR-19). |
| Seuil de surmenage atteint | Sheet de proposition à l'ouverture : « allégé car 🥵 ×3 et FC au repos +6 » ; confirmer / refuser (FR-16). |

### Autres

| Surface | État | Traitement |
|---|---|---|
| Historique | Vide | « Ton premier bilan arrivera à la fin de ce plan. » |
| Profil › coût IA | Mois en cours | Montant du mois vs budget 10 € (NFR-4) ; avertissement si > 80 %. |

## Interaction Primitives

- Tap pour agir ; un bouton principal par écran.
- **Tirer pour rafraîchir** sur Accueil, Détail Séance et Synchro : relance la détection des Courses (FR-11).
- Sheets montant du bas pour les choix courts (Ressenti, Empêchement, Modifier, Rattachement, Consigne, confirmations) ; glisser vers le bas pour fermer, sauf pendant une action en cours et sauf la sheet Ressenti (réponse obligatoire).
- Toute modification du Plan passe par une confirmation explicite ; le Plan ne change jamais en silence.
- Dépliage des semaines par tap.
- **Interdits** : carrousels, animations d'ouverture, balayage destructif ; voir aussi Inspiration & Anti-patterns.

## Accessibility Floor

Comportement ; le contraste est décrit dans `DESIGN.md` § Colors.

- Cibles tactiles ≥ 44 × 44 px (`{spacing.tap-min}`).
- VoiceOver : chaque contrôle a un rôle et un libellé ; les emojis de Ressenti sont annoncés « facile », « normal », « dur » ; les verdicts annoncent « Bloc 3, respecté » ; « Analyse en cours » est annoncé une fois, puis l'arrivée de l'avis l'est aussi.
- Graphes : texte alternatif qui résume la série (« 6 répétitions, de 4:01 à 4:12, cible 4:00–4:10 »).
- Jamais la couleur seule : chaque état a un mot.
- Taille de texte iOS respectée (tailles en `rem`) ; aucun contrôle tronqué.
- Réduire les animations : pulsation et skeleton deviennent statiques.
- Ordre de lecture = ordre visuel ; focus envoyé au titre de la sheet à son ouverture et rendu au déclencheur à sa fermeture.

## Synchronisation et données externes

- **Chaîne** : app → intervals.icu (Publication) → HealthFit → Apple Watch ; Course : Apple Watch → HealthFit → intervals.icu → app (FR-1, NFR-7). Strava absent.
- **HealthFit doit être ouvert** pour que la montre reçoive les Séances et que les Courses remontent. L'app le rappelle à trois endroits : après une Publication, dans l'état « en attente de la course », dans l'onglet Synchro (« Ouvre HealthFit pour envoyer tes séances et remonter tes courses »).
- Lectures à chaque ouverture et sur rafraîchissement ; Publications seulement à l'acceptation d'un Plan, à la confirmation d'un Ajustement ou d'une modification, et sur « Resynchroniser ».
- Les erreurs tierces (quota, clé, indisponibilité) s'affichent avec leur cause dans Synchro, sans jamais perdre le Plan.
- Le détail complet d'une Course reste dans intervals.icu ; l'app n'est pas un tableau de bord (PRD § 8).

## Inspiration & Anti-patterns

- **Registre : Runna** — clarté de l'accueil, présentation d'une Séance (nom, but, chiffres clés, pourquoi), ton des commentaires. Pris comme niveau d'exigence, pas copié.
- **Rejeté — tableau de bord d'analyse** : 1 + 2 chiffres et des graphes simples ; le reste dans intervals.icu.
- **Rejeté — gamification** (séries, badges, confettis) et **flatterie** (« Bravo ! » sans chiffre).
- **Rejeté — notifications** en v1.
- **Rejeté — calendrier mensuel** : vue semaine par semaine.
- **Rejeté — badges de récompense** (voir gamification).
- Directions visuelles et mode sombre : voir `DESIGN.md` § Brand & Style.

## Key Flows

### Flux 1 — Premier lancement : Profil coureur, Objectif, Plan (Badr, un dimanche soir, marathon d'avril en tête) — UJ-1, UJ-2

1. Badr ouvre la PWA. « Bienvenue » : il saisit son prénom.
2. « Importer ton historique » : l'app lit intervals.icu ; la progression s'affiche pendant les quelques minutes de l'import (FR-1).
3. Profil coureur prérempli (records, charge, CTL, FC au repos). Il corrige une valeur, ajoute une ancienne blessure, ses disponibilités, le dimanche pour la sortie longue, une période d'indisponibilité. « Valider mon profil » (FR-2).
4. Nouvel Objectif ([maquette](mockups/key-nouvel-objectif.html)) : marathon, 3 h 29, 4 avril 2027 via les sélecteurs natifs ; secondaire « ne pas se blesser » ; 4 jours/semaine, jours disponibles et sortie longue préremplis.
5. Si le chrono est jugé irréaliste : bannière + chrono réaliste ; il touche « Garder mon chrono », la bannière disparaît.
6. « Je prépare ton plan… » (< 2 min).
7. Plan proposé ([maquette](mockups/key-plan-propose.html)) : Méthode 80/20 + raison en une ligne ; il déplie les semaines, la barre « Valider le plan » / « Refuser » reste à l'écran.
8. Il touche « Refuser » ; Consigne obligatoire : « moins de volume en semaine 3 » ; « Régénérer ».
9. Nouveau Plan proposé ; « Valider le plan » → *actif* + Publication.
10. Retour à l'Accueil, bannière « Ouvre HealthFit pour envoyer les séances sur ta montre ».
11. **Climax** : la première Séance du jour est là, sur l'app et sur la montre — il n'a rien saisi dans intervals.icu.

Échecs : 3 générations ratées → « Plan impossible » qui nomme le garde-fou + « Modifier l'objectif ». Date à moins de 4 semaines → refusée dans le formulaire. Publication ratée → ⚠️ dans Synchro, le Plan est conservé.

### Flux 2 — Le fractionné après le travail (Badr, mardi 18 h) — UJ-3

1. 17 h 30, Badr ouvre l'app ([Accueil](mockups/key-accueil.html)) : carte « Séance du jour » — Fractionné 6×800 m, 4:05 /km, pourquoi en une ligne.
2. « Voir le détail » : il relit les Blocs et les cibles (FR-21).
3. Il s'échauffe, court avec la montre seule, s'étire.
4. Il ouvre HealthFit, puis l'app. « Synchro… » en haut.
5. Si la Course n'est pas encore arrivée : « en attente de la course » + rappel HealthFit ; il tire pour rafraîchir.
6. Course détectée et rattachée → sheet Ressenti ([maquette](mockups/key-seance-detail.html)), aucun chiffre visible. Il touche 🥵 dur.
7. Détail Séance : « Analyse en cours… » (< 30 s).
8. **Climax** : il lit « Mon avis (IA) » — point fort, à améliorer, pour la suite, chacun avec son chiffre, et l'état du jour (« à la limite, à surveiller »).
9. Il fait défiler : « Tes données » (4:05 /km sur les 800 · 9,2 km · 168 bpm), verdict par Bloc, graphe d'allure par répétition.
10. En bas, « Mes notes » : « mollet gauche un peu dur ». Il ferme l'app.

Échecs : avis rejeté 3 fois → « avis indisponible », données et Conformité seules. Tours ≠ Blocs → analyse globale annoncée. Course la veille d'une Séance non réalisée → sheet Rattachement. Troisième 🥵 d'affilée + FC au repos +6 → proposition d'Ajustement à l'ouverture suivante.

### Flux 3 — Empêchement puis « Reprendre » (Badr, jeudi matin, fiévreux) — UJ-4

1. Depuis l'Accueil, Badr ouvre la Séance du jour (encore *à venir*), touche le raccourci « Empêchement » (aussi accessible en haut de Plan › Plan actuel), puis « Malade ».
2. Une question : « Symptômes au-dessus ou en dessous du cou ? » → en dessous (fièvre).
3. Proposition : repos aujourd'hui, suite décalée, raison en une ligne.
4. **Climax** : un tap « Confirmer » — le Plan est republié, la Séance du jour devient « Repos ». Rien d'autre à faire.
5. Variante « Blessé » deux semaines plus tard : « Suspendre le plan » → l'Accueil affiche « Plan en pause ».
6. Rétabli, il touche « Reprendre » ; nouvelle génération (< 2 min), Plan proposé, il valide.

Échecs : « Refuser » → Plan inchangé. Plan régénéré hors garde-fous après 3 tentatives → échec explicite, le Plan reste *suspendu*.

### Flux 4 — Modifier le Plan (Badr, dimanche, un mariage samedi prochain)

1. Plan › Plan actuel ([maquette](mockups/key-plan-actuel.html)) › « Modifier » (en haut de l'écran).
2. Mode **Déplacer** : il déplace la sortie longue du dimanche au vendredi via le sélecteur de jour.
3. L'app vérifie les garde-fous (FR-22) : aucune règle de sécurité franchie ; vendredi n'est pas son jour de sortie longue → ligne d'information « Hors de tes jours habituels · ton profil ne change pas ».
4. **Climax** : « Enregistrer » → Plan republié, la semaine reste lisible et cohérente.
5. Variante **Modifier à la main** : il passe la sortie longue de 20 à 24 km → refusée, « Sortie longue à 24 km : +20 % au lieu de 10 % max » ; « Enregistrer » désactivé, il revient à 22 km.
6. Variante **Demander à l'IA** : « réduis la semaine 6, je suis en déplacement » → proposition + pourquoi, conforme à tous les garde-fous → « Confirmer ».

### Flux 5 — Fin de Plan (Badr, le lendemain du marathon)

1. Le lendemain de la dernière Séance, le Plan passe à l'état *terminé* (FR-7).
2. Accueil : « Plan terminé » + « Voir ton bilan » + « Définir un objectif ».
3. « Voir ton bilan » ou le hub Plan ouvre **Plan › Bilan** ([maquette](mockups/key-bilan.html)) (FR-8).
4. **Climax** : il voit tout son chapitre résumé en un écran, sans rien calculer.
5. Le Plan rejoint l'Historique avec son Bilan ; plus tard, sa ligne d'Historique rouvre ce Bilan.
6. Nouvel Objectif → le pourquoi du nouveau Plan cite le Bilan précédent (SM-6).

## Questions ouvertes

Questions closes : voir `.memlog.md`. Aucune question ouverte : les modes de modification ont été intégrés au PRD (FR-22, 22/11).
