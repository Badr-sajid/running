---
title: "Réconciliation PRD × recherche domaine Apple Watch / science de l'entraînement"
created: 2026-11-25
sources:
  - prd.md (final, 22/11)
  - addendum.md (20/11)
  - .memlog.md (25/11)
  - ../../research/domain-donnees-apple-watch-entrainement-course-2026-11-25/research.md
  - ../../architecture/architecture-running-2026-11-22/ARCHITECTURE-SPINE.md (impact aval seulement)
status: extraction — aucune décision prise
---

# Réconciliation PRD × recherche du 25/11

Extraction de sources pour la mise à jour du PRD. Rien n'est tranché ici : chaque écart est formulé comme une **question ouverte pour Badr / le PM**.

**Légende de la force de la preuve**
- **vérifiée** : affirmation contrôlée à la source primaire ou par une seconde source (registre de la recherche).
- **non vérifiée** : source unique, convention de coaching ou choix de conception de la recherche.
- **forum seulement** : comportement d'outil rapporté sur le forum intervals.icu.

---

## 1. Conflits

### C-1 — Garde-fou « volume hebdomadaire +10 % » classé sécurité physique
- **PRD** : FR-5 « le volume hebdomadaire augmente de 10 % au plus d'une semaine à l'autre » ; FR-22 le range dans la famille **sécurité physique** (refus infranchissable).
- **Memlog** : « Garde-fous FR-5 en deux familles… volume +10 %/sem… infranchissables » ; §14 arbitrage 2.
- **Recherche** : R1 — « Ne pas utiliser la règle des 10 % par semaine comme sécurité ; tout au plus comme garde-fou secondaire ». La variation hebdomadaire n'est pas associée aux blessures [65] ; essai GRONORUN négatif [84].
- **Preuve** : vérifiée (RUNSAFE = une seule cohorte, autocontrôlée ; GRONORUN = ECR).
- **Question** : rétrograder ce garde-fou (famille « secondaire », avertissement, ou paramètre non bloquant), ou le garder infranchissable par prudence (principe 1), en assumant qu'il n'a pas de base de preuve ?

### C-2 — Exceptions au garde-fou par sortie : accord explicite vs blocage total
- **PRD** : FR-22 « une modification qui augmente le risque et en franchit un est **refusée**… rien n'est enregistré ni publié » ; §14 arbitrage 2 « sans contournement ».
- **Memlog** : « blocage total (option 3)… Annule le 'Enregistrer quand même' de l'UX ».
- **Recherche** : R1 — « Toute exception exige ton accord explicite et un avertissement » ; viser **nettement moins de 10 %** pour les sorties longues (ex. +5 %, valeur non sourcée) [65].
- **Preuve** : vérifiée sur le risque du pic par sortie ; non vérifiée sur la valeur exacte et sur le mécanisme d'exception (choix de conception).
- **Question** : maintenir le blocage total, ou rouvrir une exception confirmée avec avertissement ? Faut-il un plafond plus strict (paramètre) pour la sortie longue que pour les autres Séances ?

### C-3 — Seuil de surmenage : une seule mesure objective, ligne de base de 7 jours, +5 bpm
- **PRD** : FR-16 « 3 Courses 🥵 d'affilée… et une FC au repos supérieure d'au moins 5 bpm à sa moyenne des 7 jours précédents » ; §13 arbitrage 9.
- **Memlog** : « Seuil de surmenage chiffré : 3 🥵 d'affilée + FC repos ≥ +5 bpm vs moyenne 7 j ».
- **Recherche** : R7 — ligne de base ≥ 4 semaines [114], moyenne glissante 7 j comparée à la variation individuelle (0,5–1 × CV) [70] ; alerte = **≥ 2 signaux objectifs** concordants + ressenti dégradé répété [31][70][76]. « +5 bpm » sans source primaire ; FC repos varie ~10 % d'un jour à l'autre, moyenner ≥ 3–4 jours [70].
- **Preuve** : vérifiée pour la structure (baseline personnelle, concordance) ; les seuils numériques ne sont étayés nulle part.
- **Question** : passer d'une valeur du jour vs 7 j à une moyenne 7 j vs baseline ≥ 28 j exprimée en CV ? Exiger un 2e signal objectif (sommeil, VFC — voir C-6) ou garder la règle actuelle comme v1 « simple » avec seuils paramétrables ?

