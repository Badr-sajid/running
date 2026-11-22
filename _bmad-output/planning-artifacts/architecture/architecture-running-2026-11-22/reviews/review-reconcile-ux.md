---
type: review
lens: reconcile-ux
target: ../ARCHITECTURE-SPINE.md
sources:
  - ../../../ux-designs/ux-running-2026-11-21/EXPERIENCE.md
  - ../../../ux-designs/ux-running-2026-11-21/DESIGN.md
  - ../../../ux-designs/ux-running-2026-11-21/.memlog.md
  - ../.memlog.md
date: 2026-11-23
---

# Revue de réconciliation : spine d'architecture vs spécification UX

## Verdict

Le spine couvre bien les invariants lourds que l'UX suppose : pas de push (AD-6), notes exclues des prompts (AD-7), garde-fous en deux familles avec `{refused, informational}` (AD-12), jours sélectionnables fournis par l'API et lecture seule hors ligne (AD-18), outbox et ⚠️ (AD-5). En revanche, il ne dit pas **quels états et quels déclencheurs l'API expose au front**. Or presque toute l'expérience après la Course en dépend : quand la sheet Ressenti s'ouvre, quels chiffres sont visibles avant elle, ce que contient « Analyse en cours », les relances, les sheets en attente. Le spine laisse aussi ouverts la latence de la chaîne de synchro et les transitions qui dépendent du temps. Deux équipes construiraient ici deux produits différents, et une exigence UX (graphes FC et allure, % du temps en zone) n'a pas de source de données. **Il manque 3 AD à ajouter et 6 AD à resserrer avant de découper les epics.**

Légende de sévérité : **Haute** = l'exigence est bloquée, ou des builders divergent de façon certaine ; **Moyenne** = une divergence est probable et coûteuse à corriger ensuite ; **Basse** = une précision suffit.

## Synthèse

| # | Sujet | Sévérité | Correction proposée |
|---|---|---|---|
| R-1 | États d'affichage de la Séance et sheets en attente : pas de contrat API | Haute | Nouvel AD-20 |
| R-2 | Déclenchement de la synchro, latence de la chaîne, files du worker | Haute | Resserrer AD-6 |
| R-3 | Transitions liées au temps (manquée, terminé) sans déclencheur ; Bilan après « Arrêter » | Haute | Resserrer AD-4 et AD-14, nouvel AD-21 (ou ajout à AD-6) |
| R-4 | Séries temporelles (streams) absentes du modèle de faits : graphes et % en zone bloqués | Haute | Resserrer AD-2 (table des entités) |
| R-5 | Cycle de vie du Feedback : moment du calcul de Conformité, relances, « Arrêter l'analyse » | Moyenne | Nouvel AD-22 (ou extension d'AD-7) |
| R-6 | Vérification à blanc des modifications et de l'Objectif (« Enregistrer » désactivé, ligne d'information) | Moyenne | Resserrer AD-4 et AD-18 |
| R-7 | Étapes de génération : contrat non défini | Moyenne | Resserrer AD-6 |
| R-8 | Contenu de `sync_status` et « Resynchroniser » | Moyenne | Resserrer AD-5 |
| R-9 | Notes perso : isolement vis-à-vis de dbt, sauvegarde automatique hors ligne | Moyenne | Resserrer AD-2, AD-7 et AD-18 |
| R-10 | Ressenti immuable et sheet non fermable vs hors ligne et double envoi | Moyenne | Resserrer AD-15 et AD-18 |
| R-11 | Coût IA : budget, seuil de 80 %, mois de référence | Basse | Resserrer AD-11 et la convention « Argent » |
| R-12 | Carte Bilan, Historique, stabilité du Bilan | Basse | Resserrer AD-2 et AD-4 |
| R-13 | Hors ligne : horodatage du cache, périmètre, sondage des jobs | Basse | Resserrer AD-18 |
| R-14 | « streaming » dans la carte des capacités, ambigu face à AD-6 | Basse | Reformulation |

