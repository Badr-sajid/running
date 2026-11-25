---
type: review
lens: adversarial
target: ../ARCHITECTURE-SPINE.md
focus: [AD-12, AD-21, AD-26, AD-27, AD-28, AD-29]
companions: [../.memlog.md, ../../../prds/prd-running-2026-11-20/prd.md, review-adversarial.md]
created: '2026-11-25'
---

# Revue adversariale (update du 25/11) : Architecture Spine « running »

## Verdict

Les nouveaux AD fixent bien **qui possède** le Seuil d'allure, le Mode Reprise et le détecteur. Ils ne fixent pas **comment ces objets se croisent dans le temps** : plusieurs versions de seuil en attente, un Plan *proposé* résolu avec un seuil qui n'est pas encore confirmé, des propositions périmées confirmées sur un Plan qui a bougé, un calibrage qui ne peut pas rejouer la règle qu'il calibre. Deux agents qui respectent chaque AD à la lettre livrent des unités incompatibles sur au moins quatre points critiques (T1 à T4). **Il faut fermer T1 à T4 avant de découper les epics Seuil d'allure, Mode Reprise, Imprévus et Surmenage.**

## Méthode

Pour chaque trou, je construis deux unités, des epics ou des stories confiées à deux agents différents. Chacune respecte tous les AD actuels, et leur assemblage casse quand même. Chaque trou se termine par une correction rédigée pour être collée dans le spine.

Sévérité :
- **Critique** : donnée fausse, règle de sécurité ou de NFR-10 violée, ou critère de mise en usage impossible à atteindre.
- **Élevé** : comportement divergent et visible, ou reprise coûteuse.
- **Moyen** : ambiguïté qu'une story tranchera probablement bien, sans aucune garantie.

---

## T1 : un Plan en Mode Reprise est résolu avec le seuil d'avant l'arrêt, et « Reprendre » n'a pas de moment de confirmation (**Critique**)

**Constat.**
- AD-8 dit qu'une référence d'allure se résout « à partir du **Seuil d'allure confirmé** ».
- AD-28 dit que la version `recovery` est confirmée **à l'acceptation** du Plan.
- Pendant toute la relecture, le Plan *proposé* en Mode Reprise n'a donc pas d'autre seuil confirmé que celui d'avant l'arrêt.
- Pour « Reprendre », AD-21 fait passer `suspendu → actif` en une seule transition, avec des Séances régénérées. Il n'y a **pas d'acceptation**, donc aucun moment ne permet de confirmer la version `recovery`. De plus, une régénération par le LLM dure jusqu'à 2 min (AD-6) et ne peut pas tenir dans la transaction de la transition.

**Paire incompatible.**
- *Unité A, story « Constructeur de Blocs »* : elle résout les cibles du Plan candidat avec le dernier seuil `confirmed`, conformément à AD-8. Le Plan de reprise sort avec les allures d'avant l'arrêt.
- *Unité B, story « Mode Reprise »* : elle crée la version `recovery` au statut `estimated` et s'attend à ce que les Blocs l'utilisent. À l'acceptation, elle la confirme, sans re-résolution puisque AD-21 ne prévoit une re-résolution qu'à la confirmation d'un Ajustement.
- **Résultat.** Le Plan accepté et publié porte les allures d'avant l'arrêt alors que le seuil en vigueur est celui de la reprise. Les cibles sont trop rapides au moment le plus risqué, ce qui viole FR-24.
- *Variante « Reprendre ».* L'agent A confirme la version `recovery` dès le clic, donc en silence, ce qui viole FR-23 (« jamais appliqué en silence »). L'agent B attend une acceptation qui n'existe pas : la version reste `estimated` pour toujours et les Séances régénérées utilisent l'ancien seuil.

