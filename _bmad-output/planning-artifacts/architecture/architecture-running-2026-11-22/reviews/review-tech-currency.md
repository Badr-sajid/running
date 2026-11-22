---
type: review
lens: tech-currency (fraîcheur et réalité des choix techniques)
target: ../ARCHITECTURE-SPINE.md
companion: ../.memlog.md
date: 2026-11-23
---

# Revue « fraîcheur technique » — ARCHITECTURE-SPINE (running)

## Verdict

Les numéros de version de la Stack sont **exacts et à jour** : tous ont été revérifiés aujourd'hui sur les registres. En revanche, plusieurs **mécanismes** sur lesquels le spine s'appuie ont été affirmés sans être testés, et deux d'entre eux sont **faux ou fragiles en l'état**. D'abord, openapi-typescript 7.13 ne fonctionne pas avec TypeScript 7.0. Ensuite, l'upsert intervals.icu par `external_id` est réservé aux événements créés par une application OAuth, alors que le spine prévoit une clé API. Il faut corriger ces deux points avant de lancer les epics, et ajouter deux spikes courts (DuckLake/dbt et intervals.icu).

## Méthode

- **Registres interrogés le 23/11/2026** : PyPI JSON, npm registry, nodejs.org/dist, postgresql.org/versions.json, python.org API et GitHub API (releases, CI, issues).
- **Spécification OpenAPI officielle d'intervals.icu** : `https://intervals.icu/api/v1/docs`, lue anonymement, sans aucune donnée personnelle.
- **Inspection de paquets** :
  - le tarball d'openapi-typescript 7.13.0, pour ses imports `typescript` ;
  - le wheel anthropic 2.12.0, pour `messages.parse` et `output_config`.
- **Recherches web** : DuckLake, dbt-duckdb, TypeScript 7, Tailscale et le forum intervals.icu.
- **Référence Anthropic** : skill claude-api (cache du 03/11/2026).

## 1. Versions de la Stack — contrôle croisé

| Composant | Spine | Registre (23/11) | Constat |
| --- | --- | --- | --- |
| Python | 3.14.10 | 3.14.10 (25/10). 3.15.2 existe aussi. | OK. Rester en 3.14 est prudent, car 3.15 est sortie en octobre. |
| FastAPI | 0.165.0 | 0.165.0 (18/11) | OK |
| uvicorn | 0.54.0 | 0.54.0 | OK |
| pydantic / pydantic-settings | 2.14.4 / 2.16.3 | identiques | OK |
| SQLAlchemy / Alembic | 2.1.9 / 1.20.1 | identiques | OK |
| psycopg | 3.3.9 | 3.3.9 (12/11) | OK |
| Procrastinate | 3.12.0 | 3.12.0 (19/11). CI 3.11 à 3.15. `requires-python >=3.11.5`. `psycopg[pool]` non borné. | OK : Python 3.14 et psycopg 3.3 sont couverts (question 3). |
| anthropic | 2.12.0 | 2.12.0 (20/11) | OK, voir §5 |
| dbt-core | 1.12.7 | 1.12.7. Classifiers jusqu'à Python 3.14. | OK |
| dbt-duckdb | 1.11.1 | 1.11.1 (26/10). `duckdb>=1.0.0` **non borné**. CI Python 3.14. | Voir F3 |
| DuckDB | 1.5.6 | Dernière version 2.0.1 (20/11). 1.5.6 du 28/09, avec wheels cp314 linux aarch64. | Choix défendable, voir F3 |
| DuckLake | « 1.0 » | 1.0 avec DuckDB 1.5.x. **DuckLake 1.1 existe** mais exige DuckDB ≥ 2.0.0. | À préciser, voir F3 |
| PostgreSQL | 18.7 | 18.7 (fin de vie 14/11/2030). 19.0 GA. | OK |
| Node.js | 26.13.1 LTS | 26.13.1 « Lithium » (10/11) | OK |
| pnpm | 12.21.0 | 12.21.0 | OK |
| TypeScript | 7.0.2 | 7.0.2 (08/07). **Pas d'API JS classique.** | **Bloquant**, voir F1 |
| React / Vite / plugin-react | 19.3.1 / 8.3.8 / 6.3.0 | identiques. plugin-react : peer `vite ^8`. | OK |
| vite-plugin-pwa | 2.0.0 | 2.0.0. Peer `vite … ^8.0.0`. Workbox 7.4. | OK : compatible Vite 8 et indépendant de TS. |
| openapi-typescript | 7.13.0 | 7.13.0 (11/02/2026). **Peer `typescript ^5.x`.** | **Bloquant** avec TS 7, voir F1. Aucune release depuis 9 mois. |
| Vitest | 5.1.1 | 5.1.1. Engines node `>=26` acceptés. | OK |
| Playwright | 1.65.1 | 1.65.1 | OK |
| pytest | 9.2.0 | 9.2.0 | OK. **Absent de l'entrée (version) du memlog** : version vérifiée par cette revue seulement. |
| Caddy | 2.11.9 | v2.11.9 (05/11) | OK |
| Modèle LLM | claude-opus-5-5 | Modèle courant par défaut (skill claude-api, 4 $/20 $ par MTok) | OK |

