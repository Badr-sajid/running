# Extraction des sources pour l'UX (2026-11-21)

Abréviations des sources :
- **PRD** = `planning-artifacts/prds/prd-running-2026-11-20/prd.md`
- **PRD-add** = `planning-artifacts/prds/prd-running-2026-11-20/addendum.md`
- **DIG** = `planning-artifacts/prds/prd-running-2026-11-20/research-digest.md`
- **BRF** = `planning-artifacts/briefs/brief-running-2026-11-18/brief.md`
- **BRF-add** = `planning-artifacts/briefs/brief-running-2026-11-18/addendum.md`
- **RDM** = `brainstorming/brainstorm-app-running-perso-2026-11-17/roadmap.md`
- **PTY** = `party-mode/revue-brief-2026-11-20.md`

Le PRD (status: final) fait foi ; ses arbitrages §13 tranchent les contradictions entre BRF, RDM et PTY.

---

## 1. Plateforme et forme

- Application web **mobile-first**, utilisée sur **iPhone en PWA** ; d'abord en local sur le Mac, puis auto-hébergée. **Pas d'app iOS native.** (PRD §6 NFR-1 ; BRF « Forme et dépendances »)
- PWA servie **derrière Tailscale** : rien d'exposé sur Internet, **aucune authentification** en v1, clés jamais côté client. (PRD-add §3 ; PTY §4 ; PRD NFR-2)
- Mono-utilisateur, sans comptes ni inscription ; « porte ouverte au partage » : plusieurs Profils coureur possibles plus tard (aucun identifiant figé). (PRD NFR-6, §4.2)
- Écran Apple Watch : **hors périmètre de l'app** ; la montre affiche les Séances via HealthFit / app Exercice (WorkoutKit). (PRD §1 ; DIG §2)
- Temps de réponse : Feedback < 30 s après le Ressenti, Plan < 2 min, **avec un état d'attente visible**. (PRD NFR-9)
- Première synchro : progression affichée, peut durer plusieurs minutes sans être une erreur. (PRD FR-1)

## 2. Persona / protagoniste