Points vérifiés **sans écart** : les jours grisés de Déplacer et de la sortie longue sont fournis par l'API (AD-18) ; la ligne « Hors de tes jours habituels » est le résultat `informational` d'AD-12 ; un Ressenti et un Feedback au plus par activité (AD-15) ; l'absence de notification et de websocket (AD-6) ; le mode clair seul, la police système et l'absence de police ou de CDN tiers, compatibles avec AD-17 (le front n'appelle que l'`api`) ; les unités et le formatage dans le front ; les tentatives invisibles (AD-7 les compte, l'UX ne les montre pas). Les tokens de DESIGN.md n'ont aucune portée architecturale.

## Constats détaillés

### R-1 — États d'affichage de la Séance et sheets en attente : pas de contrat API (Haute)

**UX.** EXPERIENCE.md § State Patterns › Séance définit 9 états visibles : *à venir*, *en attente de la course*, Course détectée sans Ressenti, Analyse en cours, Erreur d'analyse, *réalisée* · Feedback prêt, Avis indisponible, *manquée*, *annulée*, plus Course non planifiée. Trois règles en découlent :

- « Aucun chiffre affiché avant » le Ressenti ; « Tes données » est « masqué tant que le Ressenti n'est pas posé ».
- La sheet Ressenti s'ouvre « automatiquement une fois la Course synchronisée et rattachée ».
- « Une seule sheet à la fois » : Ressenti, Rattachement (« C'est ta séance de … ? ») et Proposition de surmenage (« à l'ouverture ») peuvent tous être en attente en même temps.

**Spine.** AD-4 prévoit une machine à états de la Séance, et AD-18 interdit toute règle métier dans le front. Mais rien ne dit :

- quels états sont **stockés** (cycle de vie : à venir, réalisée, manquée, annulée) et quels états sont **dérivés** (en attente de la course, analyse en cours, avis indisponible) ;
- qui décide de masquer les chiffres avant le Ressenti ;
- comment le front apprend qu'une sheet est à ouvrir, et dans quel ordre.

`run_attachment` n'a pas non plus d'état « candidat à confirmer » pour la sheet Rattachement, alors que sa contrainte d'unicité (AD-15) suppose un rattachement déjà décidé.

**Risque.** Un builder masque les chiffres côté front, un autre côté API, et les chiffres partent alors dans le cache hors ligne. Un autre encore déduit « en attente de la course » de la date dans le front, ce qui viole AD-18. Les priorités entre sheets sont réinventées à chaque écran.

**Correction : nouvel AD-20, « États d'affichage et actions en attente calculés par l'API ».**

- L'API renvoie pour chaque Séance un `display_state` (énumération fermée, celle de l'UX) calculé par une fonction du domaine. Le cycle de vie stocké (AD-4) reste distinct.
- Tant qu'aucun `feeling` n'existe, l'API **ne renvoie pas** `compliance`, « Tes données » ni les séries.
- Un endpoint unique (par exemple `GET /home` ou `GET /pending-actions`) renvoie la liste ordonnée des actions en attente : `feeling_required` > `attachment_confirmation` > `overload_proposal`. Le front n'ouvre que la première.
- `run_attachment` porte un statut `proposed | confirmed | rejected` (un rattachement à la veille ou au lendemain est `proposed`), et l'unicité ne porte que sur les statuts non rejetés.

### R-2 — Déclenchement de la synchro, latence de la chaîne, files du worker (Haute)

**UX.** « Tirer pour rafraîchir sur Accueil, Détail Séance et Synchro : relance la détection des Courses » ; « Lectures à chaque ouverture et sur rafraîchissement » ; un indicateur « Synchro… » s'affiche sur le contenu en cache. NFR-9 exige un Feedback en moins de 30 s après le Ressenti. SM-8 demande que la manipulation se limite à ouvrir l'app et poser un Ressenti.

**Spine.** AD-6 fait passer chaque synchro par ingestion → `dbt build` → détection, sous un verrou `lake` unique, sur un VPS de 2 vCPU. Le spine ne dit rien :

- de l'endpoint qui déclenche la synchro, ni de ce qui se passe quand l'utilisateur ouvre l'app puis tire aussitôt (deux jobs, ou un seul ?) ;
- de la portée du `dbt build` : tout le projet, ou seulement les modèles concernés ? ;
- de la concurrence du worker. Un Feedback (LLM) mis en file derrière un `dbt build` complet ou derrière une génération de Plan de 2 minutes manque sa cible de 30 s ;
- de la façon dont le front sait que la détection est terminée pour ouvrir la sheet Ressenti.

**Risque.** Des jobs de synchro s'empilent à chaque ouverture ou geste de rafraîchissement, avec des appels redondants à intervals.icu (quotas, AD-15). La sheet Ressenti met des minutes à apparaître. NFR-9 est manqué dès qu'une synchro tourne.