**Correction : amender AD-8, AD-21 et AD-28.**
- AD-8 : la résolution reçoit **une version de seuil explicite**. Pour un Plan *actif*, c'est la version en vigueur. Pour un Plan *proposé*, c'est la version **candidate épinglée sur le Plan** (`plan.pace_threshold_id`), qui peut être `estimated` si elle a l'origine `recovery`.
- AD-21, nouvelle ligne : `Plan suspendu → suspendu (reprise en génération)` sur « Reprendre ». La régénération est mise en file, et son résultat est une **révision proposée du même Plan**, présentée en relecture. La transition `suspendu → actif` a lieu à l'acceptation de cette révision. Sur le refus, le Plan reste *suspendu*.
- AD-28 : la version `recovery` porte le `plan_id` qui l'a créée. Elle est confirmée par l'acceptation de ce Plan ou de cette révision. Elle passe `rejected` quand ce Plan proposé est remplacé, supprimé ou refusé. Une acceptation vérifie que `plan.pace_threshold_id` est bien la version confirmée, et sinon re-résout avant le commit.

---

## T2 : plusieurs versions de seuil en attente, et un « en vigueur » sans clé d'ordre (**Critique**)

**Constat.** Rien n'interdit plusieurs versions `estimated` à la fois. Rien ne dit non plus si « dernier seuil confirmé à une date donnée » se trie sur la `date` de l'estimation ou sur l'instant de confirmation. Par ailleurs, les déclencheurs de AD-27 utilisent la fenêtre de 16 semaines même en Mode Reprise, alors que AD-28 veut une réestimation « sur les Courses de la reprise ».

**Paire incompatible, scénario A.**
- Une estimation `periodic` est en attente. Elle a été calculée sur 16 semaines qui contiennent les efforts d'avant l'arrêt.
- L'utilisateur accepte un Plan en Mode Reprise, ce qui confirme la version `recovery`.
- Le lendemain, il confirme la proposition `periodic`, qui est toujours valide jusqu'à la fin du jour où elle a été faite (AD-4), ou qui a été recréée.
  - *Agent A (store `pace_threshold`)* trie sur la `date` d'estimation : la version `recovery`, plus récente, reste en vigueur, mais la confirmation `periodic` a déjà re-résolu et republié les Séances (AD-27) avec l'ancienne valeur.
  - *Agent B (Conformité)* trie sur `confirmed_at` : la version `periodic` d'avant l'arrêt est en vigueur.
  - Les cibles publiées, le seuil affiché dans le Profil et la Conformité divergent. Dans les deux cas, le facteur de désentraînement est effacé.

**Paire incompatible, scénario B.**
- Quatre à six semaines après la reprise, `advance_calendar` constate que le seuil `recovery` confirmé dépasse la période de revue.
- *L'agent « estimateur »* applique AD-27 à la lettre : il prend 16 semaines de meilleurs efforts, efforts d'avant l'arrêt compris, et propose de revenir au seuil d'avant l'arrêt.
- *L'agent « Mode Reprise »* s'attend à une estimation limitée aux Courses de la reprise.
- Le déclencheur « nouveau meilleur effort » ne se déclenche jamais pendant la reprise, puisque les efforts d'avant l'arrêt dominent la fenêtre.

**Correction : amender AD-27 et AD-21.**
- **Au plus une version `estimated` par athlète** (index unique partiel). Une nouvelle estimation fait passer la précédente à `rejected`, avec le motif `superseded`. Confirmer une version fait passer toutes les autres versions `estimated` à `rejected`.
- **Clé d'ordre du seuil en vigueur** : `effective_from` est la **date locale de confirmation**, jamais rétroactive. La version en vigueur à une date D est celle dont `effective_from ≤ D` est le plus grand, et `confirmed_at` départage deux versions de même date.
- **Fenêtre d'estimation** : `max(today − 16 sem, plan.recovery_started_on)` tant que le Plan *actif* a `recovery_mode`, puis `max(today − 16 sem, recovery_started_on)` jusqu'à ce que cette borne soit dépassée. Si les données sont trop pauvres pour la régression (moins de 2 distances distinctes, paramètre), aucune proposition n'est faite, et la raison est journalisée.
- **Revue périodique** : elle est suspendue en Mode Reprise. Sa sortie (AD-28) déclenche une seule estimation.

