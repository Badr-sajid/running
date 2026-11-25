---
title: "Revue de réconciliation : spine ↔ PRD + addendum (mise à jour du 25/11)"
reviewed: ARCHITECTURE-SPINE.md (updated 2026-11-25, copie de travail non commitée)
against: prd.md (final, updated 2026-11-25), addendum.md (updated 2026-11-25)
scope: FR-1, FR-3, FR-4, FR-5, FR-13, FR-14, FR-16, FR-17, FR-21, FR-22, FR-23, FR-24, NFR-10, §12 Q1-Q11, §15, addendum §1.2, §2.2, §3.1, §5
date: 2026-11-25
---

# Revue de réconciliation PRD (25/11) → spine

## Verdict

Le spine absorbe bien la charpente de la mise à jour du 25/11 : Seuil d'allure possédé par l'app (AD-27), Mode Reprise figé sur le Plan (AD-28), hiérarchie des données imposée par deux barrières (AD-26), détecteur de surmenage unique rejoué pour le calibrage (AD-29), garde-fous réorganisés en familles et en moments (AD-12). Aucune contradiction avec les décisions délibérées (Q10 option A, seuil en vigueur = dernier confirmé, blessures non déterministes, 80/20 retiré, bascule à 70 km/sem).

Il reste toutefois **quatre trous structurels** que deux équipes combleraient différemment :

1. le Ressenti et le Feedback sont accrochés à `run_attachment`, ce qui exclut les Courses non planifiées alors que FR-16 les compte ;
2. le Seuil d'allure n'a pas d'amorçage (aucun seuil confirmé au premier Plan) et le Mode Reprise le « dé-réduit » via la fenêtre de 16 semaines ;
3. la « raison en une ligne » du Plan (Méthode honnête, Mode Reprise, Bilan pour SM-6, Objectifs secondaires) n'a pas de propriétaire ;
4. le plafond par sortie en Mode Reprise et la conversion durée → distance des Séances ne sont pas définis, alors que le garde-fou de sécurité en dépend.

À cela s'ajoute une série de règles discrètes (nag quotidien après un refus, Conformité « non évaluable », zone d'allure contre ±5 s/km, CTL qui rentre dans la génération via le Bilan) qu'il vaut mieux fixer dans le spine.

Critère : un point est classé **spine** s'il traverse au moins deux epics ou touche un invariant (état, propriété d'un calcul, frontière de données). Il est classé **story** s'il vit dans un seul cas d'usage et qu'un paramètre ou un test suffit.

## Synthèse des constats