### C-4 — Conformité des Blocs ciblés en FC (sortie longue, footing)
- **PRD** : FR-9 « tous les autres (échauffement… sortie longue en endurance) sont ciblés en FC » ; FR-13 « respecté si au moins 80 % de sa durée se passe dans la zone cible ».
- **Memlog** : « Une seule cible par Bloc : allure… fractionnés, FC pour footings » ; « Conformité : allure ±5 s/km ; FC ≥ 80 % du temps en zone ».
- **Recherche** : R8 — la note par Bloc est « une métrique propre à définir, **basée sur l'allure** » ; dérive cardiaque : « les zones FC surestiment l'intensité en fin de sortie longue » [34] ; chaleur rend FC/allure non comparables [106] ; GAP en terrain vallonné [104].
- **Preuve** : vérifiée pour la dérive (physiologie établie) ; non vérifiée pour la règle de conception R8.
- **Question** : la sortie longue ciblée en FC risque d'être jugée « non respectée » à tort en fin de sortie. Faut-il une tolérance spécifique, une correction de dérive, un jugement sur l'allure (GAP), ou accepter et documenter le biais ?

### C-5 — Règle maladie : décision sur la Séance du jour vs protocole de reprise multi-jours
- **PRD** : FR-17 « *Malade* : règle au-dessus ou en dessous du cou (rhume léger : footing possible ; fièvre : repos) » ; UJ-4 propose « footing léger » ou « repos et décaler la suite ».
- **Memlog** : aucune décision spécifique au-delà de FR-17 (draft v0).
- **Recherche** : R3 — fièvre / sous le cou : annulation, **≥ 5 jours de repos**, paliers ≥ 48 h, pas d'entraînement normal avant ~J10 ; au-dessus du cou : séance allégée ou annulée, reprise normale ~J5 ; signal d'alarme → arrêt et avis médical [86][88]. Le « neck check » n'a pas de base scientifique solide [87].
- **Preuve** : vérifiée mais non indépendante (guide BJSM écrit pour le COVID, transposition) ; « pas de reprise avant fin de fièvre » soutenu par 2 sources [86][88].
- **Question** : l'Imprévu « malade » doit-il produire un **protocole de reprise sur plusieurs jours** (repos J1–J5, paliers) plutôt qu'une décision sur la seule Séance du jour ? Faut-il afficher un renvoi médical sur signaux d'alarme ?

### C-6 — Sommeil reporté en v3 alors qu'il est un signal fiable de niveau 2
- **PRD** : §9.2 « v3 et au-delà : nutrition, sommeil » ; FR-3 « emplacements nutrition et sommeil, vides en v1 ».
- **Memlog** : aucune (hérité du brief).
- **Recherche** : R6 — hiérarchie : (2) « FC en effort stable, FC de repos, **durée de sommeil** » ; durée de sommeil fiable [6][21] ; R7 utilise la durée de sommeil dans la baseline concordante. VFC : tendance seulement, SDNN biaisé, pas validé la nuit [72][73].
- **Preuve** : vérifiée (white paper Apple + étude indépendante n=35).
- **Question** : avancer la **lecture de la durée de sommeil** (sans fonctionnalité « sommeil » dédiée) pour servir de 2e signal objectif au Seuil de surmenage (C-3) ? Ou garder v3 et accepter un détecteur à un seul signal objectif ?

### C-7 — Choix de la Méthode par le LLM et sa justification
- **PRD** : FR-5 « La Méthode est choisie par l'application dans la liste fermée et affichée avec sa raison en une ligne » ; glossaire Méthode (80/20, Daniels, polarisé) ; §13 arbitrage 29.
- **Memlog** : « méthode LLM + bandes » validé.
- **Recherche** : R5 — méthode **par défaut pyramidale, inspirée de Daniels**, variante polarisée pour volume élevé ; ligne d'explication **honnête** (« les études montrent peu d'écart… ») [46][47] ; aucun algorithme publié ne choisit une méthode d'après un profil. La distribution réalisée dérive vers le pyramidal quelle que soit la prescription [46].
- **Preuve** : vérifiée sur « peu de différence » ; non vérifiée (non significatif) sur la préférence pyramidale chez l'amateur.
- **Question** : garder le choix libre du LLM, ou fixer une Méthode par défaut avec règle déterministe de bascule ? La raison en une ligne doit-elle dire explicitement que l'écart entre Méthodes est faible ? Faut-il ajouter « pyramidal » à la liste fermée ?