---

## T3 : propositions périmées confirmées sur un Plan qui a bougé (**Critique**)

**Constat.** AD-4 rejette une proposition seulement si « la version a changé **et** qu'elle ne passe plus les garde-fous ». Or les garde-fous ne sont pas la seule cause de péremption. Le spine ne dit pas non plus ce que contient une `proposal` (des valeurs résolues, un diff, une intention ?). Enfin, « ne passe plus » peut s'entendre comme un verdict absolu ou comme une comparaison avant / après (AD-12).

**Paires incompatibles.**
1. **Seuil d'allure contre demande à l'IA.** Une demande à l'IA est calculée sur `plan.version = 7`, avec des Blocs résolus contenant des valeurs absolues, ce que AD-8 rend naturel puisque le `step` stocke la valeur résolue. L'utilisateur confirme d'abord le nouveau seuil (version 8, Séances re-résolues), puis la demande à l'IA. Les garde-fous passent, car une allure ne change aucune distance. La demande à l'IA **réécrit les Séances touchées avec les allures de l'ancien seuil**. L'agent « IA » a stocké des valeurs, l'agent « Seuil » pensait que toute Séance à venir était à jour.
2. **Surmenage contre Malade.** Une proposition de surmenage (allègement de la Séance X) est en attente. L'utilisateur déclare « Malade » et confirme : X passe *annulée*. Il confirme ensuite le surmenage : c'est une réduction, donc les garde-fous passent. L'agent « Ajustement » met à jour les Blocs de X et écrit un `upsert` dans l'outbox. L'agent « Machine à états » n'a aucune transition *annulée* → *à venir*. Selon l'implémentation, on obtient une Séance annulée modifiée ou une Séance ressuscitée dans intervals.icu.
3. **Verdict absolu contre avant / après.** Une Séance longue est manquée, et la fenêtre de 30 jours perd sa référence. La sortie longue suivante dépasse désormais 110 %. Une proposition de réduction est confirmée. L'agent A revérifie avec un verdict absolu et **rejette la réduction**. L'agent B compare avant et après, conformément à AD-12, et l'accepte.

**Correction : amender AD-4, avec une forme de `proposal` fixée.**
- Une `proposal` a les champs `kind` (`disruption`, `overload`, `ai_edit`, `pace_threshold`), `scope` (`plan` ou `athlete`), `base_plan_version` (nul si `scope = athlete`) et `payload`.
- Le `payload` est une **intention en références** : Séances visées, opérations, références d'intensité. Il ne contient **jamais de valeur résolue**. La résolution (AD-8) a lieu à l'application.
- **Confirmation.** Un seul cas d'usage, `confirm_proposal`, sous le verrou du Plan.
  - (a) Toute Séance visée doit encore être *à venir* et ne pas avoir été modifiée depuis `base_plan_version` (`workout.revision`). Sinon, la proposition est rejetée avec le code `PROPOSAL_STALE`.
  - (b) Les propositions déterministes (`disruption`, `overload`, `pace_threshold`) sont **recalculées** sur le Plan courant, et non rejouées.
  - (c) Les garde-fous sont revérifiés par **la même fonction** `check(before, after, moment)` que AD-12, avec `before` égal au Plan courant. Il n'y a jamais de verdict absolu.

---

## T4 : le calibrage ne peut pas rejouer le détecteur, faute de Ressentis dans l'historique (**Critique**)

