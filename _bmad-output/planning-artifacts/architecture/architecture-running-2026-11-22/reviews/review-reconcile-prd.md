---
title: "Revue de réconciliation : spine d'architecture ↔ PRD + addendum"
reviewed: ARCHITECTURE-SPINE.md (updated 2026-11-23)
against: prd.md (final, 2026-11-22), addendum.md (2026-11-20)
decision_log: .memlog.md (2026-11-23T00:15)
date: 2026-11-23
---

# Revue de réconciliation PRD → spine

## Verdict

Le spine couvre bien la charpente du PRD : hexagone, deux magasins, outbox, garde-fous chiffrés, cibles par référence, fuseau unique, paramètres versionnés. On n'y trouve aucune contradiction bloquante avec une décision produit. Il reste en revanche **un défaut de structure sur le garde-fou des 30 jours** (AD-3 donne à `gold` un calcul qui doit inclure des Séances encore non persistées) et **un trou sur NFR-9** (rien ne garantit qu'un Feedback ne reste pas en file derrière une synchro ou une génération). Plusieurs règles discrètes du PRD (le Ressenti et son ordre, les relances, les transitions liées au temps, la réduction de charge en FR-22) restent assez floues pour que deux epics les implémentent différemment.

Critère retenu : on ne signale un point que si la structure du spine bloque l'exigence ou laisse diverger les builders. Les détails de pur produit qui vivent dans les stories sont exclus.

## Synthèse des constats