| # | Sévérité | Exigence | Constat | Niveau |
| --- | --- | --- | --- | --- |
| 1 | **critique** | FR-12, FR-13, FR-16 | `feeling` et `feedback` dépendent de `run_attachment` (ER, AD-21) : une Course non planifiée n'a ni Ressenti ni Feedback global, et la série de 3 🥵 « non planifiées comprises » est impossible | spine |
| 2 | **critique** | FR-23, FR-24 | Pendant et après le Mode Reprise, la réestimation (16 semaines) inclut les meilleurs efforts d'avant l'arrêt et remonte le seuil ; « Reprendre » n'a pas d'acceptation, donc le seuil `recovery` n'est jamais confirmé | spine |
| 3 | **haute** | FR-23, FR-2, FR-5 | Aucun amorçage du Seuil d'allure : sans version *confirmée*, AD-8 ne peut résoudre aucune cible d'allure. Pas d'origine `initial`, ni de correction manuelle | spine |
| 4 | **haute** | FR-4, FR-5, FR-24, NFR-5, SM-6 | La raison du Plan en une ligne n'a ni propriétaire (LLM ou gabarit) ni champ dans la sortie structurée ; la validation des nombres n'est garantie que pour FR-14 | spine |
| 5 | **haute** | FR-24, FR-5 | Plafond par sortie en Mode Reprise : « Courses réalisées seulement » (PRD) contre calendrier unifié (AD-12). À la génération, il n'existe aucune Course de reprise | spine |
| 6 | **haute** | FR-5, FR-22, FR-8 | Les garde-fous se mesurent en distance, mais une Séance ou un Bloc peut être en durée. La conversion durée → distance (quelle allure ?) n'est pas fixée | spine |
| 7 | moyenne | FR-13, AD-3, AD-8 | Les deux premiers tiers et la zone résolue stockée sont incompatibles avec un `gold` qui donne le temps en zone par lap entier avec les zones courantes ; la « dérive indicative » rangée en `gold` viole AD-3 | spine |
| 8 | moyenne | FR-13, FR-9, AD-8 | Une cible d'allure est une **zone** (plage) ou une allure Daniels, mais la Conformité dit « ±5 s/km de la cible » : par rapport au centre, aux bornes ou à une valeur ? | spine |
| 9 | moyenne | FR-23, FR-16, §7 | Après un refus, le déclencheur périodique du seuil se relance chaque nuit (le seuil confirmé reste « trop vieux ») ; même risque pour le surmenage (3 🥵 toujours vraies). Ce harcèlement contredit « pas d'intrusion » | spine |
| 10 | moyenne | FR-13, FR-15, FR-8, FR-16 | Conformité « non évaluable » (laps ≠ Blocs) : la case de la Matrice, la Conformité moyenne du Bilan et le statut « respectée » ne sont pas définis | spine |
| 11 | moyenne | FR-5, FR-22, §15.1 | Règle secondaire : « +10 % d'une semaine à l'autre » (PRD) contre « par rapport à la dernière semaine non allégée » (AD-12). L'écart est silencieux, et l'exemple du PRD (« +14 % par rapport à la semaine 4 ») ne colle plus | spine + amendement PRD |
| 12 | moyenne | NFR-10, FR-8, FR-5 | La CTL est `descriptive` et va au Bilan, mais le Bilan est lu **obligatoirement** par la génération (AD-24). La CTL entre donc dans une décision | spine |
| 13 | moyenne | FR-16, §11, NFR-8 | Le calibrage « avant la mise en usage » (ou observation seule par défaut) n'est pas une condition de AD-19 ; la valeur par défaut du drapeau n'est pas fixée | spine |
| 14 | moyenne | FR-22, FR-11, FR-5 | « Deux Séances ne tombent jamais le même jour » n'est pas un invariant (contrainte ou garde-fou), alors que la génération LLM, les déplacements et le rattachement (FR-11) en dépendent | spine |
| 15 | moyenne | FR-16, FR-22 | Le surmenage n'a pas de point de déclenchement (synchro ? `record_feeling` ?). Le contenu de l'Ajustement (normal ou « plus ferme ») et le marqueur « Séance allégée par surmenage » (règle des 7 jours) ne sont pas définis | spine (déclencheur, marqueur) / story (ampleur) |
| 16 | basse | FR-1, FR-16, FR-23, AD-29 | La profondeur de l'import initial n'est pas fixée : le calibrage a besoin de tout l'historique, alors que les fenêtres (6 mois, 16 semaines) suggèrent moins | spine |
| 17 | basse | FR-21, FR-22 | Après une modification à la main (type changé), le « but dans le Plan » produit par le LLM devient périmé ; aucune règle ne dit qui le réécrit | spine |
| 18 | basse | FR-24, addendum §2.2, AD-8 | Les facteurs de Daniels sont des multiplicateurs de **VDOT**, alors que AD-8 réserve la VDOT au chrono. La façon de les appliquer à la CS n'est pas fixée | story (avec note dans AD-28) |
| 19 | basse | FR-5, addendum §2.2 | La bande Daniels est mesurée « en temps prévu par zone » puis « % du volume » ; le classement d'un Bloc FC ou Daniels dans Z1-Z5 n'est pas fixé ; l'indice de Treff est abandonné sans mention | story (avec note dans AD-12) |
| 20 | basse | divers | Règles de message et de ton, paramètres manquants, spikes sans repli : voir la section « Niveau story » | story |

## Détail des constats « spine »

### 1. Ressenti et Feedback des Courses non planifiées (critique)