### C-8 — Écriture du seuil dans intervals.icu vs « seule écriture = Publication »
- **PRD** : glossaire Publication « C'est la **seule écriture** vers intervals.icu » ; principe 6 « intervals.icu est le pivot ».
- **Memlog** : aucune décision sur les seuils.
- **Recherche** : R4 — ancrer zones et allures sur la **vitesse critique** maintenue par l'app et « **reporter la CS ou l'allure seuil dans intervals.icu**, qui ne la met pas à jour automatiquement en course » [105][108].
- **Preuve** : CS vérifiée [23][105] ; comportement d'intervals.icu **forum seulement** [108], à vérifier dans l'app (§9 de la recherche).
- **Question** : autoriser une 2e écriture (réglages de sport / seuil) vers intervals.icu, ou laisser Badr mettre à jour le seuil à la main, ou ne maintenir la CS que côté app ? À trancher après la vérification empirique [108].

### C-9 — Chrono irréaliste par VDOT sur 6 mois
- **PRD** : FR-4 « équivalence de performance calculée sur les records des 6 derniers mois » ; §13 arbitrage 22 (type VDOT).
- **Memlog** : « chrono VDOT non bloquant » validé.
- **Recherche** : coefficients VDOT relevés seulement sur un calculateur secondaire [38] (faible) ; équivalences type Riegel trop optimistes d'≥ 10 min au marathon pour la moitié des amateurs, le kilométrage hebdo améliore la prédiction [111] ; CS sur 16 semaines + découplage améliore la prédiction marathon [102][105].
- **Preuve** : non vérifiée pour [38] ; vérifiée-moyenne pour [111][102].
- **Question** : garder la VDOT (avertissement non bloquant, donc faible enjeu) ou baser l'estimation sur la CS / le volume ? Faut-il au moins ajouter une correction liée au kilométrage (déjà en « Deferred » d'architecture) ?

### C-10 — « Tes données » : 1 + 2 chiffres vs « 3–5 indicateurs » de la recherche
- **PRD** : FR-14 « 1 chiffre principal et 2 secondaires » ; §13 arbitrage 17 ; SM-C2 (plafond).
- **Memlog** : « Tes données = 1 chiffre principal + 2 secondaires (vs 3-5 du brief) ».
- **Recherche** : R6 destination « PRD (« Tes données », 3–5 indicateurs) » — repris du brief, antérieur à l'arbitrage.
- **Preuve** : aucune (référence éditoriale périmée, pas une preuve).
- **Question** : conflit apparent seulement ; confirmer que le format 1 + 2 reste, et que seule la **hiérarchie** de R6 (quels indicateurs sont éligibles) est à reprendre (voir L-4).

---

## 2. Lacunes

### L-1 — Règle de reprise quand la fenêtre de 30 jours est vide (R2)
- **PRD** : FR-5 garde-fou par sortie suppose une « plus longue Course… des 30 jours précédents » ; FR-17 « Reprendre » passe par la génération ; « retour de blessure progressif » en v1.2 (§9.2). Rien pour une fenêtre vide.
- **Recherche** : R2 — ratio non calculable (catégorie « NP ») [65] ; repartir d'un volume réduit, appliquer le facteur VDOT de Daniels (× 0,973 à 14 j, × 0,931 à 28 j…) [95], revalider la CS.
- **Preuve** : non vérifiée (convention Daniels + données Strava observationnelles [94]).
- **Question** : quel comportement de FR-5 / FR-17 « Reprendre » quand aucune Course n'existe sur 30 jours (après maladie, blessure, ou au premier Plan) ?

### L-2 — Drapeaux médicaux non diagnostiques (R3, §5.4, §5.6)
- **PRD** : FR-17 « Blessé : suspendre le Plan, ou l'annuler » ; aucun renvoi vers un professionnel.
- **Recherche** : R3 — douleur ≤ 5/10 tolérable [89] ; douleur osseuse, à la marche ou qui modifie la foulée → arrêt + orientation pro [90] ; signaux d'alarme maladie (douleur thoracique, palpitations) → avis médical [86] ; REDs : drapeau non diagnostique [96].
- **Preuve** : vérifiée-moyenne (ECR Achille [89], revue niveau IV [90], consensus CIO [96]).
- **Question** : ajouter une question « type de douleur » et un message d'orientation dans l'Imprévu « Blessé » / « Malade » ? Cela reste-t-il compatible avec « une question au plus » de FR-17 ?

