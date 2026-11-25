---
title: "Réconciliation finale PRD × recherche du 25/11 (après mise à jour)"
created: 2026-11-25
sources:
  - prd.md (mis à jour 25/11)
  - addendum.md (mis à jour 25/11)
  - .memlog.md (décisions du 25/11, lignes 39 à 63)
  - reconcile-research-2026-11-25.md (liste d'écarts initiale)
  - ../../research/domain-donnees-apple-watch-entrainement-course-2026-11-25/research.md
status: contrôle, aucune modification du PRD
---

# Réconciliation finale PRD × recherche du 25/11

Contrôle de la version mise à jour du PRD et de l'addendum. On vérifie trois choses : la prise en compte de R1 à R10 et du §9 de la recherche, la traduction fidèle des décisions du memlog du 25/11, et le sort des idées qualitatives. Les écarts voulus et logués (un seul signal objectif dans FR-16, aucune exception au plafond par sortie, VFC exclue) ne sont **pas** comptés comme des écarts.

**Bilan rapide** : les 19 décisions du 25/11 figurent toutes dans le PRD ou l'addendum. R1, R3, R5, R6, R9 et R10 sont bien repris. Il reste **deux contresens sur la recherche**, de vrais enjeux de sécurité ou d'exactitude (A-1 et A-2), une incohérence de règle (A-3) et plusieurs citations mal placées.

---

## (a) Recommandations non reprises ou mal représentées

### A-1 — FR-24 applique le facteur de Daniels au volume et au plafond par sortie (contresens) — **sévérité haute**
- **PRD** : FR-24 dit « Le volume et les allures repartent d'un niveau réduit par un facteur de désentraînement […] (facteurs de Daniels [95]) » et « Le plafond par sortie […] se calcule sur la plus longue Course connue, multipliée par ce même facteur ». L'addendum §2 (Mode Reprise) dit la même chose.
- **Recherche** : §5.5 dit que la convention Daniels sert à « ajuster les **allures** » et qu'elle « **ne dit rien sur le volume** ». Selon R2, il faut « repartir d'un volume réduit » **et** appliquer le facteur VDOT **aux allures**.
- **Risque** : après plus de 72 jours d'arrêt, le facteur vaut × 0,80. Le plafond devient alors 80 % de la plus longue Course *connue* : par exemple 24 km pour une sortie de 30 km faite des mois plus tôt. La règle de prévention la mieux prouvée [65] est ainsi contournée au moment exact où le risque est le plus haut.
- **Origine** : la décision du memlog (L-1) contient déjà ce contresens, et le PRD la reprend fidèlement. Il faut donc la réviser, pas seulement le texte.
- **À faire** : séparer deux choses. D'un côté, le facteur VDOT s'applique aux allures. De l'autre, une règle de volume de reprise à part, à définir comme paramètre `[ASSUMPTION]` non sourcé (par exemple un plafond absolu, ou une fraction bien plus faible que 0,8).

### A-2 — Addendum : « +5 % de prudence si les efforts n'étaient pas maximaux » (contresens) — **sévérité moyenne**
- **Addendum** §2, Seuil d'allure (FR-23).
- **Recherche** : §6.4 dit que « Corriger les efforts non maximaux **relève** la CS d'environ 5 % » [105]. C'est une correction vers une allure **plus rapide** (des efforts d'entraînement sous-maximaux sous-estiment la vraie CS). Ce n'est pas une marge de prudence.
- **À faire** : décrire la correction telle qu'elle est (elle relève la CS) et décider si on l'applique. Vu la posture « prudence d'abord », ne pas la faire est défendable, mais il faut le dire.

### A-3 — R2 n'est repris qu'en partie : la fenêtre « trop courte » et les pauses de 14 à 30 jours
- **PRD** : FR-24 ne se déclenche que si **aucune** Course n'existe sur 30 jours.
- **Recherche** : R2 couvre « quand la fenêtre des 30 jours est vide **ou trop courte** après une pause ». Les facteurs de Daniels commencent à 14 jours (× 0,973).
- **Trou** : après une pause de 15 à 29 jours, la sortie longue d'avant la pause reste dans la fenêtre. Le plafond par sortie reste donc plein, et les allures ne sont pas recalées.

### A-4 — R8 : la note globale de Conformité ne vient pas d'intervals.icu, et ce choix n'est pas tracé
- **PRD** : FR-13 calcule une note globale (% de Blocs respectés). La Conformité native d'intervals.icu (charge réalisée / prévue, 80 à 120 %) ne sert qu'à titre de comparaison (addendum §1).
- **Recherche** : R8 dit que « la conformité de la séance entière vient d'intervals.icu ; la note par bloc est une métrique propre ».
- **Constat** : l'écart est défendable, mais il n'est ni logué au memlog ni écrit au §15. Il faut l'acter ou le justifier.

### A-5 — R8 : facteur d'efficience (EF) éligible dans l'addendum, absent de FR-14
- **Addendum** §2, « Indicateurs de tendance (FR-14) » : on y lit « découplage Pa:HR et facteur d'efficacité (EF) ».
- **PRD** : FR-14 et la décision L-5 du memlog ne retiennent comme éligibles que la FC à allure comparable et le découplage.
- **À faire** : aligner les deux documents, soit en retirant l'EF de l'addendum, soit en l'ajoutant à FR-14. Corriger aussi le terme : la recherche dit « facteur d'efficience », l'addendum dit « efficacité ».

### A-6 — R8 / §6.2 : la FC à allure comparable n'est pas conditionnée par la pente et la chaleur
- **PRD** : FR-14 rend la « FC à allure comparable [70] » éligible sans condition.
- **Recherche** : §6.2 demande de « Comparer à pente et chaleur comparables ». §6.3 précise que « la chaleur rend la FC et l'allure non comparables d'une séance à l'autre ».
- **Constat** : la température est bien enregistrée (FR-13), mais elle n'encadre pas cet indicateur.

### A-7 — R4 : l'ancrage des zones FC n'est pas défini
- **PRD** : FR-9 cible la majorité des Blocs en FC, et le glossaire dit que le Seuil d'allure ancre « les zones ». Rien ne dit sur quoi reposent les **zones FC**.
- **Recherche** : §2.1 dit que « les pourcentages fixes de FC max […] placent mal les gens ». Il faut un seuil mesuré (CS, LTHR) et une FC max observée. Seul l'addendum (FC max observée plutôt que Tanaka) en parle, et indirectement.
- **À faire** : fixer l'ancrage des zones FC dans FR-23 ou en architecture (§12).

### A-8 — R7 : l'interdiction de l'ACWR n'est pas écrite dans le PRD
- La recherche demande de « ne pas utiliser l'ACWR, ni la Form d'intervals.icu seule ». La Form est bien couverte (addendum §1 : « jamais un garde-fou »). L'ACWR est seulement dit « contesté » (addendum §2), sans être exclu dans §8 ni dans NFR-10.

### A-9 — §9 de la recherche : vérifications et questions perdues
- **Indice de polarisation** (vérification 5 de la recherche) : il figure dans l'addendum §2, mais pas au PRD §12 ni dans la liste des vérifications à faire avant de coder (§9.1 : « Q4, Q9, Q11 »).
- **§9.1 et §12 se contredisent** : §9.1 liste Q4, Q9 et Q11, alors que l'introduction du §12 liste Q4, Q9, Q10 et Q11.
- **Règle des 10 % appliquée à l'intensité** (question ouverte du §9) : elle n'est mentionnée nulle part, même comme limite connue du garde-fou.
- **Tester ses propres Plans contre R1, R9 et R10** (§9, LLM de 2026) : non repris. SM-5 compare la crédibilité perçue, pas le respect des règles.
- **Revérification d'avril 2027 / Refresh** (§12, [108] à revérifier en mars 2027) : aucune échéance n'est notée.

### A-10 — Citations : numéros mal placés ou portée exagérée

| Emplacement | Citation | Problème | Correction suggérée |
|---|---|---|---|
| PRD FR-13 | « ±5 s/km de la cible [104] » | [104] documente le modèle GAP (Minetti / Strava), pas une tolérance de ±5 s/km. La tolérance est un arbitrage (§13.18) sans source. | Placer [104] après « GAP », et marquer ±5 s/km comme paramètre non sourcé. |
| PRD NFR-10 | « allure, FC en effort stable, FC au repos et durée de sommeil [6][21][72] » | [6] et [21] portent sur le sommeil, [72] sur la FC au repos. Aucune source ne couvre la FC en effort ([12] est la bonne référence). L'allure n'a **aucune** validation indépendante (§1.1 : « lacune »). | Ajouter [12], et noter que l'allure GPS est fiable par usage, sans validation publiée. |
| PRD NFR-10 | « la FC optique réagissant en retard [112] » | [112] est une source presse de **faible** confiance. La recherche précise : « aucune étude publiée et évaluée par des pairs ». Le PRD l'énonce comme un fait. | Atténuer (« selon des tests presse »). |
| PRD FR-4 | « cette estimation tend à être optimiste [111] » | [111] porte sur **Riegel**, pas sur la VDOT. L'addendum dit d'ailleurs « Riegel / VDOT ». Les coefficients VDOT eux-mêmes viennent d'une source faible [38]. | Écrire « les équivalences de type Riegel tendent à être optimistes [111] » et citer [38] comme source faible. |
| PRD FR-17 Blessé | « douleur ≤ 5/10 » [89] | [89] est un essai sur la **tendinopathie d'Achille**, que le PRD généralise à toute douleur. R3 généralise aussi, mais la recherche précise bien que c'est l'Achille. | Signaler la généralisation dans l'addendum. |
| Addendum §2 | « Garde-fou de charge : selon Frandsen (BJSM 2025) » | Aucun numéro de référence ([65]). Le texte parle encore du « digest » du 20/11 et ne mentionne pas le reclassement de la règle hebdomadaire. | Ajouter [65] et renvoyer à FR-5 et §15.1. |
| Addendum §2 | La puce « Chrono irréaliste » apparaît deux fois (lignes 39 et 41) | Doublon, sans erreur de citation. | Fusionner les deux puces. |

Toutes les autres citations ont été contrôlées et correspondent à la table des sources : [65][84][46][47][44][56][95][34][106][70][102][31][76][114][64][86][88][90][59][8][10][14][105][108][15][19] dans le PRD, et [39]–[41][97][98][107][36][43][53][54][23][26][35][109][29][101][22][28][72][73][7][11][20] dans l'addendum.

---

## (b) Décisions du memlog (25/11) absentes ou mal traduites

| Décision (memlog) | Où elle est | État |
|---|---|---|
| C-1 volume hebdomadaire en garde-fou secondaire | FR-5, FR-22, §15.1 | OK |
| C-2 plafond par sortie sans exception, sortie longue à +5 % `[ASSUMPTION]` | FR-5, FR-22, §15.2 | OK |
| C-3 surmenage : 1 signal suffit, 2 signaux donnent une proposition renforcée | FR-16, §15.3 | OK |
| C-6 durée de sommeil lue en v1 | FR-1, FR-3, §9.2, §15.4 | OK |
| C-4 et L-7 : deux premiers tiers, GAP, température | FR-13, §15.5 | OK, **avec réserve B-3** |
| C-5 Malade multi-jours | FR-17, UJ-4, §15.6 | OK |
| L-2 Blessé, **« REDs hors périmètre v1 »** | FR-17 | **B-1 : la mention REDs manque** |
| L-1 Mode Reprise | FR-24 | Fidèle au memlog, mais la décision contient un contresens (voir A-1) |
| L-4 NFR-10, L-10 sans objet | NFR-10, addendum §5 | OK |
| C-7 Méthode déterministe, **« 80/20 reste dans la liste fermée »** | FR-5, glossaire, addendum §2 | **B-2 : 80/20 n'est jamais sélectionnable** |
| L-3 / C-8 Seuil d'allure, Q10 | FR-23, §12 Q10 | OK |
| C-9 VDOT avec avertissement d'optimisme | FR-4, addendum | OK (citation : voir A-10) |
| C-10 et L-5 : format 1 + 2, éligibilité, sensations attendues | FR-14, FR-21 | OK (l'addendum ajoute l'EF : voir A-5) |
| L-6 affûtage | FR-5, addendum §2 | OK |
| L-8 / L-9 vérifications | §12 Q4, Q9, Q11, addendum §5 | OK (incohérence §9.1 / §12 : voir A-9) |
| L-11 calibrage avant mise en usage | §11, FR-16 | OK |
| 15 points hors PRD vers l'addendum | addendum §1 à §3 | OK, les 15 sont présents |
| §13 et §14 annotés | §13.9, 13.18, 13.29 annotés | **B-4 : §14.2 n'est pas annoté** |

- **B-1 — REDs hors périmètre v1** (memlog L-2) : ni le §8 (Non-objectifs) ni le §9.2 ne le disent. La recherche (§5.6) recommande un drapeau non diagnostique. Il faut écrire l'exclusion et sa raison, ou la reporter en v1.x.
- **B-2 — 80/20 inatteignable** (C-7) : la règle déterministe ne connaît que deux branches, Daniels par défaut et polarisé au-delà d'un volume. 80/20 reste dans la liste fermée (glossaire) et a sa propre bande (addendum §2), mais aucune règle ne le choisit. Il faut lui donner une condition de sélection, ou le retirer de la liste.
- **B-3 — Source de la GAP ambiguë** (L-7) : le memlog dit « GAP fournie par intervals.icu ». L'addendum §2 dit « GAP : polynôme de Minetti », ce qui suggère un calcul par l'application, et l'addendum §1 dit « à confirmer dans les streams ». Il faut dire qui la calcule.
- **B-4 — Annotation manquante au §14.2** : le §15 dit préciser « l'arbitrage 2 du §14 », mais le §14.2 ne porte pas la mention « *(précisé par §15.1–15.2)* », alors que le §13 est annoté. Écart mineur.
- **B-5 — FR-17 Blessé : cas « non, mais douleur > 5/10 » non couvert, et deux questions en une** : la décision L-2 pose une seule question (type de douleur), puis utilise une deuxième information (≤ 5/10) qui n'est demandée nulle part. Ce point heurte aussi la règle « une question au plus », et le cas « douleur ni osseuse ni à la marche, mais > 5/10 » n'a pas de branche.

---

## (c) Idées qualitatives abandonnées sans le dire

1. **Les limites de la règle par sortie** (§5.2) : selon les auteurs, elle « ne doit pas être utilisée seule », car une progression étalée sur plusieurs séances peut aussi être excessive. De plus, FR-5 inclut les Séances **planifiées** dans la fenêtre glissante (arbitrage 30), alors que [65] mesure les sorties **réalisées**. Le plafond peut ainsi grimper sur des Séances jamais courues. Le PRD ne dit ni l'un ni l'autre.
2. **Le prescrit n'est pas le réalisé** (§3.1) : il faut « suivre la charge réalisée ». La bande d'intensité de FR-5 ne contrôle que le Plan **prévu**, et le Bilan de fin de Plan (FR-8) n'inclut pas la répartition des intensités réellement faite.
3. **Mise en garde de Buchheit** (§4.4) : « une FC élevée ne doit pas être utilisée comme marqueur clair de fatigue ». FR-16 en fait pourtant un signal objectif suffisant. Le choix est logué, mais la mise en garde n'apparaît ni dans le « pourquoi » ni dans §11.
4. **L'écart entre ressenti et FC n'est pas forcément de la fatigue** (§6.2, R8) : la case « Non respectée × 🥵 = fatigue probable » de la Matrice (FR-15) affirme une fatigue que la recherche demande de croiser avec le contexte (chaleur, pente).
5. **L'échelle de Ressenti à 3 niveaux n'est pas validée** (synthèse, §4.4) : le PRD s'appuie sur [76] sans dire que l'échelle simplifiée n'a pas été testée.
6. **La blessure antérieure, premier facteur de risque** (§5.1, [79][80]) : le Profil enregistre les blessures passées, mais FR-5 ne respecte que « les blessures du Profil », et Q8 ne parle que des blessures actuelles. Rien ne prévoit une progression plus prudente pour un coureur déjà blessé.
7. **Les conventions de structure de plan** (§3.3, [57]) : sortie longue d'environ 25–30 % du volume hebdomadaire et d'au plus 2 h 30 à 3 h, plans marathon de 12 à 18 semaines. Ces repères n'apparaissent nulle part, ni comme paramètre ni comme avertissement, alors que FR-4 accepte un Plan marathon de 4 semaines.
8. **Le guide maladie est une transposition** (§5.3) : il a été écrit pour le COVID. FR-17 l'applique à toute maladie sans le dire, alors que la recherche insiste sur ce point.
9. **La taxonomie des risques du coach LLM** ([61] : surmenage, image du corps, **conseils hors de portée**) et l'absence de questions de clarification dans les plans LLM [58] : le §7 (Ton et voix) n'interdit pas les conseils hors de portée.
10. **Ton prudent sur les données incertaines** : la recherche formule à chaque fois le niveau de confiance (« indicatif », « repère de coach », « à calibrer »). Le PRD le fait pour les seuils de surmenage (FR-16) et pour la Méthode, mais pas pour le **découplage** (le seuil de 5 % de Friel n'est pas validé, §6.2) ni pour la tolérance de Conformité. Il faut une règle générale de ton : un repère de coach se présente comme un repère, pas comme une norme.

---

## Priorités suggérées

1. **A-1** : corriger FR-24 et la décision L-1 du memlog (sécurité).
2. **A-2** : corriger le sens de la correction de CS dans l'addendum.
3. **B-2 et B-5** : rendre 80/20 sélectionnable ou le retirer, et compléter l'arbre « Blessé ».
4. **A-3 et c-1** : couvrir les pauses de 14 à 30 jours, et décider si la fenêtre glissante compte les Séances planifiées ou seulement les Courses réalisées.
5. **A-10** : corriger les citations ([104], [112], [111], NFR-10).