- **PRD.** FR-12 : « Dès qu'une Course est récupérée, l'utilisateur pose son Ressenti », pour toute Course. FR-13 : « si la Course n'était pas planifiée, le Feedback est global ». FR-16 : « 3 Courses 🥵 d'affilée (**non planifiées comprises**) ». FR-11 : les Courses en surplus d'un même jour sont « traitées comme non planifiées ».
- **Spine.** Le diagramme ER pose `RUN_ATTACHMENT ||--o| FEELING` et `RUN_ATTACHMENT ||--o| FEEDBACK`. AD-21 fait naître le Feedback au « rattachement d'une Course éligible ». AD-23 parle d'« une seule ligne par activité », ce qui va dans le bon sens mais contredit le diagramme.
- **Divergence.** L'epic Feedback n'ouvrira de Ressenti que pour les Courses rattachées. L'epic Surmenage comptera des Ressentis qui n'existent pas. Autre cas : une Course courue entre deux Plans (aucun Plan actif).
- **Correctif.** Clé de `feeling` et `feedback` = l'activité (`icu_activity_id` + `athlete_id`), avec `run_attachment` facultatif. AD-21 : « Feedback — → *en attente du Ressenti* » sur **détection** d'une Course éligible (AD-15), rattachée ou non. Fixer aussi le comportement sans Plan actif (Ressenti demandé ? Feedback global sans niveau « Plan / Objectif » ?).

### 2. Mode Reprise et réestimation du Seuil d'allure (critique)