Les numéros sont donc tous réels et cohérents avec le memlog. Les problèmes viennent de la **compatibilité entre composants**, qui n'a pas été vérifiée.

## 2. Constats

### F1 — BLOQUANT : TypeScript 7.0 casse openapi-typescript 7.13 (AD-18)

- **Fait vérifié** : le paquet `typescript@7.0.2` n'exporte plus que `./lib/version.cjs` et des sous-chemins `./unstable/*`. Il n'y a ni `main` ni `types`. Microsoft l'annonce : « TypeScript 7.0 does not ship with an API », et une nouvelle API est prévue en 7.1.
- **Ce qu'utilise openapi-typescript 7.13.0** : il importe `typescript` à l'exécution (`dist/lib/ts.mjs` et tous les `transform/*.mjs`, avec `ts.factory.create…`). Il déclare aussi `peerDependencies: { typescript: "^5.x" }`.
- **Conséquence** : la génération du client API, seule source du contrat selon AD-18, échoue ou produit un avertissement de peer, puis plante. D'autres outils qui chargent l'API TS (typescript-eslint, par exemple) sont touchés de la même façon.
- **Correctif, à choisir et à écrire dans la Stack** :
  1. *Recommandé* : **TypeScript 6.0.3** pour le projet, puis passer à 7.x quand la 7.1 publiera une API et que openapi-typescript la supportera.
  2. Ou garder TS 7 pour `tsc` via un alias, et fournir une API TS 5 ou 6 aux outils qui en ont besoin :
     - le paquet de compatibilité `@typescript/typescript6` existe (6.0.2) ;
     - dans pnpm, alias `typescript` → `npm:@typescript/typescript6` et `@typescript/native` → `npm:typescript@7.0.2`, ou `packageExtensions`/`overrides` pour forcer `typescript@5.9.3` chez openapi-typescript.
  - Dans les deux cas, ajouter un test CI : « `openapi-typescript` régénère le client sans diff ».
- **Point annexe** : openapi-typescript n'a rien publié depuis février 2026. Ce n'est pas bloquant, mais c'est à surveiller, et `@hey-api/openapi-ts` peut servir d'alternative.

### F2 — MAJEUR : mécanique intervals.icu d'AD-5 partiellement fausse

Constats tirés de la spécification OpenAPI officielle (`/api/v1/docs`).

