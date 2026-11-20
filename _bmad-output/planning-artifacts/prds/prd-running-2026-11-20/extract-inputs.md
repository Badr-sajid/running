---
title: "Extraction des entrées pour le PRD : application de course à pied personnelle"
created: 2026-11-20
sources:
  - B = planning-artifacts/briefs/brief-running-2026-11-18/brief.md
  - A = planning-artifacts/briefs/brief-running-2026-11-18/addendum.md
  - BM = planning-artifacts/briefs/brief-running-2026-11-18/.memlog.md
  - PM = party-mode/revue-brief-2026-11-20.md
  - BI = brainstorming/brainstorm-app-running-perso-2026-11-17/brainstorm-intent.md
  - R = brainstorming/brainstorm-app-running-perso-2026-11-17/roadmap.md
  - BRM = brainstorming/brainstorm-app-running-perso-2026-11-17/.memlog.md
---

# Extraction des entrées pour le PRD

Légende des sources : **B** brief, **A** addendum, **BM** memlog du brief, **PM** revue party mode (2026-11-20), **BI** brainstorm-intent, **R** roadmap, **BRM** memlog du brainstorming. Les contradictions sont signalées par **⚠ CONTRADICTION**.

---

## 1. Vision & problème

- **Problème** : chaque plan d'entraînement coûte **~1 h 30** : ~1 h de conversation IA (prompt dédié + re-partage du contexte : niveau, historique, objectifs, contraintes) + ~30 min de traduction au format intervals.icu et saisie manuelle de chaque séance (B, BM). Coût repayé à chaque nouveau plan et à chaque grand imprévu (maladie, blessure). Strava ne dit ni si la séance a été respectée ni ce qu'elle révèle de la forme (B).
- **Le problème n'est pas la qualité des plans** (jugés « presque toujours cohérents » après relecture) mais le fait que chacun repart de zéro et passe par une saisie manuelle (B, BM).
- **Douleurs, par ordre de gêne** : 1) friction (1 h 30 répétitive), 2) continuité (aucun plan ne tient compte du précédent ni de l'historique), 3) feedback (aucun verdict structuré après une course) (B).
- **Solution** : trois moments reliés par un **profil vivant** qui s'enrichit : (1) créer un plan, (2) après chaque course, (3) gérer un imprévu (B).
- **Vision long terme** : coach unique, une seule histoire d'entraînement ; saisons de plans enchaînés (préparation → course → récupération → préparation suivante) avec bilan du plan précédent ; coach avec qui discuter qui retient ce qui compte ; le profil accueille ensuite nutrition et sommeil. « L'application décide et explique, je confirme » (B).
- **Pourquoi / motivation** : le prix de Runna (~120 $/an) a déclenché le projet (« s'il était gratuit, je n'aurais pas lancé ce projet ») ; la continuité en est le vrai moteur (B, BM). PM ajoute : **Badr veut construire son propre outil, bien construit**, en plus du gain de temps ; un chat + serveur MCP intervals.icu peut servir de prototype mais pas de produit — le « pourquoi » du PRD doit le dire franchement (PM).
- Double objectif : l'app + montée en compétence (data engineer → modern data stack), mais **l'application prime** (BM).

## 2. Utilisateur & contexte

- **Utilisateur unique** : Badr, coureur et développeur. Ni comptes, ni multi-utilisateur, ni conformité au-delà de ses propres données (B, BI).
- **Matériel / outils** : Apple Watch **Series 10**, **HealthFit** (iPhone), **intervals.icu** (B, BI). iPhone (PWA cible) et Mac (dev / exécution locale) (B).
- **Routine actuelle** : conversation IA → plan + prompt → collé à la main dans intervals.icu → synchro montre → résultats consultés sur **Strava** (BRM, BI).
- **Strava** : à abandonner pour l'analyse ; **non connecté à intervals.icu** (doublons, activités Strava non exposées par l'API) (B, A).
- **Runna** : référence appréciée (clarté de l'accueil, parcours de génération du plan, présentation des séances, commentaires générés) ; frustrations : prix, trop peu de graphiques par course, pas de nutrition, impossible d'enchaîner les plans, nouveau plan ignorant l'historique (BI, BRM, B).
- **Rythme** : 3 à 4 sorties par semaine (PM, implicite dans le calcul du seuil) ; ~1 génération de plan/mois et ~15 feedbacks/mois (PM, hypothèse coût).
- **Prochain objectif de course** : marathon début avril 2027 en moins de 3 h 30 — **peut changer et ne doit pas orienter le périmètre** (PM). La prépa marathon sera faite avec l'ancienne méthode et servira de plan de référence au test de qualité (PM).
- **Contexte projet** : pas d'échéance ; priorité à la solidité, pas de suite de versions instables (PM, BM).

## 3. Capacités envisagées pour la v1

### 3.1 Profil vivant (colonne vertébrale)
- Import de l'historique depuis intervals.icu et HealthFit à la première utilisation ; préremplissage : records, charge, forme (CTL), blessures, disponibilités, jour de la sortie longue (B, BI, R).
- Validation / correction par l'utilisateur ; viser un **profil complet, pas un démarrage rapide** ; la première synchro peut prendre du temps (B, BM).
- **Profil = import (records, charge, CTL) + formulaire** pour blessures et disponibilités, qui ne viennent pas des données importées (PM). ⚠ voir §8.
- Champs listés : historique, records personnels, charge actuelle, CTL/forme, blessures (actuelles et passées), disponibilités, jour de sortie longue, vacances / périodes d'indisponibilité (R, A).
- **Places réservées pour nutrition et sommeil** dans le modèle (vides en v1), branchables plus tard sans refonte (B, BI, R).
- Mémoire : le plan suivant part de la forme réelle (continuité « option A » en v1) (B, BM).
- Les données servent uniquement à la course (B, BI, R).

### 3.2 Objectif
- Six types : distance, durée, chrono sur une distance, récupération après une course, condition physique générale, reprise après blessure (A, BI, R). ⚠ PM recommande de réduire le nombre de types en v1 (« chrono sur une distance » candidat évident).
- Un objectif principal + des objectifs secondaires (ex. « 10 km en 45 min » + « ne pas se blesser ») (A, B).
- Entrées : historique, records, forme actuelle, blessures actuelles et passées, jours disponibles, jour de la sortie longue, périodes d'indisponibilité (A, BI).

### 3.3 Génération du plan
- Plan généré à partir des données réelles, pas d'une conversation (R, BI).
- Progression visible et logique d'une semaine à l'autre ; chrono cible réaliste (BI, R).
- Méthode choisie par l'application (80/20, Daniels, polarisé…), affichée **en une ligne avec sa raison** (B, BI, R).
- Relecture : **accepter le plan entier, ou le refuser avec une consigne en langage naturel** (« moins de volume en semaine 3 ») → régénération. Jamais de modification manuelle d'une séance (B, BM, A).
- Garde-fous de validation du plan : volume hebdomadaire +10 % max, semaine allégée toutes les 3 ou 4 semaines, répartition des intensités cohérente avec la méthode annoncée (PM).
- Squelette vertical d'abord : un objectif, un plan de **4 semaines**, publié, couru, commenté (PM).

### 3.4 Publication / synchronisation
- Une fois accepté, le plan est écrit dans le calendrier intervals.icu en séances structurées, puis atteint la montre via HealthFit, sans saisie (B).
- Republication à chaque modification confirmée ; ce sont les seules synchros après création : **rares et volontaires** (B, A).
- Lecture des courses réalisées depuis intervals.icu (B).
- Écran **« état de la synchro »** : dernière séance envoyée, dernière course reçue, statut ✅/⚠️, bouton « resynchroniser » (B, BI, R). Reste au MUST (A) — noté « [à confirmer] » dans BM.
- Republication sans doublons (upsert + external_id) (PM — voir §9).

### 3.5 Feedback après chaque course (moment 2)
- **Question de ressenti** unique, 3 emojis (😌 facile / 😐 normal / 🥵 dur), avant tout chiffre (B, A, BI, R). ⚠ moment de la question : voir §8.
- Ressenti saisissable **avant l'arrivée de la course**, puis rattaché à elle ; le modèle de données doit permettre un ressenti sans course associée (PM).
- **État d'attente** prévu quand l'app est ouverte avant que HealthFit ait exporté la course (PM).
- **Respect prévu vs réalisé bloc par bloc** (échauffement, chaque fractionné, récupération) : verdict d'une ligne par bloc + note de conformité globale (A, BI, R). Alignement prévu/réalisé index par index (un lap par bloc) (PM).
- **Trois niveaux** : la séance, l'objectif de la séance (rôle dans le plan), l'objectif global (A, BI, R).
- **Structure** : 1 point fort, 1 point à améliorer, 1 conséquence pour la suite ; si séance non respectée : ce qui a manqué + comment faire mieux (A, BI, R).
- **Deux couches visuellement séparées** : « Mon avis (IA) » et « Tes données » ; détail complet laissé à intervals.icu (B, A). ⚠ « 3 à 5 indicateurs » (B, A, BI, R) → **1 chiffre principal + 2 secondaires** (PM).
- Validateur de feedback : chaque nombre du texte IA doit exister dans les données d'entrée, sinon rejet et régénération (PM).

### 3.6 Matrice respect × ressenti / surmenage
- Lit la forme du jour (ex. « respectée + 🥵 » vs « non respectée + 😌 ») ; alimente le feedback et la détection du surmenage (A, B).
- **Ne touche pas au plan**, sauf au-delà d'un seuil de signaux concordants → déclenche le moment 3 (grand imprévu, synchro lourde) (B, A, BM).
- Seuil candidat : **trois séances 🥵 d'affilée + FC au repos en hausse** ; à calibrer dans le PRD (A, PM).
- Signaux à capter : douleurs, récupération plus longue, effort perçu en hausse à charge égale (écart ressenti vs FC/allure), jambes lourdes, baisse de motivation (A, BI, BRM).
- Toute proposition affiche son « pourquoi » en une ligne et attend confirmation (BI, R ; ex. « allégé car 🥵 hier + FC au repos +6 », BRM).

### 3.7 Bouton « Empêchement » (moment 3)
- Cas : **dernière minute** → annuler ou déplacer ; **malade** → règle au-dessus / en dessous du cou (rhume léger = footing possible, fièvre = repos) ; **blessé** → annuler le plan ou le suspendre jusqu'au rétablissement (A, BI, R).
- Déclencheurs du moment 3 : bouton Empêchement ou seuil de surmenage (B).
- L'application évalue : annuler/déplacer une ou plusieurs séances, ou régénérer le plan ; propose puis attend confirmation ; republie dans intervals.icu (B, A).
- Implémenté comme **arbre de décision déterministe** ; le LLM n'intervient que pour régénérer (PM).
- « Un imprévu, une maladie ou une blessure se gère en deux taps » (R, DoD).

### 3.8 Accueil
- Minimaliste : le plan (séance du jour) s'affiche dès l'ouverture ; chaque course et son feedback à quelques clics ; **zéro notification** (B, BI, R). Inspiré de la clarté de Runna (R).
- Idée SCAMPER : carte unique « séance du jour » (but, durée, statut sur la montre) (BRM, non tranchée).

## 4. Hors périmètre v1 / plus tard

| Horizon | Contenu | Source |
|---|---|---|
| **v1.1 (SHOULD, motivation & lisibilité)** | Jauge de faisabilité (mise à jour à chaque séance, ex. « sub-45 sur 10 km : 72 % → en hausse ») ; alerte objectif trop dur / pas assez ambitieux ; prédiction de chrono en fourchette avec tendance (ex. « 44:30 – 46:10 ») ; rappel du « vrai pourquoi » ; échauffements/étirements par séance ; vue « escalier » ; 3 notifications (séance non envoyée sur la montre, modification en attente, surmenage) | R, BI, B |
| **v1.2 (SHOULD, continuité & dialogue)** | Saison de plans enchaînés + bilan de fin de plan ; conversation IA rattachée au feedback (contexte préchargé, points mémorisés dans le profil, avant/après avant confirmation) ; retour de blessure progressif + recalcul de faisabilité | R, B, BM |
| **v2 (COULD)** | Renforcement ; empêchements « moins de temps » / « voyage » ; carnet de blessures vivant ; rapport kiné + suggestions santé ; chaussures ; parcours ; journal de bord ; simulation de course ; tableau de bord personnalisable / graphiques enrichis | R, BI, B |
| **v3+ (WON'T pour l'instant)** | Nutrition, sommeil, calendrier, météo ; objectifs maintien / trail-dénivelé / régularité | R, BI |
| **Écartés** | App iOS native ; éditeur manuel de séances ; ajustement léger après chaque course ; choix manuel de la méthode ; version « minimum vital » quand la motivation manque ; WorkoutKit/HealthKit | B, A, BI, R |

Ordre utilisateur des « en plus » : 1) échauffements/étirements → 2) renfo → 3) nutrition, sommeil, calendrier, météo (R, BRM).

