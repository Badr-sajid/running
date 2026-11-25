---
type: review
lens: tech-currency (fraîcheur et réalité des choix techniques), passe 2
target: ../ARCHITECTURE-SPINE.md (updated 2026-11-25)
companion: ../.memlog.md
previous: review-tech-currency.md (2026-11-23)
date: 2026-11-25
---

# Revue « fraîcheur technique » (passe 2) — ARCHITECTURE-SPINE (running)

## Verdict

La Stack tient toujours. Aucune version épinglée n'est devenue fausse ou dangereuse depuis le 23/11, et les corrections de la passe 1 sont en place : TypeScript 5.9.3, DuckDB 1.5.6 épinglé, spikes OQ-1 et OQ-2. Les nouvelles affirmations sur l'API intervals.icu sont **confirmées dans la spécification OpenAPI officielle** pour `sleepSecs`, `threshold_pace` et la GAP en résumé d'activité. La syntaxe d'allure absolue (`5:40/km Pace`) est **confirmée dans la documentation du forum**.

Deux points restent **non confirmés ou mal posés** :

- les **streams** GAP et température. Les noms de stream ne sont pas documentés, la GAP dépend d'un réglage `gap_model` que le spine ne lit pas, et une Apple Watch ne mesure pas la température ambiante ;
- le **repli Minetti de AD-3**, qui crée deux définitions de la GAP selon la disponibilité de la donnée.

Il faut aussi noter un bug DuckLake de dbt-duckdb 1.11.1 : il est corrigé sur `main`, mais aucune version publiée ne contient le correctif.

## Méthode (25/11/2026)

- **Registres interrogés directement** : PyPI JSON, npm registry, nodejs.org/dist, python.org API, postgresql.org/versions.json et GitHub API (releases, commits, issues).
- **Spécification OpenAPI officielle d'intervals.icu** : `https://intervals.icu/api/v1/docs`, lue anonymement. Schémas `Wellness`, `SportSettings`, `Activity` et endpoint `/activity/{id}/streams`.
- **Recherches web et forum intervals.icu** : syntaxe du Workout Builder, GAP, streams.
- **Référence Anthropic** : skill claude-api (cache du 03/11/2026), pour le modèle, `output_config` et `effort`.

## 1. Versions de la Stack : écarts depuis le 23/11

| Composant | Spine | Registre au 25/11 | Constat |
| --- | --- | --- | --- |
| Python | 3.14.10 | 3.14.10 (25/10). 3.15.2 existe. | OK |
| FastAPI | 0.165.0 | 0.165.0 (18/11) | OK |
| uvicorn | 0.54.0 | 0.54.0 | OK |
| pydantic / pydantic-settings | 2.14.4 / 2.16.3 | identiques | OK |
| SQLAlchemy / Alembic | 2.1.9 / 1.20.1 | identiques | OK. L'extra `sqlalchemy` de Procrastinate demande `~=2.0`, compatible avec 2.1. |
| psycopg | 3.3.9 | 3.3.9 | OK |
| Procrastinate | 3.12.0 | 3.12.0 (19/11). CI jusqu'à Python 3.15 et PostgreSQL 19. | OK |
| **anthropic** | 2.12.0 | **2.14.0 (24/11)**. 2.13.0 est sortie le 23/11 à 18:34 UTC. | **Dépassé (mineur).** Les deux versions n'apportent rien de cassant pour l'usage prévu. 2.14.0 change seulement la gestion de `proxy=` / `NO_PROXY`. Passer à 2.14.0. |
| dbt-core | 1.12.7 | 1.12.7 | OK |
| dbt-duckdb | 1.11.1 | 1.11.1 (26/10). `duckdb>=1.0.0` non borné. | OK, mais voir F3 |
| DuckDB | 1.5.6 | 2.0.1 (20/11) est la dernière. 1.5.6 a un wheel cp314 manylinux aarch64. Le support de 1.5 est prolongé jusqu'en mars 2027. | OK : choix défendable et cohérent avec Deferred |
| DuckLake | 1.0 | La matrice indique extension 1.0 à partir de DuckDB 1.5.2. DuckLake 1.1 est livré avec DuckDB 2.0. | OK |
| PostgreSQL | 18.7 | 18.7, dernière mineure de la 18 (fin de vie 14/11/2030). 19.0 est la version courante. | OK |
| Node.js | 26.13.1 LTS | 26.13.1 « Lithium » | OK |
| **pnpm** | 12.21.0 | **12.23.0 (22/11)**. 12.22.0 date du 17/11. | **Affirmation inexacte** : le 23/11, la 12.21.0 n'était déjà plus la dernière version. Mineur. Passer à 12.23.0. |
| TypeScript | 5.9.3 | `latest` = 7.0.2. 6.0.3 existe. 7.1.0-beta est publiée. | OK : 5.9.3 respecte le peer `typescript ^5.x` d'openapi-typescript. |
| React / Vite / plugin-react / vite-plugin-pwa | 19.3.1 / 8.3.8 / 6.3.0 / 2.0.0 | identiques | OK |
| openapi-typescript | 7.13.0 | 7.13.0. Peer `typescript ^5.x`. Aucune version publiée depuis février 2026. | OK, à surveiller (voir Deferred « TypeScript 7 ») |
| pytest | 9.2.0 | 9.2.0 | OK |
| **Vitest** | 5.1.1 | **5.1.2 (25/11)** | Correctif publié aujourd'hui. Mineur. |
| Playwright | 1.65.1 | 1.65.1 | OK |
| Caddy | 2.11.9 | v2.11.9 | OK |
| restic | « dernière stable au déploiement » | v0.20.0 (19/11) | Conseil : épingler une version dans l'image de sauvegarde, pour que les tests de restauration de AD-19 restent reproductibles. |
| Modèle LLM | claude-opus-5-5 | Toujours le modèle par défaut et courant (4 $ / 20 $ par MTok). | OK, voir §3 |
| Tailscale Personal, Hetzner CAX11 | — | Non revérifiés depuis le 22/11 (memlog). | Rien de nouveau à signaler, mais rien n'a été revérifié. |

