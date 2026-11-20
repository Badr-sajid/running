# Digest de recherche : coach de course perso (LLM + intervals.icu + HealthFit / Apple Watch)

Date : 2026-11-20. Recherche web faite pour appuyer le PRD. Niveau de confiance : **[Officiel]** = doc ou page de l'éditeur ; **[Forum]** = forum intervals.icu, développeur ou utilisateurs ; **[Tiers]** = blog, revue ou comparateur ; **[À vérifier]** = non confirmé, à tester lors d'un spike.

---

## 1. API intervals.icu

### Authentification
- **Clé API personnelle** : à générer dans Settings > Developer Settings. Elle s'utilise en HTTP Basic avec l'utilisateur `API_KEY` et la clé comme mot de passe. Avec l'id athlète `0`, l'API agit sur l'athlète propriétaire de la clé. **[Forum, officiel admin]**
- **OAuth (Bearer)** : à réserver aux apps utilisées par plusieurs personnes. Les scopes cités sont `CALENDAR:WRITE`, `ACTIVITY:READ/WRITE` et `WELLNESS:READ/WRITE`. **[Forum]**
- **Conclusion pour notre cas (un seul utilisateur)** : la clé API en Basic auth suffit, OAuth n'est pas nécessaire.
- **Point d'attention** : Cloudflare peut bloquer certains User-Agent par défaut, comme `Python-urllib`. Il faut fixer un User-Agent explicite. **[Forum]**
- Base URL : `https://intervals.icu/api/v1`. La référence OpenAPI/Swagger est sur `https://intervals.icu/api-docs.html`. Un fil du forum signale que cette doc peut être en retard sur l'API réelle. **[Forum]**

### Endpoints utiles
| Besoin | Endpoint |
|---|---|
| Lister les séances planifiées et événements | `GET /athlete/{id}/events?oldest=YYYY-MM-DD&newest=YYYY-MM-DD` |
| Créer une séance planifiée | `POST /athlete/{id}/events` |
| Créer ou mettre à jour en masse (idempotent) | `POST /athlete/{id}/events/bulk?upsert=true` avec un tableau d'events portant chacun un `external_id` |
| Modifier ou supprimer une séance | `PUT` / `DELETE /athlete/{id}/events/{eventId}` ; suppression en masse avec `PUT /athlete/{id}/events/bulk-delete` (par `external_id` ou `id`) |
| Bibliothèque de workouts | `GET/POST /athlete/{id}/workouts`, dossiers `/folders` |
| Lister les activités | `GET /athlete/{id}/activities?oldest=&newest=` (ou `activities.csv`) |
| Détail d'une activité avec intervalles détectés | `GET /activity/{id}?intervals=true` ; fichiers : `/activity/{id}/file` (original), `/fit-file` |
| Wellness (HRV, FC repos, sommeil, poids…) | `GET /athlete/{id}/wellness?oldest=&newest=`, `GET/PUT /athlete/{id}/wellness/{date}`, `PUT /athlete/{id}/wellness-bulk` |

**Exemple d'event planifié** (exemple d'utilisateur du forum, déclaré fonctionnel, pas officiel) :
```json
{ "category": "WORKOUT", "start_date_local": "2026-02-03T17:00:00", "type": "Run",
  "name": "...", "description": "<texte workout intervals.icu>", "moving_time": 2760,
  "external_id": "<notre clé primaire>" }
```
- Le texte du workout va dans `description`, et intervals.icu en déduit la structure (`workout_doc`). **[Forum]**
- La correspondance par `external_id` ne fonctionne que sur les events créés par notre propre application. **[Forum]**
- Un utilisateur signale que des workouts créés par l'API ne se synchronisaient pas vers Garmin alors que ceux créés à la main passaient. Il faut tester la chaîne de bout en bout. **[Forum, À vérifier]**
- Champs wellness observés : `restingHR`, `hrv` (RMSSD), `hrvSDNN`, `sleepSecs`, `sleepScore`, `sleepQuality`, `avgSleepingHR`. Ils peuvent être `null`. Avec une Apple Watch, la HRV et la FC de repos ne remontent souvent qu'avec un suivi du sommeil actif. **[Tiers/Forum]**
- **Limite d'accès** : les données Strava ne sont pas exposées par l'API. Si les activités arrivent uniquement via Strava, l'API ne les verra pas. Il faut donc faire entrer les activités par HealthFit, upload FIT ou une autre source directe. **[Forum, officiel admin]**