## 5. Critères de succès / métriques / barres de qualité

Critères v1 (B, validés BM) :
1. **Boucle fermée en conditions réelles** : un plan complet généré, publié, couru et commenté, **sans aucune saisie manuelle dans intervals.icu**.
2. **Fin de l'ancienne routine** : plus de conversation IA ni de Strava pour planifier ou analyser.
3. **Temps** : un plan suivant ≤ **10 min de manipulation active**, contre **1 h 30** ; le temps de lecture/analyse choisi n'est pas compté. (Chiffre proposé par le coach, puis validé — BM.)
4. **Coût** : **< 10 € par mois** d'appels IA (sous ~120 $/an de Runna).
5. **Crédibilité** : **aucun chiffre faux** dans les feedbacks — rendu mesurable par le validateur de feedback (PM).

**Test de qualité du plan** : pour un même objectif, comparer un plan de l'app et un plan de l'ancienne méthode ; celui de l'app doit paraître **au moins aussi crédible**, sinon la v1 n'est pas prête (B). PM : test **à l'aveugle** (anonymiser, mélanger) ; référence = prépa marathon faite par l'ancienne méthode.

**Ajouts recommandés par PM** :
- Critère de continuité : **« le plan 2 part réellement des données et du bilan du plan 1 »**.
- Décider si la fiabilité de la synchro devient un critère explicite.
- **Définition de « fini »** : v1 terminée quand les 5 critères + le critère de continuité passent.
- **Portes de qualité** : aucune story terminée sans ses tests ; aucun appel intervals.icu sans test de contrat ; aucun chiffre du feedback sans test qui le recalcule.