- **PRD.** FR-23 : « En Mode Reprise, il est réduit par le facteur de désentraînement, **puis réestimé sur les Courses de la reprise** ». FR-24 : « une Course d'avant l'arrêt n'ouvre jamais de droit ».
- **Spine.** AD-27 réestime sur une fenêtre de 16 semaines, par deux déclencheurs. AD-28 : « le Seuil d'allure est alors réestimé sur les Courses de la reprise **par les déclencheurs habituels** ». Or ces déclencheurs utilisent la fenêtre de 16 semaines, qui contient encore les efforts d'avant l'arrêt (arrêt de 14 jours ou plus, puis reprise de 30 jours ou plus : il reste 6 à 10 semaines d'efforts d'avant). La revue périodique (4 à 6 semaines) peut aussi tomber **pendant** le Mode Reprise. Dans les deux cas, elle propose de revenir au seuil d'avant l'arrêt.
- **Deuxième trou.** Dans AD-21, « Reprendre » passe directement de *suspendu* à *actif* et régénère. Le seuil `recovery` n'est confirmé qu'à « l'acceptation d'un Plan en Mode Reprise », et « Reprendre » n'a pas d'acceptation. Les Séances régénérées seraient donc résolues avec l'ancien seuil, non réduit. Par ailleurs, FR-19 exige que toute régénération soit proposée puis confirmée.
- **Correctif.**
  - Fenêtre d'estimation bornée à `max(aujourd'hui − 16 sem, recovery_started_on)` tant qu'un Plan porte `recovery_started_on` dans les 16 dernières semaines, et non seulement pendant le Mode Reprise.
  - Revue périodique suspendue pendant le Mode Reprise.
  - « Reprendre » produit un Plan régénéré sous forme de `proposal` (ou de Plan *proposé* rattaché au même `plan_id`). Sa confirmation confirme le seuil `recovery`.

### 3. Amorçage du Seuil d'allure (haute)

- **Spine.** AD-8 résout toute cible d'allure « à partir du Seuil d'allure **confirmé** ». AD-21 ne connaît que deux confirmations : un Ajustement, ou un Plan en Mode Reprise. Origines possibles : `best_effort`, `periodic`, `recovery`.
- **Trou.** Au premier Plan, aucun seuil n'est confirmé : il n'y a pas de Plan, donc pas d'Ajustement à confirmer. FR-2 (« corrige les valeurs importées ») et la consultation de FR-3 supposent un seuil visible et corrigible dès la validation du Profil. Faute de règle, chaque epic inventera son amorçage (confirmation silencieuse, blocage de la génération, seuil d'intervals.icu, pourtant interdit par AD-27).
- **Correctif.** La validation du Profil (FR-2) affiche le seuil estimé ; elle le confirme (origine `initial`) ou accepte une correction (origine `manual`). Une génération est refusée tant qu'aucun seuil n'est confirmé. Fixer aussi le cas où la régression est impossible (moins de deux distances dans la fenêtre) : avec quel repli, et quel message ?

### 4. Raison du Plan en une ligne sans propriétaire (haute)

- **PRD.** La raison doit :
  - citer la Méthode et dire « honnêtement que les études montrent peu d'écart » (FR-5) ;
  - dire que le Plan démarre en Mode Reprise, et pourquoi (FR-24) ;
  - citer les Objectifs secondaires qui l'influencent (FR-4) ;
  - faire référence au Bilan précédent (SM-6, bloquant).
- **Spine.** AD-7 ne prévoit, pour le Plan, que le squelette (`justification` **par Séance**). AD-24 dit « le cite dans sa raison » sans dire qui écrit cette raison. L'extracteur de nombres « alimente le validateur de FR-14 », rien n'est dit pour les textes du Plan ni pour le « pourquoi » d'une demande à l'IA.
- **Divergence.** Une équipe écrit la raison par gabarit, l'autre la demande au LLM. SM-6 peut échouer sans test, et un nombre inventé dans une justification passe.
- **Correctif.** Composer la raison du Plan en deux parties :
  - une partie déterministe, par gabarit backend : Méthode et phrase honnête, Mode Reprise, référence au Bilan ;
  - une partie LLM, `plan_rationale` dans la sortie structurée : Objectifs secondaires et récit.

  Le validateur de nombres (extracteur unique) s'applique à **toute** sortie textuelle du LLM : `justification`, `plan_rationale`, « pourquoi » d'une demande à l'IA. Un test vérifie que la raison cite le `plan_review` quand il existe.

### 5. Plafond par sortie en Mode Reprise (haute)

- **PRD.** FR-24 : « il se calcule sur les **seules Courses réalisées** depuis la reprise, comme en FR-5 ». FR-5 inclut pourtant les Séances à venir.
- **Spine.** AD-12 : « En Mode Reprise, seules les Courses depuis la reprise comptent ». Rien ne dit si les Séances à venir du Plan candidat comptent.
- **Divergence.** À la génération, aucune Course de reprise n'est réalisée.
  - Lecture « réalisé seulement » : seule la règle de la première Séance borne le Plan, et rien ne borne les suivantes. Le plafond est alors indéfini, ou toutes les Séances sont plafonnées au niveau de la première.
  - Lecture « calendrier unifié limité à la date de reprise » : chaîne de +10 % de Séance en Séance.

  Les epics Génération et Modification trancheront différemment.
- **Correctif.** En Mode Reprise, le calendrier unifié de AD-12 est tronqué à `recovery_started_on` : réalisé depuis la reprise et prévu à venir. Le cas vide est couvert par la règle de la première Séance. Le noter comme une précision à reporter dans FR-24.

### 6. Conversion durée → distance (haute)

- **Spine.** AD-12 : « Volume mesuré en distance (m) », plafond « distance d'une Séance ≤ 110 % ». FR-9 et le glossaire : un Bloc a « une durée **ou** une distance ». Le squelette porte un `volume` sans unité fixée.
- **Trou.** Une sortie en endurance ciblée en FC (« 50 min Z2 ») n'a pas de distance. Il faut une allure de conversion (allure E du seuil confirmé ? allure moyenne récente ?) pour le plafond, la règle hebdomadaire, la semaine allégée, l'affûtage, le volume prévu du Bilan (FR-8) et le choix de la Méthode.
- **Correctif.** Une seule fonction du domaine, `planned_distance(workout)`. Pour un Bloc en durée, elle utilise l'allure de référence résolue du Bloc (borne lente de la zone, par prudence), avec l'instantané de AD-8. Préciser aussi l'unité de `volume` dans le squelette (AD-9).

### 7. Conformité FC sur les deux tiers et zones résolues (moyenne)

- **Spine.** AD-3 confie à `gold` le temps passé par zone FC, **par lap**, « calculé depuis les streams » (donc avec les zones courantes d'intervals.icu). AD-8 : « la Conformité compare toujours le réalisé à la valeur résolue stockée ». AD-11 : les deux tiers sont un paramètre. AD-26 range la « dérive cardiaque indicative » parmi les faits `descriptive`, donc en `gold`.
- **Conflits.**
  1. Les deux premiers tiers d'un lap ne se déduisent pas d'un agrégat par lap.
  2. Si les zones FC d'intervals.icu ont changé depuis la résolution, `gold` mesure contre d'autres bornes que la cible stockée.
  3. La dérive se calcule sur une découpe paramétrée : la mettre en `gold` y introduit un paramètre métier (contraire à AD-3). C'est en outre presque le même calcul que le découplage, qui appartient au domaine.
- **Correctif.** `gold` expose les streams par lap (séries brutes). Le domaine calcule le temps en zone contre la plage résolue stockée, sur la fraction paramétrée, ainsi que la dérive et le découplage. La dérive reste « indicative » par usage (affichage seulement), pas par son lieu de calcul.

### 8. Cible d'allure : plage ou valeur ? (moyenne)

- **Spine.** AD-8 : une cible est « une zone parmi 5 (allure ou FC), ou une allure Daniels ». Le `step` stocke « la valeur résolue ».
- **PRD.** FR-13 : « GAP à ±5 s/km **de la cible** ».
- **Divergence.** Pour une zone d'allure (plage), une équipe juge par rapport au centre ± 5 s, une autre par rapport aux bornes élargies de 5 s. Ce qui est publié sur la montre (plage ou valeur ± tolérance) doit aussi correspondre à ce qui est jugé.
- **Correctif.** Une valeur résolue = `{low, high}` en s/km (une allure Daniels = plage dégénérée ou centre ± demi-largeur paramétrée). La Conformité accepte `[low − tol, high + tol]`. La Publication envoie exactement `{low, high}`.

### 9. Propositions répétées après un refus (moyenne)

- **Spine.** AD-27 : le déclencheur périodique s'active si « le seuil confirmé est plus ancien que la période de revue ». AD-21 : refus ou expiration → *rejeté*, sans rien d'autre. AD-4 : la proposition expire à la fin du jour local.
- **Effet.** Après un refus, le seuil confirmé reste « trop vieux » : `advance_calendar` (chaque nuit) recrée une proposition le lendemain, puis encore le suivant. Même mécanique pour le surmenage tant que les trois derniers Ressentis restent 🥵 et le signal objectif actif. Cela contredit §7 (« pas d'intrusion ») et gonfle SM-C1.
- **Correctif.** La période de revue court depuis la dernière **estimation** (confirmée ou rejetée), pas depuis la dernière confirmation. Une proposition de surmenage n'est reproposée que si un **nouveau** fait s'ajoute (nouveau 🥵 ou signal qui s'aggrave). À ranger dans AD-21 ou AD-29.

### 10. Conformité « non évaluable » (moyenne)

- **PRD.** FR-13 : si les laps ne correspondent pas aux Blocs, le Feedback est global et sans verdict.
- **Trou.** Une telle Séance est rattachée (FR-15 : « chaque Course rattachée »), mais n'a pas de statut respectée / non respectée. Sa case de Matrice, sa prise en compte dans la « Conformité moyenne » du Bilan (FR-8), et la question de savoir si elle passe *réalisée* ne sont pas fixées. AD-22 calcule la Conformité « une seule fois » : il faut un résultat persisté qui dise « non évaluable ».
- **Correctif.** `compliance.status ∈ {evaluated, not_evaluable}` avec un code de raison (`LAP_MISMATCH`, `UNPLANNED`). Matrice : ligne omise et lecture « Ressenti seul ». Bilan : moyenne sur les seules Séances `evaluated`, avec le compte des `not_evaluable` affiché.

### 11. Référence de la règle hebdomadaire (moyenne, amendement PRD)

- **PRD.** FR-5 : « augmente de 10 % au plus **d'une semaine à l'autre** ». Exemple FR-22 : « +14 % **par rapport à la semaine 4** ».
- **Spine.** AD-12 : « ≤ +10 % par rapport à la **dernière semaine non allégée** ».
- **Écart.** Après une semaine allégée à 75 %, le spine autorise +47 % par rapport à la semaine précédente. C'est défendable en coaching, mais ce n'est ni le PRD ni l'exemple de message. Le choix n'apparaît pas dans les décisions délibérées.
- **Correctif.** Soit aligner le spine sur le PRD, soit garder la règle et l'ajouter à la liste des amendements du PRD (FR-5, FR-22 et son exemple), comme pour les blessures. Le message doit nommer la semaine de référence. Fixer aussi le traitement des semaines d'affûtage et des premières semaines de reprise, qui ne sont pas des semaines « normales ».

### 12. La CTL entre dans la génération via le Bilan (moyenne)

- **PRD.** NFR-10 : « La CTL reste descriptive et ne sert jamais de base à une décision ». FR-8 : le Bilan contient « l'évolution de la CTL ».
- **Spine.** AD-26 : les faits `descriptive` (CTL) « ne vont qu'aux lectures d'affichage et au Bilan ». AD-24 : la génération « lit **obligatoirement** le dernier `plan_review` ». Le Plan est une décision, et le chemin CTL → Bilan → prompt de génération contourne la barrière de AD-26.
- **Correctif.** `plan_review` sépare ses champs `decision` (volumes, Conformité, Séances manquées, records, Ressentis) de ses champs `descriptive` (CTL). Seuls les premiers entrent dans le prompt de génération. Étendre le test CI de AD-26 à ce cas. Classer aussi explicitement la **charge par Course, l'ATL et la Form** (importées par FR-1, addendum §1.2) en `descriptive`.

### 13. Calibrage et porte de mise en usage (moyenne)

- **PRD.** FR-16 et §11 : calibrage « **avant la mise en usage** », avec un critère de fin ; à défaut, détecteur en observation seule.
- **Spine.** AD-29 fournit l'outil et le drapeau, mais AD-19 (la porte de mise en usage) ne cite ni le calibrage ni la valeur par défaut du drapeau.
- **Correctif.** Ajouter à AD-19 : « `parameter_set` de prod issu de `tools/calibrate` avec un critère de fin atteint, **ou** `observation_only = true` ». Valeur par défaut dans le code : `observation_only = true`.

### 14. Une Séance par jour (moyenne)

- **PRD.** FR-22 : « deux Séances ne tombent jamais le même jour ». FR-11 suppose « la Séance prévue le même jour » (au singulier).
- **Spine.** C'est vérifié pour un déplacement (« jours sélectionnables »), mais ce n'est ni une contrainte de données ni un garde-fou de génération. Un squelette LLM avec deux Séances le même jour passerait la validation.
- **Correctif.** Unicité partielle `(plan_id, date)` sur les Séances non *annulées*, et rejet au validateur du squelette (comptée comme une tentative).

### 15. Surmenage : déclencheur, marqueur, ampleur (moyenne)

- **Déclencheur.** AD-27 fixe ses déclencheurs ; AD-29 non. Le troisième 🥵 arrive par `record_feeling` (api), après la synchro ; la FC au repos et le sommeil arrivent par la synchro. Si le détecteur ne tourne que dans la synchro, la proposition arrive à l'ouverture suivante. **Fixer :** détecteur exécuté après `record_feeling` et en fin de synchro, idempotent.
- **Marqueur.** La règle des 7 jours (AD-12) suppose de savoir qu'une Séance a été « allégée par un Ajustement de surmenage » et à quelle date. **Fixer :** référence `adjustment_id` (type `overload`) sur la Séance, ou requête normative sur `adjustment`. Préciser si une demande à l'IA compte comme « hausse manuelle ».
- **Ampleur (story).** Allègement « normal » ou « plus ferme » quand les deux signaux concordent : pourcentage, nombre de jours, Séances touchées. Ce sont des paramètres absents de la liste de AD-11.

### 16. Profondeur de l'import initial (basse)

FR-1 dit « l'historique ». AD-29 rejoue le détecteur sur « l'historique importé » pour retrouver des épisodes de maladie anciens ; AD-13 regarde 6 mois et AD-27 16 semaines. Une équipe qui importe 6 mois rend le calibrage impossible. **Fixer :** import de tout l'historique disponible pour les activités et le wellness ; les streams peuvent être limités (paramètre) pour tenir le quota de 5 000 requêtes par jour, avec une reprise sur plusieurs jours visible dans la progression. La première synchro en cours ne doit pas mettre `sync_status` en ⚠️ (FR-1).

### 17. « But dans le Plan » après une modification (basse)

FR-21 affiche le but de la Séance en une ligne. Il vient de la `justification` du LLM (AD-9). Après un changement de type à la main (déterministe, sans LLM, FR-19), la ligne devient fausse. **Fixer :** une Séance construite ou modifiée de façon déterministe reçoit le but du gabarit de son type, et la `justification` LLM est effacée.

## Niveau story (à laisser aux stories, avec paramètres et tests)

Ces points ne demandent pas d'invariant, mais une story doit les trancher explicitement. Les paramètres cités doivent rejoindre `parameter_set` (AD-11).

**FR-4 / AD-13 — chrono irréaliste**
- Fixer quel chrono « jugé réaliste » s'affiche : VDOT actuelle seule, ou VDOT + marge de progression. Le chiffre affiché doit être cohérent avec la condition d'alerte.
- « Le meilleur effort des 6 derniers mois » : préciser s'il s'agit de la VDOT maximale sur toutes les distances, et si elle est calculée en GAP (série `gold`) ou sur les temps bruts (« records » du PRD).
- Le texte de l'alerte doit mentionner l'optimisme des équivalences [111] et la faiblesse de la source VDOT [38]. C'est un gabarit backend, pas le LLM.
- Seuil des 4 semaines : 28 jours ou 4 semaines ISO ? La convention « Semaine N » du spine rend la question réelle.

**FR-5 — génération**
- Méthode : préciser comment repérer une semaine « allégée » dans l'historique **réalisé** (la définition à 75 % porte sur le Plan), et le comportement avec moins de 8 semaines d'historique.
- Bandes : la bande Daniels est exprimée en distance ou en temps (AD-12 dit les deux). Il faut une table de correspondance « référence de Bloc (zone d'allure, zone FC, E/T/I/R/M) → zone Z1-Z5 » pour la bande polarisée. L'indice de Treff (addendum §2.2) est abandonné au profit de seuils simples : le noter dans AD-12.
- Affûtage : préciser la semaine de référence de la réduction de 41 à 60 % (pic d'avant l'affûtage ?), la définition de « fréquence maintenue » (nombre de Séances par semaine) et la règle de correspondance entre la distance de l'Épreuve et la durée d'affûtage (plages de distance, AD-10).
- Échec après 3 tentatives : la tâche se termine avec le `id` du garde-fou violé (résultat de AD-12) et la proposition de modifier l'Objectif. Aucun Plan *proposé* n'est créé.

**FR-24 — Mode Reprise**
- Définir « la dernière semaine normale avant l'arrêt » : dernière semaine ISO complète non allégée et hors affûtage ?
- Les facteurs de Daniels sont des facteurs de VDOT : choisir entre « CS × facteur » et une conversion par la VDOT. Ce second choix est contraire à AD-8 (« La VDOT ne sert qu'à l'alerte de chrono ») et demande donc une exception écrite dans AD-28.
- Interpolation entre 14, 28 et 72 jours : par paliers ou linéaire ?
- Trou du PRD à signaler : un Plan resté *actif* (Malade « oui », prolongé) après 14 jours sans Course n'entre jamais en Mode Reprise, puisque l'entrée n'a lieu qu'à la génération ou au « Reprendre ».

**FR-13, FR-14 — Conformité et Feedback**
- Seuil des 60 min : durée prévue ou réalisée ? En faire un paramètre.
- Après 3 échecs du validateur, le PRD affiche directement « avis indisponible ». AD-21 passe à *en erreur* puis demande « Arrêter l'analyse ». Choisir la lecture et l'écrire dans la table d'états.
- FC à allure comparable : plage d'allure et fenêtre d'historique à paramétrer.
- HRRc (addendum §2.2) : seulement si la FC a dépassé la LTHR, et jamais seule à la base d'un verdict. Le facteur d'efficience (EF) n'est pas retenu en v1. Ni l'un ni l'autre n'est éligible à « Tes données » en v1.
- Q5 : la table « type de Séance → chiffres » de EXPERIENCE.md doit être mise à jour (indicateurs de tendance, sensations attendues) **avant** l'epic Feedback. Le spine ne suit pas ce prérequis.
- Règles de prompt : « aucun compliment sans chiffre », « si non respectée, ce qui a manqué et comment faire mieux », trois niveaux (Séance, rôle dans le Plan, Objectif).

**FR-16 — surmenage**
- Variabilité : coefficient de variation (addendum) ou écart-type ? Fenêtre de la Ligne de base : 28 jours glissants, avec ou sans les 7 derniers jours ?
- Le « pourquoi » cite l'écart mesuré, jamais le seuil comme un fait établi. C'est un gabarit déterministe.
- Le critère de fin du calibrage (« majorité des épisodes, sans fausse alerte ») doit être produit par `tools/calibrate`.
- En observation seule, préciser où le signal est affiché.

**FR-17 — Imprévus**
- Malade « oui » : traduire les paliers d'au moins 48 h entre J5 et J10 en transformations déterministes (footing ? % de volume ?).
- Malade « non » : la « reprise normale vers J5 » touche-t-elle les Séances de J2 à J4 ?
- Blessé « Autre » → annuler : ajouter explicitement ce déclencheur à la transition *actif* → *annulé* de AD-21.
- Textes d'alarme et de renvoi vers un professionnel : gabarits fixes, non diagnostiques. La règle « non diagnostique » s'applique aussi aux justifications LLM qui mentionnent une blessure : en faire une convention de prompt.

**FR-21 — sensations attendues**
- La table est indexée par l'`intensity_ref` de la **Séance** (squelette), et non par Bloc. Le confirmer dans la story du constructeur.

**FR-22 — modifications**
- Vérifier de façon déterministe, après le LLM, le périmètre d'une demande à l'IA : une semaine ISO au plus, ni ajout ni suppression. Ce point avait déjà été relevé (#8) dans la revue du 23/11.
- Classer le « déplacer » de FR-17 comme `user_modification` et non comme `adjustment` (SM-C1).

**FR-23 — Seuil d'allure**
- Ensemble des distances de meilleurs efforts utilisées pour la régression (addendum : 400, 800, 5 000 m) et nombre minimal de points : paramètres.
- À la confirmation, exclure de la nouvelle résolution la Séance du jour (déjà sur la montre, peut-être déjà courue mais pas encore synchronisée). Sinon, sa Conformité serait jugée contre des cibles que le coureur n'a pas eues.
- L'historique du Profil affiche les changements **confirmés** (date et raison) ; les estimations rejetées restent hors de la vue par défaut.

**NFR-10, addendum §1.2 et §3.1**
- La FC max et la LTHR viennent des zones d'intervals.icu (AD-2). La règle « FC max observée plutôt que Tanaka » ne s'applique donc pas : le noter pour qu'aucune story ne calcule une FC max.
- Appariement natif d'intervals.icu (charge réalisée / prévue, refus sous 50 %) : `attach_run` l'ignore. Le préciser pour qu'aucune story ne lise `paired_event_id`.

**§12 et §5 — spikes**
- OQ-4 : aucun repli si la température est absente (contexte de FR-13). Proposer « non affichée, et l'app le dit ».
- OQ-5 : aucun repli si les laps HealthFit ne suivent pas les Blocs ou si la FC est trop peu échantillonnée. FR-13 retomberait toujours en Feedback global : le dire.
- OQ-6 : IntervalsWellnessSync (addendum §3.1) est un repli possible avant de renoncer au signal sommeil, à évaluer au regard de NFR-2.
- Q1 (une cible par étape sur la montre) n'est pas suivie dans le spine. C'est sans conséquence, puisque FR-9 s'applique quoi qu'il arrive.
- Correction par kilométrage du chrono (addendum §2.2) : différée, à ajouter à « Deferred » à côté de la correction propre à une distance.

## Ce qui est bien couvert (pour mémoire)

- **§15.1, §15.2, §15.16** : familles, moments `always` / `generation_only`, fenêtre de 30 jours sans Séances manquées ou annulées, définition de « augmente le risque », réduction jamais refusée (AD-12).
- **§15.3, §15.4, Q11** : signaux objectifs, Ligne de base ≥ 28 jours, sommeil ingéré, signal ignoré et signalé (AD-2, AD-29, OQ-6).
- **§15.5** : GAP, avec Minetti comme repli dans le domaine, et température ingérée (AD-2, AD-3, OQ-4).
- **§15.7** : facteur sur les allures seulement, repères figés, sortie par `advance_calendar` (AD-28), sous réserve des constats 2 et 5.
- **§15.8, NFR-10** : signaux exclus, non extraits de `bronze`, et `meta.usage` testé en CI (AD-26), sous réserve du constat 12.
- **§15.9, Q3** : Méthode déterministe, liste fermée, bascule à 70 km paramétrée (AD-12, AD-11).
- **§15.10, Q10** : Seuil d'allure possédé par l'app, jamais lu ni écrit chez intervals.icu (AD-27, AD-5).
- **§15.13** : sensations attendues tirées d'une table du code (AD-9).
- **§15.14** : affûtage `generation_only`, paramètre par distance (AD-12, AD-10).
- **§15.15** : spikes avant le code (OQ-4, OQ-5, OQ-6) et outil de calibrage (AD-29), sous réserve du constat 13.
- **FR-3** : emplacement nutrition, Seuil d'allure daté avec sa raison (AD-2, AD-27).
- **Q2** : seuil d'écart du chrono fixé (AD-13).
- **Q7** : semaine allégée chiffrée (AD-12).