**Constat.**
- AD-29 rejoue « ce même détecteur » sur l'historique importé.
- Le détecteur exige toujours un ressenti dégradé (FR-16 : 3 🥵 d'affilée, §15.3).
- AD-15 exclut l'historique importé du cycle Ressenti, donc **aucun Ressenti n'existe sur la période de calibrage**.

**Paire incompatible.**
- *Agent A, outil `calibrate`* : pour atteindre le critère de FR-16 (« déclenche sur la majorité des épisodes »), il rejoue seulement les signaux objectifs. Il réécrit donc une variante du détecteur, ce que AD-3 et AD-29 interdisent.
- *Agent B, détecteur* : il exige la condition de ressenti. Sur l'historique, le rejeu ne déclenche jamais, et le calibrage ne peut jamais réussir. AD-19 n'a pas de chemin vers la mise en usage autre que l'observation seule.

**Correction : amender AD-29.**
- Le détecteur est composé de **deux fonctions pures publiques** :
  - `objective_signals(series, params) → {rhr, sleep, baseline_status}` ;
  - `overload_decision(feelings, objective_signals, params)`.
- Le calibrage rejoue **uniquement** `objective_signals`, et son critère porte sur cette fonction : déclenchement sur la majorité des épisodes datés, sans fausse alerte. `overload_decision` est testée par des cas unitaires, pas calibrée.
- `calibrate` n'écrit jamais dans `app` (AD-4). Il produit un fichier de paramètres versionné, qu'un cas d'usage `import_parameter_set` charge.

---

## T5 : le détecteur n'a pas de déclencheur unique ni de dédoublonnage (**Élevé**)

**Constat.** AD-29 définit la fonction mais pas **quand** elle tourne, ni combien de propositions un même épisode peut produire. Le troisième 🥵 arrive par l'`api` (`record_feeling`), alors que la FC au repos du matin arrive par la synchro (file `lake`).

**Paire incompatible.**
- *Agent A* lance le détecteur dans `record_feeling`. Il lit un `gold` de la veille et rate le signal de FC au repos du jour.
- *Agent B* le lance en fin de synchro. Les deux implémentations coexistent, et deux propositions de surmenage sortent pour le même épisode.
- Un quatrième 🥵 le lendemain relance une troisième proposition. Le mode « observation seule » doit « signaler », mais aucune entité ne porte ce signal : l'agent A écrit seulement un log, l'agent B crée une table `overload_signal` absente du tableau de propriété.

**Correction : amender AD-29 et AD-2.**
- Un seul cas d'usage, `evaluate_overload`, appelé en fin de synchro (après `attach_run`) et après `record_feeling`. Il est idempotent.
- L'entité `overload_episode` (`app`) a la clé `(athlete_id, date de déclenchement)`. Au plus un épisode ouvert : un épisode ouvert, ou un allègement de surmenage de moins de 7 jours, empêche toute nouvelle proposition.
- En observation seule, l'épisode est enregistré avec `observation_only = true`, sans `proposal`.

---

## T6 : une re-résolution du seuil est-elle une « hausse d'intensité » ? Et la Séance du jour ? (**Élevé**)

**Constat.**
- AD-12 définit une hausse de risque comme une « hausse de l'intensité d'un Bloc ». Il interdit aussi toute hausse d'une Séance allégée pendant 7 jours après un surmenage.
- AD-4 revérifie les garde-fous dans **tout** cas d'usage qui touche un Plan.
- La confirmation d'un seuil plus rapide augmente les allures absolues des Séances allégées.
- AD-27 re-résout « les Séances à venir », sans dire si la Séance du jour en fait partie.

**Paires incompatibles.**
- *Agent « Garde-fous »* compare les valeurs résolues. Pendant les 7 jours qui suivent un surmenage, la confirmation du seuil est **refusée en bloc** par la revérification de AD-4.
- *Agent « Seuil d'allure »* compare les références, qui sont inchangées. Il passe.
- **La Séance du jour.** À 7 h, la montre a déjà reçu les anciennes cibles. Le seuil est confirmé à 8 h et la Séance du jour est re-résolue. La Course de 18 h est jugée contre la nouvelle valeur stockée (AD-8) et tombe en *non respectée*.

**Correction : amender AD-12 et AD-27.**
- L'intensité se compare **en références** (zone ou allure Daniels), jamais en valeurs résolues. Une re-résolution n'est pas une modification au sens de AD-12 : elle n'exécute aucun garde-fou.
- La re-résolution porte sur les Séances **dont la date est strictement postérieure à aujourd'hui** (`Clock`), et seulement sur les références d'allure.
- La Conformité utilise toujours la valeur résolue stockée sur le `step` (AD-8). « Seuil en vigueur à sa date » (AD-27) ne s'applique qu'aux Courses non planifiées et aux zones d'analyse.

---

## T7 : le moment des garde-fous n'est pas fixé pour chaque cas d'usage (**Élevé**)

**Constat.** AD-12 distingue deux moments : « génération » (`generation_only`, plus le secondaire toujours respecté) et « modification de l'utilisateur » (`always`, plus un avertissement). Plusieurs cas d'usage ne tombent clairement dans aucun des deux :
- la demande à l'IA (une modification de l'utilisateur, mais produite par le LLM) ;
- la régénération après « Reprendre » ;
- les Ajustements déterministes (Malade, Blessé adapté, surmenage) ;
- la re-résolution du seuil ;
- `advance_calendar`.

