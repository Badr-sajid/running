---
type: review
lens: adversarial
target: ../ARCHITECTURE-SPINE.md
companions: [../.memlog.md, ../../../prds/prd-running-2026-11-20/prd.md]
created: '2026-11-23'
---

# Revue adversariale — Architecture Spine « running »

## Verdict

Le spine fixe bien les **frontières** (magasins, sens des dépendances, outbox, LLM validé), mais il laisse ouverts presque tous les **comportements dans le temps** : qui déclenche les transitions temporelles, quelles transitions existent, qui écrit `run_attachment`, comment le Bilan de fin de Plan est figé, et quelle version de `parameter_set` s'applique. Deux équipes qui respectent chaque AD à la lettre peuvent donc livrer des unités qui ne fonctionnent pas ensemble. **Le spine n'est pas prêt à découper en epics tant que les trous C1 à C4 ne sont pas fermés.**

## Méthode

Pour chaque trou, je construis une **paire d'unités** (epics ou stories) confiées à deux agents différents. Chacune respecte tous les AD existants, et pourtant leur assemblage casse. Chaque trou se termine par un AD nouveau ou durci, rédigé pour être collé dans le spine.

Sévérité :
- **Critique** : données fausses ou perdues, règle du PRD violée en production.
- **Élevé** : comportement divergent et visible, ou reprise coûteuse.
- **Moyen** : ambiguïté qu'une story tranchera sans doute bien, mais sans garantie.

---

## C1 — Personne ne possède les transitions temporelles (Séance *manquée*, Plan *terminé*) — **Critique**

**Constat.** FR-18 (« passe à l'état *manquée* le lendemain ») et FR-7 (« *terminé* le lendemain de la dernière Séance ») sont des transitions **déclenchées par l'horloge**. AD-4 dit que toute transition passe par la machine à états, dans un cas d'usage. AD-14 dit que le jour se calcule dans le fuseau de l'athlète. Mais **aucun AD ne nomme le déclencheur** : ni tâche périodique, ni évaluation à la lecture, ni port Horloge. Le domaine est pur (AD-1), donc il ne peut pas lire l'heure lui-même.

**Paire incompatible.**
- *Unité A — epic « Séances manquées » (FR-18)* : l'agent ajoute une tâche Procrastinate périodique à 00 h 05 UTC qui appelle `mark_missed_workouts`. Il respecte AD-4 (cas d'usage, transaction) et AD-6 (worker).
- *Unité B — epic « Accueil » (FR-20)* : l'agent dérive l'état à la lecture. Si `date < today` et qu'il n'y a pas de rattachement, il renvoie `missed` sans rien écrire, puisque rien ne lui interdit un état calculé. Il respecte AD-18, car c'est l'API qui fournit l'état.
- **Résultat.** À 00 h 30 heure de Paris (23 h 30 UTC), B affiche « manquée » alors que A n'a encore rien écrit. Le Bilan (dbt, qui lit `app`) compte 0 manquée, l'accueil en montre 1. Un autre agent, qui prend `date.today()` du serveur au lieu du fuseau, viole AD-14 sans qu'aucun test d'architecture ne le voie, faute de port Horloge.