Autres barres chiffrées : garde-fous du plan (+10 % de volume hebdo max, semaine allégée toutes les 3–4 semaines) (PM) ; « Tes données » = 1 + 2 chiffres (PM) ; seuil matrice = 3 × 🥵 d'affilée + FC repos en hausse ≈ une semaine de signaux à 3–4 sorties/semaine (A, PM). DoD coureur (R) : profil et objectif configurés une fois ; plan sur la montre sans toucher intervals.icu ; un emoji puis feedback clair, chiffré, crédible, sans Strava ; changements expliqués et confirmés ; imprévu géré en deux taps ; usage quotidien quasi nul.

## 6. Contraintes & préoccupations

- **Forme produit (form factor)** : **application web personnelle, conçue pour le mobile (mobile-first)**, d'abord **en local sur le Mac**, puis **auto-hébergée et ouverte sur l'iPhone en PWA**. **Pas d'app iOS native** (pas d'abonnement Apple Developer à 99 $/an ; sideload gratuit expire à 7 jours). Choix des technologies renvoyé à l'architecture (B, BM, A). ⚠ BM a d'abord noté « démarrage en outil local pour développeur, bascule possible vers une app iOS — forme finale non tranchée », puis tranché pour la PWA.
- **Hébergement** : PWA servie derrière **Tailscale** ; rien d'exposé sur Internet ; clé API côté serveur ; pas d'authentification à écrire (PM).
- **Plateforme / dépendances** : intervals.icu = pivot (écriture des séances + lecture des courses) ; HealthFit = transport vers la montre (bêta : **il faut ouvrir l'app HealthFit** pour recevoir la séance ; délai de quelques minutes à quelques heures) et **seule source des courses** (B, A). Dépendance à deux outils tiers non maîtrisés (B). Plan B transport : **Watchletic** (PM).
- **Principe amendé** : « usage quotidien quasi nul » admet l'exception de l'ouverture de HealthFit (A, BM).
- **API intervals.icu** : clé personnelle (Basic auth) ; quotas ~5 000 req/jour, 2 500 / 15 min, 10 req/s ; 429 + Retry-After (A — sources forums, à vérifier).
- **IA** : LLM appelé par API, payé à l'usage, génère plans et feedbacks (B). Crédibilité : chiffres exacts, sans flatterie ni hallucination (B, R). Principe « le LLM raconte, le code compte » (PM).
- **Coût** : < 10 €/mois ; pas d'abonnement (B). Initialement « pas de budget défini, plafond raisonnable » (BM). Estimé largement tenable, à mesurer (PM).
- **Vie privée / données** : utilisateur unique, données uniquement pour la course, aucune conformité au-delà de ses données (B, BI). Clé API jamais exposée côté client (PM).
- **Synchro** : deux rythmes ; synchro lourde rare (création du plan, grands imprévus) ; **pas de synchro légère après chaque course** (A, BM).
- **Simplicité** : « Simple et efficace d'abord » ; l'apprentissage technique ne justifie jamais la complexité (B, BM).
- **Données manquantes possibles** : FC au repos — vérifier que HealthFit l'envoie à intervals.icu (PM).

## 7. Décisions prises

### Depuis le brainstorming (BRM, 2026-11-17)
- MoSCoW validé tel que proposé (MUST/SHOULD/COULD/WON'T, cf. §3–4).
- Ressenti avant les chiffres, 3 emojis, une seule question ; question affichée pendant le chargement du feedback.
- Toute modification du plan passe par une confirmation ; « pilote automatique transparent ».
- Méthode choisie par l'app (choix manuel rejeté) ; version « minimum vital » rejetée.
- Échauffements/étirements et saison enchaînée passés en SHOULD pour une v1 légère.
- Import / préremplissage depuis HealthFit et intervals.icu validé.
- Cas d'empêchement maladie et blessure gardés.
- Les 3 indispensables : génération du plan, synchro montre, feedback après chaque course.

### Depuis le brief (BM, 2026-11-18)
- Brief = entrée pour bmad-prd puis bmad-architecture (pas de lecteur externe).
- Forme : app web mobile-first, locale puis auto-hébergée en PWA ; pas d'iOS natif.
- intervals.icu pivot ; HealthFit seule source ; Strava non connecté.
- Fiabilité de la synchro **non** élevée au rang de critère de succès (« pas besoin pour l'instant ») ; écran état de la synchro au MUST [à confirmer].
- Emoji posé à l'ouverture de l'app après détection d'une nouvelle course, **pas à chaud**.
- Synchro uniquement lourde et rare ; matrice = lecture de la forme, ne modifie qu'au-delà d'un seuil (remplace « réajustement après chaque course » du brainstorming).
- Relecture du plan : accepter / refuser + consigne ; pas d'éditeur.
- Continuité option A : mémoire en v1, saison en v1.2.
- Douleur n°2 renommée « continuité » (au lieu de « qualité du plan »).
- Critères de succès et test de qualité validés.
- Graphiques et nutrition retirés des arguments contre Runna.

### Depuis la revue party mode (PM, 2026-11-20) — changements recommandés vs brief
- **Risques transport montre et remontée par bloc : éliminés** (vérifiés en conditions réelles : séances structurées courues sans problème ; un lap par bloc prévu). Le brief les classait « à prototyper en premier ».
- Plan B transport : Watchletic.
- Profil = import + formulaire (vs « préremplissage » des blessures/disponibilités dans le brief).
- « Tes données » : 1 chiffre principal + 2 secondaires (vs 3 à 5).
- Ressenti saisissable tout de suite, à chaud, avant l'arrivée de la course (vs « pas à chaud » de BM).
- État d'attente post-course à prévoir.
- Réduire les types d'objectifs en v1 (chrono sur une distance).
- Ajouter le critère de continuité ; reconsidérer la synchro comme critère.
- Test de qualité à l'aveugle.
- Principes de conception : LLM raconte / code compte ; validateur de feedback ; plan JSON validé puis traduit ; tests de round-trip ; Empêchement en arbre de décision ; Tailscale.
- Portes de qualité, squelette vertical (4 semaines, post-course inclus dès le début), définition de « fini ».
- Séquence : PRD → UX (avant l'architecture, car le rituel post-course impacte le modèle de données) → architecture → epics/stories → test design → sprint planning → build.

## 8. Questions ouvertes / risques / points non tranchés

**À préparer pour le PRD (PM)** : le nombre de types d'objectifs en v1 ; le seuil de la matrice que Badr juge juste.

**Questions ouvertes**
- La fiabilité de la synchro devient-elle un critère de réussite explicite ? (PM vs A/BM)
- Quels types d'objectifs en v1 (6 ou sous-ensemble) ? Chaque type = son générateur et ses règles (PM).
- Seuil exact de la matrice ; calibrage pour éviter sur-réactions et alertes bruyantes (A, B, R, PM).
- La FC au repos remonte-t-elle de HealthFit vers intervals.icu ? (PM)
- Moment exact de la question de ressenti (voir contradiction ci-dessous) et modèle de données « ressenti sans course » (PM).
- Comment l'app « importe depuis HealthFit » alors qu'une PWA ne peut pas lire HealthFit directement (HealthFit est la source qui alimente intervals.icu) ? Les sources disent « intervals.icu et HealthFit » sans préciser le mécanisme (B, BI, R) — non tranché.
- Comment est détectée une « nouvelle course » (polling intervals.icu ? à l'ouverture ?) — non spécifié.
- Comportement si la course réalisée ne correspond pas à la séance prévue (nombre de laps ≠ nombre de blocs, course non planifiée) — non traité ; PM suppose un alignement index par index.
- Notifications de v1.1 : lesquelles garder (BI) ; impossibles à pousser en PWA ? (non évoqué).
- Carte « séance du jour » vs « plan de la semaine » à l'accueil (BRM, non tranché).
- Coût réel de l'IA : à mesurer (B, PM).

**Risques (B, R, PM)**
- Crédibilité de l'IA (chiffres exacts, sans flatterie).
- Qualité du plan généré (validée ou invalidée par le test comparatif).
- Dépendance à intervals.icu et HealthFit ; HealthFit import en bêta.
- Modèle de données du profil : vérifier tôt que nutrition, sommeil et saison s'y branchent sans refonte (R).
- Charge de configuration initiale acceptable (R).
- Quotas et format de l'API (sources forums, à vérifier) (A).
- Transport montre et remontée par bloc : listés comme risques dans B/R mais **éliminés** selon PM.

**⚠ CONTRADICTIONS entre documents**
1. **Synchro comme critère** : A/BM « la synchro n'est pas un critère de réussite » vs critère n°1 et principe « la confiance avant les fonctionnalités » qui reposent dessus (PM) ; BI/R/BRM faisaient de la « synchro fiable » le cœur de la confiance.
2. **Continuité** : présentée comme vrai moteur (B) mais aucun critère v1 ne la vérifie (PM).
3. **Préremplissage du profil** : B/BI/R « préremplit blessures, disponibilités » ; PM : ces champs ne viennent pas de l'import → formulaire.
4. **Six types d'objectifs** en v1 (A, BI, R, BRM) vs PM : en choisir un sous-ensemble.
5. **Moment du ressenti** : BRM/BI/R « pendant le chargement du feedback » ; BM/A « à l'ouverture qui suit la détection d'une nouvelle course, **pas à chaud** » ; PM « tout de suite, plus honnête **à chaud**, avant l'arrivée de la course ».
6. **Nombre d'indicateurs « Tes données »** : 3 à 5 (B, A, BI, R) vs 1 principal + 2 secondaires (PM).
7. **Réajustement** : BI/R/BRM « réajustement par la matrice » au MUST et synchro « automatique dans les deux sens » ; B/A/BM : pas d'ajustement après chaque course, matrice en lecture seule sauf seuil, synchros rares et volontaires. (Changement assumé et tracé dans BM.)
8. **Saison de plans enchaînés** : BRM « retenu pour la v1 » (décision initiale) puis passée en SHOULD (arbitrage MoSCoW) → v1.2 (R, B).
9. **Échauffements/étirements** : BRM « Combiner (v1) » vs SHOULD v1.1 (R, arbitrage).
10. **Mémoire** : B place la mémoire du profil en v1 ; R liste « mémoire profil » en v1.2 (il s'agit de la mémoire des conversations, mais le libellé prête à confusion).
11. **Forme produit** : BM d'abord « bascule possible vers une app iOS, non tranchée », puis décision PWA sans iOS natif (B). Résolu en faveur de la PWA.
12. **Douleur n°2** : BM « qualité du plan », renommée « continuité » dans B.
13. **Statut des risques montre/remontée** : « à prototyper en premier » (B, R) vs « éliminés » (PM).
14. **Écran état de la synchro** : MUST affirmé (A, B) mais marqué « [à confirmer] » dans BM.
15. **Usage quotidien quasi nul** (BI, R) vs nécessité d'ouvrir HealthFit (A, amendé).

## 9. Contenu technique (pour l'addendum d'architecture, pas le PRD)

- **API intervals.icu** : `POST /api/v1/athlete/{id}/events` ; `events/bulk?upsert=true` avec `external_id` pour republier sans doublons ; Basic auth par clé personnelle ; quotas ~5 000/jour, 2 500/15 min, 10 req/s, 429 + Retry-After (A, PM).
- **Format des séances** : syntaxe texte du champ `description` (format le plus sûr) ; éviter `workout_doc` (non documenté ; FIT vide signalé fin 2026) (A, PM).
- **Remontée des courses** : `GET /api/v1/activity/{id}?intervals=true` → `icu_intervals` ; un lap par bloc, alignement index par index (A, PM).
- **Pipeline plan** : LLM → JSON structuré (semaines, séances, blocs, cibles) → validateur de garde-fous → fonction déterministe de traduction vers la syntaxe `description`. Le LLM n'écrit jamais cette syntaxe (PM).
- **Pipeline feedback** : calculs déterministes (allure par bloc, écart à la cible, note de conformité, dérive cardiaque) passés comme faits au LLM ; validateur qui rejette tout nombre absent des entrées et régénère (PM).
- **Empêchement** : arbre de décision codé ; LLM seulement pour régénérer (PM).
- **Tests** : round-trip intervals.icu (envoyer, relire, comparer au JSON source) ; tests de contrat sur chaque appel ; tests de recalcul de chaque chiffre (PM).
- **Hébergement** : local Mac → auto-hébergé, PWA derrière Tailscale, clé API côté serveur, pas d'auth (B, PM).
- **Modèle de données** : profil vivant avec emplacements nutrition/sommeil ; ressenti pouvant exister sans course associée (B, PM).
- **Écartés techniquement** : WorkoutKit/HealthKit (compte dev payant, sideload 7 jours) ; Intervals Companion (cibles d'allure douteuses) (A).
- Transport alternatif : Watchletic (PM).
- Stack technologique : **non choisie**, renvoyée à l'architecture (B).

## 10. Ton / voix / ressenti qualitatif

- **Motivant et réaliste** : ni démotivant ni flatteur ; **aucun compliment sans chiffre** qui le justifie (pas de « super séance ! ») (B, BI, BRM).
- **Concret** : chaque affirmation cite une donnée mesurée ou ressentie (ex. « les 3 derniers fractionnés à 4'12 au lieu de 4'20 prévu ») (BRM, A).
- **Clair, compréhensible immédiatement** ; données digestes pour que l'utilisateur se forge son propre avis (BRM).
- **L'IA donne un avis, les données restent séparées** : « Mon avis (IA) » vs « Tes données » (B, BI).
- **Explicable** : chaque décision (méthode, plan, ajustement) expliquée en **une ligne** (ex. « allégé car 🥵 hier + FC au repos +6 ») (B, BRM).
- **Pilote automatique transparent** : l'app décide, l'utilisateur garde un droit de veto mais ne fait pas le travail (B).
- **Pas d'intrusion** : zéro notification en v1, accueil minimaliste, quasiment aucune manipulation au quotidien (B, BI, R).
- **Le ressenti avant les chiffres**, pour que les données n'influencent pas le ressenti ; le temps d'attente du chargement « sert à quelque chose » (BRM).
- **Prédictions en fourchette**, jamais un chiffre unique trompeur (v1.1) (R, BRM).
- **La confiance est le produit** : fiabilité, cohérence, crédibilité avant les fonctionnalités (R, BRM).
- **Clarté à la Runna** : accueil, présentation des séances, commentaires générés (BI, R).
- La motivation est un levier mental (le « vrai pourquoi »), pas un dosage de la séance : pas de séance au rabais (BI, R).