**Paire incompatible.**
- *Agent « Demander à l'IA »* traite la demande comme une génération. Une proposition qui dépasse +10 % est rejetée par le validateur, réessayée 3 fois, puis abandonnée en échec.
- *Agent « Garde-fous »* la traite comme une modification : elle est enregistrée avec un avertissement.
- L'utilisateur voit un refus pour la demande à l'IA et un simple avertissement pour la même modification faite à la main.
- Inversement, la régénération après « Reprendre » vérifie la bande de la Méthode sur un calendrier fait de semaines suspendues à volume nul, qui comptent toutes comme semaines allégées, et elle échoue de façon imprévisible.

**Correction : amender AD-12 avec une matrice fermée.**

| Cas d'usage | `always` (avant / après) | `generation_only` | Secondaire |
| --- | --- | --- | --- |
| génération, refus avec Consigne, régénération après « Reprendre » | oui (absolu) | oui, sur les semaines **régénérées seulement** | refus |
| modification à la main, déplacement, demande à l'IA | oui | non | avertissement |
| Ajustement déterministe (Imprévu, surmenage) | oui | non | rien |
| re-résolution du seuil, `advance_calendar` | non | non | rien |

Les avertissements sont **persistés** sur `user_modification` (`warnings[]`) pour SM-C1.

---

## T8 : AD-26 laisse fuir des faits descriptifs vers la décision (**Élevé**)

**Constat.** Le marquage `decision` / `descriptive` porte sur un modèle `gold` entier. Trois chemins le contournent.

**Paires incompatibles.**
1. **Modèle mixte.** L'agent « Données » publie un seul `gold_run_streams` (FC, allure, GAP, altitude, température) marqué `decision`, puisque la Conformité en a besoin. La température entre alors dans la famille `decision`. Un autre agent le marque `descriptive`, et la Conformité ne peut plus lire la FC.
2. **Dérivé du domaine.** AD-3 place le découplage allure / FC dans le domaine, et AD-26 classe la « dérive cardiaque indicative » en `descriptive`. Le découplage est calculé à partir de streams `decision`, donc le test CI ne voit rien. L'agent « Matrice / Feedback » l'utilise pour conclure « fatigue probable ».
3. **Blanchiment par le Bilan.** Le Bilan lit les deux familles (AD-26). La génération lit **obligatoirement** le Bilan (AD-24). La tendance de CTL figurant dans `plan_review` arrive dans le prompt du Plan, puis dans les volumes. C'est exactement ce que NFR-10 interdit.

**Correction : amender AD-26.**
- Un modèle `gold` d'interface a **un seul usage**. La CI refuse un modèle qui mélange des colonnes `decision` et `descriptive`. Les streams sont donc séparés (`gold_run_streams_decision`, `gold_run_streams_descriptive`).
- Les sorties du domaine portent aussi un usage. Le découplage est `descriptive` en v1 et ne peut alimenter ni la Matrice, ni un verdict, ni le Seuil de surmenage.
- `plan_review` sépare `decision_facts` et `descriptive_facts`. La génération et la demande à l'IA ne lisent que `decision_facts`. Le prompt Feedback peut recevoir des faits `descriptive`, étiquetés comme tels, mais jamais comme base d'un verdict.