**Deuxième paire : la Course en retard.** HealthFit est ouvert le surlendemain (NFR-7 l'accepte). La Séance du lundi est déjà *manquée* quand la Course du lundi arrive mercredi. L'unité « Détection » (AD-15) applique la règle du même jour et rattache. L'unité « Machine à états » n'a pas prévu `manquée → réalisée`, et lève une erreur ou laisse une Séance *manquée* avec une Course rattachée. Le PRD ne tranche pas le cas, et le spine non plus.

**AD proposé — AD-20 « Horloge et transitions temporelles ».**
- Le domaine reçoit `today` (date locale de l'athlète) en paramètre. Seul un **port `Clock`** le fournit, via la fonction unique de AD-14. Aucun appel à `date.today()`, `datetime.now()` ou `now()` SQL hors de l'adaptateur `Clock`, avec une règle d'import ou de lint en CI.
- Les états *manquée* et *terminé* sont **stockés**, jamais dérivés à la lecture. Un seul cas d'usage idempotent, `advance_calendar(athlete_id, today)`, applique toutes les transitions temporelles.
- Il est appelé (1) au début de **chaque** chaîne de synchro, avant la détection, et (2) par une tâche Procrastinate périodique quotidienne à 00 h 15 **heure locale de l'athlète**, avec un `queueing_lock` par athlète.
- *En attente de la course* est un **état dérivé d'affichage** (`date == today`, sans rattachement, état stocké *à venir*). Il n'est jamais persisté.
- La transition `manquée → réalisée` est **permise** par rattachement d'une Course datée du jour de la Séance (ou de la veille ou du lendemain, après confirmation), tant que le Plan n'est pas *terminé* ou *annulé*. Le Bilan est alors recalculé (voir C3).

---

## C2 — Machines à états non spécifiées : chaque epic inventera ses transitions — **Critique**

**Constat.** AD-4 dit qu'« une machine à états du domaine fixe seule les transitions permises », mais le spine ne donne **pas la table**. Les états du PRD laissent beaucoup de cas sans réponse.

| Question | Interprétation A | Interprétation B |
| --- | --- | --- |
| Refus avec Consigne (FR-6) | nouveau `plan` *proposé*, l'ancien est supprimé | même `plan`, nouvelle version |
| « Reprendre » après blessure (FR-17, FR-19 : passe par la génération) | même Plan, `suspendu → actif`, Séances régénérées | nouveau Plan, l'ancien est *annulé* |
| Séances d'un Plan *suspendu* dont la date passe | deviennent *manquées* (FR-18 littéral) | restent *à venir*, gelées |
| Dernière Séance *annulée* ou déplacée | *terminé* le lendemain de la dernière Séance **non annulée** | le lendemain de la date initiale |
| Plan *suspendu* qui dépasse la date de l'Objectif | passe à *terminé* | reste *suspendu* sans fin |
| Plusieurs Plans *proposés* en même temps (régénérations, nouvel Objectif) | autorisé | un seul à la fois |
| Séances du Plan annulé par l'acceptation d'un nouveau Plan | *annulées* | supprimées |

**Paire incompatible.**
- *Unité A — epic « Imprévus » (FR-17)* : « Reprendre » crée un nouveau Plan, ce qui produit de nouveaux UUID de Séance et donc de nouveaux `external_id`.
- *Unité B — epic « Publication » (AD-5)* : elle ne publie qu'un upsert des Séances du Plan actif et ne connaît pas les Séances de l'ancien Plan.
- **Résultat.** Les anciennes Séances restent dans le calendrier intervals.icu et sur la montre, en doublon des nouvelles. Cela viole FR-9 (aucun doublon) et SM-7, alors que A et B respectent AD-4 et AD-5. Dans le même temps, le Bilan (FR-8) compte les Séances suspendues comme *manquées* selon l'interprétation, ce qui fausse SM-6.

**AD proposé — AD-21 « Tables de transitions normatives ».** Le spine contient la table complète (état source, événement, état cible, garde, effets). Voici le minimum :

- **Plan** :
  - `proposé → actif` (acceptation : publie toutes les Séances, annule l'ancien Plan actif ou suspendu dans la même transaction) ;
  - `proposé → (supprimé)` (refus avec Consigne : la régénération crée un **nouveau** `plan` *proposé* ; au plus un Plan *proposé* par athlète) ;
  - `actif → suspendu` (Blessé) ;
  - `suspendu → actif` (Reprendre : **même** `plan`, Séances futures régénérées ; les Séances remplacées passent à *annulée* et émettent leur suppression) ;
  - `actif|suspendu → annulé` (Blessé > annuler, ou acceptation d'un autre Plan) ;
  - `actif → terminé` (via `advance_calendar` : `today` > date de la dernière Séance non annulée) ;
  - `suspendu → annulé` automatique quand `today` > date de l'Objectif.
- **Séance** :
  - `à venir → réalisée` (rattachement) ;
  - `à venir → manquée` (`advance_calendar`, **seulement si le Plan est *actif***) ;
  - `à venir → annulée` (Imprévu, Ajustement, annulation ou suspension du Plan) ;
  - `manquée → réalisée` (rattachement tardif, voir C1) ;
  - `réalisée`, `annulée` : terminaux.
  - Pendant une suspension, les Séances futures **restent *à venir*** et ne sont pas marquées *manquées*.
- **Règle de publication liée** : toute transition vers *annulée*, et toute Séance d'un Plan qui passe à *annulé*, émet dans la même transaction une ligne d'outbox `delete(external_id)`.

---

## C3 — `plan_review` : dans `gold`, « stocké dans le Profil », lu par la génération suivante, mais jamais figé ni prêt à temps — **Critique**

**Constat.** AD-2 place `plan_review` dans `gold`, calculé par dbt à partir des faits et de `app`. FR-8 le veut « stocké dans le Profil » et **lu par la génération du Plan suivant**. Trois défauts en découlent.

1. **Ordre temporel impossible.** FR-7 permet de créer un nouvel Objectif *pendant* un Plan actif. La génération du Plan 2 a lieu **avant** l'annulation du Plan 1, qui n'intervient qu'à l'acceptation. Le Bilan du Plan 1 n'existe donc pas encore au moment où FR-5 doit le lire. Même pour un Plan *terminé*, le Bilan n'apparaît dans `gold` qu'au prochain `dbt build`, c'est-à-dire à la prochaine synchro, puisque AD-6 ne lance dbt que dans la chaîne de synchro.
2. **Non figé.** `gold` est « reconstructible » : chaque `dbt build` recalcule le Bilan à partir de l'état courant de `app` et des faits. Plusieurs événements modifient alors rétroactivement un Bilan déjà consulté, et déjà injecté dans la raison du Plan 2 (SM-6) : un rattachement tardif (C1), une activité modifiée ou supprimée dans intervals.icu, une nouvelle version de `parameter_set` lue par dbt (C6).
3. **Cycle de dépendance déguisé.** Le cycle part de `gold` (meilleur effort, plus longue sortie) qui sert aux garde-fous du domaine. Ceux-ci écrivent `plan`, `compliance` et `workout` dans `app`. dbt relit ces tables pour produire `gold_plan_review`, que relit la génération suivante. Ce n'est pas une boucle dans un même build, mais l'exactitude du Bilan dépend de l'**ordre** entre transaction `app` et `dbt build`, et aucun AD ne le fixe.

**Paire incompatible.**
- *Unité A — epic « Bilan » (FR-8)* : un modèle `gold_plan_review` matérialisé en table, recalculé à chaque build. Il respecte AD-2 et AD-3.
- *Unité B — epic « Génération » (FR-5)* : elle lit « le dernier Bilan » via le port Faits et, s'il manque, envoie un prompt sans Bilan, puisque rien ne l'interdit.
- **Résultat.** Le Plan 2 généré juste après l'acceptation du terme du Plan 1, sans synchro intermédiaire, ignore le Bilan du Plan 1. SM-6, critère **bloquant**, échoue en silence.

**AD proposé — AD-22 « Bilan de fin de Plan figé ».** Je recommande l'option (a).
- **(a) Recommandé : le Bilan est un instantané `app`.** Le Bilan passe dans la colonne *calculs par Plan* de AD-3 (domaine). À la transition `→ terminé|annulé`, le cas d'usage calcule le Bilan en Python à partir de l'état `app` et d'agrégats **lus** dans `gold` via le port Faits (CTL et records aux bornes du Plan). Il l'écrit en **ligne immuable** `plan_review` dans `app`, dans la même transaction, avec `parameter_set_id` et `facts_snapshot_id` (l'identifiant du snapshot DuckLake lu). `gold` ne contient plus de Bilan. Un rattachement tardif après *terminé* ne le modifie pas, puisque la transition C1 est interdite dans ce cas.
- **(b) Alternative, si le Bilan doit rester dans dbt** : la transition `→ terminé|annulé` enfile dans la même transaction un `dbt build --select gold_plan_review+` sous le verrou `lake`. Le modèle est **incrémental en ajout seul**, une ligne par `plan_id`, jamais recalculée. La génération (FR-5) **échoue explicitement** (`PLAN_REVIEW_PENDING`) si le dernier Plan clos n'a pas de Bilan.
- Dans les deux cas, on ajoute **la règle du Plan de remplacement** : pour un nouvel Objectif créé pendant un Plan actif, la génération reçoit un **Bilan provisoire** du Plan en cours, calculé par la même fonction avec `as_of = today` et non persisté. Il faut amender FR-5 et FR-8 pour le dire.
- On retire de AD-2 la ligne « `plan_review` → lake (`gold`) », ou on la remplace selon l'option retenue.

---

## C4 — `run_attachment` a plusieurs écrivains et aucune règle de concurrence — **Critique**

**Constat.** `run_attachment` est écrit (1) par la détection dans la chaîne de synchro (worker, sous verrou `lake`), (2) par la confirmation d'un rattachement veille ou lendemain (api, FR-11), et (3) indirectement par un déplacement (FR-22), qui change la date de référence. AD-15 ne pose l'unicité que sur l'**activité**, pas sur la **Séance**. AD-4 impose une transaction, mais aucun verrou ni contrôle de version sur le Plan pour ces écritures. La version de Plan n'est définie que pour les propositions.

**Paire incompatible 1 : écriture contre détection.**
- *Unité A — FR-22 « Déplacer »* (api) : elle lit la Séance du jour, sans rattachement donc modifiable, et la déplace à jeudi en une transaction.
- *Unité B — Détection* (worker) : au même instant, elle voit une Course du jour et rattache la Séance d'aujourd'hui, qu'elle a lue avant le commit de A.
- **Résultat.** On obtient une Séance déplacée à jeudi avec une Course de lundi rattachée, ou une Séance « verrouillée » quand même déplacée, puis une Publication d'une Séance réalisée sur une date future. Les deux unités respectent AD-4.

**Paire incompatible 2 : la plus longue du jour.**
- FR-11 dit que « la plus longue est rattachée ». La détection de 8 h rattache la Course de 5 km, puis l'utilisateur pose son Ressenti et le Feedback est généré. À 19 h, une Course de 12 km arrive. Une unité « Détection » littérale **réattache** la Séance à la Course de 12 km, et les Ressenti et Feedback de la Course de 5 km pointent vers une Séance qui n'est plus la leur. Une autre unité laisse la Course de 12 km non planifiée. Les deux lisent FR-11 correctement.

**Paire incompatible 3 : rattachement proposé et Ressenti.** FR-12 demande le Ressenti « dès que la Course est récupérée ». FR-11 demande pour la veille ou le lendemain une confirmation d'un tap. Si l'unité « Ressenti » demande le Ressenti d'abord et que l'unité « Conformité » calcule à la détection, la Conformité d'une Course en attente de confirmation est calculée comme **non planifiée** (Feedback global). Elle n'est jamais recalculée après la confirmation, car AD-15 limite à « au plus un » Feedback.

**AD proposé — AD-23 « Rattachement : un écrivain, verrou par Plan, immuable après Ressenti ».**
- Toutes les écritures sur `run_attachment` passent par **un seul cas d'usage**, `attach_run`, appelé par la détection et par la confirmation.
- **Verrou par Plan** : tout cas d'usage qui touche un Plan ou ses Séances (rattachement, déplacement, modification, Ajustement, `advance_calendar`, Publication enfilée) commence par `SELECT … FROM plan WHERE id = :id FOR UPDATE` et incrémente `plan.version`. La version de Plan d'AD-4 est **cette** colonne : elle s'incrémente à **toute** modification des Séances, de leurs dates, de leurs Blocs ou de leurs états, rattachement compris.
- Contrainte d'unicité partielle `UNIQUE (workout_id) WHERE workout_id IS NOT NULL`.
- **Premier arrivé, définitif** : une fois un Ressenti posé sur un rattachement, celui-ci est immuable. La règle « la plus longue » ne s'applique qu'entre les Courses **détectées dans le même lot de synchro**. Une Course arrivée plus tard le même jour est non planifiée.
- **Ordre** : un rattachement *proposé* (veille ou lendemain) doit être tranché **avant** la demande de Ressenti de cette Course. L'écran Ressenti affiche d'abord la question de rattachement d'un tap, puis les 3 emojis. Conformité et Feedback ne sont calculés qu'après la pose du Ressenti (voir C5).

---

## C5 — Le verrou Ressenti → Feedback et le budget de 30 s n'ont pas de propriétaire — **Élevé**

**Constat.** FR-12 interdit d'afficher **le moindre chiffre** avant le Ressenti. AD-18 interdit les règles métier dans le front, donc c'est l'API qui doit retenir les chiffres. Mais aucun AD ne dit **quels endpoints** sont concernés, ni **quand** Conformité et Feedback sont calculés. NFR-9 impose moins de 30 s entre Ressenti et Feedback, et aucun AD ne relie ce budget à la chaîne `worker`, LLM et lecture de `gold`.

**Paire incompatible.**
- *Unité A — « Synchro » (FR-10)* : elle affiche « dernière Course reçue : 10,2 km en 52 min ». Elle respecte AD-18, puisque la donnée vient de l'API.
- *Unité B — « Profil » (FR-2, FR-3)* : elle montre les records et la plus longue sortie sur 30 jours, déjà mis à jour par le `dbt build` de la synchro qui vient de récupérer la Course.
- *Unité C — « Ressenti »* : elle masque correctement les chiffres sur l'écran Feedback.
- **Résultat.** Les chiffres de la Course fuient avant le Ressenti par deux autres écrans, ce qui viole FR-12, alors que chaque unité est conforme.

**Paire temporelle.**
- *Unité D — « Conformité »* : elle calcule la Conformité au moment de la **détection** et la stocke, ce qui va plus vite.
- *Unité E — « Feedback »* : elle recalcule la Conformité dans la tâche Feedback, par souci de cohérence de version.
- **Résultat.** On obtient deux `compliance` pour un rattachement (l'unicité de AD-15 porte sur Ressenti et Feedback, pas sur la Conformité), ou une violation de contrainte en production. Par ailleurs, si E enfile la tâche Feedback **après** le commit du Ressenti dans une autre transaction, un crash entre les deux laisse un Ressenti sans Feedback, sans jamais de relance.

**AD proposé — AD-24 « Porte du Ressenti et chaîne Feedback ».**
- **Porte côté API** : tant qu'une Course rattachée ou non planifiée n'a pas de `feeling`, **aucune** réponse d'API n'expose de valeur dérivée de cette Course : distance, durée, allure, FC, Conformité, mais aussi records, plus longue sortie ou CTL mis à jour par elle. Les endpoints Profil et Synchro renvoient pendant ce temps les valeurs `gold` **du snapshot précédant l'ingestion de cette Course**, ou un indicateur `pending_feeling`. Un test d'intégration parcourt tous les endpoints GET pour le vérifier.
- **Création** : `record_feeling` crée dans **la même transaction** la ligne `feeling` et la tâche Procrastinate `generate_feedback`, enfilée sur la même connexion. `compliance` est calculée **une seule fois**, dans cette tâche, avec `UNIQUE(run_attachment_id)`.
- **Budget de 30 s** : la tâche Feedback ne prend **jamais** le verrou `lake` et ne lance jamais dbt. Elle lit `gold` dans l'état présent, ce qui suffit puisque la détection a eu lieu après le `dbt build`. Elle tourne dans une file Procrastinate `interactive`, distincte de `sync`, avec au moins un slot réservé, pour qu'un `dbt build` ou une génération de Plan ne la bloque pas.
- **Tentatives** : `retry=0` côté Procrastinate pour toute tâche qui appelle le LLM. Les 3 tentatives de AD-7 et les 2 relances de FR-14 sont comptées par `llm_call` (`feedback_id`, `cycle`, `attempt`). Un redémarrage du worker en cours de tâche reprend au cycle courant sans dépasser ces limites.

---

## C6 — `parameter_set` : quelle version s'applique, et dbt peut-il l'utiliser ? — **Élevé**

**Constat.** AD-11 rend les versions immuables, et chaque `plan`, `compliance` et `feedback` enregistre la sienne. Mais le spine ne dit pas **laquelle choisir** quand la version courante de l'athlète diffère de celle du Plan, ni **si dbt a le droit** d'utiliser des paramètres.

**Paire incompatible 1 : version épinglée ou courante.**
- *Unité A — « Modification à la main » (FR-22)* : elle revérifie les garde-fous avec la version **du Plan**.
- *Unité B — « Ajustement de surmenage » (FR-16)* : elle utilise la version **courante**.
- **Résultat.** Après un recalibrage (par exemple +10 % ramené à +8 %), un même Plan est accepté par A et refusé par B pour la même semaine. Si l'on applique partout la version courante, un recalibrage rend d'un coup le Plan actif invalide, et **toute** modification est refusée à cause de violations antérieures (AD-4 revérifie « toutes les Séances restantes »).

**Paire incompatible 2 : dbt et paramètres.**
- *Unité C — modèle `gold` FC au repos* : elle expose `resting_hr_avg_7d` avec la fenêtre de 7 jours **codée en dur** dans le SQL, ce qui est un agrégat et relève donc de dbt selon AD-3.
- *Unité D — Seuil de surmenage (domaine)* : elle lit la fenêtre dans `parameter_set` (FR-16 : « les valeurs du seuil sont des paramètres »).
- **Résultat.** Si la fenêtre passe à 10 jours, D croit lire une moyenne sur 10 jours et C sert toujours 7 jours. C'est la divergence Python/SQL que AD-3 voulait justement empêcher. Le même risque vaut pour la « plus longue sortie sur 30 jours », dont la fenêtre et le seuil de 110 % sont des garde-fous paramétrables.

**AD proposé — AD-25 « Résolution des paramètres et paramètres interdits à dbt ».**
- **Règle de résolution** :
  - garde-fous d'un Plan existant (FR-19, FR-22, FR-16) : version **épinglée sur le Plan** ;
  - génération et régénération : version **courante**, qui devient l'épinglage du nouveau Plan ou de la nouvelle version ;
  - Conformité, Matrice, Feedback : version **courante** au moment du calcul, enregistrée.
- **dbt n'utilise aucun paramètre de `parameter_set`.** `gold` expose des **séries brutes ou journalières** : FC au repos par jour, distance de chaque Course par jour, CTL par jour, meilleur effort par Course. Toute fenêtre, tout seuil et toute tolérance s'appliquent dans le domaine. Si un modèle `gold` a besoin d'une constante, elle est fixe, nommée dans `meta`, et ne figure **pas** dans `parameter_set`.
- Conséquence sur AD-3 : la liste des agrégats `gold` devient « séries et historiques non paramétrés ». La « plus longue sortie sur 30 jours » devient « distance maximale par jour » ; la fenêtre de 30 jours et la fusion avec les Séances planifiées se font dans le domaine (voir C7).

---

## C7 — Garde-fou « 110 % de la plus longue Course ou Séance planifiée » : impossible dans `gold` — **Élevé**

**Constat.** AD-3 met la « plus longue sortie sur 30 jours » dans `gold`. Or FR-5 et AD-12 demandent le maximum sur les **Courses et Séances planifiées**, fenêtre qui **inclut les Séances du Plan en cours de génération**. Ce Plan n'est pas encore persisté : il est en mémoire, dans le validateur. dbt ne peut pas le voir. Il ne verrait de toute façon les Séances persistées qu'au prochain build.

**Paire incompatible.**
- *Unité A — dbt* : elle construit `gold_longest_30d` à partir de `silver_runs` **et** de `app.workout`, que dbt peut lire (AD-2).
- *Unité B — validateur de garde-fous* : elle lit cette valeur et la combine avec les Séances candidates.
- **Résultat.** Les Séances déjà persistées sont comptées deux fois, ou celles qui ont été annulées ou déplacées depuis le dernier build sont comptées à tort. Les verdicts diffèrent entre génération (Plan en mémoire) et modification (Plan persisté plus `gold` obsolète).

La même ambiguïté touche « +10 % par rapport à la **dernière semaine non allégée** » : semaine **prévue** ou **réalisée** ? Sur les semaines passées d'un Plan, une unité prendra le volume couru (`gold`), une autre le volume planifié (`app`).

**AD proposé — durcir AD-12 et AD-3.**
- Les garde-fous de volume s'évaluent sur un **calendrier unifié en mémoire**, construit par une seule fonction du domaine : pour chaque jour passé, la distance **courue** lue dans `gold`, série brute par jour ; pour chaque jour présent ou futur, la distance **planifiée**, issue des Séances candidates ou persistées non annulées.
- « Semaine » signifie semaine ISO, du lundi au dimanche, dans le fuseau de l'athlète.
- « Dernière semaine non allégée » se détermine sur ce calendrier unifié.
- dbt ne lit jamais `app.workout` pour un calcul de garde-fou.

---

## C8 — Outbox : ordre, coalescence et « suppression de l'ancienne date » contradictoire — **Élevé**

**Constat.** AD-5 fixe `external_id` = UUID de la Séance, stable après déplacement, **et** « un déplacement émet aussi la suppression de l'ancienne date ». Avec un upsert par `external_id`, il n'existe qu'**un** événement intervals.icu par Séance. Supprimer « l'ancienne date » revient à supprimer **cet** événement, c'est-à-dire la Séance déplacée. AD-5 ne dit pas non plus si la ligne d'outbox porte un **payload** (l'état au moment de l'écriture) ou une **référence**, ni comment ordonner deux lignes pour la même Séance.

**Paire incompatible.**
- *Unité A — « Déplacer » (FR-22)* : elle émet `upsert(new)` puis `delete(external_id)`, comme AD-5 le dit littéralement.
- *Unité B — publisher* : il traite les lignes en parallèle, avec deux workers Procrastinate en concurrence, ou en réessayant d'abord la plus ancienne en échec.
- **Résultat.** Selon l'ordre, la Séance disparaît de la montre, ou une version obsolète écrase la nouvelle (déplacer, modifier, puis le réessai du déplacement passe en dernier). SM-7 est violé, et `sync_status` peut rester ✅ puisque chaque appel a réussi.

**Deuxième trou : `sync_status` a deux écrivains.** La Publication (worker) le passe à ⚠️ en cas d'échec d'écriture. L'ingestion (worker, chaîne de synchro) devra signaler les échecs de lecture (FR-10, NFR-7). Une unité écrase le ⚠️ de l'autre par un ✅.

**AD proposé — durcir AD-5.**
- Une ligne d'outbox porte `(workout_id, plan_version, op ∈ {upsert, delete})` et **aucun payload**. Le publisher relit l'état **courant** de la Séance au moment de publier : la dernière écriture gagne.
- Les lignes sont **coalescées par `workout_id`** : seule la plus récente est publiée. Un **verrou Procrastinate `publication:{athlete_id}`** sérialise la Publication.
- **Un déplacement est un simple upsert** : l'événement garde son `external_id` et change de date. `delete` n'est émis que pour une Séance qui passe à *annulée* ou une Séance d'un Plan *annulé* (voir C2). Le test aller-retour sur le compte de test (AD-19) doit vérifier qu'un upsert avec une nouvelle date **déplace** bien l'événement sans le dupliquer. À défaut, la règle devient « delete puis create », dans le même lot, de façon ordonnée.
- On sépare `sync_status` en `publication_status`, avec une valeur par Séance (dernière tentative, erreur), et `ingest_status`, avec une valeur par athlète. Le ✅ ou ⚠️ de FR-10 est **dérivé** des deux par l'API, et chaque écrivain ne touche que sa ligne.

---

## C9 — Lecture `app` par dbt et lecture `gold` par l'app : cohérence et cycle — **Moyen**

**Constat.** AD-2 autorise dbt à lire `app` « comme source ». Deux risques en découlent.
1. **Instantané incohérent** : dbt lit `plan`, `workout`, `compliance` et `feeling` par requêtes séparées pendant que l'API écrit. Une même exécution peut voir une Séance *réalisée* sans sa Conformité. C'est pire encore si l'option (b) de C3 est retenue.
2. **Couplage de schéma caché** : Alembic peut renommer une colonne `app` dont dépend un modèle dbt. La convention « aucun ne touche les tables de l'autre » ne couvre pas les **lectures**. Une migration verte en CI back casse le `dbt build` en production.

L'app, de son côté, lit `gold` au fil de l'eau. Une requête qui lit deux modèles `gold` pendant un `dbt build` peut combiner deux snapshots DuckLake différents, par exemple les records avant et la CTL après.

**AD proposé — durcir AD-2.**
- dbt ne lit `app` qu'au travers de **vues d'export déclarées** (`app_export.*`, possédées par Alembic, avec un contrat versionné et un test CI qui exécute `dbt build` après `alembic upgrade head`). Il les lit dans **une seule transaction `REPEATABLE READ`**, par exemple via un `ATTACH` Postgres à snapshot unique, ou une copie en `bronze_app__*` au début du build.
- Le port Faits ouvre **un snapshot DuckLake par requête de cas d'usage** (`AT (VERSION => …)`) et l'expose (`facts_snapshot_id`) pour qu'il soit enregistré dans `feedback` et `plan_review`.
- Règle anti-cycle : **aucun modèle `gold` lu par le domaine ne dépend d'une table `app` écrite à partir de ce même modèle**, vérifié par une sélection `dbt ls` en CI. Si l'option (a) de C3 est retenue, dbt n'a plus besoin de lire `app`, et la règle se simplifie : dbt ne lit plus `app` du tout, AD-2 devient strictement unidirectionnel.

---

## C10 — Synchros multiples et import initial — **Moyen**

**Constat.** FR-11 lance une lecture « à chaque ouverture » et à chaque rafraîchissement. AD-6 sérialise par **verrou** (`lock`) mais ne **dédoublonne** pas la file. Trois ouvertures en 10 s enfilent trois chaînes ingestion → `dbt build` → détection, qui s'exécutent l'une après l'autre sur un VPS de 2 vCPU. La file `interactive` (C5) et la génération de Plan (moins de 2 min) en pâtissent. L'import initial (FR-1), possiblement long, peut aussi bloquer la première synchro.

**Paire incompatible.**
- *Unité A — Accueil* : elle enfile une synchro à chaque ouverture et sonde son `job_id`.
- *Unité B — Synchro* : elle utilise `queueing_lock`, si bien que la deuxième demande est **refusée** (Procrastinate lève `AlreadyEnqueued`).
- **Résultat.** A reçoit une erreur à la place d'un `job_id`, et l'accueil affiche une erreur de synchro.

**AD proposé — durcir AD-6.**
- `request_sync(athlete_id)` est idempotent : s'il existe une synchro *en attente* ou *en cours*, il renvoie **son** `job_id`, avec `queueing_lock = sync:{athlete_id}` et rattrapage de `AlreadyEnqueued`.
- On définit les files Procrastinate `sync`, `interactive` et `publication` avec leur concurrence.
- L'import initial est une synchro comme les autres, avec une fenêtre plus longue. Il ne contourne pas le verrou `lake`.

---

## C11 — Faits supprimés ou modifiés en amont — **Moyen**

**Constat.** AD-15 rend `bronze` en ajout seul, et `silver` déduplique par identifiant. Rien ne dit ce qui arrive quand une activité est **modifiée** (laps recoupés dans intervals.icu) ou **supprimée** après le rattachement, le Ressenti et le Feedback.

**Paire incompatible.**
- *Unité A — `silver`* : elle garde la dernière version par `fetched_at` et retire les activités absentes de la dernière fenêtre lue.
- *Unité B — `run_attachment`* : elle pointe toujours vers l'activité via une clé étrangère logique.
- **Résultat.** Le Feedback affiche des chiffres que `gold` ne contient plus, le Bilan (C3, option b) change, et le test « chaque chiffre a un test qui le recalcule » n'est plus vérifiable.

**AD proposé — compléter AD-15.**
- `silver` garde la version la plus récente par identifiant et porte un indicateur `deleted_upstream`, sans jamais retirer de ligne.
- `feedback` et `compliance` stockent les **valeurs calculées et le `facts_snapshot_id`** (C9). Ils ne sont jamais recalculés automatiquement.
- Une activité supprimée en amont garde son rattachement, et l'écran Synchro la signale.

---

## C12 — Définitions orphelines (à trancher dans le spine) — **Moyen**

| Terme | Ambiguïté | Proposition |
| --- | --- | --- |
| `plan.version` (AD-4) | qu'est-ce qui l'incrémente ? | voir AD-23 : toute écriture sur le Plan ou ses Séances, rattachement et transitions temporelles compris |
| « Toutes les Séances restantes » (AD-4) | à partir de `today` ou de demain ? Séance du jour sans Course ? | `date ≥ today` et état *à venir* |
| « 3 🥵 d'affilée » (FR-16) | qui calcule la série, domaine ou `gold` ? sur quel ordre (date ou `fetched_at`) ? | domaine, par ordre chronologique des Courses, à partir de `feeling` (`app`) |
| Déclencheur du Seuil de surmenage | à chaque Ressenti ? à la synchro ? | dans la tâche Feedback, après Conformité, dans la même transaction que la création de `proposal` |
| `proposal` en attente | expire-t-elle ? combien en même temps ? | une seule *en attente* par Plan ; expire dès que `plan.version` change ; la confirmation revérifie de toute façon |
| « Ajustement de surmenage : 7 jours sans hausse » (AD-12) | 7 jours à partir de la proposition, de la confirmation, en jours locaux ? | à partir de la confirmation, 7 dates locales, via le port `Clock` |
| Relances de Feedback « tant que le Plan est actif » (FR-14) | Course non planifiée sans Plan ? Plan suspendu ? | le statut est lu dans la tâche, `actif` seulement ; pour une Course non planifiée, il faut un Plan actif ou suspendu (à confirmer avec le PRD) |

---

## Récapitulatif des corrections

| # | Sévérité | Correction | Nouveaux AD ou AD touchés |
| --- | --- | --- | --- |
| C1 | Critique | Port `Clock`, `advance_calendar` unique, états temporels stockés, `manquée → réalisée` | **AD-20**, AD-14 |
| C2 | Critique | Tables de transitions Plan et Séance normatives | **AD-21**, AD-4 |
| C3 | Critique | Bilan figé (instantané `app` recommandé), Bilan provisoire pour le Plan de remplacement | **AD-22**, AD-2, AD-3 |
| C4 | Critique | `attach_run` unique, `FOR UPDATE` sur le Plan, `plan.version` défini, rattachement immuable après Ressenti | **AD-23**, AD-4, AD-15 |
| C5 | Élevé | Porte du Ressenti côté API sur tous les endpoints, Feedback enfilé dans la même transaction, file `interactive` | **AD-24**, AD-6, AD-18 |
| C6 | Élevé | Règle de résolution de version, dbt sans paramètres | **AD-25**, AD-11, AD-3 |
| C7 | Élevé | Calendrier unifié réalisé et planifié dans le domaine, semaine ISO locale | AD-12, AD-3 |
| C8 | Élevé | Outbox par référence, coalescée, sérialisée ; déplacement = upsert ; `sync_status` scindé | AD-5 |
| C9 | Moyen | Vues `app_export` en snapshot unique, snapshot DuckLake par cas d'usage, règle anti-cycle | AD-2 |
| C10 | Moyen | `request_sync` idempotent, files nommées | AD-6 |
| C11 | Moyen | Suppressions et modifications en amont, valeurs figées dans Feedback | AD-15 |
| C12 | Moyen | Définitions orphelines | AD-4, AD-12, AD-16 |

**Amendements PRD induits** :
- FR-5 et FR-8 : Bilan provisoire pour le Plan de remplacement.
- FR-11 et FR-12 : ordre rattachement → Ressenti, et rattachement définitif une fois le Ressenti posé.
- FR-18 : Séances d'un Plan suspendu non marquées *manquées*, et rattachement tardif.
- FR-7 : sort d'un Plan suspendu au-delà de la date de l'Objectif.