**Correction : resserrer AD-6.**

- `POST /sync` est **idempotent et regroupé** : si une synchro est en file ou en cours, il renvoie son `job_id` au lieu d'en créer une autre (verrou `lake` + `queueing_lock` Procrastinate). Les ouvertures rapprochées (moins de N s, paramètre) ne relancent pas d'ingestion.
- La synchro déclenchée par l'utilisateur fait une ingestion **incrémentale** (depuis le dernier `fetched_at`) et un `dbt build --select` limité aux modèles en aval des données modifiées. Le build complet est nocturne.
- Il y a au moins deux files Procrastinate : `lake` (concurrence 1) et `llm` (concurrence ≥ 1, indépendante). Le Feedback ne prend jamais le verrou `lake` : il lit `gold` au dernier instantané validé.
- Le résultat du job de synchro contient les Courses nouvellement détectées et rattachées. Le front relit alors `pending-actions` (R-1).
- Ajouter un budget de latence chiffré : depuis la fin de l'ingestion jusqu'à la détection en moins de X s sur la machine cible, vérifié une fois avant la mise en usage (AD-19).

### R-3 — Transitions liées au temps sans déclencheur ; Bilan après « Arrêter » (Haute)

**UX.** Une Séance est *manquée* « le lendemain » (FR-18). Un Plan est *terminé* « le lendemain de la dernière Séance », puis l'Accueil propose « Voir ton bilan ». « Arrêter le plan » mène à : Plan *annulé*, Séances à venir retirées, **Bilan calculé**.