## 2. Constats

### F1 — MAJEUR : streams GAP et température affirmés sans support documenté (AD-2, AD-3, OQ-4)

Le spine ingère des « **streams** FC, allure, GAP, altitude et température » (AD-2).

- **Ce que confirme l'OpenAPI.**
  - `Activity` expose `gap` (moyenne), `gap_model` (enum `NONE` | `STRAVA_RUN`), `gap_zone_times` et `use_gap_zone_times` ;
  - `IntervalGroup` et `Interval` exposent `gap` par lap ;
  - les températures sont `average_temp`, `min_temp` et `max_temp` (capteur), ainsi que `average_weather_temp`, `min_weather_temp` et `max_weather_temp` (météo).
- **Ce que l'OpenAPI ne confirme pas.** `GET /api/v1/activity/{id}/streams{ext}` prend un paramètre `types` libre. `ActivityStream.type` est une simple `string`, sans enum. **Aucun nom de stream GAP n'est documenté.** Seules des sources tierces citent `temp` comme nom de stream, et aucune ne cite de stream GAP.
- **La GAP dépend d'un réglage.** `SportSettings.gap_model` peut valoir `NONE`. Or AD-2 ne lit dans les réglages de sport que les zones FC et la FC seuil. Si le réglage vaut `NONE`, la GAP manque, et rien ne permet de distinguer cette absence d'une absence de donnée.
- **La température est probablement vide.** L'Apple Watch n'a pas de capteur de température ambiante, donc les champs capteur des FIT HealthFit sont probablement vides. La seule température réaliste est la température **météo** d'intervals.icu (`*_weather_temp`), qui est un résumé et non un stream.
- **Correctifs.**
  - Dans AD-2, écrire « GAP et température **en résumé d'activité et de lap**, stream si disponible ».
  - Ajouter la lecture de `gap_model` aux réglages de sport ingérés.
  - Ajouter à OQ-4 : « nom exact du stream GAP, ou absence de ce stream ; `gap_model` du compte ; source de la température (capteur ou météo) ».

### F2 — MAJEUR : le repli Minetti crée deux définitions de la GAP (AD-3, AD-27)

- **Le problème.** AD-3 dit : « si intervals.icu ne fournit pas la GAP, le domaine la calcule depuis l'altitude (polynôme de Minetti) ». Or la GAP d'intervals.icu suit le **modèle Strava** (`gap_model: STRAVA_RUN`). C'est un modèle empirique fondé sur la FC, plus généreux en descente, et ce n'est pas Minetti.
- **Conséquence.** La série `gold` des meilleurs efforts en GAP, qui nourrit le Seuil d'allure (AD-27) et la VDOT (AD-13), mélangerait deux modèles selon les activités ou les périodes. Le calcul de la Conformité en GAP pourrait aussi changer de base sans prévenir. C'est l'inverse de « un seul propriétaire par calcul ».
- **Correctif.** Choisir **une seule source pour toute la série** :
  - (a) toujours Minetti, calculée par le domaine à partir de l'altitude. C'est le modèle de l'étude de vitesse critique citée par la recherche [104][105] ;
  - (b) toujours la GAP d'intervals.icu, avec `gap_model` contrôlé dans `sync_status`, et un ⚠️ si une activité n'a pas de GAP.

  Le spike OQ-4 doit trancher. Le choix retenu doit être écrit dans AD-3.