- **Upsert** : `POST /api/v1/athlete/{id}/events/bulk?upsert=true` existe bien. Sa définition est « *Update events with matching external_id **and created by the same OAuth application*** ». Un paramètre alternatif `upsertOnUid` existe aussi.
- **Suppression** : `PUT /api/v1/athlete/{id}/events/bulk-delete` supprime par `id` ou `external_id`. Là encore, « *If external_id is supplied then the event must have been created by the calling OAuth application* ».
- **Ce que prévoit le spine** : AD-16 dit que les identifiants intervals.icu sont lus via des variables d'environnement, et le memlog ne parle que de clé API. Avec une **clé API personnelle**, il n'y a pas d'application OAuth. Il n'est donc **pas garanti** que l'upsert par `external_id` retrouve l'événement : risque de doublons, contraire à SM-7 et à ce qu'AD-5 veut empêcher.
- **Déplacement de Séance** : AD-5 dit qu'« un déplacement émet aussi la suppression de l'ancienne date ». Or, si l'upsert par `external_id` fonctionne, l'événement existant est **mis à jour**, nouvelle date comprise : il n'y a rien à supprimer. Si l'upsert ne fonctionne pas, il faut supprimer **par `id` intervals.icu**. Il faut alors stocker `icu_event_id` sur `workout` ou dans l'outbox, ce que le modèle de données ne prévoit pas.
- **Intervalles d'activité** : `GET /api/v1/activity/{id}/intervals` existe. C'est conforme à ce que la Conformité suppose.
- **Quotas** : aucune limite n'est documentée dans la spec, et il n'y a pas d'en-tête `Retry-After` documenté. Sur le forum, le personnel indique environ **10 requêtes/s par IP**, avec une petite rafale, et **sans en-têtes de rate limit** sur ce plafond. Il existe aussi un ancien brouillon de quotas par 15 min et par jour avec `Retry-After`. AD-15 s'appuie sur `Retry-After`, ce qui n'est pas garanti : le limiteur doit avoir un repli (backoff exponentiel plus plafond local d'environ 5 req/s).
- **Correctif** :
  - **Spike n°1 (½ jour, sur le compte de test D1)** : avec la clé API, créer en bulk avec `external_id`, refaire un bulk `upsert=true` en changeant la date, puis vérifier l'absence de doublon. Tester aussi `bulk-delete` par `external_id`.
  - **Si le spike échoue** :
    - (a) enregistrer une application OAuth intervals.icu, même pour un seul athlète, et adapter le port credentials ;
    - ou (b) utiliser `upsertOnUid` ou un PUT par `icu_event_id` stocké.
  - Amender AD-5 (supprimer ou conditionner « suppression de l'ancienne date ») et AD-15 (repli quand il n'y a pas de `Retry-After`).

### F3 — MAJEUR : DuckDB 1.5.6 + DuckLake (catalogue Postgres) + dbt-duckdb, affirmé mais jamais exécuté

Questions 1 et 2 de la mission.

- **Ce qui est confirmé** :
  - l'extension `ducklake` est disponible depuis DuckDB 1.5.2 et implémente DuckLake 1.0 ;
  - les catalogues Postgres, SQLite et DuckDB sont supportés ;
  - dbt-duckdb accepte un `attach` avec `path: "ducklake:postgres:"`, un secret `postgres` et `data_path`.
- **Ce qui ne l'est pas** :
  - la seule source citée (cookbook MotherDuck) charge l'extension depuis **`repo: core_nightly`** et ne donne aucune version ;
  - aucune source ne montre le trio « dbt-duckdb 1.11.1 + DuckDB 1.5.6 + `ducklake:postgres` stable » en fonctionnement ;
  - la CI de dbt-duckdb épingle MotherDuck en `duckdb==1.5.5`.
- **Problèmes connus et ouverts** :
  - **dbt-duckdb #857** : avec un catalogue DuckLake sur Postgres, la liste des relations parcourt `system.information_schema.tables`, environ 11 s par schéma sur un lake de 800 tables. Le lake du projet sera petit, donc l'effet sera faible, mais c'est un coût fixe sur chaque `dbt build` de la chaîne de synchro (NFR-9).
  - **dbt-duckdb #748** : échecs intermittents `__dbt_tmp does not exist` quand `threads > 1` sur DuckLake. **Il faut `threads: 1`.**
- **Piège de dépendances** : dbt-duckdb exige `duckdb>=1.0.0` sans borne haute. Un `uv add dbt-duckdb` sans épinglage tire **DuckDB 2.0.1**, et l'extension `ducklake` passe alors en 1.1. Il faut épingler explicitement `duckdb==1.5.6` dans `pyproject.toml` (pas seulement dans le lock).
- **Montée en 2.0 (Deferred)** :
  - DuckLake 1.1 exige DuckDB 2.0. Un catalogue 1.0 reste utilisable, mais la migration (`AUTOMATIC_MIGRATION`) est **irréversible**.
  - Ajouter à la ligne Deferred : faire un `pg_dump` de la base `lake` avant toute montée, et monter `api` et `worker` **ensemble**, parce qu'un processus en 1.5 ne lira pas un catalogue migré en 1.1.
