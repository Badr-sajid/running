---
title: "Réconciliation : brief du 18/11 → PRD du 20/11"
created: 2026-11-20
sources:
  - ../../briefs/brief-running-2026-11-18/brief.md
  - ../../briefs/brief-running-2026-11-18/addendum.md
  - ../../briefs/brief-running-2026-11-18/.memlog.md
cible:
  - prd.md
  - addendum.md
---

# Réconciliation brief → PRD

Périmètre : ce qui, dans le brief (et son addendum et son memlog), manque, est affaibli ou contredit dans le PRD et son addendum. Ne sont **pas** retenus comme écarts : les arbitrages validés du PRD §11 (un seul type d'Objectif, Ressenti dès la récupération de la Course, HealthFit lu via intervals.icu, porte ouverte au partage, SM-7 secondaire, etc.), les éléments reportés au §8.2 et les non-objectifs du §7.

Classement : **[Majeur]** influence l'architecture, l'UX ou un critère de succès ; **[Moyen]** perte de sens ou de traçabilité ; **[Mineur]** nuance.

---

## A. Principes, ton, intention (qualitatif)

### A1. [Majeur] Les principes directeurs ont disparu en tant que section d'arbitrage
- **Source (brief, Principes directeurs)** : « Ces principes tranchent les arbitrages du PRD et de l'architecture. » Sept principes suivent.
- **PRD** : aucune section « Principes ». Seuls survivent le pilote automatique transparent (§1, NFR-5), le ton (§6), la confidentialité (NFR-2) et « aucune modification sans confirmation » (FR-16). Or l'addendum §5 envoie le PRD à l'UX puis à l'architecture : ces étapes n'ont plus de grille d'arbitrage explicite.
- **Correction suggérée** : ajouter un §1.1 « Principes directeurs » qui reprend les sept principes tels quels (une ligne chacun) et précise qu'ils tranchent les arbitrages UX et architecture. Renvoyer chaque principe vers les FR/NFR qui l'incarnent.

### A2. [Majeur] « Simple et efficace d'abord » et l'objectif d'apprentissage sont absents
- **Source (brief)** : « Le projet sert aussi à apprendre des technologies modernes, mais quand une approche simple qui marche s'y oppose, l'application gagne. » **Memlog** : « double objectif = l'app de course + montée en compétence (data engineer vers modern data stack) » ; « la montée en compétence est un bénéfice secondaire, jamais un motif de complexité ».
- **PRD** : absent. Le JTBD « Bâtisseur » (§2.1) et la phrase « un outil qu'on construit soi-même, et bien » (§1) parlent de possession, pas d'apprentissage, et n'énoncent pas la règle de priorité. Combiné avec NFR-6 (porte ouverte au partage), le PRD pousse plutôt vers plus d'ambition technique, sans garde-fou contre la sur-ingénierie.
- **Correction suggérée** : ajouter le principe dans A1, et une NFR « Simplicité » : à fonctionnalité égale, l'architecture retient la solution la plus simple ; l'apprentissage d'une technologie ne justifie jamais à lui seul un composant. Préciser que NFR-6 n'autorise pas de travail anticipé au-delà de « ne pas figer d'identifiant ».

### A3. [Majeur] « La confiance avant les fonctionnalités » est absent
- **Source (brief)** : « Une synchro fiable et des recommandations crédibles comptent plus qu'une fonctionnalité de plus. »
- **PRD** : absent comme principe. La crédibilité est bien couverte (FR-12, NFR-3, SM-5), mais la règle de priorisation (fiabiliser avant d'ajouter) ne l'est pas. Avec SM-7 désormais non bloquant (arbitrage §11.12, légitime), plus rien n'empêche d'élargir le périmètre avant que la synchro soit fiable.
- **Correction suggérée** : l'intégrer dans A1 et l'appliquer au §8.1 : « on n'élargit pas après le squelette vertical tant que la synchro et le Feedback ne sont pas fiables en conditions réelles ».

### A4. [Moyen] « Le stress et la récupération se dosent ; chaque séance part de l'état réel du jour »
- **Source (brief)** : « La prévention des blessures est une contrainte de base, et chaque séance part de l'état réel du jour. »
- **PRD** : la prévention est bien traduite en garde-fous (FR-5). En revanche, « chaque séance part de l'état réel du jour » n'apparaît nulle part : la Matrice (FR-13) lit la forme mais n'agit que dans le Feedback, et le Plan ne bouge qu'au Seuil de surmenage. Le principe est donc soit abandonné, soit réinterprété sans le dire.
- **Correction suggérée** : expliciter l'interprétation retenue, par exemple : « l'état réel du jour est pris en compte à la génération (Profil vivant) et au Seuil de surmenage ; la carte Séance du jour (FR-17) rappelle la lecture de forme de la dernière Course ». Sinon, noter l'abandon dans §11.

### A5. [Moyen] La métaphore de continuité et la vision long terme sont perdues
- **Source (brief)** : « Je veux que mes plans soient les chapitres d'une même histoire. » ; Vision : « l'application devient mon coach unique, avec une seule histoire d'entraînement ».
- **PRD** : la Vision (§1) ne garde que la boucle v1 et ajoute le partage. La continuité est réduite à FR-3 et à une `[NOTE FOR PM]`. Or c'est le « vrai moteur » selon le brief : sans ce cap, l'UX risque de traiter chaque Plan comme un objet isolé.
- **Correction suggérée** : ajouter au §1 une phrase de vision long terme (« coach unique, une seule histoire d'entraînement, chaque Plan est un chapitre ») et en tirer une consigne UX : le Plan courant se présente comme la suite du précédent (rappel de son point de départ).

### A6. [Mineur] La raison d'être de la séparation des deux couches a disparu
- **Source (brief)** : « L'avis de l'IA reste séparé des données brutes, **pour que je garde mon propre jugement**. »
- **PRD** : FR-12 impose la séparation sans en donner la raison. Le « motivant » du ton (§6) n'est relié à aucune FR.
- **Correction suggérée** : ajouter la raison à la description du §4.4, et relier §6 à FR-12 (« le ton de « Mon avis (IA) » suit le §6 »).

---

## B. Spécifications affaiblies ou contredites

### B1. [Majeur] « Tes données » : 3 à 5 indicateurs dans le brief, exactement 3 dans le PRD, sans arbitrage
- **Source (brief + addendum)** : « « Tes données » (3 à 5 indicateurs clés) ».
- **PRD** : FR-12 « 1 chiffre principal et 2 secondaires » ; SM-C2 en fait un **plafond** (« le format 1 + 1 + 1 / 1 + 2 est un plafond »). Ce resserrement ne figure pas dans les arbitrages du §11. Il vient sans doute de la revue party mode, mais il contredit une spécification marquée « validée lors du brainstorming ».
- **Correction suggérée** : soit ajouter un arbitrage n°17 au §11 (« Tes données ramené à 1 + 2, contre 3 à 5 dans le brief »), soit revenir à « 3 à 5 indicateurs dont 1 principal ». Faire confirmer par Badr.

### B2. [Majeur] Signaux de surmenage et calibration du seuil abandonnés
- **Source (addendum du brief)** : « Signaux de surmenage à capter pour calibrer la matrice : douleurs ; récupération plus longue ; effort perçu en hausse à charge égale ; jambes lourdes ; baisse de motivation. La plupart de ces signaux sont subjectifs, d'où le poids de l'emoji. » ; « Le seuil exact est à calibrer dans le PRD. » **Brief (risques)** : « Les seuils de la matrice : à calibrer pour éviter les propositions trop fréquentes ou les alertes inutiles. »
- **PRD** : FR-14 fige le seuil candidat (3 🥵 + FC au repos en hausse, §11.9). Aucun des cinq signaux n'est capté ni stocké ; « FC au repos en hausse » n'est pas quantifié (sur quelle fenêtre, de combien ?) ; aucun mécanisme de calibration. SM-C1 surveille l'excès d'Ajustements mais pas les alertes manquées. Le seuil 3 🥵 est par ailleurs peu sensible pour un coureur à 3 ou 4 sorties par semaine (au moins une semaine de 🥵).
- **Correction suggérée** : (1) quantifier « FC au repos en hausse » (ex. moyenne 7 j > moyenne 28 j + X bpm) dans FR-14 ou en question ouverte ; (2) prévoir dans FR-3 un emplacement pour au moins « douleur » (oui/non, zone), signal de sécurité le plus important ; (3) ajouter une question ouverte « calibration du Seuil après le squelette vertical » et journaliser les déclenchements et quasi-déclenchements.

### B3. [Majeur] Contradiction interne sur le « bilan du Plan précédent »
- **Source (brief)** : « la mémoire en v1 (le plan suivant part de ma forme réelle), la saison de plans enchaînés en v1.2, **avec le bilan de chaque plan** » ; memlog : « la saison de plans enchaînés + bilan de fin de plan reste en SHOULD (v1.2) ».
- **PRD** : FR-3 intègre « le bilan du Plan précédent » et SM-6 (bloquant) exige que « le Plan 2 part[e] réellement des données et du bilan du Plan 1 », mais §8.2 range « bilan de fin de Plan » en v1.2. Aucune FR ne définit ce qu'est un bilan ni qui le produit.
- **Correction suggérée** : trancher. Soit définir en v1 un bilan minimal déterministe (Conformité moyenne, distribution des Ressentis, Objectif atteint ou non), ajouter « Bilan » au glossaire et retirer « bilan » du §8.2 ; soit retirer « bilan » de FR-3 et de SM-6 (la continuité v1 = données réelles seulement), conformément au brief.

### B4. [Moyen] Contrainte « HealthFit seule source, Strava non connecté à intervals.icu » et déduplication des Courses absentes
- **Source (brief, Forme et dépendances)** : « HealthFit … est la seule source des courses. Strava n'est pas connecté à intervals.icu, pour éviter les doublons et les activités que l'API ne renvoie pas. » ; risque « sans doublons ».
- **PRD** : le §7 écarte Strava côté application, mais ne dit pas que Strava ne doit pas être branché à intervals.icu. FR-9 rattache la Course « à la Séance prévue le même jour » sans traiter le cas de deux Courses le même jour ou d'un doublon. L'addendum du PRD ne reprend pas la note « l'API n'expose pas les activités venues de Strava ».
- **Correction suggérée** : ajouter à NFR-7 la configuration prérequise (« Strava non connecté à intervals.icu »), et à FR-9 une règle pour plusieurs Courses le même jour (rattacher la plus proche de la Séance en durée, signaler les autres comme non planifiées). Reporter la note API dans l'addendum §1.

### B5. [Moyen] L'évaluation « annuler ou régénérer » de l'Empêchement est implicite
- **Source (addendum du brief)** : « **Dans tous les cas**, l'application évalue s'il faut régénérer le plan ou seulement annuler une ou plusieurs séances. » ; brief : « l'application évalue s'il suffit d'annuler ou de déplacer une séance, ou s'il faut régénérer le plan ».
- **PRD** : FR-15 liste les options par cas, et §4.6 dit que l'IA ne sert « que pour régénérer un Plan », mais aucune règle ne dit **quand** l'arbre déterministe bascule vers une régénération (ex. Imprévu > N jours, Séance clé manquée, reprise après suspension).
- **Correction suggérée** : ajouter à FR-15 ou FR-16 un critère de bascule vers la régénération, ou le lister en question ouverte §10.

### B6. [Mineur] « Pendant le chargement du feedback »
- **Source (addendum du brief)** : la question de Ressenti est posée « pendant le chargement du feedback et avant tout chiffre ».
- **PRD** : FR-10 garde « avant tout chiffre », mais pas l'idée que le Ressenti masque le temps de calcul. Si le Feedback LLM dépend du Ressenti (Matrice), il ne peut se calculer qu'après : l'intention UX (zéro attente perçue) est à reformuler.
- **Correction suggérée** : préciser dans FR-10 que les calculs déterministes démarrent dès la récupération de la Course, et que seule la rédaction de l'avis attend le Ressenti. Indiquer l'attente maximale acceptable.

---

## C. Risques et inconnues non repris

### C1. [Moyen] Pas de section risques ; deux risques du brief n'ont plus de propriétaire
- **Source (brief, Inconnues et risques)** : sept risques, dont « Les seuils de la matrice … à calibrer » et « Le coût réel de l'IA : inconnu tant que la génération et le feedback n'ont pas été mesurés ».
- **PRD** : transport et remontée bloc par bloc sont déclarés vérifiés (addendum §3) ; crédibilité, qualité du plan et dépendances sont couverts par FR-12, SM-5 et NFR-7. En revanche, la calibration des seuils (voir B2) et le coût réel (NFR-4 dit seulement « mesurable », sur une base supposée de 15 Feedbacks par mois avec possibles régénérations en boucle par les validateurs de FR-5 et FR-12) n'ont aucun suivi.
- **Correction suggérée** : ajouter un §10 bis « Risques » court, ou compléter §10 : calibration du Seuil ; coût réel, avec un plafond de régénérations par validateur (ex. 3 tentatives, puis erreur visible) pour borner le coût.

### C2. [Mineur] Classement des douleurs non repris
- **Source (brief)** : « Par ordre de gêne : 1. la friction ; 2. la continuité ; 3. le feedback. »
- **PRD** : le §1 cite friction et continuité, sans ordre. Utile pour prioriser les epics.
- **Correction suggérée** : une ligne au §1 ou au §8.1 qui rappelle l'ordre, pour guider le découpage en epics.