### L-3 — Maintien automatique du seuil / de la CS dans le Profil vivant (R4)
- **PRD** : FR-3 Profil vivant intègre Courses, Ressenti, Bilan ; rien sur la mise à jour du seuil qui ancre toutes les cibles.
- **Recherche** : R4 + §7 constat 3 — « Le maintien automatique de la CS ou de l'allure seuil est plus important que la méthode affichée » ; un seuil périmé fausse allures, charge et Conformité [105][108].
- **Preuve** : vérifiée pour la méthode CS ; forum seulement pour l'absence de mise à jour d'intervals.icu.
- **Question** : ajouter une exigence (FR-3 ou nouvelle FR) « le seuil d'allure est réestimé automatiquement et visible dans le Profil » ? Fréquence (recherche : à chaque meilleure performance + revue 4–6 semaines, choix de conception) ? Lien avec C-8.

### L-4 — Hiérarchie des données : ce qui ne doit jamais fonder un verdict (R6)
- **PRD** : FR-14 « règle déterministe selon le type de Séance » ; table en UX ; aucune exclusion explicite.
- **Recherche** : R6 — VO2max et VFC en tendance seulement ; puissance, dynamique de foulée, stades et scores de sommeil Apple en affichage seulement ; fractionnés courts jugés sur l'allure [8][10][14][72].
- **Preuve** : vérifiée à non vérifiée selon la métrique (§1.1).
- **Question** : inscrire dans le PRD (NFR-3 ou FR-14) que puissance, VO2max, VFC, dynamique de foulée et scores Apple ne sont jamais une cible, une source de charge ni la base d'un verdict ?

### L-5 — Indicateurs de tendance et consigne « raison + sensations attendues » (R8)
- **PRD** : FR-14 « Tes données » par type ; FR-21 « son but dans le Plan, en une ligne ». Pas d'indicateur de tendance ni de « sensations attendues ».
- **Recherche** : R8 — FC à allure comparable (marqueur n° 1 [70]), découplage sur sorties longues [102], EF ; HRRc et écart ressenti/FC jamais seuls pour un verdict ; « associer chaque consigne à sa raison et aux sensations attendues » [59].
- **Preuve** : vérifiée [70] ; [59] faible (étude de cas n = 1, prépublication).
- **Question** : ajouter les « sensations attendues » au détail d'une Séance (FR-21) ? Autoriser des indicateurs de tendance parmi les chiffres éligibles de « Tes données » (en respectant le plafond 1 + 2) ?