- **Extensions dans l'image Docker** : exécuter `INSTALL ducklake; INSTALL postgres;` **au build** de l'image ARM. Sinon, le premier `ATTACH` télécharge les extensions en prod, ce qui pose un problème de reproductibilité (NFR-8).
- **Correctif** : **spike n°2 (½ jour)**. Dans un conteneur Python 3.14 / linux-arm64 avec Postgres 18.7, lancer `dbt build` sur un modèle bronze → silver → gold vers `ducklake:postgres:`. Pendant ce `build`, lire `gold` depuis un second processus DuckDB, ce qui teste le côté `api` d'AD-2. Mesurer la durée du `build` à vide. Écrire dans la Stack : « DuckLake 1.0 (extension livrée avec DuckDB 1.5.6, dépôt `core`, pas nightly) », `threads: 1` et l'épinglage `duckdb==1.5.6`.

### F4 — MINEUR : détails LLM à jour mais à préciser (AD-7)

Question 7 de la mission.

- **Confirmé dans le wheel anthropic 2.12.0** : `client.messages.parse()` existe hors namespace beta (2 définitions, sync et async). Le paramètre `output_config` est présent. `messages.stream()` accepte aussi un format de sortie, donc on peut combiner streaming et sortie structurée pour la génération de Plan.
- **Précisions à apporter pour éviter une dérive** :
  - utiliser `output_config.format` / `messages.parse`, car le paramètre `output_format` est **déprécié** ;
  - sur Opus 5.5, l'effort par défaut est **`medium`** et la réflexion ne peut pas être désactivée (400). AD-11 rend l'effort paramétrable : fixer une valeur explicite par usage ;
  - `tool_choice` forcé (`any`/`tool`) renvoie 400 sur Opus 5.5. C'est cohérent avec le choix des sorties structurées : ne pas faire de tool use forcé ;
  - refus : vérifier `stop_reason == "refusal"` avant de lire `content`. Le fallback serveur recommandé (`betas=["server-side-fallback-2026-07-01"]`, `fallbacks="default"`) est **en beta**. Le memlog dit « fallback serveur » : l'indiquer comme beta et décider si un refus suivi d'un fallback réussi compte comme une tentative ;
  - version d'API `2026-10-21` : `temperature`, `top_p` et `top_k` renvoient 400. Ne pas les envoyer.

### F5 — MINEUR : Tailscale Serve convient au service worker, avec des prérequis à écrire (AD-17)

Question 6 de la mission.

- **Ce qui convient** : Serve sert un certificat **Let's Encrypt valide** pour `<machine>.<tailnet>.ts.net`. Le contexte est sécurisé, donc Safari iOS accepte l'enregistrement du service worker et l'installation de la PWA. Serve renouvelle lui-même le certificat, alors qu'avec `tailscale cert` le renouvellement est à la charge de l'utilisateur.
- **Prérequis à écrire dans le seed de déploiement** :
  - activer **MagicDNS** et **HTTPS Certificates** dans la console d'administration ;
  - le nom de machine apparaît dans les journaux publics de **Certificate Transparency**. Choisir un nom neutre, cohérent avec NFR-2 ;
  - Let's Encrypt limite le nombre de certificats émis, avec un blocage possible d'environ 34 h. Ne pas recréer le nœud en boucle : conserver le volume d'état de `tailscaled` dans Docker ;
  - l'iPhone doit avoir le VPN Tailscale actif pour joindre l'app. Sans VPN, le mode hors ligne d'AD-18 (cache GET) prend le relais, ce qui est cohérent.

### F6 — INFO : points sans source dans le memlog

- **DuckLake 1.0 en avril 2026** : d'abord noté « source secondaire », puis confirmé (ducklake.select, 13/04/2026). OK.
- **Hetzner CAX11 à environ 6 €/mois et Tailscale Personal gratuit** : non revérifiés ici, car ils ne bloquent rien. À confirmer au moment de commander le VPS.
- **Tarifs Anthropic** : viennent du cache du skill (03/11), donc vieux de 20 jours. Acceptable pour une estimation NFR-4.

## 3. Actions demandées