### F3 — MOYEN : dbt-duckdb 1.11.1 a un bug DuckLake corrigé seulement sur `main` (OQ-2)

- **Le bug.** Le ticket #617 « [ducklake] postfix for tables » montre qu'une table matérialisée par dbt garde le chemin physique `<modèle>__dbt_tmp/`, même après le renommage logique. DuckLake fixe le chemin au `CREATE`.
- **Le correctif.** Il est fusionné le 14/11 (PR #908). La PR #910 du 18/11 le restreint aux modèles SQL : les modèles Python gardent l'ancien comportement. Aucune version publiée ne le contient (dernière : 1.11.1 du 26/10).
- **Impact.** Les fichiers Parquet de `silver` et `gold` sont rangés sous des chemins `__dbt_tmp`, ce qui gêne la lecture humaine du lake, les scripts de maintenance et de sauvegarde (AD-25), et un éventuel nettoyage manuel. Le fonctionnement n'est pas cassé : le catalogue reste juste.
- **Correctif.** Ajouter à OQ-2 la vérification « chemin physique d'un modèle table après deux builds ». Si le bug gêne, attendre la 1.11.2, ou épingler un commit de `main`. Le Deferred DuckDB 2.0 reste valable :
  - la CI nightly de dbt-duckdb teste DuckDB 2.0 depuis le 20/10 (PR #858, incluse dans 1.11.1) ;
  - le ticket #870 sur les dépôts d'extensions signés de DuckDB 2.0 est encore ouvert ;
  - aucune compatibilité 2.0 n'est **déclarée**.

### F4 — MINEUR : allure absolue confirmée, mais son export vers la montre ne l'est pas (AD-8, AD-9, OQ-4)

- **Confirmé.** La syntaxe `5:00/km Pace`, les plages `4:30/km-4:40/km Pace` et les variantes `/mi` et `/400m` figurent dans le Workout Builder Syntax Quick Guide (mars 2026). La fonction est annoncée depuis le 17/11/2025 (« Specify workouts using absolute pace »). Elle a été conçue justement pour les plans construits hors d'intervals.icu, sans synchroniser le seuil. Cela va dans le sens de AD-8 et AD-27.
- **Non confirmé.** Le transport vers la montre n'est pas vérifié. Sur Garmin, des cibles d'allure se perdent, et un fil signale « FIT import strips pace targets ». Pour Zwift, intervals.icu envoie quand même son seuil d'allure en surcharge. Le comportement de HealthFit vers WorkoutKit est inconnu.
- **Correctif.** OQ-4 traite déjà le cas du `threshold_pace` vide. Ajouter explicitement : « une Séance avec cible **absolue** arrive-t-elle sur la montre avec la bonne plage, que `threshold_pace` soit renseigné ou non ? ».

### F5 — MINEUR : écarts de versions et affirmation « vérifié le 23/11 »

- **anthropic.** La 2.14.0 est sortie (voir le tableau).
- **pnpm.** La 12.23.0 était sortie dès le 22/11, la veille de la vérification annoncée.
- **Vitest.** La 5.1.2 est sortie aujourd'hui.

Aucun de ces écarts n'est bloquant. Comme le spine déclare « vérifié sur les registres 23/11 », il vaut mieux soit corriger les trois numéros, soit écrire « version minimale, mise à jour au lockfile ».

## 3. Affirmations intervals.icu et Anthropic : état de vérification

| Affirmation du spine | État | Source |
| --- | --- | --- |
| Syntaxe `description` avec allure absolue (`5:40/km Pace`) | **Confirmée** (documentation du forum, pas l'OpenAPI) | Quick Guide, fil « absolute pace » |
| GAP disponible sur les activités | **Confirmée en résumé** (`Activity.gap`, `Interval.gap`). **Non confirmée en stream.** Dépend de `gap_model`. | OpenAPI `/api/v1/docs` |
| Stream de température | **Non confirmé.** Probablement vide avec une Apple Watch. La météo `average_weather_temp` est confirmée en résumé. | OpenAPI, sources tierces |
| `SportSettings.threshold_pace` | **Confirmé** (`number`, avec `pace_units` à part, et aussi `Activity.threshold_pace`). Le spine ne le lit pas : cohérent avec AD-27. Si l'écriture est rouverte, vérifier l'unité (probablement des m/s). | OpenAPI |
| Wellness `sleepSecs` (durée de sommeil) | **Confirmé** (`integer`). Existent aussi `sleepScore`, `sleepQuality` et `avgSleepingHR`, que AD-26 exclut à juste titre. Le remplissage par HealthFit reste à vérifier (OQ-6). | OpenAPI |
| Wellness `restingHR`, `ctl` ; SportSettings `lthr`, `hr_zones` | **Confirmés** | OpenAPI |
| `claude-opus-5-5`, `output_config.format`, effort explicite | **Confirmés.** Sur Opus 5.5, l'effort vaut `medium` par défaut, donc la règle « toujours explicite » de AD-7 est justifiée. Thinking non désactivable. `tool_choice` forcé → 400, sans impact puisque la sortie est structurée. | skill claude-api (03/11/2026) |
| Refus du modèle compté comme une tentative (AD-7) | Cohérent. Le guide recommande en plus le paramètre serveur `fallbacks` (beta `server-side-fallback-2026-07-01`) sur Opus 5.5. À envisager. | skill claude-api |

## 4. Recommandations, par ordre

1. **AD-2 et AD-3** : remplacer « streams GAP et température » par « résumés d'activité et de lap, stream si disponible ». Lire `gap_model`. Fixer **une seule** source de GAP pour toute la série (F1, F2).
2. **OQ-4** : ajouter le nom du stream GAP, `gap_model`, la source de la température et l'export d'une cible absolue vers la montre (F1, F4).
3. **OQ-2** : ajouter le chemin physique DuckLake après rebuild, ainsi que la règle de choix entre la 1.11.2 et un commit épinglé (F3).
4. **Stack** : anthropic 2.14.0, pnpm 12.23.0, Vitest 5.1.2, et restic épinglé (F5).

## Sources

- [intervals.icu OpenAPI (officielle)](https://intervals.icu/api/v1/docs)
- [Workout Builder Syntax Quick Guide](https://forum.intervals.icu/t/workout-builder-syntax-quick-guide/123701)
- [Specify workouts using absolute pace](https://forum.intervals.icu/t/specify-workouts-using-absolute-pace/115846)
- [Run, intervals using pace - pace not synced to garmin connect](https://forum.intervals.icu/t/run-intervals-using-pace-pace-not-synced-to-garmin-connect/118130)
- [FIT import strips pace targets](https://forum.intervals.icu/t/fit-import-strips-pace-targets/120680)
- [Garmin : pace targets perdus pour les séances externes](https://forums.garmin.com/apps-software/mobile-apps-web/f/garmin-connect-web/438551/externally-generated-running-workouts-lose-pace-targets-while-hr-targets-remain-functional)
- [Gradient adjusted pace + Pace training load (modèle Strava)](https://forum.intervals.icu/t/gradient-adjusted-pace-pace-training-load/4031)
- [GAP discrepancies, Strava vs Intervals.icu](https://forum.intervals.icu/t/gap-discrepancies-strava-vs-intervals-icu/132337)
- [Access Activities Streams via API](https://forum.intervals.icu/t/access-activities-streams-via-api/101065)
- [Reverse-engineering Strava's GAP (Minetti et Strava)](https://aaron-schroeder.github.io/reverse-engineering/grade-adjusted-pace.html)
- [dbt-duckdb #908 Fix DuckLake table materialization paths](https://github.com/duckdb/dbt-duckdb/pull/908), [#910](https://github.com/duckdb/dbt-duckdb/pull/910), [#617](https://github.com/duckdb/dbt-duckdb/issues/617), [#858](https://github.com/duckdb/dbt-duckdb/pull/858), [#870](https://github.com/duckdb/dbt-duckdb/issues/870)
- [dbt-duckdb releases](https://github.com/duckdb/dbt-duckdb/releases), [PyPI dbt-duckdb](https://pypi.org/project/dbt-duckdb/)
- [DuckDB release calendar](https://duckdb.org/release_calendar), [DuckDB releases](https://github.com/duckdb/duckdb/releases), [Announcing DuckDB 2.0.0](https://duckdb.org/2026/10/21/announcing-duckdb-200)
- [DuckLake release calendar / matrice de compatibilité](https://ducklake.select/release_calendar)
- [anthropic-sdk-python releases](https://github.com/anthropics/anthropic-sdk-python/releases), [PyPI anthropic](https://pypi.org/project/anthropic/)
- [Procrastinate supported_versions.py](https://github.com/procrastinate-org/procrastinate/blob/main/.github/supported_versions.py)
- [npm pnpm](https://www.npmjs.com/package/pnpm), [npm vitest](https://www.npmjs.com/package/vitest), [npm typescript](https://www.npmjs.com/package/typescript), [npm openapi-typescript](https://www.npmjs.com/package/openapi-typescript)
- [PostgreSQL versions.json](https://www.postgresql.org/versions.json), [Node.js dist index](https://nodejs.org/dist/index.json), [restic releases](https://github.com/restic/restic/releases)
