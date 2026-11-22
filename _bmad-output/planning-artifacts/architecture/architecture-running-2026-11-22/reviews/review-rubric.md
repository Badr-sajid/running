# Revue rubrique — ARCHITECTURE-SPINE.md (running, 2026-11-22)

- **Objet revu** : `../ARCHITECTURE-SPINE.md` (status `draft`, AD-1..AD-19), avec `../.memlog.md`
- **Spécification de référence** : `prds/prd-running-2026-11-20/prd.md` (FR-1..FR-22, NFR-1..NFR-9, SM-1..SM-8)
- **Altitude déclarée** : `feature` (garder les epics cohérents), purpose `build-substrate`
- **Date** : 2026-11-23

## Verdict

Le spine est solide et bien au-dessus de la moyenne : paradigme clair, frontières de magasins nettes, propriétaire unique par calcul, garde-fous chiffrés, versions vérifiées. Il **n'est pas encore prêt** à servir de contrat aux epics. Trois points de divergence réels restent ouverts : l'horloge (qui fait avancer les états liés au temps, et quand), la définition de la « semaine » et de la fenêtre des 30 jours, et des entités sans propriétaire (Imprévu, zones et seuils). L'enveloppe opérationnelle n'est que partiellement décidée, et la sauvegarde vers S3 contredit NFR-2 et AD-17 tels qu'ils sont écrits. Un passage correctif ciblé suffit : aucune refonte n'est nécessaire.

## Grille

| Critère | Évaluation | Commentaire |
| --- | --- | --- |
| Fixe les vrais points de divergence, n'en manque aucun | Partiel | Excellente couverture (dépendances, magasins, calculs, mutations, outbox, LLM, cibles, temps, idempotence). Il manque : l'horloge et les tâches planifiées, les bornes de semaine, la fenêtre de 30 jours mixte planifié/réalisé, le propriétaire des zones et seuils, l'entité Imprévu, et le texte des messages (FR ou code). |
| Chaque Rule est applicable et empêche sa divergence | Majoritairement | AD-1 (CI d'import) et AD-15 (contrainte d'unicité) sont exemplaires. AD-2 « lecture seule » n'a aucun mécanisme. AD-5 « suppression de l'ancienne date » est contradictoire avec l'upsert par `external_id`. AD-6 ne dit pas comment les synchros concurrentes sont dédoublonnées. |
| Rien sous Deferred ne permet une divergence | Presque | Seul le mapping Daniels E/T/I/R/M → zones est différé « à la story du constructeur de Blocs » alors que trois epics le consomment (Blocs, garde-fous de bande, Conformité). |
| Techno nommée vérifiée et à jour | Oui, avec 2 réserves | Le memlog porte des entrées `(version)` datées du 23/11 pour toute la pile. Réserves : compatibilité openapi-typescript 7.x avec TypeScript 7.0 (portage natif), et dbt-core 1.12.7 sous Python 3.14 (seule la CI de dbt-duckdb sous 3.14 est attestée). |
| Couvre les capacités de la spec | Oui | FR-1..FR-22 tous présents dans la carte. NFR-3 et NFR-5 n'ont pas de ligne dédiée : ils sont couverts par AD-3, AD-7 et AD-11, mais l'origine des « pourquoi » en une ligne des décisions déterministes n'est pas tranchée (voir M-5). |
| Chaque dimension de l'altitude est décidée, différée ou en question ouverte | Partiel | Il n'y a pas de section « Open Questions ». Dimensions silencieuses : rotation des secrets, rétention, maintenance DuckLake, reprise sur échec des tâches et de l'outbox, rollback de migration, mise à jour OS et images, registre et architecture des images, alerte en cas d'échec de sauvegarde. |
| Décisions, pas justifications ; mermaid valides ; aucun placeholder | Presque | Le ton est décisionnel. L'erDiagram contient une cardinalité fausse et des libellés accentués non quotés. Il reste des formulations non tranchées : « type Hetzner CAX11 », « offre Personal ». |

## Constats

Sévérités : **Critique** (bloque les epics), **Majeur** (divergence probable entre epics ou contradiction avec la spec), **Moyen** (ambiguïté ou trou opérationnel), **Mineur** (forme).

### Majeurs