| # | Gravité | Action | Où |
| --- | --- | --- | --- |
| 1 | Bloquant | Remplacer `TypeScript 7.0.2` par `6.0.3`, ou documenter l'alias `@typescript/typescript6` pour les outils. Ajouter un test CI de régénération du client. | Stack, AD-18 |
| 2 | Majeur | Spike intervals.icu sur le compte de test : upsert et bulk-delete par `external_id` **avec une clé API**. Selon le résultat, passer à une app OAuth ou stocker `icu_event_id`. Amender AD-5 (déplacement) et AD-15 (pas de `Retry-After` garanti, plafond local d'environ 5 req/s). | AD-5, AD-15, AD-16, modèle `workout` |
| 3 | Majeur | Spike DuckLake : dbt-duckdb 1.11.1 + DuckDB 1.5.6 + `ducklake:postgres` sur arm64. Épingler `duckdb==1.5.6` dans `pyproject.toml`, fixer `threads: 1`, installer les extensions au build de l'image. | Stack, AD-6, deploy |
| 4 | Mineur | Montée DuckDB 2.0 / DuckLake 1.1 : migration irréversible, donc `pg_dump lake` d'abord et montée simultanée de `api` et `worker`. | Deferred |
| 5 | Mineur | AD-7 : `output_config.format`/`messages.parse`, effort explicite (le défaut `medium` d'Opus 5.5), fallback de refus en beta, pas de paramètres d'échantillonnage. | AD-7, AD-11 |
| 6 | Mineur | Déploiement : MagicDNS + HTTPS activés, nom de machine neutre (Certificate Transparency), état `tailscaled` persistant. | Structural Seed |
| 7 | Info | Ajouter pytest 9.2.0 à l'entrée (version) du memlog. | .memlog.md |

## Sources

- PyPI JSON : fastapi, uvicorn, pydantic, pydantic-settings, sqlalchemy, alembic, psycopg, procrastinate, anthropic, dbt-core, dbt-duckdb, duckdb, pytest (consultés le 23/11/2026).
- npm registry : typescript, @typescript/typescript6, vite, vite-plugin-pwa, @vitejs/plugin-react, openapi-typescript (tarball 7.13.0 inspecté), react, vitest, @playwright/test, pnpm.
- https://nodejs.org/dist/index.json · https://www.postgresql.org/versions.json · python.org API · GitHub caddyserver/caddy.
- Procrastinate : `.github/workflows/ci.yml` et `pyproject.toml` (branche main) ; releases GitHub.
- dbt-duckdb : releases 1.10.0 à 1.11.1 ; issues [#857](https://github.com/duckdb/dbt-duckdb/issues/857), [#748](https://github.com/duckdb/dbt-duckdb/issues/748).
- [DuckDB 1.5 — extension ducklake](https://duckdb.org/docs/1.5/core_extensions/ducklake) · [DuckLake 1.0](https://ducklake.select/2026/04/13/ducklake-10/) · [DuckLake docs (1.1 exige DuckDB 2.0)](https://ducklake.select/docs/stable/duckdb/introduction) · [Migration DuckDB 2.0](https://duckdb.org/docs/2.0/operations_manual/migration) · [Annonce DuckDB 2.0.0 LTS](https://duckdb.org/2026/10/21/announcing-duckdb-200)
- [MotherDuck cookbook dbt + DuckLake local](https://motherduck.com/docs/cookbook/dbt-local-ducklake/) (extension `core_nightly`).
- [Annonce TypeScript 7.0](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/) · [Nx — TypeScript 7](https://nx.dev/docs/kb/typescript-7)
- Spécification OpenAPI intervals.icu : https://intervals.icu/api/v1/docs (endpoints `events/bulk`, `events/bulk-delete`, `activity/{id}/intervals`) · [Forum — API access](https://forum.intervals.icu/t/api-access-to-intervals-icu/609/716) · [Integration Cookbook](https://forum.intervals.icu/t/intervals-icu-api-integration-cookbook/80090)
- [Tailscale — Enabling HTTPS](https://tailscale.com/kb/1153/enabling-https/)
- [Anthropic — Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) ; wheel `anthropic-2.12.0` inspecté (`resources/messages/messages.py`) ; skill claude-api (cache du 03/11/2026).