### L-6 — Affûtage (R9)
- **PRD** : FR-5 n'impose aucune structure de fin de Plan ; seule la semaine allégée est prévue.
- **Recherche** : R9 — 2 semaines (jusqu'à 3 pour le marathon), volume −41 à −60 %, intensité et fréquence maintenues [44][56] ; R10 : à vérifier en code.
- **Preuve** : vérifiée (deux méta-analyses).
- **Question** : ajouter un garde-fou déterministe d'affûtage quand l'Objectif vise une Épreuve ? Paramètre selon la distance (AD-10 interdit les branches) ?

### L-7 — Température et pente (contexte de la Conformité)
- **PRD** : FR-13 « allure moyenne à ±5 s/km de la cible », sans correction ; météo en v3 (§9.2).
- **Recherche** : §6.3 — GAP sur terrain vallonné [104] ; « l'app doit au moins enregistrer la température » [106].
- **Preuve** : vérifiée-moyenne.
- **Question** : la Conformité d'allure doit-elle utiliser la GAP (déjà fournie par intervals.icu) ? Enregistrer la température dès la v1 sans fonctionnalité météo ?

### L-8 — Vérification empirique : contenu du FIT HealthFit (§9, check 1)
- **PRD / addendum** : addendum §5 « Vérifications en attente » ne liste que « une cible par étape » ; §3 dit le bloc par bloc « vérifié en conditions réelles ».
- **Recherche** : aucune doc du FIT (fréquence FC, puissance, tours) [15][19] ; « trou principal », test d'une soirée avant de coder.
- **Preuve** : non vérifiée (absence de documentation).
- **Question** : ajouter ce test aux vérifications en attente (addendum §5) avant les stories Feedback ?

### L-9 — Vérification empirique : comportements d'intervals.icu (§9, check 4 ; §12)
- **PRD** : §12 Q4 « Mécanismes intervals.icu : à vérifier en architecture » (publication, quotas seulement).
- **Recherche** : formule de charge allure [39], Form le jour même [41], zones de Form [64], bandes de conformité [97], absence de mise à jour du seuil [108] — tous périmés ou forum.
- **Preuve** : forum seulement.
- **Question** : étendre §12 Q4 à ces points, au moins [108] qui conditionne C-8 / L-3 ?

### L-10 — Vérification empirique : valeur de VFC envoyée par HealthFit (§9, check 2)
- **PRD** : la VFC n'est utilisée nulle part.
- **Recherche** : moyenne de la journée ou nocturne ? [15] ; alternative IntervalsWellnessSync [20] (faible).
- **Preuve** : forum seulement.
- **Question** : sans objet tant que la VFC n'entre pas dans le Seuil de surmenage ; à lancer seulement si C-3 retient la VFC comme 2e signal.

### L-11 — Calibrage des seuils sur l'historique (§9, question « seuils du détecteur »)
- **PRD** : §11 « recalibrage en v1.2 » ; FR-16 « valeurs… ajustables ».
- **Recherche** : calibrer par rétro-analyse des périodes de maladie / fatigue de l'historique, puis ajuster ; seuils = paramètres, « pas des faits à citer » (§7 constat 4).
- **Preuve** : non vérifiée (choix de conception).
- **Question** : prévoir une rétro-analyse sur l'historique importé (FR-1) avant la mise en usage, plutôt qu'en v1.2 ? Le « pourquoi » de FR-16 doit-il éviter de présenter « +5 bpm » comme un fait ?

---

## 3. Confirmations

- **Principe 4 / addendum §2** (« le LLM raconte, le code compte ») : confirmé par R10 et [58][59] (garde-fous en prompt non respectés).
- **FR-5 garde-fou par sortie ≤ 110 % de la plus longue des 30 j** (arbitrage 14, 30) : confirmé par R1 [65] — la règle la mieux prouvée.
- **FR-12 Ressenti avant les chiffres** : confirmé par [76] (mesures subjectives > capteurs) et R8 ; réserve : échelle à 3 niveaux non validée en tant que telle.
- **FR-9 fractionnés ciblés en allure** : confirmé par §1.1 (FC optique en retard sur les efforts courts [112]).
- **FR-16 / FR-19 proposition jamais silencieuse, confirmée** : confirmé par R7.
- **FR-16 logique de concordance** (ressenti + signal objectif) : structure confirmée [31][70] ; seuils à recalibrer (C-3).
- **FR-3 Profil vivant, mémoire persistante** : confirmé par [59].
- **NFR-5 raison en une ligne** : confirmé par [59] (consigne + raison courte).
- **FR-17 fièvre → repos** : confirmé par [86][88] ; l'étendue de la reprise est en C-5.
- **FR-13 / FR-16 / §11 seuils paramétrables** : confirmé par §4.5 et §7 constat 4.
- **CTL en description seulement** (FR-3, FR-8) : confirmé par §4.1 (Banister mal validé, pas un détecteur).
- **Liste fermée de Méthodes sans MAF ni norvégienne** : confirmé par R5 [53][54].
- **§12 Q7 semaine allégée chiffrée / FR-5 toutes les 3–4 semaines** : cohérent avec la convention [57], qui n'est pas une preuve ; à garder paramétrable.
- **§8 pas de lecture directe HealthKit, FR-1 via intervals.icu** : compatible ; la recherche ne l'exige pas.
- **Absence d'ACWR dans le PRD** (addendum §2 « contesté ») : confirmé par [42][65].
- **Renforcement en v2** : cohérent, les essais propres à la course sont mitigés [82][83].

---

## 4. Hors PRD (vers addendum / architecture)

- **VFC SDNN vs RMSSD, biais −8,3 ms, pas de validation nocturne** [22][28][72][73] → addendum §5 (si VFC retenue) / architecture (modèle wellness).
- **Contenu du FIT HealthFit, réglage « Synchronisation automatique », VFC moyenne journalière, IntervalsWellnessSync** [15][19][20] → addendum §3 et §5.
- **Algorithme CS** (d = CS·t + D′, 400/800/5 000 m, 16 semaines, GAP Minetti, +5 % si efforts non maximaux) [23][105] → architecture, service de seuils / AD-8, AD-11.
- **CS / D′ calculés par intervals.icu sur la courbe d'allures** [107] → addendum §1.
- **FC max observée > Tanaka (208 − 0,7 × âge)** [26] ; protocoles LTHR de Friel non mélangeables [35][109] → architecture (résolution des zones, AD-8).
- **Formule de charge intervals.icu (h × IF² × 100), HRSS, Form le jour même, zones de Form −30** [39][40][41][64] → addendum §1.
- **Conformité intervals.icu (charge réalisée / prévue, vert 80–120 %), appariement refusé si charge < 50 %** [97][98] → addendum §1 ; impact AD-22 (rattachement).
- **Indice de polarisation (formule à vérifier dans Treff 2019)** [36][43] → addendum §2 (bandes des Méthodes).
- **Baseline ≥ 28 j, moyenne 7 j, seuil 0,5–1 × CV** [70][71][114] → architecture AD-3 (séries `gold`) et AD-11 (`parameter_set`).
- **Facteurs VDOT après arrêt (× 0,973 / 0,931 / 0,80)** [95] → architecture AD-11 (paramètres de reprise).
- **Découplage Pa:HR, EF, HRRc (calculé si FC ≥ seuil)** [29][101][102] → architecture domaine (« Tes données ») / addendum §1.
- **Pourcentages d'affûtage (−41 à −60 %)** [44][56] → addendum §2 (garde-fous chiffrés).
- **Score de sommeil non comparable entre versions watchOS 26.2** [7] ; Vitals (VFC Series 12 seulement) [11] → architecture (modèle de données sommeil) / addendum §3.
- **Polynôme GAP de Minetti, modèle Strava non publié** [104] → architecture (Conformité allure).
- **Puissance / dynamique de foulée sans validation** [14] → architecture (ingestion : descriptif seulement).

---

## 5. Impact aval

| Conflit / lacune | Architecture | UX |
|---|---|---|
| **C-1** volume hebdo +10 % | AD-12 : famille de l'`id` « volume hebdomadaire » (sécurité → secondaire/informational) ; résultat `refused` vs `informational`. | Messages de refus FR-22 ; jours grisés en `dry_run`. |
| **C-2** exception confirmée | AD-12 « Modification de l'utilisateur » (refus infranchissable) ; AD-4 si une confirmation d'exception est ajoutée ; nouveau paramètre plafond sortie longue (AD-11). | Réintroduire un « Enregistrer quand même » avec avertissement (retiré le 22/11). |
| **C-3 / C-6** surmenage, sommeil | AD-2 : ingestion wellness à élargir (durée de sommeil, éventuellement VFC) ; AD-3 : séries `gold` sur 28 j+ ; AD-11 : CV, fenêtres ; `runner_profile` sommeil n'est plus « vide » ; Deferred « Nutrition et sommeil (v3) » à revoir. | Ligne « pourquoi » de FR-16 ; affichage éventuel du signal sommeil. |
| **C-4 / L-7** Conformité FC, GAP, température | AD-8 (Conformité contre la valeur résolue), AD-9 (ciblage FR-9) ; streams GAP/température à ingérer (AD-2). | Verdict par Bloc et texte « avis » sur les sorties longues. |
| **C-5 / L-2** maladie, blessure | AD-21 machines à états (Imprévu multi-jours), AD-4 propositions ; arbre de décision Imprévus. | Flux « Empêchement » (deux taps, une question au plus) ; message d'orientation médicale. |
| **C-7** Méthode | AD-7 (le LLM propose), AD-12 bandes (ajout éventuel « pyramidal »), AD-11. | Libellé de la raison en une ligne. |
| **C-8 / L-3** seuil / CS | AD-2 (lake en lecture seule ; frontière intervals.icu), AD-5 (outbox : nouveau type d'écriture « réglages de sport »), AD-8 (zones et seuil d'intervals.icu comme **seule** source), AD-11 (snapshot), AD-13. | Profil coureur consultable (FR-3) : seuil et sa date de mise à jour. |
| **C-9** chrono irréaliste | AD-13 (VDOT actuelle + min(semaines ÷ 5 ; 4)) ; Deferred « correction de prédiction propre à une distance ». | Ligne d'avertissement FR-4. |
| **L-1** reprise | AD-12 (garde-fou par sortie non calculable sur fenêtre vide), AD-11 (facteurs). | Flux « Reprendre » (FR-17). |
| **L-6** affûtage | AD-12 (nouveau garde-fou), AD-10 (paramètre par distance, pas de branche). | Vue progression du Plan (FR-5). |