- **Badr** : coureur et développeur, 3 à 4 sorties/semaine, **Apple Watch Series 10**. (PRD §4.3)
- Prochain objectif : **marathon début avril 2027 en moins de 3 h 30** ; peut changer, ne fixe pas le périmètre. (PRD §4.3 ; PTY §2)
- Possède HealthFit avec l'achat intégré « Fanatical Supporter ». (PRD-add §3)
- Il construit son propre outil, « et bien » ; pas de date limite, priorité à la solidité. (PRD §1 ; PTY §2)
- Non-utilisateurs v1 : toute autre personne ; coureurs sans Apple Watch, HealthFit ou intervals.icu. (PRD §4.2)
- JTBD : fonctionnel (Plan sur la montre sans rien saisir ; savoir après chaque Course si la Séance est respectée, sans Strava), contextuel (Imprévu en **deux taps**), émotionnel (confiance dans les chiffres, aucune flatterie, garder son jugement, rester motivé), bâtisseur (posséder son outil plutôt qu'un abonnement de type Runna). (PRD §4.1)

## 3. Intégrations

- **intervals.icu = pivot** : seule écriture (Publication) et source de lecture (Courses, records, charge, CTL, FC au repos). Clé API personnelle en Basic auth côté serveur. (PRD §2, principe 6, FR-1 ; PRD-add §1)
- **HealthFit** : transport vers la montre (import des Séances planifiées) et **seule source des Courses** ; **jamais lu directement**, uniquement via intervals.icu. Il faut **ouvrir HealthFit** pour que la montre reçoive les Séances (accepté en v1). (PRD FR-1, NFR-7, §13 arb. 10 et 15)
- **Apple Watch / Apple Health** : atteints uniquement via HealthFit ; **pas de lecture directe de HealthKit ni WorkoutKit**. (PRD §8 ; PRD-add §3)
- **Strava : exclu.** Non connecté à intervals.icu (doublons, activités Strava non exposées par l'API). (PRD NFR-7, §8 ; BRF-add « Contraintes techniques »)
- **Garmin : non utilisé.** Mentionné seulement comme comparable (Garmin Coach) ou anecdote forum. (DIG §1, §3)
- **LLM** appelé par API, payé à l'usage ; seul autre tiers autorisé avec intervals.icu. (PRD NFR-2 ; BRF)
- Plan B transport : **Watchletic**. Écartés : WorkoutKit/HealthKit, Intervals Companion. (PRD-add §3)
- Transport montre et remontée bloc par bloc (un lap par Bloc) **vérifiés en conditions réelles**. (PTY §1 ; PRD-add §3)
- Contexte : HealthFit propose un widget « Next Scheduled Workout ». (DIG §2)
- Quotas intervals.icu : erreurs visibles dans l'état de la synchro. (PRD NFR-7)

## 4. Glossaire à reprendre **tel quel** (PRD §2)

Profil coureur · CTL · Objectif (principal / secondaires) · Épreuve (≠ Course) · Méthode (80/20, Daniels, polarisé) · Plan (états : *proposé*, *actif*, *suspendu*, *terminé*, *annulé* ; un seul actif ou suspendu) · Bilan de fin de Plan · Consigne · Séance (états : *à venir*, *en attente de la course*, *réalisée*, *manquée*, *annulée*) · Bloc (une seule cible : allure ou FC) · Publication · Course · Ressenti (😌 facile, 😐 normal, 🥵 dur) · Conformité · Feedback · Matrice respect × ressenti · état du jour · Seuil de surmenage · Imprévu · Ajustement (annuler, déplacer, alléger, suspendre, reprendre, régénérer).

Libellés UI cités verbatim :
- « Séance du jour » (carte d'accueil, PRD FR-20)
- « Mon avis (IA) » / « Tes données » (PRD FR-14)
- « en attente de la course » (PRD FR-11, UJ-3)
- « Empêchement », puis « dernière minute » / « malade » / « blessé » (PRD FR-17)
- « Reprendre » (bouton, PRD FR-17)
- « resynchroniser » (bouton) et statut ✅ / ⚠️ (PRD FR-10)
- « avis indisponible » (PRD FR-14)
- « état de la synchro » (PRD §5.3, FR-10)
- Exemple de pourquoi : « allégé car 🥵 ×3 et FC au repos +6 » (PRD FR-16)
- Exemples : « semi en 1 h 40 le 12 avril » (Objectif), « moins de volume en semaine 3 » (Consigne) (PRD UJ-2)

## 5. Principes directeurs utiles à l'UX (PRD §3)

1. La confiance avant les fonctionnalités.
2. Simple et efficace d'abord.
3. **Pilote automatique transparent** : l'app décide et explique **en une ligne** ; veto de l'utilisateur.
4. **Le LLM raconte, le code compte** : aucun nombre issu du LLM.
5. Chaque Plan part de l'état réel (on ne réexplique rien).
6. intervals.icu pivot, transport interchangeable.
- NFR-5 Explicabilité : Méthode, Plan, Ajustement toujours avec leur raison en une ligne. (PRD §6)
- « Aucune modification sans confirmation. Le plan ne change jamais en silence. » (BRF « Principes directeurs »)

## 6. Parcours utilisateur (PRD §4.3)

- **UJ-1** Configuration unique du Profil : import intervals.icu (records, charge, CTL) → formulaire (blessures, disponibilités, jour de la sortie longue, périodes d'indisponibilité) → validation. Synchro initiale potentiellement longue.
- **UJ-2** Création de Plan : saisie Objectif → Plan proposé avec Méthode + raison en une ligne → refus avec Consigne → régénération → acceptation → Séances dans intervals.icu puis montre. **Climax : il n'a rien saisi.**
- **UJ-3** Course puis Feedback : ouverture → Course récupérée → **Ressenti avant tout chiffre** → verdict bloc par bloc → « Mon avis (IA) » (point fort, point à améliorer, conséquence) + « Tes données » (1 principal, 2 secondaires). **Cas limite** : Course pas encore remontée → Séance « en attente de la course ».
- **UJ-4** Maladie : « Empêchement » → « Malade » (deux taps) → au-dessus / en dessous du cou → proposition (footing léger, ou repos + décalage) avec raison → confirmation d'un tap → Plan republié.

## 7. Fonctionnalités et écrans (FR)

### 7.1 Profil coureur / onboarding (PRD §5.1)
- **FR-1 Import initial** : historique des Courses, records, charge, CTL, FC au repos depuis intervals.icu ; progression affichée.
- **FR-2 Complément et validation** — champs du formulaire : **blessures actuelles et passées, disponibilités par jour, jour de la sortie longue, périodes d'indisponibilité** ; correction des valeurs importées. **Aucun Plan avant validation une fois.** Profil modifiable ensuite.
- **FR-3 Profil vivant** : intègre chaque Course, Ressenti, Bilan de fin de Plan. Emplacements nutrition et sommeil vides en v1 (pas d'UI attendue).
- On vise un Profil **complet, pas un démarrage rapide** (SM-C3 : ne pas raccourcir UJ-1). (PRD §5.1, §10)
- Le digest recommande des « questions de clarification obligatoires » à la création du plan (objectif, historique, blessures, disponibilités). (DIG §4 garde-fou 9)

### 7.2 Objectif et Plan (PRD §5.2)
- **FR-4 Saisie de l'Objectif** — entrées : **chrono + distance + date obligatoire** (Épreuve ou date cible) ; **Objectifs secondaires en texte libre**. Date < 4 semaines refusée. Si chrono irréaliste (équivalence type VDOT sur records 6 mois) : avertissement en une ligne + chrono jugé réaliste ; **non bloquant** (l'utilisateur peut maintenir).
- **FR-5 Génération** : entrées = Profil (+ dernier Bilan) + Objectif ; Méthode choisie par l'app (liste fermée) + raison en une ligne. Garde-fous déterministes (≤ +10 %/semaine, semaine allégée toutes les 3–4 semaines, aucune Séance > +10 % de la plus longue sur 30 j glissants, bande d'intensité de la Méthode, respect disponibilités / sortie longue / blessures). Jusqu'à 3 tentatives invisibles ; après 3 échecs : **échec explicite qui nomme le garde-fou violé**, propose de modifier l'Objectif, ne publie rien. **Progression semaine par semaine lisible avant acceptation.**
- **FR-6 Relecture** : accepter le Plan entier, ou refuser avec Consigne → régénération. **Aucune modification manuelle d'une Séance.**
- **FR-7 Cycle de vie** : accepter = *actif* + Publication ; nouvel Objectif possible pendant un Plan actif/suspendu → **confirmation** puis annulation de l'ancien et retrait de ses Séances à venir ; *terminé* le lendemain de la dernière Séance → l'accueil propose un nouvel Objectif.
- **FR-8 Bilan de fin de Plan** : volume réalisé vs prévu, Conformité moyenne, Séances manquées, évolution CTL et records, répartition des Ressentis ; **consultable**.

### 7.3 Publication et état de la synchro (PRD §5.3)
- Publications rares et volontaires : acceptation d'un Plan, confirmation d'un Ajustement, bouton « resynchroniser ». Lectures à chaque ouverture.
- **FR-9 Publication** : un Bloc = une cible ; allure pour fractionné, tempo, seuil, allure de course ; FC pour échauffement, récupération, retour au calme, footing, sortie longue en endurance. Pas de doublon.
- **FR-10 État de la synchro** : dernière Séance publiée, dernière Course reçue, statut ✅ / ⚠️, bouton « resynchroniser » ; échec signalé **avec sa cause**, sans perte du Plan.

### 7.4 Feedback après chaque Course (PRD §5.4)
- **FR-11 Détection et rattachement** : à l'ouverture et sur **rafraîchissement manuel**, **sans notification**. Rattachement à la Séance du jour ; plusieurs Courses → la plus longue, les autres non planifiées. Course la veille/le lendemain d'une Séance non réalisée → **rattachement proposé, confirmé d'un tap**. Sinon Séance « en attente de la course ».
- **FR-12 Ressenti** : une seule question, 3 emojis, **avant tout chiffre** ; Feedback affiché seulement après.
- **FR-13 Conformité bloc par bloc** : verdict d'une ligne par Bloc + note globale (% de Blocs respectés) ; allure ±5 s/km, FC ≥ 80 % du temps en zone ; Séance « respectée » si ≥ 80 %. Si laps ≠ Blocs ou Course non planifiée : **Feedback global, sans verdict par Bloc, et l'app le dit.**
- **FR-14 Feedback en deux couches** visuellement séparées : « Mon avis (IA) » (3 niveaux : Séance, rôle dans le Plan, Objectif ; **exactement** 1 point fort, 1 point à améliorer, 1 conséquence ; si non respectée : ce qui a manqué et comment faire mieux) ; « Tes données » (**1 chiffre principal + 2 secondaires**, règle déterministe par type de Séance ; détail complet dans intervals.icu). Validation des nombres, 3 tentatives, sinon « avis indisponible » (seuls « Tes données » et Conformité). **Aucun compliment sans chiffre.**
- Recommandation digest : niveau de confiance et citation des métriques dans l'interprétation IA. (DIG §4 garde-fou 7 — non repris par le PRD)

### 7.5 Matrice et surmenage (PRD §5.5)
- **FR-15 État du jour** — 6 cases (libellés à affiner en UX, PRD §13 arb. 27) :
  - Respectée : 😌 « marge, en progrès » · 😐 « dans le Plan » · 🥵 « à la limite, à surveiller »
  - Non respectée : 😌 « Séance mal calibrée ou trop prudente » · 😐 « exécution à revoir » · 🥵 « fatigue probable »
- **FR-16 Seuil de surmenage** : 3 Courses 🥵 d'affilée (non planifiées comprises) **et** FC au repos ≥ +5 bpm vs moyenne 7 j → proposition d'Ajustement avec pourquoi en une ligne. Sous le seuil, le Plan ne bouge jamais.

### 7.6 Empêchements et Séances manquées (PRD §5.6)
- **FR-17 Déclarer un Imprévu** : deux taps (« Empêchement » → type), **une question au plus**, confirmation d'un tap.
  - *dernière minute* : annuler ou déplacer ;
  - *malade* : au-dessus / en dessous du cou (rhume léger → footing ; fièvre → repos) ;
  - *blessé* : suspendre ou annuler ; reprise **uniquement** via bouton « Reprendre » touché par l'utilisateur.
- **FR-18 Séance manquée** : passe à *manquée* le lendemain ; le Plan ne bouge pas ; prise en compte dans Feedback suivant et Bilan.
- **FR-19 Ajustement confirmé** : proposé avec pourquoi en une ligne, appliqué après confirmation, puis republié ; refuser = Plan inchangé ; tout Plan modifié repasse les garde-fous.
- Digest : garde-fou médical « consulter un professionnel » pour douleur/blessure, sans diagnostic. (DIG §4 garde-fou 8 — non repris explicitement par le PRD)

### 7.7 Accueil et Séances (PRD §5.7)
- **FR-20 Séance du jour** : carte unique (but, durée, état) + accès au Plan, aux Feedbacks récents, à l'état de la synchro. Sans Plan actif : proposer de définir un Objectif. Minimaliste, **aucune notification**.
- **FR-21 Détail d'une Séance** : but dans le Plan en une ligne + liste des Blocs (durée ou distance, cible).
- Référence de clarté : **Runna** (accueil, présentation des Séances, commentaires). (PRD §5.7, §7 ; BRF « Pourquoi ne pas utiliser Runna »)

### 7.8 Écrans implicites à prévoir (déduits)
- Coût IA « mesuré et consultable » (PRD NFR-4) → un endroit pour le consulter.
- États d'attente génération Plan (< 2 min) et Feedback (< 30 s) (PRD NFR-9).
- Confirmation de remplacement d'un Plan actif (PRD FR-7).
- Plan *suspendu* avec bouton « Reprendre » (PRD FR-17).
- Accueil après Plan *terminé* → nouvel Objectif + accès au Bilan (PRD FR-7, FR-8).
- Tolérances et Seuil « paramètres ajustables » (PRD FR-13, FR-16) : pas d'exigence d'UI explicite.

## 8. Notifications

- **v1 : zéro notification.** Détection des Courses à l'ouverture et sur rafraîchissement. (PRD §5.7, §7, FR-11, §13 arb. 6 ; RDM v1 étape 8)
- **v1.1 : trois notifications ciblées** (les seules autorisées) : séance du jour pas envoyée sur la montre ; modification du plan en attente de confirmation ; signal de surmenage. (RDM v1.1 ; PRD §9.2)

## 9. Ton et voix (PRD §7)

- Motivant et réaliste, **jamais flatteur**. Concret : chaque affirmation cite une donnée mesurée ou ressentie (ex. « les 3 derniers fractionnés à 4'12 au lieu de 4'20 prévus »).
- Clair immédiatement, **sans jargon non expliqué** ; données digestes pour que l'utilisateur se forge son avis.
- Pas d'intrusion : aucune notification, quasiment aucune manipulation au quotidien.
- Tutoiement implicite (« Tes données », « Mon avis (IA) »). (PRD FR-14)
- Langue : français (toutes les sources).

## 10. Non-objectifs (PRD §8 ; BRF-add « Alternatives écartées »)

- Pas d'éditeur de séances ; pas de réajustement continu après chaque Course ; pas de tableau de bord d'analyse (détail dans intervals.icu).
- Pas de Strava, pas de lecture directe HealthKit / WorkoutKit.
- Pas de choix manuel de la Méthode ; pas de version « minimum vital » des Séances (rejet aussi en RDM v3+).
- Pas de produit commercial : pas de comptes, paiement, conformité réglementaire.
- Pas d'app iOS native.

## 11. Périmètre MVP vs plus tard (PRD §9)

- **MVP** : FR-1 à FR-21. Livraison en **squelette vertical** : Plan de 4 semaines, un Objectif, publié, couru, commenté, avec FR-11 à FR-14 dès le squelette. (PRD §9.1 ; PTY §7)
- **v1.1** : jauge de faisabilité continue, alerte « pas assez ambitieux », prédiction de chrono en fourchette, rappel du « vrai pourquoi », échauffements et étirements par Séance, vue « escalier », trois notifications ciblées.
- **v1.2** : saison de Plans enchaînés, conversation IA rattachée au Feedback (avant/après avant confirmation, RDM v1.2), retour de blessure progressif, signaux de surmenage supplémentaires.
- **v2** : 5 autres types d'Objectif (distance, durée, récupération après Épreuve, condition générale, reprise après blessure), transport sans ouvrir HealthFit, renforcement, empêchements « moins de temps » et « voyage », carnet de blessures, chaussures, parcours, graphiques enrichis. (RDM v2 ajoute : rapport kiné, journal de bord, simulation de course, tableau de bord personnalisable.)
- **v3+** : nutrition, sommeil, calendrier, météo ; partage. (RDM v3+ ajoute types maintien / trail / régularité.)

## 12. Critères de succès touchant l'UX (PRD §10)

- SM-1 boucle fermée sans saisie dans intervals.icu ; SM-3 Plan suivant ≤ 10 min de manipulation active ; SM-5 aucun chiffre faux ; SM-6 la raison du Plan 2 fait référence au Bilan du Plan 1 (**à rendre visible**).
- SM-8 : un jour de course, manipulation = ouvrir HealthFit, ouvrir l'app, poser un Ressenti.
- Contre-métriques : SM-C1 (peu d'Ajustements), **SM-C2 le format du Feedback est un plafond**, SM-C3 (ne pas raccourcir UJ-1).

## 13. Questions ouvertes

- **À fixer en UX** : table type de Séance → chiffres de « Tes données » ; libellés de la Matrice. (PRD §12 point 5, §13 arb. 26–27)
- Seuil d'écart du chrono irréaliste (architecture). (PRD §12.2 ; PRD-add §2)
- Une cible par Bloc sur la montre ? (PRD §12.1 ; PRD-add §5)
- Saison de Plans enchaînés = « vrai moteur », v1 n'en livre que la mémoire et le Bilan. (PRD §12.6)
- Titre de l'app : « Titre de travail, à confirmer ». (PRD en-tête)
- Horizon de synchro HealthFit non documenté ; HRV/FC au repos Apple Watch dépendent du suivi du sommeil (impact sur FR-16 si données `null`). (DIG §2, §5)

## 14. Contradictions entre sources

1. **Types d'Objectif** : 6 types en v1 (BRF « Périmètre » ; BRF-add ; RDM v1) vs **1 seul** (« chrono sur une distance ») en v1 (PRD FR-4, §13 arb. 1 ; suggéré par PTY §3.3).
2. **« Tes données »** : 3 à 5 indicateurs (BRF ; BRF-add ; RDM) vs **1 principal + 2 secondaires** (PRD FR-14, arb. 17 ; PTY §5).
3. **Moment du Ressenti** : « pendant le chargement du feedback », à l'ouverture qui suit la détection (BRF-add ; RDM) ; PTY §5 recommande de pouvoir le saisir **avant l'arrivée de la Course** (à chaud, Ressenti sans Course associée) ; **PRD tranche** : dès la récupération de la Course, ni avant, ni reporté (arb. 5, FR-12).
4. **Préremplissage du Profil** : BRF dit que blessures, disponibilités et jour de sortie longue sont préremplis par import (et import depuis « intervals.icu, HealthFit ») ; PTY §3.4 et PRD FR-1/FR-2 : import (records, charge, CTL, FC repos) via intervals.icu seulement + **formulaire** pour le reste.
5. **Réajustement** : RDM v1 étape 6 « réajustement » via la Matrice ; PRD : Matrice en lecture seule, Ajustement uniquement sur Imprévu ou Seuil de surmenage (FR-15, FR-16, FR-18, §8).
6. **Blessé** : « suspendre jusqu'au rétablissement » (BRF-add) vs reprise uniquement par bouton « Reprendre » manuel (PRD FR-17, arb. 16).
7. **Fiabilité de la synchro** : « pas un critère de réussite » (BRF-add) vs critère secondaire SM-7 (PRD §10, arb. 12) ; l'écran reste MUST partout.
8. **Alerte objectif** : RDM v1.1 regroupe « trop difficile / pas assez ambitieux » ; PRD avance l'alerte chrono irréaliste en v1 (FR-4, arb. 3), garde « pas assez ambitieux » en v1.1.
9. **Libellés du Feedback** : DIG §4 propose « Mesuré » / « Interprétation IA » avec niveau de confiance ; PRD retient « Tes données » / « Mon avis (IA) », sans niveau de confiance.
10. **Garde-fous non repris par le PRD** : DIG §4 propose limite de 2–3 séances qualitatives/semaine, jamais deux dures d'affilée, garde-fou médical « consulter un professionnel », journal des décisions. Absents des FR.
11. **Vocabulaire** : « vacances » (RDM) vs « périodes d'indisponibilité » (PRD) ; « récupération après une course » (BRF-add) vs « après une Épreuve » (PRD §9.2) — utiliser le glossaire PRD.
12. **Synchro « aller/retour automatique »** (RDM v1 étape 4) vs Publications « rares et volontaires » (PRD §5.3 ; BRF).