### Limites de débit et conditions d'utilisation
- **Clé API** : 5 000 requêtes par jour, 2 500 par fenêtre glissante de 15 minutes, et 10 appels par seconde par IP. Au-delà, l'API répond HTTP 429 avec un en-tête `Retry-After`. Les en-têtes `X-RateLimit-Limit` et `X-RateLimit-Remaining` sont renvoyés. **[Forum, officiel admin]**
- L'usage de l'API implique d'accepter les « Intervals.icu API Terms ». L'accès est limité à ses propres données et aux athlètes suivis. Pour un usage perso, ces limites sont très larges.
- **Tarifs** : la version gratuite inclut le calendrier, le workout builder, fitness/fatigue/forme (CTL/ATL/TSB) et l'analyse. **Supporter à 4 $/mois** : annual training plan builder, **« Intervals MCP Connector » pour brancher son propre assistant IA**, prévisions météo, zones entièrement personnalisées, etc. **[Officiel]**

### Format texte des workouts structurés
Source principale : « Workout Builder Syntax Quick Guide ». **[Forum, officiel]**
- **Étapes** : chaque étape est une ligne qui commence par `-`. Une ligne sans tiret sert d'en-tête, par exemple `Warmup` ou `Cooldown`. Le texte placé avant la durée devient le libellé (cue) de l'étape.
- **Durées** : `30s`, `10m`, `1m30`, `1h`. **Attention : `m` veut dire minutes.** Pour les mètres, il faut écrire `mtr` (`400mtr`). Distances : `2km`, `1mi`.
- **Allure absolue** : `5:00/km Pace`, `4:30-4:40/km Pace`, ou `5:00 Pace` (l'unité par défaut du sport s'applique alors).
- **Allure relative au seuil** : `78-82% Pace`. **Zones d'allure** : `Z2 Pace`, `Z2-Z3 Pace`. Il faut avoir renseigné la Threshold Pace dans les réglages.
- **FC** : `95% LTHR`, `90-95% LTHR`, `Z2 HR`, `Z2-Z3 HR`, `70% HR` (% de la FC max).
- **Rampe** : `- 10m ramp 60-80% Pace`.
- **Répétitions** : une ligne `Main Set 4x` ou `4x` seule, suivie des étapes. Il faut une ligne vide avant et après le bloc. **Les répétitions imbriquées ne sont pas supportées.** Les cues générées ressemblent à « Main Set 1/4 ».
- **Allure de récupération** : on peut écrire `- 90s 50-80% Pace intensity=rest`. **[Forum]**
- Exemple course :
```
Warmup
- 15m Z2 HR

Main Set 6x
- 800mtr 3:50-4:00/km Pace
- 2m Z1 HR

Cooldown
- 10m Z1-Z2 HR
```
- **Recommandation** : faire générer par le LLM un objet JSON structuré, puis produire le texte intervals.icu avec du code déterministe et le valider (parser et round-trip via l'API). Le LLM ne doit pas écrire la syntaxe directement.

Sources :
- https://forum.intervals.icu/t/api-access-to-intervals-icu/609
- https://forum.intervals.icu/t/intervals-icu-api-integration-cookbook/80090
- https://forum.intervals.icu/t/uploading-planned-workouts-to-intervals-icu/63624
- https://forum.intervals.icu/t/workout-builder-syntax-quick-guide/123701
- https://forum.intervals.icu/t/workout-builder/1163
- https://forum.intervals.icu/t/support-for-running-pace-or-speed-in-workout-builder-syntax-done/91362
- https://forum.intervals.icu/t/solved-outdated-api-documentation/114931
- https://intervals.icu/api-docs.html
- https://www.intervals.icu/pricing

---

## 2. HealthFit, intervals.icu et Apple Watch

### Fonctionnement
- Depuis le **10 oct. 2025**, HealthFit importe les séances planifiées d'intervals.icu (bêta en sept. 2025). Pour activer la fonction, il faut se déconnecter puis se reconnecter à intervals.icu dans HealthFit. Les séances sont importées dans Apple Health et deviennent des workouts planifiés sur la montre, côté app Exercice, via WorkoutKit. **[Forum, développeur ; App Store]**
- **Paywall** : le 23 mars 2026, le développeur a indiqué que **« the workout planner requires the Fanatical Supporter plan »**. Il s'agit de l'achat intégré HealthFit **Fanatical Supporter, environ 9,99 $**. La fonction avait été ouverte à tous par erreur. Un utilisateur dit être passé à WorkOutDoors à cause de ce changement. **[Forum ; App Store]**
- HealthFit propose un widget « Next Scheduled Workout ». **[App Store]**
- Dans l'autre sens, HealthFit exporte les activités de la montre vers intervals.icu. C'est le canal qui alimente les données d'activité de l'app, à la place de Strava (voir §1).

### Limites connues ou probables
- **Une seule cible (alerte) par étape côté WorkoutKit** : chaque `WorkoutStep` porte un objectif (temps, distance ou ouvert) et **une alerte au maximum**, de type plage d'allure ou de vitesse, zone ou plage de FC, cadence ou puissance. Une étape ne peut donc **pas avoir à la fois une cible d'allure et une cible de FC**. **[Doc Apple, À vérifier dans le détail]**
- **Cibles d'allure** : avec d'autres passerelles (Intervals Companion v3, Watchletic), des utilisateurs du forum voient bien les cibles FC sur la montre mais **pas les cibles d'allure**, quel que soit le format. Rien de spécifique n'a été trouvé pour HealthFit. **[Forum, À vérifier]**
- Un utilisateur Garmin note qu'il faut garder une seule métrique de cible par workout pour qu'elle apparaisse sur la montre. C'est un argument de plus pour une cible unique et cohérente par séance. **[Forum]**
- **Rampes** : WorkoutKit n'a pas d'équivalent connu. Une rampe sera probablement aplatie ou ignorée. **[À vérifier]**
- **Répétitions** : WorkoutKit gère des blocs répétés (`IntervalBlock` avec un nombre d'itérations), et intervals.icu ne permet pas de les imbriquer. Des répétitions simples devraient donc passer. **[À vérifier]**
- **Étapes « jusqu'au lap »** : la fonction `Press lap` d'intervals.icu n'existe que pour Garmin. Côté Apple, il faut utiliser un objectif « open ». **[Forum]**
- Un signalement du 13 oct. 2025 décrit des séances planifiées absentes de la montre, sans résolution publiée. L'horizon de synchronisation (combien de jours d'avance) n'est pas documenté.
- Les montres et iPhone sous **watchOS / iOS 26** ont un vrai Workout Builder natif : étapes Work/Recovery, objectif temps, distance ou ouvert, cible allure (moyenne ou instantanée), FC (zone ou plage), cadence ou puissance, et répétitions groupées. Les cibles côté WorkoutKit sont donc bien supportées. Avec **watchOS 27**, les zones FC et puissance arrivent dans HealthKit. **[Tiers DC Rainmaker ; WWDC]**

### Conséquences pour le PRD
1. Fixer une règle : **une seule cible par étape**. Allure pour les séances qualitatives (seuil, VMA, tempo) avec une plage, pas une valeur unique ; FC (zone) pour l'endurance fondamentale et les récupérations.
2. Éviter les rampes et les répétitions imbriquées.
3. Prévoir un **spike de validation** : publier 3 à 4 workouts types puis vérifier leur rendu sur la montre (allure, FC, répétitions, distance en `mtr`).
4. Prévoir un plan B si HealthFit perd les cibles d'allure : WorkOutDoors, Watchletic, Intervals.icu Companion, ou une app compagnon WorkoutKit maison (projet beaucoup plus lourd).
5. Le prérequis est l'achat intégré HealthFit Fanatical Supporter, à inscrire dans les dépendances.

Sources :
- https://forum.intervals.icu/t/import-planned-workouts-from-intervals-icu-into-apple-health/112136
- https://apps.apple.com/us/app/healthfit/id1202650514
- https://forum.intervals.icu/t/intervals-companion-v3-updates-for-apple-health-and-workout-syncing/124208
- https://www.watchletic.com/intervalsicu
- https://developer.apple.com/documentation/workoutkit
- https://developer.apple.com/documentation/workoutkit/workoutstep
- https://developer.apple.com/videos/play/wwdc2023-10016
- https://developer.apple.com/videos/play/wwdc2026/207/
- https://www.dcrainmaker.com/2025/09/apple-watch-workout-builder-ios26-watchos26.html
- https://support.apple.com/guide/watch/apd66fcd5c5c/watchos

---

## 3. Produits comparables et positionnement

| Produit | Adaptation du plan | Retour après la séance | Prix (indicatif, fin 2026) |
|---|---|---|---|
| **Runna** (Strava) | Plans conçus par des coachs puis ajustés par un algorithme opaque. Les allures changent seulement si l'utilisateur rate ou dépasse ses cibles de façon répétée (« a single fast session is not enough »), surtout sur tempo et fractionné. Options « Not Feeling 100% » et « Adapt for Heat ». Pousse 2 semaines de séances vers Garmin. Le WSJ a rapporté des blessures liées à une progression d'allure trop agressive, et Runna a ajouté des moyens de baisser l'intensité. | Peu de questions au coureur ; l'adaptation repose surtout sur les allures enregistrées. « Runna AI » permet de revoir l'historique des ajustements. | ~19,99 $/mois, 119,99 $/an (149,99 $ avec Strava). Le statut d'une offre gratuite est contradictoire selon les sources. |
| **Garmin Coach** (adaptatif) et **Daily Suggested Workouts** | Le plan s'ajuste selon la forme, les préférences et la date de course. Les DSW se basent sur la récupération, la HRV et la charge. | L'utilisateur note la difficulté après chaque séance, ce qui influence la suite. Explications des ajustements améliorées en 2024. | Garmin Coach gratuit. Connect+ à 6,99 $/mois ou 69,99 $/an (Active Intelligence, conseils). |
| **TrainAsONE** | Planification algorithmique (« IA ») qui adapte en continu à partir des séances faites. | Évaluations automatiques. | Freemium, prix non trouvé. |
| **Athletica** | Plans IA adaptatifs (course, vélo, triathlon) basés sur charge et récupération. | Analyses de séance. | ~20 $/mois, essai 2 semaines. |
| **AI Endurance** | « Digital twin », DFA alpha 1, réajustement selon récupération et performance. | Analyses détaillées. | ~20 $/mois ou 200 $/an (oct. 2026). |
| **Runalyze** | Pas un coach. Analyse : VO2max effectif, prédictions de course, « Marathon shape » (l'éditeur précise qu'elle n'a pas de base scientifique). | Analyses riches (decoupling, quality sessions en premium). | Gratuit ; Supporter ~2,50 €/mois ; Premium ~5,50 €/mois. |
| **intervals.icu** natif | Calendrier, workout builder, CTL/ATL/forme. Annual plan builder pour les Supporters. Pas d'IA de planification native, mais un **MCP Connector** pour brancher son propre LLM (Supporter). | Analyse des intervalles et de la charge. | Gratuit ; Supporter 4 $/mois. |
| Écosystème IA sur intervals.icu | LeCoach, IntervalCoach, PlanWatts, Peaking, TrainerAI (Telegram, proactif), MyTrainPal, Intervals Pro… : coachs LLM tiers qui lisent intervals.icu et y écrivent. | Variable. | Souvent freemium. |
| **Strava Athlete Intelligence** | Aucune ; descriptif seulement (il explique le passé). | Résumé LLM après la séance, tendances sur 30 jours. | Inclus dans Strava Premium. |
| **Apple Watch / Fitness** (watchOS 26) | Pas de plan ni d'adaptation. Builder de séances personnalisées natif. | Métriques natives. | Inclus. |
| **ChatGPT et LLM génériques** | Plans statiques ; ne posent pas de questions de clarification ; aucune boucle avec les données. | À la demande. | Abonnement LLM. |

### Positionnement possible pour le projet (perso)
- **Différenciateurs** par rapport à Runna et Garmin :
  - (a) une **explication** de chaque séance et de chaque changement, ancrée dans les données réelles ;
  - (b) une **séparation explicite entre faits mesurés et opinion de l'IA** dans le retour après séance ;
  - (c) une **modification du plan seulement sur perturbation majeure et avec confirmation de l'utilisateur**. Runna, à l'inverse, ajuste automatiquement de façon opaque, et c'est ce qu'on lui reproche ;
  - (d) une mémoire du profil (blessures, préférences, contraintes) ;
  - (e) un coût marginal faible : intervals.icu gratuit ou 4 $/mois, HealthFit en achat unique, plus les tokens LLM.
- **Ce qui existe déjà** : la niche « coach LLM branché sur intervals.icu » est encombrée (LeCoach, IntervalCoach, TrainerAI…). Pour un projet perso ce n'est pas bloquant, mais on peut s'en inspirer, voire s'appuyer sur le MCP Connector officiel plutôt que de tout recoder.

Sources :
- https://the5krunner.com/2026/09/28/is-runna-ai/
- https://the5krunner.com/2026/10/14/is-runna-free/
- https://the5krunner.com/2026/02/21/runna-ai-marathon-training-injury/
- https://www.sensai.fit/blog/runna-vs-trainingpeaks-2026
- https://www.tomsguide.com/reviews/runna-app
- https://www8.garmin.com/manuals/webhelp/GUID-708A8F4D-9A78-49CF-9528-DE109BBCC472/EN-US/GUID-71798B16-A9A9-4D0F-BF64-50DD65AFC0D5.html
- https://dcrainmaker.com/2025/03/garmin-connect-plus-subscription-walkthrough.html
- https://www.shoulditrain.com/blog/garmin-connect-plus-review
- https://the5krunner.com/2024/11/12/garmin-adaptive-plans-get-improved-explanations/
- https://www.sensai.fit/blog/ai-endurance-review-2026
- https://athletica.ai/pricing
- https://apps.apple.com/pa/app/trainasone-ai-running-plans/id1541355896?l=en-GB
- https://runalyze.com/pricing
- https://blog.runalyze.com/?p=4808
- https://www.intervals.icu/pricing
- https://the5krunner.com/2026/06/12/lecoach-ai-cycling-running-coach/
- https://the5krunner.com/2026/06/23/intervalcoach-ai-training-app/
- https://forum.intervals.icu/t/trainerai-an-intervals-icu-coach-that-lives-in-telegram-writes-to-you-first-and-keeps-no-copy-of-your-data/132167
- https://forum.intervals.icu/t/any-reviews-or-comparisons-of-the-wealth-of-ai-tools-for-intervals/123739?page=3
- https://advnture.com/news/strava-launches-athlete-intelligence
- https://the5krunner.com/2026/03/30/type-to-run-weekly-coach/

---

## 4. Pièges des plans générés par LLM et garde-fous

### Ce que dit la littérature
- **JSSM 2024** (plans ChatGPT de 6 semaines notés par des coachs sur 22 critères) : les plans sont jugés **sous-optimaux**. La qualité augmente avec la quantité d'informations fournies. Les faiblesses relevées portent sur le **contrôle de la progression**, le **suivi individualisé** et la **gestion du risque de blessure**. Le modèle **ne pose pas de questions de clarification**. Les auteurs déconseillent d'utiliser ces plans sans l'avis d'un coach.
- **BMC Research Notes 2025** (étude pilote) : des coachs ont souvent du mal à distinguer un plan IA d'un plan humain. Un plan crédible n'est donc pas pour autant un plan sûr.
- **arXiv 2025** (un seul sujet, préparation semi-marathon avec un LLM) : l'adhérence et la motivation sont les points les plus durs. Les garde-fous et la personnalisation restent limités.
- Pièges typiques à couvrir :
  - **allures hallucinées** : le modèle invente une VMA ou une allure seuil sans données, ou fait des erreurs d'arithmétique (min/km vs km/h) ;
  - progression trop rapide du volume ou de l'intensité ;
  - trop d'intensité, au détriment de la répartition polarisée ou pyramidale ;
  - pas de semaines de décharge ;
  - sortie longue trop grande par rapport à l'historique ;
  - antécédents de blessures ignorés ;
  - aplomb excessif sur des sujets médicaux ;
  - incohérence d'un appel à l'autre (non-déterminisme).

### Repères de charge
- **Règle des 10 %** (volume hebdomadaire) : heuristique courante mais peu étayée.
- **Frandsen et al., BJSM 2025** : cohorte de 5 205 coureurs et 588 071 séances Garmin. Le risque de blessure de surmenage augmente quand **une séance dépasse de plus de 10 % la plus longue sortie des 30 derniers jours**. Par rapport à une hausse de 10 % ou moins :
  - +64 % de risque pour un dépassement de 10 à 30 % ;
  - +52 % pour 30 à 100 % ;
  - +128 % au-delà de 100 % (HR 2,28).

  La blessure se joue donc **à la séance**, plus qu'en accumulation hebdomadaire. Limites : étude observationnelle, blessures auto-déclarées.
- **ACWR** (ratio charge aiguë sur charge chronique, zone « idéale » souvent citée de 0,8 à 1,3) : très répandu dans les montres, mais **fortement critiqué** (Impellizzeri et al., Lolli et al.). On lui reproche un couplage mathématique (la charge aiguë fait partie de la charge chronique), l'absence de rationnel causal et un manque de preuves pour en faire un outil de prévention. Il reste utilisable comme **signal d'alerte**, pas comme règle.
- intervals.icu calcule déjà CTL (fitness), ATL (fatigue), forme (TSB) et le ramp rate. Ce sont des signaux de charge disponibles sans recalcul.

### Garde-fous recommandés pour l'app (déterministes, hors LLM)
1. **Le LLM propose, du code valide** : sortie JSON avec schéma strict, puis un validateur de règles avant toute publication sur intervals.icu.
2. **Plafond sur la sortie longue** : au plus 110 % de la plus longue sortie des 30 derniers jours (Frandsen 2025). Toute exception demande une confirmation explicite.
3. **Plafond de progression hebdomadaire** du volume (environ 10 %, paramétrable), avec une **semaine de décharge** toutes les 3 à 4 semaines.
4. **Limite d'intensité** : une part maximale de temps en zone haute (par exemple environ 20 %), au plus 2 à 3 séances qualitatives par semaine, jamais deux séances dures d'affilée.
5. **Ancrage des allures** : les allures viennent uniquement des réglages intervals.icu (threshold pace, zones) ou d'un test ou d'une course récente, et sont **calculées par le code**. Le LLM n'en invente aucune. Vérification de plausibilité (fourchettes, cohérence entre zones).
6. **Signaux d'alerte** (ACWR au-dessus de ~1,3–1,5, TSB très négatif, HRV ou FC repos dégradées, douleur signalée) : ils déclenchent **une proposition** d'allègement, jamais une modification silencieuse. C'est cohérent avec le principe « modification seulement sur perturbation majeure, avec confirmation ».
7. **Séparer données et opinion** dans le retour après séance : bloc « Mesuré » (chiffres bruts de l'API) d'un côté, bloc « Interprétation IA » de l'autre, avec niveau de confiance et citation des métriques utilisées.
8. **Garde-fou médical** : douleur, blessure ou symptôme inquiétant donnent lieu à un message type « consulter un professionnel » et à une suspension des séances intenses. L'app ne pose pas de diagnostic.
9. **Questions de clarification obligatoires** à la création du plan (objectif, historique, blessures, disponibilités), pour corriger le défaut relevé par la JSSM.
10. **Journal des décisions** : pour chaque changement de plan, conserver la raison, les données utilisées et la confirmation de l'utilisateur.

Sources :
- https://www.jssm.org/jssm-23-56.xml-Fulltext
- https://pmc.ncbi.nlm.nih.gov/articles/PMC10915606
- https://monkemanx.github.io/paper-summary/chatgpt_generated_training_plans_for_runners/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC11908068
- https://arxiv.org/html/2509.26593v1
- https://marathonhandbook.com/expert-coaches-examine-chatgpts-ability-to-create-running-training-plans/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC12421110
- https://blogs.bmj.com/bjsm/?p=11751
- https://ph.au.dk/display/artikel/everything-we-thought-about-running-injury-development-was-wrong-study-shows
- https://pmc.ncbi.nlm.nih.gov/articles/PMC7534938
- https://bjsm.bmj.com/content/53/15/921.abstract
- https://pubmed.ncbi.nlm.nih.gov/34052983/
- https://www.outsideonline.com/health/training-performance/injury-prevention-acute-chronic-workload-ratio-research/

---

## 5. Inconnues à lever (spikes recommandés)
1. Rendu sur l'Apple Watch, via HealthFit, des cibles d'**allure** (absolue ou en %) et de **FC**, des répétitions et des distances en `mtr`.
2. Horizon et fréquence de synchronisation HealthFit (combien de jours d'avance, déclenchement en arrière-plan).
3. Vérifier qu'un workout créé par l'API (`description`) est bien structuré (`workout_doc` rempli) et repris par HealthFit, comme un workout créé à la main.
4. Disponibilité des données wellness Apple Watch dans intervals.icu (HRV et FC repos nécessitent le suivi du sommeil).
5. Opportunité d'utiliser le **MCP Connector intervals.icu** (Supporter 4 $/mois) plutôt qu'un client API maison.