---

## T9 : sortie du Mode Reprise vraie dès le départ, et repères figés incomplets (**Élevé**)

**Constat.**
- **Sortie.** La condition est que « la fenêtre de 30 jours ne contient plus que des Courses postérieures à `recovery_started_on` ». Après un arrêt de 30 jours ou plus, la fenêtre ne contient **aucune** Course : la condition est vraie dès le départ.
- **Repères figés.** Le garde-fou « première Séance ≤ la plus longue Séance de cette semaine-là » a besoin de la plus longue Course de la dernière semaine normale. AD-28 ne fige que le volume. « Normale » et « première semaine » ne sont pas définies.

**Paire incompatible.**
- *Agent « advance_calendar »* applique la condition à la lettre : le Plan sort du Mode Reprise **le lendemain** de la reprise, et le plafond de 110 % se recalcule sur les Séances à venir.
- *Agent « Garde-fous »* suppose environ 30 jours de reprise.
- Pour la première Séance, l'agent A recalcule la plus longue Course depuis `gold`, ce qui viole « ne recalcule jamais ». L'agent B attend un repère figé qui n'existe pas.
- Pour la première semaine, l'un prend la semaine ISO de `recovery_started_on` (1 jour si la reprise tombe un dimanche), l'autre prend 7 jours glissants.