**M-1 — L'horloge n'a pas de propriétaire : les transitions liées au temps et les tâches planifiées sont silencieuses.**
FR-18 (« manquée le lendemain »), FR-7 (« terminé le lendemain de la dernière Séance »), FR-8 (Bilan à la fin du Plan) et AD-19 / D2 (sauvegarde nocturne) exigent tous quelque chose qui se déclenche avec le temps. AD-14 fixe *comment* calculer le jour, pas *qui* déclenche ni *quand*. Un epic peut dériver « manquée » à la lecture pendant qu'un autre l'écrit par une tâche périodique, ce qui donne des états divergents, un Bilan jamais calculé et une génération du Plan suivant qui lit un `gold_plan_review` absent ou périmé (SM-6). Le cas FR-7 est particulier : le nouveau Plan est généré *avant* l'annulation de l'ancien, donc avant que son Bilan existe.
*Correctif* : ajouter un AD « Horloge et tâches planifiées ». Les états temporels (manquée, terminé) sont **matérialisés** par une tâche Procrastinate périodique (`periodic`, par exemple toutes les heures et à l'ouverture), idempotente, qui passe par les cas d'usage (AD-4). La fin ou l'annulation d'un Plan enfile une synchro (ingestion → `dbt build`) sous le verrou `lake`. La génération d'un Plan commence par attendre une synchro fraîche. La sauvegarde est une tâche périodique du worker (ou un cron hôte, mais un seul des deux). Trancher le cas FR-7 : soit un Bilan provisoire du Plan actif est calculé au moment de la génération, soit la continuité SM-6 ne vaut que pour les Plans terminés.

**M-2 — La « semaine » et la fenêtre des 30 jours ne sont pas définies, alors que la moitié des garde-fous en dépend.**
AD-12 chiffre « +10 % par rapport à la dernière semaine non allégée », « une semaine allégée toutes les 3 ou 4 semaines », les bandes Daniels « du volume hebdomadaire » et « 110 % de la plus longue Course ou Séance planifiée sur 30 jours ». Trois points restent ouverts : (a) les bornes de semaine (semaine ISO lundi–dimanche dans le fuseau de l'athlète, ou semaine relative au début du Plan ; FR-22 parle de « la semaine 6 ») ; (b) la nature du volume comparé (réalisé, prévu, ou réalisé pour le passé et prévu pour le futur) et son unité (l'arbitrage 30 du PRD dit distance, AD-12 ne le dit pas, et les bandes mélangent temps et distance) ; (c) qui calcule la fenêtre mixte de 30 jours. AD-3 range « plus longue sortie sur 30 jours » dans `gold`. Or la fenêtre inclut les Séances planifiées, qui sont en cours de modification dans la transaction AD-4 et que `gold` ne peut pas voir.
*Correctif* : dans AD-12, ajouter « semaine = semaine ISO lundi–dimanche dans le fuseau de l'athlète (AD-14) ; "semaine N" de FR-22 = N-ième semaine ISO du Plan ; volume = distance en m ; passé = réalisé (`gold`), futur = prévu (`app`) ». Dans AD-3, préciser que `gold` fournit uniquement les maxima **réalisés** par jour, et que la fonction de domaine combine ces maxima avec les Séances planifiées en mémoire, propositions comprises.

**M-3 — La sauvegarde S3 contredit NFR-2 et AD-17 tels qu'ils sont écrits.**
AD-17 : « Les seuls tiers qui reçoivent des données sont intervals.icu et Anthropic. » Pourtant, le diagramme de déploiement et D2 envoient chaque nuit un `pg_dump` de `app` (blessures, notes personnelles, Ressentis) et les Parquet vers un fournisseur S3, lui-même différé (« n'importe quel fournisseur »). Une story d'implémentation ne peut pas satisfaire les deux règles à la fois.
*Correctif* : ajouter à AD-17 que les sauvegardes sont **chiffrées côté VPS avant envoi** (par exemple restic ou age, clé hors du fournisseur et conservée aussi hors du VPS), de sorte que le stockage objet ne reçoive que du chiffré. Amender NFR-2 au Finalize pour nommer l'hébergeur et le stockage chiffré comme sous-traitants acceptés. Fixer aussi la rétention des sauvegardes (par exemple 7 quotidiennes et 4 hebdomadaires) et la cohérence du lot : prendre le verrou `lake` pendant la copie Parquet et le dump du catalogue.

**M-4 — Des entités sans propriétaire ni magasin : Imprévu, zones et seuils, proposition de rattachement.**
- `disruption` (Imprévu) figure au glossaire, mais pas dans la table de AD-2 ni dans l'erDiagram. FR-18 a pourtant besoin de savoir qu'un Imprévu a été déclaré. Même remarque pour `race_event`, `method`, `instruction` (Consigne) et `athlete`, qui apparaît dans l'ER mais pas dans la table.
- **Zones et seuil intervals.icu** : AD-8 les résout « à partir des zones et du seuil configurés dans intervals.icu », sans dire s'ils sont lus en direct par l'adaptateur Calendrier ou ingérés dans le lake, ni s'ils sont figés au moment de la génération. Un epic lira l'API en direct pendant qu'un autre lira `gold`, et la Conformité dérivera si l'athlète change son seuil.
- AD-8 résout les allures Daniels E/T/I/R/M « depuis les zones et le seuil ». Chez Daniels, ces allures dérivent de la VDOT, que AD-13 calcule déjà. Deux sources sont donc possibles pour une même cible.
- La proposition de rattachement veille ou lendemain (FR-11, « confirmé d'un tap ») n'a pas d'état stocké.
*Correctif* : compléter la table de AD-2 avec `disruption`, `race_event`, `method`, `instruction`, `athlete` et `attachment_proposal` (ou un état `proposed` dans `run_attachment`). Ajouter `sport_settings` (zones, seuils) en `bronze` → `gold`, lu via le port Faits, avec un instantané stocké dans le `plan` ou le `step` au moment de la résolution. Trancher dans AD-8 : les allures Daniels se résolvent depuis la VDOT courante (`gold`, AD-13), ou depuis le seuil intervals.icu, mais une seule source.

### Moyens

**M-5 — La langue et la mise en forme des messages « pourquoi » et des garde-fous ne sont pas tranchées.**
AD-12 attribue à chaque garde-fou « son message en une ligne » et le PRD cite « Sortie longue à 24 km : +20 % au lieu de 10 % max ». Or la convention dit que la mise en forme (km, min/km) se fait **uniquement dans le front**, et le code est en anglais. Un epic produira des chaînes françaises côté backend, un autre les composera côté front à partir du `code`. Les « pourquoi » des Ajustements déterministes (FR-16, FR-19, NFR-5) ont le même problème.
*Correctif* : l'API renvoie `{code, params}` en unités SI (par exemple `{code: "WEEKLY_VOLUME_INCREASE", params: {distance_m: 24000, increase_pct: 20, max_pct: 10}}`), et le front possède un catalogue de messages français indexé par `code`. Le texte libre en français vient uniquement du LLM (`justification`).

**M-6 — AD-5 : « suppression de l'ancienne date » contredit l'upsert par `external_id`, et la sémantique de l'outbox est incomplète.**
L'`external_id` étant stable, `events/bulk?upsert=true` *déplace* l'événement existant. Émettre en plus « la suppression de l'ancienne date » risque de supprimer l'événement déplacé si la suppression se fait par `external_id`, ou de laisser un doublon si elle se fait par date. Rien n'est dit non plus de l'ordre ou de la fusion de plusieurs lignes d'outbox pour la même Séance, ni de la politique de reprise (automatique avec backoff, ou seulement via « resynchroniser »).
*Correctif* : « une ligne d'outbox référence une Séance ; le worker publie **l'état courant** de la Séance (idempotent) ; plusieurs lignes en attente pour une même Séance sont fusionnées ; la suppression ne sert qu'aux Séances annulées ou retirées (FR-7), par `external_id` ; reprise automatique ×N avec backoff, puis ⚠️ et reprise manuelle ». Vérifier par un test de contrat le comportement réel d'intervals.icu sur l'upsert avec changement de date.

**M-7 — Enveloppe opérationnelle : plusieurs dimensions sont silencieuses.**
Le déploiement, les environnements, l'hébergement et la sauvegarde sont décidés. Rien n'est décidé, différé ni posé en question sur les points suivants :
- **Rotation et expiration des secrets** : clés intervals.icu et Anthropic, et surtout **expiration des clés de nœud Tailscale** (180 jours par défaut), qui couperait l'accès en prod sans bruit. Décider : désactiver l'expiration de clé sur le nœud VPS, et documenter une procédure de rotation (modifier le fichier d'environnement puis redémarrer).
- **Maintenance DuckLake** : une synchro à chaque ouverture produit beaucoup de petits fichiers et d'instantanés. Décider d'une tâche périodique, sous le verrou `lake`, qui fusionne les petits fichiers, expire les anciens instantanés et nettoie les fichiers orphelins.
- **Rétention** : `bronze` en ajout seul (doublons à chaque relecture), table des tâches Procrastinate, `llm_call`, logs Docker (rotation). Fixer une durée pour chacun, ou écrire « conservé indéfiniment ».
- **Reprise après échec** : sémantique d'une tâche interrompue (Procrastinate peut relancer une tâche bloquée, ce qui pose la question d'une double génération ou d'un double coût LLM). Exiger que les tâches soient idempotentes par `job_id` et que leur résultat soit écrit en une seule transaction.
- **Migrations** : `alembic upgrade` au déploiement, mais pas de règle de rollback. Décider : migrations uniquement additives (expand/contract) et sauvegarde juste avant chaque déploiement. Côté lake, décider qu'un changement de modèle dbt se fait par `--full-refresh` depuis `bronze`.
- **Images** : le VPS est ARM et le Mac de dev aussi, mais rien ne dit que la CI construit en `linux/arm64`, ni quel registre est utilisé (par exemple GHCR). Les images de base (`python:3.14-slim`, `postgres:18.7`) ne sont pas épinglées.
- **Mises à jour de l'OS et de Docker** sur le VPS : mises à jour de sécurité automatiques ou manuelles ?
- **Alerte d'échec de sauvegarde** : avec la seule supervision par logs et écran Synchro, une sauvegarde qui échoue reste invisible. Décider : afficher l'état de la dernière sauvegarde dans l'écran Synchro.
*Correctif* : ajouter une sous-section « Opérations » sous Structural Seed (une ligne par dimension), ou une section « Open Questions » pour ce qui n'est pas tranché.

**M-8 — AD-2 et AD-6 : deux règles sans mécanisme.**
- « dbt lit la base `app` en lecture seule » et « aucun composant n'écrit dans le magasin de l'autre » reposent uniquement sur la discipline. *Correctif* : rôles Postgres distincts. `app_rw` est propriétaire de la base `app`. `dbt` a `SELECT` seul sur `app` et écrit sur `lake`. L'API lit `lake` avec un rôle en lecture seule.
- « Synchro à chaque ouverture » avec un verrou `lake` : plusieurs ouvertures empilent des synchros. *Correctif* : utiliser `queueing_lock="sync:<athlete_id>"` (au plus une synchro en attente) en plus de `lock="lake"`, et faire renvoyer à `POST /sync` le `job_id` existant.

**M-9 — Mapping Daniels E/T/I/R/M ↔ zones différé à une story.**
Ce mapping est consommé par le constructeur de Blocs (AD-9), la bande Daniels (AD-12) et la Conformité (AD-8). Le différer « à la story du constructeur de Blocs » crée une dépendance d'ordre implicite et un risque d'interprétations parallèles.
*Correctif* : le déclarer comme une valeur de `parameter_set` dont le module propriétaire est nommé (`domain/targets`), et exiger que l'epic du constructeur de Blocs précède ceux des garde-fous et de la Conformité. Le fixer directement dans le spine est encore mieux.

### Mineurs

**m-1 — erDiagram : une cardinalité fausse et des libellés fragiles.**
- `WORKOUT ||--o| RUN_ATTACHMENT` impose exactement un `workout` par `run_attachment`, ce qui contredit la phrase juste en dessous (« Une Course non planifiée a un `run_attachment` sans `workout` »). Écrire `WORKOUT |o--o| RUN_ATTACHMENT`.
- Les libellés de relation non quotés et accentués (`génère`, `reçoit`) passent mal dans le lexer `erDiagram` de plusieurs versions de Mermaid. Quoter tous les libellés : `GOAL ||--o{ PLAN : "génère"`.
- `DISRUPTION`, `RACE_EVENT` et `INSTRUCTION` sont absents (voir M-4).

**m-2 — Flowcharts : valides, quelques durcissements conseillés.**
- `subgraph Entrées` : l'identifiant de subgraph est accentué. Écrire `subgraph ENT[Entrées]` (même chose pour `Adaptateurs`, par cohérence).
- L'arête `PROD -- sauvegarde nocturne : pg_dump app+lake, Parquet --> S3` contient `:`, `+` et `,` dans un libellé non quoté. Écrire `PROD -->|"sauvegarde nocturne chiffrée : pg_dump app+lake, Parquet"| S3`.
- Le reste (`-. … .->`, `[( … )]`, `<-->`, les enchaînements) est correct.

**m-3 — Réserves de compatibilité de version à lever dans le memlog.**
- openapi-typescript 7.13.0 avec TypeScript 7.0.2 : TS 7 est le portage natif, dont l'API JS de compilation n'est pas celle de 5.x. Vérifier que le générateur fonctionne, ou épingler `typescript@5.x` comme dépendance de génération.
- dbt-core 1.12.7 sous Python 3.14 : le memlog atteste la CI de dbt-duckdb sous 3.14, pas celle de dbt-core. Ajouter une entrée `(version)`.
- Le `Stack` indique « type Hetzner CAX11 » et « offre Personal » : ce sont des intentions, pas des épinglages. Écrire « Hetzner CAX11 (arm64) » ou déplacer en Deferred « choix de l'hébergeur ».

**m-4 — Carte des capacités : NFR-9 mentionne un « streaming » ambigu.**
AD-6 interdit websocket et push, et le front interroge le statut. « Streaming » ne peut donc désigner que le flux entre le worker et Anthropic. Le dire (« streaming SDK worker ↔ Anthropic ; le front interroge `GET /jobs/:id` et ses étapes »), et ajouter des lignes NFR-3 et NFR-5 renvoyant à AD-3, AD-7, AD-11 et M-5.

**m-5 — Pas de section « Open Questions ».**
Le memlog garde `purpose: build-substrate (à confirmer)`, et l'amendement PRD sur les blessures est renvoyé au Finalize. Lister explicitement : confirmation du purpose, amendements PRD (FR-5, FR-22, §12 Q8 sur les blessures ; FR-8 « stocké dans le Profil » alors qu'il est désormais dans `gold` ; NFR-2 sur les sauvegardes), et les points de M-7 non tranchés.

**m-6 — Versionnage des prompts.**
`llm_call` journalise le modèle et les tokens, mais pas la version du prompt. Pour SM-5 (comparaison à l'aveugle) et pour recalculer un Feedback, ajouter `prompt_version` à `llm_call` et au `parameter_set` (ou un hash du gabarit).

## Points forts à conserver

- AD-2 et AD-3 (deux magasins, propriétaire unique par calcul) tranchent la tension principale du memlog proprement, avec une table de propriété.
- AD-7, AD-8 et AD-9 (« le LLM propose, le domaine valide » ; références d'intensité résolues par le code ; squelette LLM et Blocs déterministes) ferment les risques de crédibilité du PRD (§11) et servent aussi NFR-4 et NFR-9.
- AD-12 chiffre les garde-fous ouverts par le PRD (§12 Q7) et règle explicitement le conflit sur les blessures (§12 Q8), avec un amendement PRD prévu.
- AD-19 traduit NFR-8 en porte de mise en usage vérifiable (tag + CI verte, compte intervals.icu de test, restauration testée).
- Les versions sont toutes adossées à une entrée `(version)` datée, et le choix DuckDB 1.5.6 / PostgreSQL 18.7 est justifié par NFR-8, avec une échéance de montée en version différée et datée.

## Corrections prioritaires (ordre conseillé)

1. M-1 : AD « Horloge et tâches planifiées ».
2. M-2 : définitions de semaine, de volume et de fenêtre de 30 jours dans AD-12 et AD-3.
3. M-4 : compléter la table de AD-2 (Imprévu, zones et seuils, propositions) et trancher la source des allures Daniels.
4. M-3 et M-7 : sauvegarde chiffrée et sous-section « Opérations ».
5. M-5, M-6 et M-8 : messages `{code, params}`, sémantique de l'outbox, rôles Postgres et `queueing_lock`.
6. Mineurs : erDiagram, quotage mermaid, réserves de version, Open Questions.