| # | Sévérité | Exigence | Constat | Correctif proposé |
| --- | --- | --- | --- | --- |
| 1 | **critique** | FR-5 (arbitrage 30), FR-22 | La fenêtre glissante de 30 jours inclut des Séances planifiées, mais AD-3 donne « plus longue sortie sur 30 jours » à `gold` | Resserrer AD-3 et AD-12 |
| 2 | **haute** | NFR-9 | Aucune règle de file, de concurrence ni de budget de temps : un Feedback peut attendre derrière `dbt build` ou derrière une génération | Compléter AD-6 et AD-11 |
| 3 | **haute** | FR-14, §7, SM-5 | Le Feedback n'a pas de sortie structurée, le validateur des nombres n'a pas de normalisation et le Feedback n'a pas d'états (relances, Plan actif) | Compléter AD-7, ajouter un état `feedback` à AD-4 |
| 4 | **haute** | FR-12, FR-1, FR-16 | Rien ne borne les Courses qui exigent un Ressenti (l'historique importé en déclencherait des centaines) ; ordre chronologique, immuabilité et verrou côté serveur non fixés | Compléter AD-15 et AD-18 |
| 5 | **moyenne** | FR-7, FR-11, FR-18, FR-8/SM-6 | Les transitions liées au temps n'ont pas de déclencheur ; `manquée → réalisée` n'est pas prévu ; l'unicité du Plan actif n'est pas structurelle ; l'annulation ne dit pas qu'elle supprime dans intervals.icu ; le Bilan peut être périmé à la génération | Compléter AD-4, AD-5, AD-14 |
| 6 | **moyenne** | FR-22 | La règle « une réduction n'est jamais refusée » entre en conflit avec la revérification de toutes les Séances restantes ; la règle des 7 jours est plus large dans AD-12 que dans le PRD | Resserrer AD-12 |
| 7 | **moyenne** | FR-13 | Le critère « 80 % de la durée dans la zone » demande un temps en zone par lap : l'ingestion ne dit pas qu'elle lit les flux FC | Compléter AD-15 et AD-2 |
| 8 | basse | FR-22 (demande à l'IA) | Le périmètre (une semaine au plus, aucun ajout ni suppression) n'est pas vérifié de façon déterministe après le LLM | Compléter AD-7 et AD-9 |
| 9 | basse | FR-22, FR-7, AD-5 | L'outbox est ambiguë : intention ou diff, suppression d'une « ancienne date » alors que l'`external_id` est stable | Resserrer AD-5 |
| 10 | basse | SM-C1, FR-19 | Le classement Ajustement ou modification n'est pas fixé pour le « déplacer » de FR-17 ni pour une demande à l'IA confirmée ; il manque l'avant/après qui permet le « pourquoi » d'une régénération | Compléter AD-4 |
| 11 | basse | NFR-4 | Le tarif par modèle n'est pas un paramètre : le coût en micro-USD n'a pas de source définie | Compléter AD-11 |
| 12 | basse | FR-11 | Une Course plus longue peut arriver après qu'un Ressenti a été posé sur la première : le rattachement est-il figé ? | Question à trancher, puis AD-15 |
| 13 | basse | FR-22, AD-11 | On ne sait pas quelle version de `parameter_set` s'applique à la revérification d'un Plan existant : celle du Plan ou la courante ? | Compléter AD-11 |

---

## Détail des constats

### 1. [CRITIQUE] Fenêtre de 30 jours : AD-3 met le calcul au mauvais endroit

**PRD** : FR-5 dit « aucune Séance ne dépasse de plus de 10 % la plus longue Course **ou Séance planifiée** des 30 jours précédents (fenêtre glissante qui inclut les Séances du Plan) ». L'arbitrage 30 et FR-22 imposent une revérification sur toutes les Séances restantes.

**Spine** : AD-3 classe « plus longue sortie sur 30 jours » parmi les agrégats `gold` et ajoute : « Un calcul qui a besoin d'un agrégat le **lit** dans `gold` et ne le recalcule jamais ». AD-2 inscrit aussi cet agrégat dans le lake.

**Problème** : `gold` ne voit que les Courses passées, et au mieux les Séances déjà persistées dans `app` au dernier `dbt build`. Il ne voit pas le Plan candidat, qu'il vienne d'une génération ou d'une modification non encore validée. À partir de la semaine 5 d'un Plan, la fenêtre de 30 jours ne contient plus que des Séances planifiées. Un builder qui applique AD-3 à la lettre comparera chaque Séance à la plus longue Course passée. Le garde-fou devient alors faux dans les deux sens : il refuse une progression légitime et laisse passer une Séance de +10 % sur une autre déjà à +10 %. AD-3 interdit justement de recalculer en Python, ce qui bloque la bonne implémentation.

**Correctif** :
- **AD-3** : `gold` expose une **série quotidienne** (la plus longue Course par jour local) et non un maximum déjà calculé. La règle des 10 % appartient au **domaine**, qui calcule le maximum glissant sur l'union de la série `gold` (jours passés) et des Séances candidates (Plan en cours de vérification). Il faut l'écrire comme une exception explicite au principe « ne jamais recalculer ».
- **AD-12** : préciser que la référence de « +10 % par semaine » pour la première semaine du Plan est le volume réalisé (`gold`), et le volume planifié pour les semaines suivantes.

### 2. [HAUTE] NFR-9 : rien ne garantit 30 s pour le Feedback

**PRD** : « un Feedback s'affiche en moins de 30 s après la pose du Ressenti, et un Plan en moins de 2 min, avec un état d'attente visible ».

**Spine** : AD-6 met tout dans Procrastinate, avec un verrou `lake` unique pour la chaîne ingestion → `dbt build` → détection. La carte des capacités cite « worker, squelette LLM, streaming ». Le spine ne dit rien de la concurrence du worker, des files ou d'un budget de temps.

**Problème** :
- Avec une concurrence de 1 (le défaut le plus simple), le job Feedback se place derrière une synchro en cours (un `dbt build` sur un CAX11) ou derrière une génération de Plan de 2 minutes.
- Trois tentatives Opus successives (AD-7) peuvent à elles seules dépasser 30 s.
- On ne sait pas si la Conformité, « Tes données » et la Matrice sont calculées pendant le job Feedback ou dès la détection.
- « Streaming » (dans la carte) et « pas de push » (AD-6) se contredisent en apparence. Le memlog le règle : le streaming se fait côté serveur, pour la génération uniquement.

**Correctif (AD-6)** :
- Deux files Procrastinate. Une file `interactive` (Feedback, demande à l'IA), prioritaire et jamais bloquée par le verrou `lake`, qui ne fait que **lire** `gold`. Une file `batch` (synchro, dbt, génération, Publication). Concurrence du worker ≥ 2.
- Calculs déterministes du Feedback (Conformité, « Tes données ») persistés **à la détection**. Seule la Matrice attend le Ressenti, et le job Feedback se limite à Matrice + LLM + validateur.
- **AD-11** : un budget de temps par usage (`feedback` 30 s, `plan` 120 s), en paramètre. Une fois le budget dépassé, le cycle s'arrête comme après 3 échecs (« avis indisponible » avec « Tes données » et la Conformité affichées).
- Préciser que le streaming est entre le worker et Anthropic, jamais vers le front.
- Dédoublonner les synchros déclenchées à chaque ouverture (`queueing_lock` Procrastinate), aussi pour NFR-7.

### 3. [HAUTE] FR-14 : sortie libre, validateur et états du Feedback non définis

**PRD** : exactement 1 point fort, 1 point à améliorer, 1 conséquence. Chaque nombre cité doit figurer dans les données calculées, avec 3 tentatives au plus, puis « avis indisponible ». En cas d'erreur, l'utilisateur choisit « Relancer » ou « Arrêter l'analyse », avec **2 relances au plus par Feedback, uniquement tant que le Plan est actif**. Chaque relance compte dans NFR-4. §7 : aucun compliment sans chiffre. SM-C2 : ce format est un plafond.

**Spine** : AD-7 prévoit une sortie structurée **« pour le Plan et les demandes à l'IA »**, ce qui exclut le Feedback. Le validateur est seulement cité. Aucun état de Feedback, aucun compteur de relances, aucune condition « Plan actif ».

**Problème** :
- En texte libre, la structure 1/1/1 n'est pas vérifiable, et chaque builder écrira son propre extracteur de nombres. Il faut une règle commune pour « 4'12 », « 1 h 40 », « +6 », « 252 s/km », « semaine 3 », les dates et les arrondis. Sans elle, on aura soit des rejets à tort (coût, NFR-9), soit des chiffres faux qui passent (SM-5, principe 4).
- Sans machine à états, chacun inventera les siennes (`pending`, `ready`, `unavailable`, `stopped`, `error`) et sa propre règle de relance.

**Correctif** :
- **AD-7** : sortie structurée aussi pour `feedback`, `{strength, improvement, consequence}` (une phrase chacun). Le validateur du domaine est **un seul module** d'extraction et de normalisation des nombres, avec une table de formats et une tolérance d'arrondi fixées par un paramètre (AD-11). Chaque nombre normalisé doit appartenir à l'ensemble des faits transmis, y compris la lecture de la Matrice et les métadonnées du Plan (numéros de semaine, dates).
- **AD-4** : ajouter la machine à états du `feedback` : `pending → ready | unavailable`, `error → (relaunch) pending | stopped`. Stocker `relaunch_count ≤ 2` (paramètre). Le cas d'usage « Relancer » refuse si le Plan n'est pas *actif*.

### 4. [HAUTE] FR-12 : quelles Courses demandent un Ressenti, dans quel ordre, et qui bloque l'affichage

**PRD** : le Ressenti est demandé dès la récupération, avant tout chiffre. Il est obligatoire, ne peut être ni ignoré ni reporté, et n'est plus modifiable une fois posé. Plusieurs Ressentis en attente sont demandés **dans l'ordre chronologique**. FR-16 compte les 🥵 Courses non planifiées comprises. FR-1 importe tout l'historique.

**Spine** : AD-15 prévoit au plus un Ressenti par activité. AD-18 interdit toute règle métier dans le front. Rien de plus.

**Problème** :
- **Borne de départ** : l'import initial (FR-1) fait remonter des centaines de Courses historiques. Sans borne, un builder qui crée un `run_attachment` « non planifié » pour chacune, comme le suggère « Une Course non planifiée a un `run_attachment` sans `workout` », produira des centaines de Ressentis obligatoires, et la séquence 🥵 de FR-16 n'aura pas de début.
- **Ordre** : il faut l'heure de début de l'activité (UTC). Une `DATE` locale ne départage pas deux Courses du même jour.
- **Immuabilité et verrou** : AD-15 dit « au plus un », pas « non modifiable ». Comme le front n'a pas de règle métier, c'est **l'API** qui doit refuser de servir les chiffres d'une Course sans Ressenti. Sinon le verrou n'existe que dans l'UI.

**Correctif** :
- **AD-15** : seules les activités dont le début est **postérieur à la validation du Profil (FR-2)** entrent dans le cycle Ressenti/Feedback. L'historique antérieur reste un fait `gold`, sans `run_attachment`. `feeling` est en **insertion seule** (aucun endpoint de mise à jour, contrainte d'unicité).
- **AD-18** : un endpoint `pending_feelings` trié par `started_at` croissant. Les endpoints qui exposent des chiffres d'une Course sans Ressenti répondent `409 FEELING_REQUIRED`.

### 5. [MOYENNE] Transitions liées au temps, unicité du Plan actif et annulation

**PRD** :
- FR-18 : une Séance passe à *manquée* le lendemain.
- FR-7 : un Plan passe à *terminé* le lendemain de sa dernière Séance. Accepter un nouveau Plan, après confirmation, *annule* l'ancien et **retire ses Séances à venir d'intervals.icu**.
- Glossaire : un seul Plan actif ou suspendu à la fois.
- FR-11 : une Course de la veille ou du lendemain d'une Séance non réalisée est proposée au rattachement.
- FR-8/SM-6 : le Plan 2 lit le Bilan du Plan 1.

**Spine** : AD-4 fixe une machine à états. AD-14 impose une fonction de jour unique. AD-5 couvre l'upsert et la suppression lors d'un déplacement. Le Bilan est un modèle dbt `gold`.

**Problème** :
- **Déclencheur** : le spine ne dit pas qui fait passer une Séance à *manquée* ou un Plan à *terminé*, ni quand. Une tâche périodique Procrastinate, une évaluation à chaque synchro et un calcul paresseux à la lecture donnent des états différents, et dbt (qui lit `app`) verra des états périmés.
- **`manquée → réalisée`** : le jour J+1, la Séance de J est *manquée* (FR-18), mais une Course de J+1 peut encore lui être rattachée (FR-11). Si la machine à états n'autorise pas cette transition, FR-11 est bloqué. Si elle l'autorise sans borne, une Séance redevient réalisable indéfiniment.
- **Unicité** : « un seul Plan actif ou suspendu » ne figure pas comme contrainte.
- **Annulation** : AD-5 ne parle que de « modification de Séances publiées ». Il n'est pas écrit que l'annulation (FR-7, ou FR-17 « Blessé → annuler ») émet des **suppressions** dans l'outbox pour toutes les Séances à venir, ni que l'acceptation du nouveau Plan et l'annulation de l'ancien forment **une seule transaction**.
- **Bilan périmé** : le Bilan n'existe qu'après un `dbt build` postérieur à la transition. Une génération lancée juste après un passage à *terminé* lit un Bilan absent, ce qui fait échouer SM-6, critère bloquant.

**Correctif** :
- **AD-14** : les transitions liées au temps sont appliquées par **un seul cas d'usage** (`advance_calendar`), en tête de chaque synchro et en tâche périodique quotidienne, de façon idempotente.
- **AD-4** : la machine à états autorise `manquée → réalisée` uniquement par un rattachement confirmé dans la fenêtre veille/lendemain. Ajouter un index unique partiel `plan(athlete_id) WHERE state IN ('active','suspended')`. L'acceptation d'un nouveau Plan est un seul cas d'usage et une seule transaction : annulation de l'ancien, activation du nouveau, outbox.
- **AD-5** : l'outbox porte aussi des **suppressions**. L'annulation d'un Plan émet la suppression de ses Séances à venir.
- **AD-6** : la génération de Plan commence, sous le verrou `lake`, par un `dbt build` (ou par une vérification de fraîcheur) pour garantir que le Bilan du Plan précédent existe.

### 6. [MOYENNE] FR-22 : la réduction de charge face à la revérification globale, et la règle des 7 jours

**PRD** : après toute modification, les garde-fous sont revérifiés sur toutes les Séances restantes. Une modification qui **augmente le risque** et en franchit un est refusée. Une modification qui **réduit la charge n'est jamais refusée**. Après un Ajustement de surmenage, pendant 7 jours, **toute hausse manuelle d'une Séance allégée** est refusée.

**Spine** : AD-12 reprend les deux familles et le résultat `{refused, informational}`, mais écrit « pendant 7 jours après un allègement de surmenage, **aucune hausse** ».

**Problème** :
- **Effets de bord d'une réduction** : réduire la semaine 5 peut faire échouer la règle des +10 % en semaine 6, qui est comparée à la « dernière semaine non allégée ». La semaine 5 peut aussi devenir une semaine « allégée » et décaler le rythme 3/4. La revérification globale trouvera alors une violation. Faut-il refuser (contraire au PRD), signaler, ou ignorer ? Aucune règle ne dit comment classer une violation **dans `refused` ou dans `informational`**, ni comment on mesure qu'une modification « augmente le risque ».
- **Portée des 7 jours** : le PRD vise les hausses *manuelles* de Séances *allégées*. AD-12 interdit *toute* hausse, ce qui toucherait aussi une régénération ou une autre Séance. Pour appliquer la règle, il faut aussi que l'`adjustment` de surmenage enregistre les Séances allégées et son horodatage.
- **Violations déjà présentes** : un Plan existant peut déjà violer une règle sans lien avec la modification, après un changement de paramètres ou de nouvelles Courses dans `gold`. La règle doit préciser qu'on compare l'état avant et l'état après.

**Correctif (AD-12)** :
- Une vérification compare l'état **avant et après** pour chaque garde-fou. Seule une violation **nouvelle ou aggravée**, causée par une Séance dont la charge (distance, durée, zone de la référence) **augmente**, va dans `refused`. Toute autre violation va dans `informational`.
- « Hausse » se définit de façon déterministe : distance ou durée en hausse, ou référence d'intensité plus élevée.
- Restreindre la règle des 7 jours aux **hausses demandées par l'utilisateur sur les Séances listées dans l'`adjustment` de surmenage**. La durée se calcule dans le fuseau de l'athlète (AD-14) et c'est un paramètre (AD-11).

### 7. [MOYENNE] FR-13 : le temps en zone FC demande des données que l'ingestion ne lit pas

**PRD** : un Bloc ciblé en FC est respecté si au moins 80 % de sa durée se passe dans la zone cible.

**Spine / addendum** : la lecture se fait par `GET /activity/{id}?intervals=true` puis `icu_intervals`. AD-15 décrit `bronze` comme du JSON brut d'activité. Le spine ne parle nulle part de flux.

**Problème** : `icu_intervals` donne des moyennes par lap. Une FC moyenne dans la zone ne signifie pas 80 % du temps dans la zone. Sans flux FC (ou sans temps en zone par lap, s'il existe), un builder calculera sur la moyenne, ce qui viole FR-13 et NFR-3. Ce choix pèse sur le volume du lake, sur les quotas (NFR-7) et sur le modèle `silver`.

**Correctif** :
- **AD-15** : l'ingestion lit aussi les **flux FC** (`streams`) des activités soumises au cycle Ressenti (voir le constat 4), dans une table `bronze` dédiée.
- **AD-3** : le temps en zone par lap est un modèle `silver`/`gold` (fait par lap), consommé par la Conformité du domaine.
- Ajouter à « Vérifications en attente » : vérifier si intervals.icu fournit un temps en zone par intervalle, ce qui éviterait de lire les flux.

### 8. [BASSE] « Demander à l'IA » : vérifier le périmètre de façon déterministe

**PRD** : une demande porte sur une semaine au plus. Plusieurs semaines, un ajout ou une suppression de Séance, un changement d'Objectif ou une demande ambiguë reçoivent la réponse « pas pris en charge » et rien ne change. Après 3 échecs, l'application nomme le garde-fou et propose la modification à la main.

**Spine** : AD-7 et AD-9 prévoient une sortie structurée validée, mais ne disent pas que **le domaine** vérifie le périmètre.

**Correctif (AD-7)** : la sortie `ai_edit` est soit un **diff sur des `workout_id` existants** (date, type, volume, référence), soit `{unsupported: reason}`. Le domaine rejette tout diff qui crée ou supprime une Séance, touche plus d'une semaine ISO locale ou change l'Objectif. Ce rejet ne coûte pas de tentative : c'est une réponse « non pris en charge » immédiate.

### 9. [BASSE] Outbox : intention ou diff, et « suppression de l'ancienne date »

**Spine (AD-5)** : l'`external_id` est l'UUID de la Séance, stable après un déplacement, et « un déplacement émet aussi la suppression de l'ancienne date ».

**Problème** : avec un upsert par `external_id`, déplacer la Séance met simplement à jour la date de l'événement. Une suppression adressée par `external_id` pourrait effacer l'événement qu'on vient de déplacer. Une suppression adressée par date pourrait viser un autre événement. Par ailleurs, on ne sait pas si une ligne d'outbox porte un diff ou une intention de type « publier l'état courant de la Séance X ».

**Correctif (AD-5)** : une ligne d'outbox est une **intention idempotente** : `upsert(workout_id)` ou `delete(workout_id)`. Le worker publie l'**état courant** lu au moment de l'envoi, ce qui fusionne naturellement les modifications successives. On ne supprime une ancienne date que si l'upsert d'intervals.icu ne déplace pas l'événement, ce qu'il faut confirmer par le test aller-retour (AD-19).

### 10. [BASSE] SM-C1 : classement des changements et « pourquoi » d'une régénération

**PRD** : les modifications de l'utilisateur, **y compris les propositions de l'IA qu'il a demandées**, sont comptées à part des Ajustements. Le « déplacer » de FR-17 peut s'écarter des préférences. Une régénération dit ce qu'elle ne conserve pas : « ton déplacement du 4 déc. n'est pas conservé ».

**Spine** : il existe des tables `adjustment`, `user_modification` et `proposal`, mais aucune règle de classement.

**Correctif (AD-4)** :
- Une demande à l'IA confirmée va dans `user_modification`.
- Un Imprévu (FR-17, y compris « déplacer »), le surmenage (FR-16) et « Reprendre » vont dans `adjustment`.
- `user_modification` enregistre le `workout_id`, le type et les valeurs avant et après, pour que la régénération puisse citer les modifications perdues.

### 11. [BASSE] NFR-4 : d'où vient le coût

**Spine** : `llm_call` stocke le coût en micro-USD, et `parameter_set` contient le taux €/$, mais **pas le tarif par modèle**.

**Correctif (AD-11)** : ajouter au `parameter_set` le tarif par modèle (entrée, sortie, cache). Le coût est calculé une seule fois, par le domaine ou l'adaptateur LLM, à partir de l'usage renvoyé par l'API. Les appels refusés ou en erreur sont aussi journalisés, puisque chaque relance compte.

### 12. [BASSE] FR-11 : une Course plus longue arrive après le premier Ressenti

**PRD** : s'il y a plusieurs Courses le même jour, la plus longue est rattachée.

**Problème** : si une seconde Course, plus longue, remonte lors d'une synchro ultérieure, alors que la première a déjà son Ressenti et son Feedback (calculé contre la Séance), faut-il rattacher à nouveau ? Le Ressenti et le Feedback sont attachés à l'activité (AD-15), mais la Conformité dépend de la Séance.

**Correctif** : point à trancher avec Badr. La proposition la plus simple est que le rattachement est **figé une fois le Ressenti posé**, et que la Course arrivée ensuite est traitée comme non planifiée. Il faudra l'inscrire dans AD-15 et amender FR-11.

### 13. [BASSE] Version de paramètres pour la revérification

**Spine (AD-11)** : chaque `plan`, `compliance` et `feedback` enregistre sa version. Le spine ne dit pas quelle version s'applique quand FR-22 ou FR-19 revérifie un Plan existant après un recalibrage.

**Correctif** : la revérification utilise **la version courante**, et un Plan modifié prend cette version. Les écarts déjà présents relèvent de `informational` (voir le constat 6).

---

## Écarts intentionnels (consignés dans le memlog, pas des gaps)

| Sujet | PRD | Spine | Statut |
| --- | --- | --- | --- |
| Blessures comme garde-fou déterministe | FR-5, FR-22 (sécurité physique), §12 Q8 | AD-12 : pas de garde-fou déterministe en v1, justification par le LLM, Imprévu « Blessé » ; contraintes structurées en v1.1 | **Intentionnel** (memlog : décision « option A »). **Amendement du PRD en attente** (FR-5, FR-22, §12 Q8), à tracer jusqu'au Finalize |
| Bande « polarisé » | Addendum : Z1 ≥ 75 % | AD-12 : Z1-Z2 ≥ 75 % (modèle unique à 5 zones) | **Intentionnel** (memlog : décision sur les bandes). Conséquence : 80/20 et polarisé ne se distinguent plus que par Z3 et Z4-Z5 |
| Semaine allégée chiffrée | §12 Q7, ouverte | AD-12 : ≤ 75 % de la dernière semaine non allégée | Résolu (memlog) |
| Seuil du chrono irréaliste | §12 Q2, addendum §2 | AD-13 : VDOT + min(semaines/5 ; 4) | Résolu, heuristique notée `[ASSUMPTION]` dans le memlog, correction marathon différée |
| Squelette par le LLM, Blocs par le code | Addendum §2 : le LLM produit des Blocs | AD-9 : le LLM produit un squelette, le code construit les Blocs | Intentionnel (memlog C4), plus strict que le PRD |
| Bilan de fin de Plan dans `gold` | FR-8 : « stocké dans le Profil » | AD-2 : `gold_plan_review` | Intentionnel (memlog, option 2). Voir le constat 5 sur la fraîcheur |

Point mineur sans décision consignée : la bande Daniels de l'addendum demande « allure E pour la **majorité** du volume ». AD-12 ne reprend que les plafonds T, I et R. Il faut l'ajouter comme paramètre (E ≥ 50 % par exemple) ou noter qu'on l'abandonne volontairement.

## Exigences vérifiées comme couvertes

FR-1 (progression par `GET /jobs`), FR-9 (ciblage, round-trip, `external_id`), FR-10 (`sync_status`), FR-14 (validateur et 3 tentatives, notes exclues des prompts), FR-15/FR-16 (fonctions du domaine paramétrées, `run_attachment` pour les Courses non planifiées), FR-17/FR-19 (arbre déterministe, revérification à la confirmation par la version du Plan), FR-22 (préférences contournables sans changer le Profil, constructeur de Blocs unique), NFR-2 (AD-17), NFR-3 (AD-3, AD-8, AD-11), NFR-6 (AD-16), NFR-7 (limiteur 429), NFR-8 (AD-19), ton et tutoiement (conventions). Les détails restants (refus d'une date à moins de 4 semaines, libellés de la Matrice, texte des messages) relèvent des stories.

Question produit hors périmètre de l'architecture, à soumettre au PM : les Séances d'un Plan *suspendu* (FR-17) restent-elles dans intervals.icu, et donc sur la montre ? Le PRD ne le dit pas. Selon la réponse, la suspension émet ou non des suppressions dans l'outbox (voir le constat 5).