**Correction : amender AD-28.**
- Les repères figés sont :
  - `recovery_started_on` ;
  - `last_run_before_break_on` ;
  - `last_normal_week_start` (dernière semaine ISO **non allégée** au sens de AD-12, avant l'arrêt) ;
  - `last_normal_week_volume_m` ;
  - `last_normal_week_longest_m`.
- **Première semaine** : la première semaine ISO **complète** commençant le jour même ou après `recovery_started_on`. Les jours qui précèdent ce lundi sont soumis au même plafond, au prorata.
- **Sortie** : la sortie a lieu au premier jour local D tel que `D − 30 j > last_run_before_break_on` **et** `D ≥ recovery_started_on + 14 j` (paramètre). Il faut donc qu'aucune Course d'avant l'arrêt ne reste dans la fenêtre, avec une durée minimale de reprise. Badr doit valider la durée minimale, qui est une question produit.

---

## T10 : « Malade » et l'inactivité longue sur un Plan *actif* n'entrent jamais en Mode Reprise (**Moyen**, arbitrage produit)

**Constat.**
- Le Mode Reprise n'est évalué qu'à la génération et au « Reprendre ». Un « Malade, fièvre » (`actif → actif`) suivi d'une rechute, ou deux semaines de Séances *manquées*, laissent le Plan *actif* après 14 jours sans Course.
- Le plafond de 110 % reste alors calculé sur les Séances à venir, que AD-12 compte dans la fenêtre.
- Par ailleurs, un « Malade » déclaré le lendemain (« j'étais malade hier ») tombe sur une Séance déjà *manquée* (AD-20, 03:00). AD-21 n'a pas de transition *manquée* → *annulée*.

**Paire incompatible.**
- *Agent « Imprévus »* autorise une date de début passée et fait passer la Séance de *manquée* à *annulée*, une transition inventée.
- *Agent « Bilan »* compte cette Séance comme *manquée*.
- Pour l'inactivité, l'agent « Malade » suppose que le Mode Reprise prendra le relais, et l'agent « Mode Reprise » ne le déclenche jamais sur un Plan *actif*.
- Deux déclarations « Malade » qui se chevauchent empilent deux protocoles, avec un allègement appliqué sur une Séance déjà allégée.

**Correction : amender AD-20 et AD-21.**
- `advance_calendar` détecte au moins 14 jours sans Course (paramètre) sur un Plan *actif*. Il crée alors une `proposal` de type `recovery_regeneration`, qui passe par la même relecture que T1.
- Un Imprévu ne s'applique qu'à partir d'aujourd'hui. Une Séance *manquée* reste *manquée*.
- Au plus un Imprévu actif de chaque type par Plan. Une nouvelle déclaration **remplace** le protocole en cours : elle recalcule à partir du protocole et n'empile rien.

---

## T11 : revue périodique en boucle, et proposition de seuil sans Plan (**Moyen-Élevé**)

**Constat.**
- **Boucle.** Le déclencheur périodique teste « le seuil **confirmé** est plus ancien que la période de revue ». Après un refus, ou une expiration en fin de journée (AD-4), le seuil confirmé reste ancien : la condition reste vraie chaque jour.
- **Proposition sans Plan.** Le diagramme ERD rattache `proposal` à `PLAN`, alors qu'une estimation peut survenir sans Plan *actif* : entre deux Plans, ou pendant une suspension.

**Paire incompatible.**
- *Agent « advance_calendar »* repropose chaque matin, ce qui fait une sheet bloquante quotidienne.
- *Agent « UX »* attend une revue toutes les 4 à 6 semaines.
- Sans Plan *actif*, l'agent A ne fait aucune estimation, et l'agent B crée une `proposal` avec `plan_id` nul, que `confirm_proposal` rejette faute de `plan.version`.

**Correction : amender AD-27.**
- La période se mesure depuis **la dernière estimation, quel que soit son statut**.
- Les propositions `pace_threshold` ont `scope = athlete` (T3). Leur confirmation sans Plan *actif* confirme le seuil sans rien re-résoudre.
- Si un Plan *proposé* existe, il est re-résolu avant d'être affiché (T1).
- Ordre dans `pending-actions` (AD-18) : `disruption` > `overload` > `pace_threshold` > `ai_edit`.

---

## T12 : quelle version de `parameter_set` pour la résolution et le détecteur ? (**Moyen**)

**Constat.** AD-11 sépare deux cas : la version **courante** pour un « nouveau calcul », et la version **épinglée** pour revérifier un Plan. Le spine ne range ni la re-résolution des allures (table de pourcentages), ni le détecteur, ni les protocoles Malade et Blessé dans l'une de ces deux catégories.

**Paire incompatible.** Badr importe une nouvelle table de pourcentages en cours de Plan.
- *Agent « Seuil d'allure »* re-résout avec la table courante.
- *Agent « Constructeur de Blocs »* résout une Séance modifiée à la main avec la table épinglée du Plan.
- Deux Séances du même Plan, avec le même seuil et la même référence `T`, portent des allures différentes.

**Correction : amender AD-11.**
- Version épinglée par le Plan : toute résolution de cible et tout protocole d'Ajustement d'un Plan.
- Version courante : le détecteur de surmenage, l'estimation du seuil, la Conformité et le Feedback, qui l'enregistrent.
- Une nouvelle version n'atteint un Plan existant qu'à sa prochaine génération.

---

## T13 : un changement des zones FC dans intervals.icu n'a pas de propriétaire (**Moyen**)

**Constat.** Les zones FC sont relues à chaque synchro (AD-2, AD-8). AD-27 ne fait re-résoudre les Séances que sur confirmation d'un seuil d'**allure**.

**Paire incompatible.**
- *Agent A* re-résout tous les `step` à chaque re-résolution, références FC comprises. Les cibles FC changent alors **en silence** quand Badr touche ses zones dans intervals.icu.
- *Agent B* ne re-résout jamais les références FC : les Séances publiées gardent les anciennes zones indéfiniment.

**Correction : amender AD-8.** Une référence FC est résolue à la création ou à la reconstruction du `step`, puis figée. La synchro qui constate un changement des zones FC met `sync_status` en mode informatif (« zones FC modifiées dans intervals.icu, appliquées au prochain Plan »). Aucune re-résolution automatique n'a lieu.

---

## T14 : « aujourd'hui » dans le calendrier unifié (**Moyen**)

**Constat.** AD-12 dit « réalisé pour les jours passés, prévu pour les jours à venir », sans classer aujourd'hui.

**Paire incompatible.** Le `dry_run` de l'`api` classe aujourd'hui parmi les jours passés : il compte le réalisé, soit 0 avant la synchro. La confirmation dans le worker le classe parmi les jours à venir et compte la Séance prévue. Le bouton « Enregistrer » est actif, puis le refus tombe à la confirmation.

**Correction : amender AD-12.** Le jour J (`Clock`) compte les Courses réalisées de J **plus** les Séances de J encore *à venir*. Une seule fonction `unified_calendar(as_of)` sert au `dry_run` et à la confirmation.

---

## T15 : la tâche quotidienne `advance_calendar` n'a pas de file (**Moyen**)

**Constat.** AD-20 lance aussi `advance_calendar` à 03:00, mais AD-6 ne range que la synchro complète dans la file `lake`. AD-24 exige qu'une synchro précède la transition vers *terminé*. Les estimations du seuil, qui concernent l'athlète, ne sont pas sous le verrou du Plan.

**Paire incompatible.**
- *Agent A* met la tâche de 03:00 dans une file par défaut. Elle tourne en même temps qu'une synchro, et deux estimations sortent ensemble. Le Plan passe *terminé* sans synchro, et le Bilan est figé sur des faits incomplets.
- *Agent B* suppose que tout passe dans la file `lake`.

**Correction : amender AD-20 et AD-6.** La tâche de 03:00 **est une synchro complète** dans la file `lake`, avec le même `queueing_lock`. Toute écriture dans `pace_threshold` prend un verrou consultatif par athlète (`pg_advisory_xact_lock`), en plus de l'index unique de T2.

---

## Synthèse

| # | Trou | Sévérité | AD à amender |
| --- | --- | --- | --- |
| T1 | Plan de reprise résolu avec le seuil d'avant l'arrêt ; « Reprendre » sans acceptation | Critique | AD-8, AD-21, AD-28 |
| T2 | Plusieurs seuils en attente, ordre du seuil en vigueur, fenêtre d'estimation en reprise | Critique | AD-27, AD-21 |
| T3 | Propositions périmées ; forme de `proposal` ; vérification absolue contre avant / après | Critique | AD-4, AD-12 |
| T4 | Calibrage impossible sans Ressentis | Critique | AD-29 |
| T5 | Déclencheur et dédoublonnage du détecteur | Élevé | AD-29, AD-2 |
| T6 | Re-résolution vue comme une hausse d'intensité ; Séance du jour | Élevé | AD-12, AD-27 |
| T7 | Matrice des moments des garde-fous par cas d'usage | Élevé | AD-12 |
| T8 | Fuites descriptives : modèle mixte, découplage, Bilan | Élevé | AD-26, AD-24 |
| T9 | Sortie du Mode Reprise vraie dès le départ ; repères incomplets | Élevé | AD-28 |
| T10 | Malade ou inactivité sur un Plan actif sans Mode Reprise ; Malade rétroactif | Moyen | AD-20, AD-21 |
| T11 | Revue périodique en boucle ; proposition sans Plan | Moyen-Élevé | AD-27, AD-18 |
| T12 | Version de `parameter_set` pour la résolution et le détecteur | Moyen | AD-11 |
| T13 | Zones FC modifiées sans propriétaire | Moyen | AD-8 |
| T14 | « Aujourd'hui » dans le calendrier unifié | Moyen | AD-12 |
| T15 | File de la tâche quotidienne, concurrence sur `pace_threshold` | Moyen | AD-20, AD-6 |

**Questions produit pour Badr :**
- la durée minimale du Mode Reprise (T9) ;
- le déclenchement du Mode Reprise sur un Plan *actif* après 14 jours sans Course (T10) ;
- l'ordre des propositions dans `pending-actions` (T11).

Les autres corrections ne demandent pas d'arbitrage produit.