**Spine.** AD-14 fixe *comment* calculer ces règles (une seule fonction, dans le fuseau de l'athlète), mais pas *quand* elles s'appliquent. Aucune tâche périodique n'est prévue, et l'évaluation à la lecture n'est pas non plus retenue. Par ailleurs, `plan_review` est calculé par dbt (AD-2, AD-3), et dbt ne tourne que dans la chaîne de synchro (AD-6). Après « Arrêter », rien ne garantit donc que le Bilan existe quand l'utilisateur l'ouvre.

**Risque.** Un builder évalue « manquée » à la volée dans l'API, un autre écrit l'état par un cron : les deux divergent sur ce que lit dbt, donc sur le Bilan (« Séances manquées »). Le Bilan d'un Plan annulé reste introuvable jusqu'à la prochaine synchro.

**Correction.**

- **Nouvel AD-21 (ou ajout à AD-4 et AD-14)** : les transitions liées au temps sont **écrites** dans la base `app` par un cas d'usage unique, `apply_time_transitions(athlete, today_local)`, idempotent. Il s'exécute en tête de chaque synchro et dans une tâche périodique Procrastinate quotidienne (peu après minuit, dans le fuseau de l'athlète). L'API ne recalcule jamais ces états à la lecture.
- Toute transition vers *terminé* ou *annulé* met en file une synchro (ou au minimum un `dbt build --select gold_plan_review+`). L'API expose un état `plan_review: pending | ready` pour que l'écran Bilan affiche une attente au lieu d'une erreur.

### R-4 — Séries temporelles absentes du modèle de faits (Haute)

**UX.** Le tableau « Tes données » par type de Séance exige :

- le « % du temps dans la zone FC cible » (chiffre principal du footing, secondaire de la sortie longue) ;
- des graphes « FC au fil de la Course + zone », « Allure au fil de la Course », ainsi que l'« Allure par répétition » ;
- un texte alternatif qui résume la série.

**Spine.** Le lake contient « Courses, laps, FC au repos, CTL, records… » (AD-2). Il ne contient ni flux seconde par seconde (streams), ni temps par zone. Le % en zone ne se calcule pas à partir des laps, et les graphes « au fil de la Course » non plus.

**Risque.** L'exigence est bloquée. Ou bien un builder appelle intervals.icu en direct depuis l'API pour obtenir les streams, ce qui viole AD-2, AD-5 et AD-17 dans l'esprit : une seule voie de lecture, par l'ingestion.

**Correction : resserrer AD-2.** Ajouter à la table des entités du lake les **streams d'activité** (FC, allure, distance, temps), ingérés en `bronze` pour chaque Course et dédupliqués en `silver`. Ajouter un modèle `gold` d'interface **sous-échantillonné** pour les graphes (par exemple 1 point toutes les 10 s, paramètre). Le % en zone est un calcul par Course du domaine (AD-3) qui lit ces streams, ou à défaut le temps par zone fourni par intervals.icu : il faut choisir l'un des deux. Vérifier l'effet sur la taille des Parquet et sur les quotas.

### R-5 — Cycle de vie du Feedback (Moyenne)

**UX.** Pendant « Analyse en cours », « Tes données » et la Conformité sont **déjà visibles** dessous (« chiffres déterministes »). En cas d'erreur : « Relancer » (2 au plus par Feedback, tant que le Plan est actif) ou « Arrêter l'analyse » (vers « Avis indisponible »). Depuis « Avis indisponible », « Relancer » reste possible dans la même limite.

**Spine.** AD-7 fixe 3 tentatives par cycle, et `feedback` stocke le texte et les chiffres. Plusieurs choses ne sont pas fixées :

- le moment où `compliance` est calculée et persistée (pendant la synchro, au rattachement, ou dans le job de Feedback ?) ;
- le stockage du compteur de relances et de l'état du Feedback (`pending | running | failed | stopped | ready`) ;
- le sens d'« Arrêter l'analyse » sur un job Procrastinate en cours (annulation ? résultat ignoré ?) ;
- la règle « seulement tant que le Plan est actif ».

Le déclencheur du job de Feedback (la pose du Ressenti) n'est pas écrit non plus.

**Risque.** Si la Conformité est calculée dans le job LLM, l'écran « Analyse en cours » n'a rien à montrer sous la carte. Les compteurs de relance seront implémentés de façon incohérente avec le coût mesuré (NFR-4).

**Correction : nouvel AD-22, « Cycle de vie du Feedback ».**

- `compliance` et « Tes données » sont calculées et persistées **au rattachement**, dans la synchro, puis exposées dès la pose du Ressenti (R-1).
- Poser le Ressenti met en file **dans la même transaction** le job de Feedback (file `llm`).
- `feedback` porte `status`, `cycle_count` (≤ 3, soit 1 + 2 relances, en paramètre) et l'identifiant du dernier `job_id`.
- « Arrêter l'analyse » passe le statut à `stopped` et demande l'annulation du job. Un résultat qui arrive après `stopped` est ignoré, mais son coût est journalisé dans `llm_call`.
- « Relancer » est refusé (code RFC 9457 stable) si le compteur est épuisé ou si le Plan n'est pas *actif*.

### R-6 — Vérification à blanc des modifications et de l'Objectif (Moyenne)

**UX.** Dans la sheet Modifier, la bannière nommant le garde-fou et le bouton « Enregistrer » **désactivé** apparaissent *avant* l'enregistrement, tout comme la ligne « Hors de tes jours habituels » avec « Enregistrer » actif. Le formulaire Objectif refuse la date « dès la saisie », avec la première date possible ; il affiche l'aide « Soit 4:57 /km », « 20 semaines de préparation » et l'alerte « Chrono jugé réaliste : 3 h 41 — d'après ton semi… » avant « Créer mon plan ».

**Spine.** AD-4 vérifie les garde-fous « avant de valider la transaction », c'est-à-dire à l'enregistrement. AD-18 dit que les garde-fous sont « fournis par l'API », sans dire comment le front les obtient avant d'enregistrer. AD-13 n'expose pas l'alerte chrono comme une requête séparée de la création de l'Objectif.

**Risque.** Le front recode la règle des +10 % pour griser le bouton, ce qui viole AD-3 et AD-18. Ou bien l'UX dégénère en « erreur après tap », contraire à la spécification.

**Correction : resserrer AD-4 et AD-18.**

- Chaque cas d'usage de modification (Déplacer, Modifier à la main, déplacement par Empêchement) a un mode **`dry_run`**. C'est le même code, exécuté sans commit ni outbox, et il renvoie le résultat AD-12 `{refused, informational}`, exposé par exemple en `POST /workouts/:id/modifications:check`.
- Ajouter une requête `POST /goals:check` qui renvoie `min_date`, le nombre de semaines, l'allure moyenne, et l'alerte chrono (VDOT requise et actuelle, chrono réaliste, effort de référence avec sa date).
- Le front ne calcule aucun de ces chiffres. Ils relèvent de « chaque chiffre affiché a un test » (AD-19).

### R-7 — Étapes de génération : contrat non défini (Moyenne)

**UX.** Trois étapes fixes, « Profil lu · Méthode choisie · Garde-fous vérifiés », chacune faite, en cours ou pas commencée. Les tentatives sont invisibles. En cas d'échec, l'écran nomme le garde-fou violé.

**Spine.** AD-6 mentionne « statut, étapes, résultat » sans énumération ni règle de progression. Avec 3 tentatives (AD-7) et des Blocs construits par le code (AD-9), une nouvelle tentative pourrait faire *reculer* « Garde-fous vérifiés ».

**Correction : resserrer AD-6.** `GET /jobs/:id` renvoie `steps: [{key, status}]`, avec des clés stables par type de job : `plan_generation` = `profile_loaded`, `method_chosen`, `guardrails_checked` ; `initial_import` = progression numérique pour la barre de FR-1. La progression est **monotone** : une nouvelle tentative garde l'étape en cours et n'en recule aucune. Le résultat d'échec contient `guardrail_ids` et leurs messages (AD-12). Le même job sert à « Refuser + Consigne » et à « Reprendre ».

### R-8 — Contenu de `sync_status` et « Resynchroniser » (Moyenne)

**UX.** En ✅ : « Dernière Séance publiée, dernière Course reçue, horodatées ». En ⚠️ : « Cause en clair (quota intervals.icu, clé, réseau) », « Ton plan est conservé », un point d'alerte sur l'onglet, et « Resynchroniser », qui « republie le Plan », avec un état « Publication… » puis ✅ ou ⚠️.

**Spine.** AD-5 ne passe `sync_status` à ⚠️ que pour un échec de **Publication**. Un échec de **lecture** (clé révoquée, quota sur l'ingestion) n'a pas d'emplacement prévu. « Resynchroniser » n'est pas décrit : republier toutes les Séances à venir ? rejouer seulement les lignes d'outbox en échec ?

**Correction : resserrer AD-5.**

- `sync_status` a deux volets, `publication` et `ingestion`. Chacun porte `state` (`ok | error | running`), `cause_code` (énumération fermée : `ICU_QUOTA`, `ICU_AUTH`, `NETWORK`, `ICU_UNAVAILABLE`, `UNKNOWN`), `last_success_at`, et l'objet concerné (dernière Séance publiée, dernière Course reçue).
- « Resynchroniser » est un cas d'usage qui écrit dans l'outbox un upsert de **toutes** les Séances à venir du Plan actif, ainsi que les suppressions des dates obsolètes, puis lance une ingestion.
- Le point d'alerte de l'onglet est lu depuis un endpoint léger, ou depuis le `GET /home` de R-1.

### R-9 — Notes perso : isolement vis-à-vis de dbt, sauvegarde automatique hors ligne (Moyenne)

**UX.** « Mes notes » : « Pour toi seul, jamais envoyé à l'IA » ; « Enregistrement automatique à la sortie du champ ».

**Spine.** AD-7 exclut les notes des prompts, et les logs les excluent aussi. Mais dbt lit la base `app` comme source (AD-2), et `plan_review` en est dérivé. Si les notes partagent une table avec `feeling` ou `feedback`, elles peuvent finir dans `silver` ou `gold`, puis dans un contexte LLM construit à partir de `gold`, comme le Bilan cité par le pourquoi du nouveau Plan (SM-6). Enfin, AD-18 interdit toute file d'attente hors ligne : une note tapée hors ligne est perdue à la sortie du champ, sans que l'UX le dise.

**Correction.**

- Les notes vivent dans une **table dédiée** (`workout_note`), **exclue des sources dbt** et du constructeur de contexte LLM. Un test vérifie qu'aucune source dbt ni aucun assembleur de prompt ne la référence (resserrer AD-2 et AD-7).
- Pour AD-18, en exception assumée : le brouillon d'une note est gardé **localement** (sans file de mutation) tant que l'envoi échoue, et le front l'indique. Ou bien le champ est désactivé hors ligne. Il faut trancher entre les deux.

### R-10 — Ressenti immuable et sheet non fermable vs hors ligne et double envoi (Moyenne)

**UX.** La sheet Ressenti n'a ni poignée, ni glisser, ni tap sur le voile. Un seul choix, non modifiable. Un tap vaut choix et fermeture.

**Spine.** AD-15 limite à un Ressenti par activité. Rien ne dit que l'API n'expose **aucune** modification ni suppression, ni comment un renvoi réseau est traité. Une sheet impossible à fermer, ouverte depuis le cache hors ligne (AD-18) alors que l'envoi est impossible, bloque l'app.

**Correction : resserrer AD-15 et AD-18.**

- `POST /runs/:id/feeling` est idempotent. Un même choix renvoyé donne 200 avec l'existant ; un choix différent donne une erreur `FEELING_ALREADY_SET`. Il n'y a ni PUT ni DELETE.
- L'action en attente `feeling_required` (R-1) n'est **jamais** présentée à partir d'une réponse servie par le cache hors ligne.

### R-11 — Coût IA : budget, seuil de 80 %, mois de référence (Basse)

**UX.** Profil › coût IA : le montant du mois face au budget de 10 €, avec un avertissement au-delà de 80 %.

**Spine.** `llm_call` stocke le coût en micro-USD, et le taux €/$ est dans `parameter_set`. Le **budget** et le **seuil d'avertissement** n'y figurent pas, et le « mois » n'est pas défini (calendaire, dans quel fuseau ?).

**Correction.** Ajouter `ai_budget_eur_month` et `ai_budget_warning_ratio` (0,8) à `parameter_set` (AD-11). Le mois est le mois calendaire dans le fuseau de l'athlète (AD-14). L'API renvoie `{spent_eur, budget_eur, warning: bool}`. Préciser aussi si un échec ou une relance compte dans le coût : oui, d'après FR-14.

### R-12 — Carte Bilan, Historique, stabilité du Bilan (Basse)

**UX.** La carte « Bilan de ton plan » reste en haut du hub tant qu'« aucun nouveau Plan n'est lancé ». L'Historique liste les Plans *terminé* et *annulé* (« arrêté en semaine 3 »), et chaque ligne rouvre **son** Bilan.

**Spine.** `plan_review` est un modèle `gold` reconstructible (AD-2) : il est recalculé à chaque build et peut donc changer après coup, par exemple si une activité est modifiée dans intervals.icu ou si les paramètres changent. Le spine ne dit pas non plus si un Plan *proposé* puis refusé (Consigne) ou remplacé apparaît dans l'Historique, ni ce que veut dire « nouveau Plan lancé » (proposé ? actif ?).

**Correction.**

- Préciser que `plan_review` est calculé avec le `parameter_set` enregistré sur le Plan (AD-11). Accepter la dérive due aux faits, ou figer un instantané dans `app` à la clôture : il faut choisir.
- L'Historique ne liste que les Plans dont le cycle de vie est *terminé* ou *annulé*. Un Plan *proposé* puis refusé n'y entre jamais.
- La carte Bilan est exposée par l'API (`latest_review_card`) tant qu'aucun Plan n'est *actif* ou *proposé*.

### R-13 — Hors ligne : horodatage du cache, périmètre, sondage des jobs (Basse)

**UX.** La bannière « Hors ligne — données de … » donne l'heure du cache. Sinon, c'est la dernière version en cache.

**Spine.** AD-18 parle de la « dernière réponse GET en cache », sans stratégie de service worker, sans horodatage et sans exclusions.

**Correction.** Stratégie NetworkFirst sur les GET de lecture, avec l'horodatage de la réponse stocké dans le cache, et exclusion de `GET /jobs/*`, `pending-actions` et du coût IA. Le front affiche l'horodatage stocké.

### R-14 — « streaming » dans la carte des capacités (Basse)

La ligne NFR-9 de la carte indique « worker, squelette LLM, streaming ». Un lecteur peut y voir du streaming vers le front (SSE), contraire à AD-6. **Correction** : écrire « streaming de la réponse Anthropic côté worker uniquement ; le front interroge `GET /jobs/:id` ».

## Récapitulatif des modifications du spine

- **Ajouter** :
  - AD-20, États d'affichage et actions en attente (R-1, R-10) ;
  - AD-21, Transitions liées au temps et recalcul du Bilan (R-3) ;
  - AD-22, Cycle de vie du Feedback (R-5).
- **Resserrer** :
  - AD-2 : streams, `workout_note` hors dbt, stabilité de `plan_review` ;
  - AD-4 : `dry_run` ;
  - AD-5 : `sync_status` à deux volets, Resynchroniser ;
  - AD-6 : `POST /sync` regroupé, files `lake` et `llm`, build sélectif, étapes de job ;
  - AD-11 : budget IA ;
  - AD-15 : Ressenti idempotent et immuable, statut de `run_attachment` ;
  - AD-18 : hors ligne, brouillon de note, `goals:check`.
- **Reformuler** la carte des capacités (NFR-9).
