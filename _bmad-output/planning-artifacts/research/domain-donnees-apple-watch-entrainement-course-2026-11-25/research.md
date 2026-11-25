---
title: 'Recherche domaine : données Apple Watch et science de l''entraînement en course à pied'
type: 'domain'
topic: 'Données Apple Watch/HealthKit et science de l''entraînement course à pied pour un coach IA perso'
decision: 'Quelles données Apple Watch utiliser et comment, quelles règles de charge/récupération/blessure et quelle méthode appliquer pour une analyse juste et des plans cohérents'
source: 'native-run'
status: complete
claims_verified: 13
claims_unverified: 11
claims_overturned: 0
preset: 'deep'
validation: 'normal'
created: '2026-11-25'
updated: '2026-11-25'
---

# Recherche domaine : données Apple Watch et science de l'entraînement en course à pied

**Décision servie :** quelles données Apple Watch utiliser et comment les interpréter, quelles règles de charge, de récupération et de prévention des blessures appliquer, et quelle méthode d'entraînement choisir, pour que l'application analyse juste et génère des plans cohérents.

## Synthèse exécutive

**Ce que disent les preuves :** construis l'app sur **peu de signaux simples et bien mesurés**. Code en dur **une règle de sécurité** solide, ancre tout sur **un seuil tenu à jour automatiquement**, et traite le choix de méthode comme secondaire.

**Les trois constats qui décident :**

1. **Les données Apple les plus riches sont les moins fiables.**
   - **Fiables** : l'allure, la FC en effort stable, la FC de repos (−0,08 bpm face à une ceinture [72]) et la durée de sommeil [6][21].
   - **En tendance seulement** : la VO2max (−6,3 ml/kg/min et 13 % d'erreur sur Series 10 [10]) et la VFC (variabilité de la FC), stockée en SDNN et non en RMSSD (deux façons de la calculer, §2.2), sous-estimée de 8 ms au repos sur Series 9 / Ultra 2 [72], et pour laquelle je n'ai trouvé aucune validation nocturne [73].
   - **Sans validation publiée** : la puissance et la dynamique de foulée [14]. **Validés mais imparfaits, donc indicatifs** : les stades de sommeil (kappa 0,68) [6][21].
   - Les **mesures subjectives de bien-être** suivent la charge mieux que les capteurs [76]. Ta question de ressenti à 3 niveaux en est une version simplifiée, non validée en tant que telle. → **R6**
2. **La règle de sécurité la mieux prouvée porte sur chaque sortie, pas sur la semaine.** Une sortie plus de 10 % plus longue que la plus longue des 30 derniers jours augmente le risque de blessure de 1,5 à 2,3 fois, alors que la variation d'une semaine à l'autre n'y est pas associée [65]. → **R1, R2** La « règle des 10 % par semaine » a échoué dans un essai randomisé [84]. Une progression trop rapide figure aussi **parmi les erreurs relevées dans des plans écrits par ChatGPT** (génération GPT-3.5, 2023) [58].
3. **Le choix de méthode compte peu.** Polarisé et pyramidal donnent des résultats équivalents [46][47], et aucun algorithme publié ne choisit une méthode d'après un profil. En revanche, **selon des utilisateurs du forum (septembre 2025), intervals.icu ne met pas à jour automatiquement l'allure seuil en course** [108] ; c'est à vérifier dans l'app. Sans un maintien automatique de la vitesse critique (CS, l'allure limite tenable sans épuisement rapide, §2.1), calculable depuis tes meilleurs efforts d'entraînement [105], les allures, la charge et la conformité prévu / réalisé dérivent ensemble. → **R4, R5**

**La principale réserve :** les seuils d'alerte (surmenage, FC de repos, VFC, zones de Form, ACWR — le ratio charge aiguë / chronique) **n'ont pas de base numérique validée** [64][42][70]. La logique est défendable : écart à sa propre ligne de base, et alerte seulement quand plusieurs signaux concordent. Mais les chiffres sont des **paramètres à calibrer sur tes données**, pas des faits à afficher. Deux vérifications empiriques sont à faire **avant de coder** : le contenu réel du FIT exporté par HealthFit, et la valeur de VFC qu'il envoie (§9).

**Statuts :** les 10 affirmations sur lesquelles reposent les recommandations ont été vérifiées auprès de la source primaire ou d'une seconde source, et sont confirmées. **Pour trois d'entre elles, cette confirmation n'est pas vraiment indépendante** : la cohorte RUNSAFE (une seule cohorte, contrôlée sur son propre article), le guide maladie BJSM (mêmes auteurs) et le comportement d'intervals.icu (forum seulement). Le registre compte 13 affirmations vérifiées (les 10 contrôlées, plus 3 étayées par plusieurs sources dans les synthèses de sources) et 11 non vérifiées (source unique). Aucune n'a été infirmée. Un audit des citations a corrigé une référence et atténué 19 formulations trop fortes.

**Les actions** sont dans le §8 (R1 à R10).

### Avant de coder : vérifications à faire (une soirée)

1. **Contenu du FIT HealthFit** : exporter 2 ou 3 courses et lister les champs (fréquence de la FC, puissance, dynamique de foulée, tours) (§1.2, §9).
2. **VFC envoyée par HealthFit** : comparer Apple Santé et le champ wellness d'intervals.icu sur 2 semaines (moyenne de la journée ou mesure nocturne ?) (§1.2, §9).
3. **Séance planifiée de test** : vérifier que les cibles d'allure par bloc arrivent sur la montre (§9).
4. **intervals.icu, dans l'app** : la formule de charge allure [39], le calcul de Form le jour même [41], les zones de Form [64], les bandes de conformité [97] et l'absence de mise à jour automatique de l'allure seuil [108] (§12).
5. **Formule de l'indice de polarisation** dans Treff 2019 avant de la coder (§2.2).


## 1. Les données Apple Watch / HealthKit : ce qu'elles valent pour la course

**Question :** quelles données la montre produit, comment elles sont calculées, à quel point on peut s'y fier, et lesquelles arrivent réellement dans intervals.icu via HealthFit.

### 1.1 Inventaire et fiabilité, métrique par métrique

| Métrique | Comment Apple la produit | Fiabilité établie | Usage recommandé dans l'app |
|---|---|---|---|
| **Fréquence cardiaque (séance)** | Capteur optique (PPG), mesurée « en continu » pendant la séance et 3 min après [12][13] | Bonne en allure stable (Apple Watch 4 sous le seuil ANSI/CTA de 10 % d'erreur au tapis) ; la validité baisse quand l'intensité monte ; retard face à une ceinture pendant les fractionnés, documenté seulement par des tests presse [112], aucune étude publiée et évaluée par des pairs. Apple prévient elle-même que des lectures « anormalement hautes ou basses » arrivent [13]. | Fiable pour footings, sortie longue, tempo. Sur les fractionnés courts (< 2–3 min), juger l'exécution **sur l'allure**, pas sur la FC. |
| **Allure / distance (GPS)** | GPS + accéléromètre | Aucune étude indépendante trouvée sur le GPS bi-fréquence de la Series 10 (lacune). | Référence d'exécution pour les blocs, de préférence corrigée de la pente (GAP, voir §6). |
| **Puissance de course** | Modèle Apple (watts), course uniquement, watchOS 9+ [1][2] | **Aucune validation publiée et évaluée par des pairs** face à Stryd ou une plateforme de force. Seuls des tests mono-athlète de blogs (2022) existent : valeurs propres à chaque marque, montée lente sur les efforts courts [14]. | Ne pas l'utiliser comme cible ni comme source de charge. Au mieux, un indicateur de tendance. |
| **Temps de contact au sol, oscillation verticale, longueur de foulée** | Types HealthKit dédiés depuis iOS 16 [1][2] | Pas de validation publiée et évaluée par des pairs trouvée. | Affichage descriptif uniquement, jamais de verdict. |
| **Cadence** | **Pas de type HealthKit dédié** : elle doit être dérivée (pas/temps) ou lue dans le FIT exporté [1] | Non évaluée. | Descriptif. |
| **VO2max (« Forme cardio »)** | Estimation **sous-maximale** à partir de la réponse FC/vitesse. Uniquement après une marche, course ou rando en extérieur, sur terrain plat (pente < 5 %), avec un bon GPS et un effort d'environ +30 % de la réserve cardiaque [8]. | Validation Apple : +1,4 ± 4,7 ml/kg/min, ICC 0,86, mais sur une population âgée (55 ans en moyenne), pas des coureurs, et contre un test lui-même sous-maximal [8]. Études indépendantes : sous-estimation d'environ 6,3 ml/kg/min et erreur moyenne de 13,2 % **sur Series 10** (n=40) [10] ; fiabilité faible sur Series 7, avec une surestimation chez les moins entraînés et une sous-estimation chez les plus entraînés (n=19) [9]. | **Tendance uniquement**, jamais une valeur absolue pour fixer des allures. |
| **VFC (HRV)** | Stockée en **SDNN** (pas RMSSD), à des moments irréguliers et opaques [15][22] | Bon accord avec une ceinture Polar H7 au repos sur des enregistrements courts (2018, n=20), avec des battements manqués [17] ; accord N-N seulement modéré sur Series 6 (2025, n=78) [16]. **Aucune validation Apple du SDNN nocturne.** | Voir §4 : utilisable seulement en tendance individuelle, avec prudence. |
| **FC de repos** | Calculée par Apple, journalière | Pas de validation spécifique trouvée. | Tendance sur plusieurs jours, contre la ligne de base personnelle. |
| **Sommeil (durée, stades)** | **Accéléromètre seul**, époques de 30 s, 4 états : Éveil / Paradoxal / Core (N1+N2) / Profond (N3) [6] | Selon le white paper Apple (watchOS 26) : sensibilité sommeil 96,8 %, spécificité 78,9 %, kappa à 4 stades de 0,68 (contre 0,63 avant). L'erreur la plus fréquente est de classer le sommeil profond en Core [6]. Étude indépendante (Series 8, n=35) : durée proche de la polysomnographie, sensibilité par stade de 50,5 à 86,1 % [21]. Les nuits de 1 à 3 h sont enregistrées en endormi/éveillé seulement [6]. | **Durée fiable ; stades indicatifs.** Ne jamais baser un verdict sur le % de sommeil profond. |
| **Score de sommeil** (watchOS 26) | Durée + régularité de l'heure du coucher + interruptions | Les seuils ont été recalibrés en watchOS 26.2 (fin 2025), si bien que les scores d'avant et d'après ne sont **pas comparables** [7]. | À éviter comme entrée, ou à normaliser par version d'OS. Préférer la durée brute. |
| **Effort (1–10) / Training Load** | Le score d'effort estimé est stocké dans HealthKit (`estimatedWorkoutEffortScore`, iOS 18+) [4]. Le Training Load compare les 7 derniers jours aux 28 précédents [3]. | Formule non publiée ; les entrées (durée, FC, âge) ne sont connues que par la presse [5]. | Ne pas en faire un pilier : l'app a sa propre charge (intervals.icu) et son propre ressenti. |
| **Vitals (nuit)** | FC, fréquence respiratoire, SpO2, température du poignet, durée de sommeil. La plage habituelle est apprise en 7 nuits ; une alerte se déclenche quand **plusieurs** métriques sortent de la plage [11]. | Pas de seuil numérique publié. **La VFC n'est dans Vitals que sur Series 12 / Ultra 4**, donc pas sur ta Series 10 [11]. | Un signal Vitals peut servir de **drapeau de contexte** (maladie possible), pas de mesure. |

### 1.2 Ce qui passe réellement dans intervals.icu (HealthFit)

- **Les séances** : HealthFit « reflète Apple Santé », qui est sa seule source de vérité, et exporte un fichier FIT. Je n'ai trouvé **aucune documentation primaire** de son contenu (fréquence d'échantillonnage de la FC, puissance, dynamique de foulée, tours) [15][19]. C'est le **trou principal** de cette partie ; il se comble par un test empirique, pas par de la recherche (§9).
- **Le bien-être** : HealthFit synchronise la VFC (SDNN, dans un champ intervals.icu dédié), le sommeil, le poids et la SpO2 avec un réglage « Synchronisation automatique ». En 2021, des utilisateurs signalaient que la VFC envoyée était une **moyenne de la journée** et non la valeur nocturne ; je n'ai pas trouvé de réponse du développeur [15]. Alternative : IntervalsWellnessSync (2026) ne prend la VFC et la FC de repos **que pendant le sommeil** [20] (information éditeur, confiance faible à moyenne).
- **Les séances planifiées** : HealthFit importe les séances planifiées d'intervals.icu (disponible depuis octobre 2025), et depuis mars 2026 il faut l'abonnement **« Fanatical Supporter »** (9,99 $ sur l'App Store) [18][19]. Un utilisateur a signalé des séances absentes de la montre. Ni le mécanisme (WorkoutKit) ni les cibles prises en charge (allure, FC, puissance) ne sont documentés [18].
- **La « VFC de récupération »** de HealthFit (12.16, octobre 2026) exige une Series 12 ou une Ultra 4 [19] : **elle n'est pas disponible sur une Series 10**.

### 1.3 À retenir pour la décision

L'allure, la FC en effort stable, la durée de sommeil et la FC de repos sont les données sur lesquelles on peut construire. La VO2max et la VFC se lisent en **tendance individuelle**. La puissance, la dynamique de foulée, les stades de sommeil et les scores Apple (sommeil, effort) restent **descriptifs**. Les règles de VFC publiées (calibrées sur RMSSD) ne se transposent pas directement au SDNN d'Apple.

## 2. Glossaire et modèles mentaux

**Question :** quel vocabulaire faut-il maîtriser pour lire les données et écrire les règles de l'app ? Les entrées marquées *(non vérifié)* reposent sur des connaissances générales que je n'ai pas pu sourcer pendant cette recherche : elles sont à confirmer avant d'être codées.

### 2.1 Le modèle central : les trois domaines d'intensité

La physiologie moderne découpe l'effort en **trois domaines** séparés par deux seuils [25] :

- **Modéré**, sous le 1er seuil (LT1 / VT1, « seuil aérobie ») : footing, sortie longue. Lactate stable et bas.
- **Lourd**, entre les deux seuils : tempo, allure semi-marathon / marathon. Un état stable reste possible.
- **Sévère**, au-dessus du 2e seuil, la **vitesse critique (CS)** : fractionné VO2max. Plus d'état stable, l'épuisement arrive en quelques minutes.

Deux conséquences directes pour l'app. **(1)** Les **pourcentages fixes de FC max ou de VO2max placent mal les gens dans ces domaines** ; seule la CS (ou la puissance critique) délimite de façon fiable la frontière lourd/sévère [25]. Il faut donc ancrer les zones sur un **seuil mesuré** (CS, allure seuil, FC au seuil), pas sur « 220 − âge ». **(2)** Les différentes définitions du « seuil » (MLSS, 4 mmol, LT2, CS) **ne sont pas interchangeables** : la MLSS est inférieure à la CS [24][25]. L'app doit choisir **un** ancrage et le nommer.

### 2.2 Glossaire

| Terme | Définition opérationnelle | Mesure / formule | Source |
|---|---|---|---|
| **VO2max** | Débit maximal d'oxygène consommé (ml/kg/min). | Labo : test progressif avec analyse des gaz. Montre : estimation sous-maximale (voir §1). La CS se situe vers 70–90 % de la VO2max. | [23] ; valeurs typiques *(non vérifié)* |
| **vVO2max / VMA** | Vitesse minimale qui sollicite la VO2max ; elle combine VO2max et économie de course. Tenable environ 4–8 min (Tlim, très variable d'un individu à l'autre). | Terrain : test VAMEVAL / Léger-Boucher, 5–6 min à fond *(non vérifié)*. La VMA de terrain n'est pas strictement égale à la vVO2max de labo. | [32] |
| **Économie de course** | Coût en O2 à une vitesse sous-maximale donnée ; fort prédicteur de performance, entraînable. | Labo uniquement. | [33] |
| **LT1 / VT1** | 1re hausse durable du lactate, 1re cassure ventilatoire ; frontière modéré/lourd. | Labo ; la montre ne peut que l'estimer. | [25] |
| **LT2 / VT2 / « seuil anaérobie »** | 2e seuil ; ses définitions varient (4 mmol, Dmax, MLSS) et ne sont pas interchangeables. | Labo. | [25] |
| **MLSS** | Plus haute intensité où le lactate reste stable sur 30 min. | 4–5 tests de 30 min, impraticable pour l'app. | [24] |
| **Vitesse critique (CS) et D′** | CS : vitesse la plus haute tenable sans puiser dans une réserve finie D′ (en mètres) ; frontière lourd/sévère. Meilleur indice de l'état stable métabolique maximal. | **d = CS·t + D′** (régression distance-temps sur au moins 3 efforts maximaux de 2 à 15 min). Calculable à partir des données d'entraînement (§6). | [23][24] |
| **FC max** | FC maximale atteinte. | **Tanaka : 208 − 0,7 × âge** (erreur type d'environ 10 bpm) ; « 220 − âge » est moins bon. **Une FC max observée doit primer sur toute formule.** | [26] |
| **Réserve cardiaque (Karvonen)** | Fraction = (FC − FC repos) / (FC max − FC repos). FC cible = FC repos + % × (FC max − FC repos). | Formule HRR sourcée ; l'attribution à Karvonen n'est pas vérifiée. | [27] |
| **FC au seuil (LTHR)** | FC tenable environ 1 h. | Protocole actuel de Friel : 20 min seul à fond, moyenne − 5 %. Ses anciens ouvrages décrivent un test de 30 min (moyenne des 20 dernières) : **les deux protocoles diffèrent, ne pas les mélanger.** L'erreur du test de 30 min est d'environ 8 bpm (§6). | [35] |
| **Zones (modèle à 3 zones de Seiler)** | Z1 ≤ LT1 (≈ 2 mmol), Z2 entre LT1 et LT2, Z3 > LT2 (≈ 4 mmol). Les élites font environ 80 % de leur volume en Z1. | Les modèles à 5 zones (Friel) ou 7 zones (Coggan) sont des subdivisions. | [37] |
| **VDOT (Daniels)** | « Pseudo-VO2max » dérivée d'un résultat de course, qui donne les allures E / M / T / I / R. | Équations Daniels-Gilbert (1979), vitesse en m/min. **Coefficients relevés uniquement sur un calculateur secondaire.** Les % de chaque allure ne sont pas vérifiés. | [38] (faible) |
| **RPE (Borg CR10)** | Effort perçu sur une échelle de 0 à 10 ; la version CR-10 modifiée sert au sRPE. | Recueilli 15 à 30 min après la séance. | [27] |
| **sRPE (Foster)** | Charge = RPE × durée (min). | Corrélation avec les TRIMP de r ≈ 0,46 à 0,96 au sein d'un même individu. | [27] |
| **TRIMP de Banister** | Charge FC exponentielle. | **TRIMP = D × HRR × A × e^(B×HRR)** ; hommes A = 0,64, B = 1,92 ; femmes A = 0,86, B = 1,67. | [27] |
| **TRIMP d'Edwards** | Σ minutes par zone × poids (1 à 5, par tranches de 10 % de la FC max à partir de 50 %). | | [27] |
| **TSS / rTSS / hrTSS** | Charge TrainingPeaks : 1 h au seuil = 100 ; variantes puissance, allure (rTSS, sur NGP) et FC (hrTSS, la moins précise selon l'éditeur). | | [29] |
| **Charge intervals.icu (allure)** | **Charge = heures en mouvement × IF² × 100**, avec IF = vitesse GAP moyenne / vitesse seuil. Variante FC (HRSS) : un TRIMP normalisé, 100 = 1 h au seuil, qui exige FC repos et LTHR. La priorité puissance / allure / FC se règle par sport. | Code publié par le développeur en 2022, à revérifier ; l'équation exacte du HRSS n'est pas publiée. | [39][40] |
| **CTL / ATL / TSB = Fitness / Fatigue / Form** | Moyennes exponentielles de la charge sur 42 j (fitness) et 7 j (fatigue) ; Form = Fitness − Fatigue. **Dans intervals.icu, Form se calcule le jour même** (TrainingPeaks utilise la veille), et les constantes sont modifiables. | Form en % = (Fitness − Fatigue) × 100 / Fitness. | [29][41] |
| **Ramp rate** | Variation de la CTL sur environ 7 j. Pas de seuil numérique publié par l'éditeur. | | [29] |
| **ACWR** | Charge aiguë (7 j) / chronique (28 j). **Très contesté** (voir §4). | | [42] |
| **Monotonie / contrainte (Foster)** | Monotonie = moyenne / écart-type de la charge quotidienne sur une semaine ; contrainte (strain) = charge hebdo × monotonie. Les seuils de maladie étaient **individuels** ; le seuil courant « monotonie > 2 » n'est pas retrouvé dans le résumé de l'article. | | [30] |
| **Dérive cardiaque** | À effort constant, la FC monte après 10–20 min (débit systolique en baisse), d'autant plus avec la déshydratation. **Les zones FC surestiment donc l'intensité en fin de sortie longue.** | | [34] |
| **Découplage (Pa:HR)** | Écart du ratio allure/FC entre la 1re et la 2e moitié d'une séance stable. | (ratio 1re moitié − ratio 2e moitié) / ratio 1re moitié. | [29][41] |
| **Facteur d'efficience (EF)** | Allure normalisée (NGP) ÷ FC moyenne ; **une hausse** à séances comparables signifie un progrès. | | [29] |
| **Indice de polarisation** | PI = log10((Z1/Z2) × Z3 × 100), où Zi est la fraction du volume dans la zone i ; PI > 2 = polarisé. intervals.icu regroupe S1 = Z1+Z2, S2 = Z3+Z4, S3 = Z5+. *Formule à vérifier dans Treff 2019 avant de coder : les deux synthèses divergent.* | | [36][43] |
| **VFC : SDNN vs RMSSD** | **RMSSD** : indice principal du tonus vagal (parasympathique), stable sur 1 à 5 min ; c'est l'indice des protocoles de « readiness ». **SDNN** : variabilité totale, sympathique et parasympathique ; la norme clinique se mesure sur 24 h. Sur une fenêtre ultra-courte au repos (le cas Apple), il reflète surtout l'arythmie respiratoire vagale, et **ses normes ne sont pas comparables** aux normes 24 h. | RMSSD = √(moyenne((NNᵢ₊₁ − NNᵢ)²)). | [28] |
| **Surmenage : FOR / NFOR / OTS** | Un continuum. Le surmenage fonctionnel (FOR) est une baisse de quelques jours suivie d'une surcompensation. Le surmenage non fonctionnel (NFOR) est une stagnation de plusieurs semaines à plusieurs mois. Le syndrome de surentraînement (OTS) dure plus de 2 mois et se diagnostique par exclusion. **Aucun marqueur isolé ne suffit.** | | [31] |
| **Affûtage (taper)** | 2 semaines (jusqu'à 3), volume réduit de 41 à 60 %, **intensité et fréquence maintenues**. | | [44] |
| **Périodisation** | Traditionnelle : plusieurs qualités travaillées en parallèle. Par blocs (Issurin) : qualités travaillées successivement, avec une base de preuves jugée faible par Kiely. La distribution d'intensité (polarisée, pyramidale) n'est **pas** un modèle de périodisation. | | [45] |
| **Types de séances** | Footing (Z1, conversationnel), sortie longue, tempo / seuil, fractionné VO2max (3–5 min à vVO2max), répétitions (courtes et rapides), lignes droites, fartlek, progressive. | *(non vérifié)*, à reprendre dans *Daniels' Running Formula*. | — |

## 3. Méthodes d'entraînement : ce que disent les preuves et comment choisir

**Question :** quelles méthodes reposent sur des preuves, pour qui, et quelle règle de sélection automatique est défendable ?

### 3.1 Distribution de l'intensité : polarisé, pyramidal ou seuil

- **Polarisé et pyramidal donnent des résultats équivalents.** La méta-analyse sur données individuelles de Rosenblat et al. (2025 ; 13 études, 348 athlètes) ne trouve aucune différence sur la VO2max (SMD 0,06, p = 0,68) ni sur les contre-la-montre (SMD 0,05, p = 0,34) [46]. Oliveira et al. (2024 ; 17 études) trouvent un léger avantage du polarisé sur la VO2peak, limité aux athlètes très entraînés et aux protocoles de moins de 12 semaines, et aucun avantage sur la performance [47]. Rosenblat critique la méthode d'Oliveira (participants comptés deux fois, regroupement d'interventions hétérogènes) [46]. *Confiance : élevée sur « peu de différence ».*
- **La nuance « pyramidal pour les amateurs » est fragile.** Le sous-groupe récréatif penche vers le pyramidal (SMD 0,32, IC −0,14 à 0,79, **p = 0,18, non significatif**). Il repose sur environ 5 petites études, dont 2 non randomisées, et l'effet du niveau ne se distingue pas de celui de la VO2max de départ [46]. *À présenter comme une tendance, pas comme un fait.*
- **Trop de seuil fait moins bien.** Une distribution dominée par la zone 2 du modèle à 3 zones (entre les deux seuils) fait moins bien que le polarisé ou le pyramidal [48]. Un essai randomisé chez des amateurs (10 km) a donné l'avantage au polarisé sur l'entraînement « entre les seuils » ; c'est un seul laboratoire [49].
- **Repère de volume.** Au moins environ 70 % du volume à basse intensité et au plus 30 % au seuil ou en fractionné pour les coureurs de demi-fond et de fond [50] *(confiance moyenne, résumé seul)*.
- **Ce qui est prescrit n'est pas ce qui est fait.** Mesurée à la FC, la distribution réellement suivie par les participants dérivait vers le pyramidal, quelle que soit celle qui leur avait été prescrite. Il faut donc **suivre la charge réalisée**, et l'allure quand elle est disponible [46].

### 3.2 Les méthodes « de marque »

| Méthode | Principe | Niveau de preuve |
|---|---|---|
| **Daniels (VDOT)** | Un résultat de course donne le VDOT, qui donne 5 allures (E/M/T/I/R). En général 2 séances de qualité + une sortie longue par semaine, avec des plafonds par séance (T ≤ 10 % du volume hebdo, I ≤ 8 %, R ≤ 5 %). | Système de coaching ; aucun essai sur le « package » [51] (source secondaire). |
| **80/20 (Fitzgerald)** | Environ 80 % bas / 20 % modéré à élevé, avec des zones ancrées sur le seuil. Plans par niveau 0 à 3 selon l'expérience. | Inspirée des données d'élites (Seiler) ; pas d'essai direct [52]. |
| **Norvégienne (double seuil)** | Gros volume facile + 3–4 séances au seuil guidées par la lactatémie (≈ 2–4,5 mmol). | Description élite ; ses auteurs disent eux-mêmes que des essais contrôlés manquent. La version « singles » pour amateurs n'existe que sur des blogs [53]. |
| **MAF (180 − âge)** | Plafond de FC pour tout le travail aérobie. | Formule **non validée** ; le seul article publié est une « hypothèse » [54]. |
| **Course-marche (Galloway)** | Pauses de marche régulières. | Un essai randomisé (marathon, n = 42) : temps similaires, moins de douleurs musculaires après la course [55]. |
| **Pfitzinger, Hansons, Lydiard** | Systèmes de coaching marathon. | Sources primaires non consultées pendant cette recherche. |

**Aucun algorithme publié ne choisit une méthode à partir du profil du coureur.** Le plus proche, ce sont les niveaux d'entrée du 80/20 et le sous-groupe de Rosenblat [46][52].

### 3.3 Affûtage et structure de plan

- **Affûtage :** 2 semaines (jusqu'à 3 pour le marathon), volume réduit de 41 à 60 %, intensité et fréquence maintenues. Le résultat est confirmé par une méta-analyse de 2023 limitée aux sports d'endurance [44][56]. *Confiance élevée.*
- **Structure :** trois repères courants relèvent de la **convention de coaching**, pas de la preuve [57] : une sortie longue d'environ 25–30 % du volume hebdomadaire et d'au plus 2 h 30 à 3 h ; des plans de 12 à 18 semaines pour le marathon ; une semaine allégée toutes les 3 à 4 semaines. Je n'ai pas trouvé d'étude sur la durée optimale d'un plan selon la distance ni sur le nombre de séances de qualité.

### 3.4 Les plans générés par IA : erreurs connues

- **Quand ChatGPT (génération GPT-3.5, 2023) écrit des plans**, la note médiane des experts passe de 2 à 3 puis 4 sur 5 à mesure que les informations fournies s'enrichissent, et aucun plan n'est jugé optimal. **Les experts relèvent notamment une progression trop rapide** (risque de blessure). S'y ajoutent un dépistage santé et un suivi jugés seulement neutres, l'absence de questions de clarification et des conseils populaires mais non étayés [58]. *Le résultat porte sur un modèle ancien ; sa transposition aux modèles de 2026 est incertaine.*
- **Une étude de cas sur un semi-marathon (2025, n = 1, prépublication)** relève un taux d'adhérence de 70 à 80 %. Elle pointe trois failles : des garde-fous appliqués **seulement dans la conversation, jamais imposés** ; **aucune mémoire persistante de l'athlète** ; une motivation laissée à l'initiative de l'utilisateur. Elle recommande d'associer à chaque consigne **une raison courte et les sensations attendues** [59].
- **Sur le format du feedback**, les données sont minces. Ajouter du feedback à l'auto-suivi augmente l'activité physique (d = 0,29), mais aucun format ne ressort comme meilleur [60]. Dans un essai de 4 semaines (N = 54, prépublication CHI 2026), un coach GPT-4o n'augmente pas plus l'activité qu'une app sans LLM, mais améliore l'état d'esprit et l'engagement ; ses auteurs proposent une taxonomie des risques (surmenage, image du corps, conseils hors de portée) [61].

## 4. Charge, fatigue et récupération : quels signaux, quels seuils

**Question :** quels modèles et quels seuils sont assez solides pour déclencher une alerte de surmenage et proposer un ajustement du plan ?

### 4.1 Le modèle Fitness / Fatigue / Form (Banister, CTL/ATL/TSB)

- **Ce qu'il est.** Fitness et Fatigue sont deux moyennes exponentielles de la charge, sur 42 et 7 jours ; Form en est la différence. intervals.icu la calcule **le jour même**, et les deux constantes sont modifiables [41].
- **Ce qu'il vaut.** Malgré plus de 45 ans d'usage, sa validité est mal étudiée. Ses paramètres sont instables et mal conditionnés [62], et en simulation, un bruit de mesure de plus de 4 % rend ses prédictions insatisfaisantes [63]. *Confiance moyenne.*
- **Les zones de Form ne reposent sur aucune donnée.** Le développeur d'intervals.icu explique que les zones viennent de Joe Friel, qui les a définies pour un TSB absolu. Le développeur les a lui-même appliquées à la Form exprimée en %, et les décrit comme **« une règle empirique plutôt que de la science »** [64]. Par défaut, la frontière entre la zone verte et la zone rouge est à −30 [41].
- **Conclusion pour l'app :** utiliser Fitness / Fatigue / Form **pour décrire** la tendance de charge et pour planifier la progression, **pas comme détecteur de surmenage** à lui seul.

### 4.2 L'ACWR : à ne pas utiliser comme prédicteur de blessure

- **Les critiques.** Le ratio crée un couplage mathématique entre ses deux termes, puisque la charge aiguë fait partie de la charge chronique [42]. La courbe en U de la « zone idéale 0,8–1,3 » n'est apparue qu'après l'ajout de données AFL non publiées [42]. En simulation, l'ACWR reste « associé » aux blessures même quand celles-ci en sont indépendantes par construction [42]. L'Australian Institute of Sport recommande de ne pas l'utiliser comme indicateur de risque [42].
- **Les données de course le confirment.** Dans la cohorte Garmin-RUNSAFE, l'ACWR n'était pas associé aux blessures dans le sens attendu [65].
- **Il existe un camp opposé**, minoritaire : un éditorial de 2021 rappelle que la littérature reste disputée [66]. *Le consensus actuel est défavorable.*

### 4.3 VFC : ce que valent les protocoles, et pourquoi ils se transposent mal à Apple

- **L'entraînement guidé par la VFC apporte au mieux un petit avantage.** Trois méta-analyses vont dans ce sens. Granero-Gallegos (2020) trouve un petit effet positif sur la VO2max (ES 0,40), plus marqué chez les amateurs et les femmes [67]. Düking (2021) trouve des effets petits et non significatifs sur la performance, mais **moins de non-répondeurs** [68]. Manresa-Rocamora (2021) trouve des effets petits et non significatifs : « si c'est supérieur, c'est de peu » [69]. *Les résultats sont discordants sur la significativité et convergent sur la petitesse de l'effet.* Je n'ai trouvé aucune méta-analyse publiée de 2024 à 2026.
- **La méthode de référence** suit la moyenne glissante sur 7 jours du ln(RMSSD), **mesuré au réveil**, et la compare à la variation individuelle [71]. Le « plus petit changement important » vaut 0,5 × CV (coefficient de variation individuel) (Le Meur) ou 1 × CV (Plews). Buchheit précise toutefois : *« rien ne prouve actuellement qu'un changement supérieur à une quelconque fraction du CV soit réellement significatif en pratique »* [70]. D'un jour à l'autre, le ln(RMSSD) varie de 10 à 20 % et la FC de repos d'environ 10 %. Il faut donc moyenner **au moins 3 à 4 jours** [70].
- **Le problème Apple, en trois points :**
  1. **SDNN au lieu de RMSSD.** Sur des mesures courtes au repos, les deux donnent « globalement le même type de résultats » [22], mais les règles publiées sont calibrées sur le RMSSD [70].
  2. **La mesure est biaisée.** Les Series 9 et Ultra 2 **sous-estiment le SDNN de 8,3 ms** en moyenne face à une ceinture Polar H10, hors de la marge d'équivalence de ±10 ms. **La FC de repos, elle, concorde à −0,08 bpm** [72]. Je n'ai trouvé **aucune validation nocturne** du SDNN Apple [72][73].
  3. **L'échantillonnage est irrégulier et opaque** [22]. Une mesure environ toutes les 2 h pendant le sommeil, selon un blog d'éditeur non vérifié [113].
  
  Un praticien juge la mesure nocturne Apple peu adaptée au suivi de l'entraînement et recommande une mesure le matin [74] (source praticien, intérêt commercial).
- **Conclusion pour l'app.** La **FC de repos est mieux mesurée que la VFC** sur Apple Watch. La VFC ne doit compter que comme **un signal parmi d'autres**, et seulement en écart à sa propre ligne de base sur au moins 7 jours, jamais seule ni en valeur absolue.

### 4.4 FC de repos, sommeil, ressenti : les signaux complémentaires

- **FC de repos.** Je n'ai trouvé **aucun seuil en bpm qui soit sourcé** : le « +5 bpm » courant n'a pas de source primaire. Buchheit raisonne en CV individuel, avec un plus petit changement d'environ −2 % pour une adaptation positive, et avertit qu'*« une FC élevée ne doit pas être utilisée comme marqueur clair de fatigue »*. Une FC en hausse **accompagnée** d'une VFC en baisse suggère une fatigue accumulée [70].
- **Sommeil.** Chez des adolescents, dormir moins de 8 h multipliait les risques de blessure par 1,7 (IC 1,0–3,0 ; étude rétrospective et faible) [75]. Il n'existe pas de seuil validé chez l'adulte coureur.
- **Le ressenti bat les capteurs.** Une revue systématique de 56 études conclut que les mesures subjectives de bien-être suivent la charge aiguë et chronique **avec plus de sensibilité et de régularité** que les mesures objectives, et que les deux types de mesures concordent rarement [76]. *Confiance élevée.* Ta question de ressenti (😌 / 😐 / 🥵) s'appuie sur ce principe, mais c'est une version simplifiée à 3 niveaux, que la revue n'a pas testée.
- **Surmenage.** Selon le consensus ECSS/ACSM, **aucun marqueur ne remplit tous les critères**, et le diagnostic passe par l'exclusion d'autres causes (maladie, infection) [31]. Une app ne diagnostique pas : elle **signale** une tendance qui converge.
- **Monotonie / contrainte (Foster).** Les seuils associés à la maladie étaient **individuels** ; je n'ai pas retrouvé le « > 2 » souvent cité [30].

### 4.5 Une logique de concordance défendable

Rien dans la littérature ne fixe une règle numérique validée. La structure la plus défendable combine ce que les sources soutiennent, et c'est celle qu'Apple applique elle-même dans Vitals, qui alerte quand **plusieurs** métriques sortent de leur plage personnelle sur 7 nuits [11] :

1. **Une ligne de base personnelle** de 4 semaines ou plus pour la FC de repos, la VFC et la durée de sommeil [114], avec moyenne glissante sur 7 jours [71][11].
2. **Un signal** = une moyenne sur 7 jours sortie de la plage de variation individuelle (seuil à 0,5–1 × CV, à calibrer et non à imposer) [70].
3. **Une alerte** = au moins 2 signaux objectifs qui concordent, **plus** un ressenti dégradé sur plusieurs séances. Ce nombre est un **choix de conception** : les sources soutiennent la combinaison de plusieurs marqueurs [31][70][76], et Apple applique le même principe dans Vitals (« plusieurs métriques ») [11]. Un signal isolé est seulement affiché.
4. **Les seuils de cette règle sont des paramètres de conception, pas des faits** : ils sont à calibrer sur tes propres données.

## 5. Prévention des blessures et gestion des imprévus

**Question :** quelles règles le générateur de plan et le module « Empêchement » doivent-ils imposer ?

### 5.1 Le risque de base

- **Fréquence.** Un coureur débutant se blesse environ 17,8 fois par 1 000 h de course, un coureur loisir environ 7,7 fois [77]. Environ 40 % des coureurs se blessent. Les blessures les plus fréquentes sont la tendinopathie d'Achille, le syndrome tibial médial, le syndrome fémoro-patellaire et la fasciite plantaire [78].
- **Le facteur de risque le plus constant est une blessure antérieure** ; deux revues indépendantes concordent sur ce point [79][80]. Les facteurs biomécaniques ne sont pas confirmés chez les non-élites [81].
- **Le renforcement musculaire** réduit les blessures tous sports confondus : elles tombent à moins d'un tiers, et les blessures de surcharge baissent de près de moitié [82]. Les essais propres à la course sont **mitigés** : un programme hanches et tronc a protégé des débutants, un programme pied et cheville non [83].

### 5.2 La règle de progression : par sortie, pas par semaine

- **La meilleure donnée actuelle est la cohorte Garmin-RUNSAFE** (5 205 coureurs, 18 mois, 588 071 séances). Elle compare chaque sortie à **la plus longue des 30 jours précédents** et trouve un risque de blessure de surcharge accru dès +10 % : rapport de risque (*hazard rate ratio*) 1,64 pour +10 à 30 %, 1,52 pour +30 à 100 %, 2,28 au-delà de +100 % [65]. Même entre +1 et +10 %, la tendance est défavorable, mais non significative (1,19). Les auteurs conseillent donc de rester **sous 10 %**, et même « bien en dessous » [65].
- **L'évolution d'une semaine à l'autre n'était pas associée aux blessures** dans cette cohorte [65]. Cela rejoint l'essai GRONORUN : un programme construit sur la « règle des 10 % par semaine » n'a pas réduit les blessures chez des débutants (environ 20,8 % contre 20,3 %) [84]. Le signal « plus de 30 % par semaine » chez les débutants n'est pas significatif (p = 0,07) [85].
- **Limites à connaître.** C'est **une seule cohorte**, observationnelle, avec des blessures auto-déclarées et une charge mesurée par la distance seule. Elle ne stratifie ni par niveau ni par intensité, et les auteurs précisent que la règle par sortie **ne doit pas être utilisée seule**, car une progression étalée sur plusieurs séances peut aussi être excessive [65]. Aucune réplication n'a été trouvée.
- **Le cas de la reprise.** Si aucune sortie n'a eu lieu dans les 30 jours, le ratio **n'est pas calculable** (catégorie « NP » de l'étude) [65]. L'app a donc besoin d'une règle à part pour la reprise.

### 5.3 Maladie

- **Le principe.** Le guide BJSM [86] prévoit un retour progressif par paliers. Ce guide est **écrit pour le COVID** ; son extension aux autres infections est une transposition, pas une recommandation du guide :
  - **Fièvre ou symptômes sous le cou / systémiques** (essoufflement, douleur thoracique, palpitations) : au moins 5 jours de repos, puis des paliers d'au moins 48 h chacun, sans entraînement normal avant environ le 10e jour.
  - **Symptômes au-dessus du cou seulement** : on saute le palier de repos initial et l'entraînement normal ne reprend pas avant environ le 5e jour.
  - **Signal d'alarme** (douleur thoracique, palpitations, essoufflement à l'effort) : arrêt et avis médical.
- **Ce que vaut la règle traditionnelle.** La règle du cou (« neck check » : symptômes au-dessus ou en dessous du cou) n'a **pas de base scientifique solide** [87], mais aucun consensus ne soutient la reprise avant la fin de la fièvre [88]. La règle « fièvre ou symptômes systémiques : pas d'entraînement » est soutenue par deux sources concordantes [86][88].

### 5.4 Blessure

- **Tendinopathie d'Achille.** Dans un essai randomisé, continuer à courir avec une douleur **≤ 5/10** pendant l'effort n'a pas nui à la guérison [89]. Le critère « la douleur doit être revenue à la normale le lendemain matin » n'a pas été vérifié.
- **Fracture de fatigue du tibia.** On ne reprend qu'une fois la sensibilité osseuse disparue et la marche indolore. La reprise se fait en course-marche, **la distance avant l'intensité**. Niveau de preuve faible (IV, sur une échelle de I à V) [90].
- **Règle pour l'app :** une douleur qui modifie la foulée, une douleur à la marche ou une douleur osseuse localisée signifient **arrêt de la course et orientation vers un professionnel**, pas une adaptation automatique.

### 5.5 Pauses et désentraînement

- **Chez des sportifs très entraînés**, la VO2max baisse de 7 % en 21 jours d'arrêt et se stabilise à −16 % vers 56 jours (n = 7) [91]. La VO2max gagnée récemment se perd entièrement en cas d'arrêt prolongé [92]. **Réduire l'entraînement en maintenant l'intensité** préserve la VO2max [93].
- **Chez des marathoniens amateurs** (environ 44 000 coureurs Strava, observationnel), un trou de 7 à 13 jours pendant la préparation est associé à un marathon environ 4 % plus lent, 2 semaines à environ 6 %, 3 semaines à environ 7,5 % [94]. *La source secondaire est un article de presse sur l'étude.*
- **Convention Daniels** pour ajuster les **allures** : VDOT × 0,973 après 14 jours d'arrêt, × 0,931 après 28 jours, × 0,80 au-delà de 72 jours, avec des facteurs plus doux en cas d'entraînement croisé [95]. Cette convention **ne dit rien sur le volume**. Les deux ordres de grandeur, Daniels et Strava, sont cohérents.

### 5.6 Signaux médicaux (REDs)

Le consensus du CIO (2023) décrit le REDs : un déficit énergétique relatif, chez l'homme comme chez la femme, qui fragilise les os et favorise les fractures de fatigue. Son outil CAT2 classe le risque par feu tricolore [96]. **L'app ne diagnostique pas.** Une aménorrhée, des fractures de fatigue à répétition ou une baisse de performance inexpliquée avec fatigue doivent déclencher un **drapeau non diagnostique et une recommandation de consulter**.

## 6. Analyse post-séance et estimation des seuils

**Question :** quels indicateurs post-séance sont valides, comment noter le respect « prévu vs réalisé », et comment estimer les seuils automatiquement ?

### 6.1 Conformité prévu / réalisé

- **intervals.icu note la séance entière**, pas chaque bloc. La conformité vaut charge réalisée × 100 / charge prévue, ou la durée à défaut de charge. Code couleur : vert de 80 à 120 % ; orange de 50 à 80 % ou de 120 à 150 % ; rouge en dessous de 50 % ou au-dessus de 150 %. Une activité non appariée dans une semaine planifiée compte 0 % [97].
  - **Appariement** : intervals.icu associe une activité à une séance prévue, une pour une. Il n'apparie pas automatiquement une activité dont la charge est inférieure à 50 % de la charge prévue [97][98].
  - **TrainingPeaks** utilise le TSS, puis la durée, puis la distance ; ses propres pages se contredisent sur la définition du rouge [99][115].
- **Aucun des deux outils ne documente un score par bloc.** La comparaison bloc par bloc prévue dans ton brief est donc **une métrique à concevoir**. Pour la définir, les autres sections fournissent trois repères : juger les blocs courts sur l'allure, non sur la FC (§1.1) ; utiliser la GAP sur terrain vallonné (§6.3) ; tenir compte du fait que la charge réalisée dérive de la charge prescrite (§3.1).

### 6.2 Quels indicateurs croire

| Indicateur | Ce que disent les preuves | Usage |
|---|---|---|
| **FC à allure sous-maximale standardisée** | C'est le marqueur le plus fiable : environ 3 % de variation d'un jour à l'autre, plus petit changement important d'environ −1 %. Il se mesure sur les 30 à 60 dernières secondes après au moins 3 à 4 min d'effort stable [70]. | **Indicateur de tendance de forme n° 1.** Comparer à pente et chaleur comparables. |
| **Récupération cardiaque (HRR60, HRRc)** | Variation d'environ 25 % d'un jour à l'autre, et une récupération plus rapide n'est **pas toujours** un bon signe [70]. Elle varie aussi avec l'âge, la température et l'intensité de l'effort qui précède (facteurs de confusion) [100]. intervals.icu ne la calcule que si la FC a atteint au moins la FC au seuil [101]. | **Jamais de verdict sur une seule séance.** |
| **Découplage allure:FC** | Sur 82 303 marathoniens Strava, l'ajouter à un modèle de CS fait passer l'erreur de prédiction du marathon de 6,45 à 5,16 % [102]. Le seuil « < 5 % = bonne base aérobie » de Friel (défini en [103]) **n'a pas de validation publiée** : je n'ai trouvé aucune étude. | Pertinent sur sorties longues stables, en tendance. Seuil de 5 % à présenter comme un repère de coach. |
| **Facteur d'efficience (EF)** | Allure normalisée / FC ; sa hausse à séances comparables indique un progrès [29]. | Tendance. |
| **Écart RPE / FC** | L'effort perçu et la FC peuvent **se dissocier** [70]. | Un écart n'est pas automatiquement de la fatigue ; à croiser avec le contexte. |
| **Ressenti post-séance** | Les mesures subjectives suivent la charge mieux que les mesures objectives [76]. | Signal principal, à afficher avant les chiffres comme prévu dans ton brief. |

### 6.3 Corriger le contexte : pente et chaleur

- **Pente.** La GAP de Minetti (2002) donne le coût énergétique de la course selon la pente, Cr(i) = 155,4i⁵ − 30,4i⁴ − 43,3i³ + 46,3i² + 19,5i + 3,6 ; le facteur GAP vaut Cr(i)/3,6 [104]. C'est cette GAP qu'utilise l'étude de Smyth [105]. Strava utilise un modèle empirique basé sur la FC, plus généreux en descente, dont les coefficients ne sont pas publiés [104].
- **Chaleur.** Les temps de marathon se dégradent quand l'indice WBGT passe de 5 à 25 °C : de 1,7 % à 4,5 % pour les meilleurs hommes (écart au record de l'épreuve), davantage chez les coureurs plus lents [106]. La dégradation par degré de WBGT n'a pas été vérifiée. **La chaleur rend la FC et l'allure non comparables d'une séance à l'autre** ; l'app doit au moins enregistrer la température.

### 6.4 Estimer les seuils automatiquement

- **Vitesse critique à partir des données d'entraînement** (Smyth & Muniz-Pumares 2020, 31 190 coureurs Strava, texte intégral lu) [105] :
  - **Données** : les meilleurs temps corrigés de la pente (GAP de Minetti) sur 400 m, 800 m, 1 000 m, 1 500 m, 3 000 m et 5 000 m, sur les **16 semaines** précédentes, sans vérifier que les efforts étaient maximaux.
  - **Modèle** : distance-temps linéaire. **La meilleure combinaison est 400 + 800 + 5 000 m**, avec une erreur de prédiction du marathon de 7,7 % ; les combinaisons de distances courtes uniquement font le moins bien. Corriger les efforts non maximaux relève la CS d'environ 5 %.
  - **Allure marathon** : environ 93 % de la CS pour un marathon en 2 h 30, environ 79 % au-delà de 5 h. Rester à **≤ 94 % de la CS dans la 1re moitié**, faute de quoi on ralentit nettement ensuite.
  - **En théorie**, le modèle demande 3 à 4 efforts maximaux de 2 à 15 min [23].
- **intervals.icu** calcule une CS et un D′ sur sa courbe d'allures, à partir des meilleurs efforts de 1 à 5 km (points d'au moins 2 min, le dernier d'au moins 15 min) [107]. En revanche, d'après des utilisateurs expérimentés (septembre 2025), **il ne met pas à jour automatiquement l'allure seuil en course à pied** [108]. **L'app doit donc s'en charger** et la reporter dans intervals.icu, sinon la charge calculée là-bas (qui dépend de l'allure seuil, §2) dérive.
- **FC au seuil.** Le test de 30 min ne diffère pas significativement du critère labo, mais l'erreur type est d'environ 8 bpm, ce qui n'est pas négligeable aux bornes des zones [109]. TrainingPeaks ne relève son seuil FC automatiquement **qu'à la hausse** [110].
- **Prédiction de temps.** La formule de Riegel prédit un marathon **trop rapide d'au moins 10 min pour la moitié des coureurs amateurs** (N = 2 303). Ajouter le kilométrage hebdomadaire améliore nettement la prédiction [111].
- **Fréquence de réestimation.** Je n'ai **trouvé aucune étude** qui la fixe.

## 7. Ce que seul le croisement des sections 1 à 6 révèle

1. **Les données les plus « riches » d'Apple sont les moins fiables, et les plus simples sont les mieux étayées.** La puissance, la dynamique de foulée, les stades de sommeil, la VFC et les scores Apple n'ont pas de validation solide (§1). Le ressenti [76], l'allure, la FC de repos [72] et la FC à allure stable [70] sont à la fois bien mesurés et bien étayés. **L'app doit donc construire son raisonnement sur peu de signaux simples**, et traiter le reste comme du décor.
2. **La règle de sécurité la mieux prouvée est simple à calculer avec ces données.** Il suffit des distances des séances pour appliquer « aucune sortie à plus de 110 % de la plus longue des 30 derniers jours » [65]. Elle est mieux étayée que le choix de méthode lui-même (§3), et une progression trop rapide figure parmi les erreurs relevées dans des plans ChatGPT [58]. **C'est la contrainte la plus rentable à coder en dur.**
3. **Le choix de méthode compte peu ; la cohérence des allures compte beaucoup.** Polarisé et pyramidal se valent [46][47], et aucun algorithme de sélection n'existe. En revanche, tout plan repose sur un seuil exact, et, selon des utilisateurs du forum, intervals.icu **ne met pas à jour automatiquement l'allure seuil** en course [108]. Un seuil périmé fausse à la fois les allures prescrites, la charge et la conformité (§2, §6). **Le maintien automatique de la CS ou de l'allure seuil est plus important que la méthode affichée.**
4. **Les seuils d'alerte n'ont pas de base numérique, mais une structure se défend.** Ni la Form intervals.icu [64], ni l'ACWR [42], ni une FC de repos à « +5 bpm » ne reposent sur des données. Ce qui est soutenu, c'est le raisonnement en **écart à sa propre ligne de base** [70] et la **concordance de plusieurs signaux** [31], logique qu'Apple applique déjà dans Vitals [11]. La matrice respect × ressenti de ton brief est donc conceptuellement juste, mais **ses seuils sont des paramètres à calibrer, pas des faits à citer**.
5. **Les garde-fous doivent vivre dans le code, pas dans le prompt.** L'étude de cas ChatGPT montre que des garde-fous seulement conversationnels ne sont pas respectés et que l'absence de profil persistant dégrade le coaching [59]. Ton brief prévoit déjà les deux remèdes : un profil vivant et la confirmation avant toute modification.

## 8. Recommandations

Chaque recommandation indique **où elle s'applique** (PRD ou architecture) et **sur quoi elle repose**.

| # | Recommandation | Destination | Base de confiance |
|---|---|---|---|
| R1 | **Contrainte dure du générateur** : aucune sortie ne dépasse 1,10 × la plus longue des 30 derniers jours, et une cible nettement inférieure à 10 % pour les sorties longues (la valeur exacte, par exemple +5 %, est un paramètre de conception non sourcé). Toute exception exige ton accord explicite et un avertissement. Ne pas utiliser la règle des 10 % par semaine comme sécurité ; tout au plus comme garde-fou secondaire. | PRD (contraintes du générateur) ; architecture (validateur de plan déterministe, hors LLM) | Élevée sur le fait qu'un pic par sortie est un risque ; **moyenne** sur la valeur exacte de 10 % (une seule cohorte) [65][84] |
| R2 | **Règle de reprise** : quand la fenêtre des 30 jours est vide ou trop courte après une pause, repartir d'un volume réduit, appliquer le facteur VDOT de Daniels aux allures, et revalider la CS sur les premières séances. | PRD (module Empêchement) | Moyenne : convention Daniels [95] et données Strava observationnelles [94] |
| R3 | **Règles maladie** : fièvre ou symptômes sous le cou entraînent l'annulation, au moins 5 jours de repos, puis une reprise par paliers d'au moins 48 h, sans entraînement normal avant environ le 10e jour. Symptômes au-dessus du cou seulement : séance allégée ou annulée, reprise normale vers le 5e jour. Signal d'alarme : arrêt et orientation médicale. Blessure : douleur ≤ 5/10 comme garde-fou, mais toute douleur osseuse ou à la marche signifie arrêt et orientation vers un professionnel. | PRD (Empêchement : choix malade/blessé) | Moyenne (guide d'experts, pas d'essai) ; l'extension du guide COVID aux autres maladies est une transposition [86][88][89][90] |
| R4 | **Ancrer toutes les zones et allures sur la vitesse critique**, estimée à partir de tes meilleurs efforts corrigés de la pente sur 16 semaines (combinaison 400/800/5 000 m), avec une FC max observée plutôt qu'une formule. **Reporter la CS ou l'allure seuil dans intervals.icu**, qui ne la met pas à jour automatiquement en course. | Architecture (service de seuils) ; PRD (profil vivant) | Élevée pour la méthode CS [23][105] ; moyenne pour le comportement d'intervals.icu (forum) [108] |
| R5 | **Méthode par défaut : pyramidale, inspirée de Daniels** (2 séances de qualité, une sortie longue, le reste facile), avec la variante polarisée pour un objectif de performance sur volume plus élevé. **La ligne d'explication doit être honnête** : « les études montrent peu d'écart entre méthodes ; celle-ci colle à ton volume et à ton objectif ». Ne jamais afficher MAF ou « norvégienne » comme validées. | PRD (génération et explication) | Élevée sur « peu de différence » [46][47] ; **faible** sur la préférence pour le pyramidal chez l'amateur (non significatif) [46] |
| R6 | **Hiérarchie des données en entrée** : (1) ressenti et allure ; (2) FC en effort stable, FC de repos, durée de sommeil ; (3) VFC SDNN et VO2max, **en tendance seulement** ; (4) puissance, dynamique de foulée, stades de sommeil et scores Apple, **en affichage seulement**. Sur les fractionnés courts, juger l'exécution sur l'allure. | PRD (« Tes données », 3–5 indicateurs) ; architecture (modèle de données) | Moyenne à élevée selon la métrique (§1) [6][8][10][14][72] |
| R7 | **Détecteur de surmenage** : ligne de base de 4 semaines ou plus et moyenne sur 7 jours. Une alerte exige au moins 2 signaux objectifs concordants plus un ressenti dégradé répété, puis une **proposition** (jamais une modification silencieuse). Les seuils sont paramétrables et documentés comme tels. Ne pas utiliser l'ACWR, ni la Form d'intervals.icu seule. | PRD (seuil de signaux concordants) ; architecture (règles, pas LLM) | Moyenne pour la structure [31][70][76] ; **les seuils numériques sont non étayés** |
| R8 | **Feedback post-séance** : la conformité de la séance entière vient d'intervals.icu ; la note par bloc est **une métrique propre à définir**, basée sur l'allure. Indicateurs : FC à allure comparable (tendance), découplage sur les séances longues, EF. HRRc et un écart ponctuel entre ressenti et FC ne justifient **jamais** un verdict à eux seuls. Associer chaque consigne à sa raison et aux sensations attendues. | PRD (spécification du feedback) | Moyenne [70][97][102][59] |
| R9 | **Affûtage** : 2 semaines (jusqu'à 3 pour le marathon), volume réduit de 41 à 60 %, intensité et fréquence maintenues. | PRD (structure de plan) | Élevée [44][56] |
| R10 | **Les garde-fous en code** : R1, R3, R7 et R9 s'appliquent comme validations déterministes sur la sortie du LLM, et non comme instructions dans le prompt. | Architecture | Moyenne (étude de cas + erreurs documentées des LLM) [58][59] |

## 9. Questions ouvertes

| Question | Pourquoi c'est important | Comment y répondre |
|---|---|---|
| Que contient **exactement** le FIT exporté par HealthFit pour une course Apple Watch (fréquence d'échantillonnage de la FC, puissance, GCT, tours) ? | Cela conditionne les calculs bloc par bloc et les charges. | **Test empirique** : exporter 2 ou 3 courses et lire le FIT, ou l'onglet *Data* d'intervals.icu. Aucune doc n'existe. |
| Quelle **valeur de VFC** HealthFit envoie-t-il : moyenne de la journée ou mesure nocturne ? | Cela décide si la VFC est exploitable même en tendance. | Comparer Apple Santé et le champ wellness d'intervals.icu sur 2 semaines, ou demander au développeur de HealthFit. |
| Les séances planifiées HealthFit gèrent-elles des **cibles d'allure par bloc** sur la montre ? | Cela conditionne le format des séances écrites dans intervals.icu. | Test d'une séance structurée simple. |
| Quels **seuils** pour le détecteur de surmenage (fraction du CV, nombre de jours) ? | Aucun seuil validé n'existe. | Calibrer sur ton historique (rétro-analyse des périodes de maladie ou de fatigue), puis ajuster. |
| La règle des 10 % par sortie tient-elle **pour l'intensité** (pas seulement la distance) et selon le niveau ? | La cohorte ne l'a pas étudié. | Surveiller les réplications (BJSM, groupe RUNSAFE) ; un Refresh dans 12 mois. |
| Les plans générés par les **LLM de 2026** commettent-ils encore les erreurs de 2023 ? | Les études portent sur GPT-3.5. | Suivre les publications (Düking et al.) ; tester tes propres plans contre R1, R9 et R10. |
| Durée de plan, nombre de séances de qualité et semaines allégées : **quelles preuves** ? | Ce ne sont aujourd'hui que des conventions. | Peu de chances qu'une étude tranche ; garder des conventions explicites et paramétrables. |
| Fréquence de **réestimation de la CS** ? | Aucune étude trouvée. | Choix de conception : recalcul à chaque nouvelle meilleure performance, et revue toutes les 4 à 6 semaines. |

## 10. Contre-preuves

La passe de contradiction systématique (red team) n'a pas été lancée : elle était désactivée et la validation est au niveau `normal`. Les contradictions trouvées en cours de route sont rapportées dans le texte, avec les deux positions sourcées : VFC guidée (§4.3), ACWR (§4.2), Oliveira vs Rosenblat (§3.1), protocoles LTHR de Friel (§2.2).

## 11. Annexe des sources

Toutes les sources ont été consultées le 2026-11-25.

| n | Affirmation soutenue | Éditeur | Date de publication | Consultée | Confiance |
|---|---|---|---|---|---|
| [1] | Types HealthKit course (runningPower, GCT…), pas de type cadence | [Apple Developer](https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/runningpower) | 2022-06 | 2026-11-25 | moyenne |
| [2] | Métriques de course Apple Watch (définitions, disponibilité) | [Apple Support](https://support.apple.com/guide/watch/apd1f24d4d35/watchos) | 2025 | 2026-11-25 | moyenne |
| [3] | Training Load : 7 j vs 28 j | [Apple Support](https://support.apple.com/guide/watch/apde4c07a6cf) | 2026 | 2026-11-25 | élevée |
| [4] | estimatedWorkoutEffortScore (iOS 18) | [Apple Developer](https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/estimatedworkouteffortscore) | 2024 | 2026-11-25 | élevée |
| [5] | Entrées du score d'effort (secondaire) | [DC Rainmaker](https://dcrainmaker.com/2024/07/apples-training-load-vitals-watchos11.html) | 2024-07 | 2026-11-25 | faible |
| [6] | White paper stades de sommeil (kappa 0,63→0,68) | [Apple](https://www.apple.com/kr/health/pdf/Estimating_Sleep_Stages_from_Apple_Watch_Oct_2025.pdf) | 2025-10 | 2026-11-25 | élevée |
| [7] | Recalibrage du score de sommeil watchOS 26.2 | [MacRumors](https://www.macrumors.com/2025/11/04/watchos-26-2-updates-sleep-score-ranges/) | 2025-11 | 2026-11-25 | moyenne |
| [8] | White paper VO2max (méthode, validation N=221) | [Apple](https://www.apple.com/healthcare/docs/site/Using_Apple_Watch_to_Estimate_Cardio_Fitness_with_VO2_max.pdf) | 2021-05 | 2026-11-25 | élevée |
| [9] | VO2max Series 7 : fiabilité faible (n=19) | [JMIR Biomed Eng (Caserman et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325102) | 2024-08 | 2026-11-25 | moyenne |
| [10] | VO2max Series 10 : biais −6,25 ml/kg/min, MAPE 13,2 % (n=40) | [Mayo Clin Proc Digital Health (Lambe et al.)](https://doi.org/10.1016/j.mcpdig.2026.100357) | 2026-04 | 2026-11-25 | élevée |
| [11] | Vitals : plage 7 nuits, alerte multi-métriques, VFC Series 12 seulement | [Apple Support](https://support.apple.com/120142) | 2026-09 | 2026-11-25 | élevée |
| [12] | Validité FC optique en course selon l'intensité | [PMC (étude relue)](https://pmc.ncbi.nlm.nih.gov/articles/PMC6732081) | 2019 | 2026-11-25 | moyenne |
| [13] | FC Apple Watch : mesure continue, lectures aberrantes possibles | [Apple Support](https://support.apple.com/en-AU/120277) | 2026-09 | 2026-11-25 | moyenne |
| [14] | Comparaisons puissance de course Apple/Stryd (blog) | [the5krunner](https://the5krunner.com/2022/08/06/running-power-comparisons-more-comparisons-with-apple-garmin-stryd-coros-polar/) | 2022-08 | 2026-11-25 | faible |
| [15] | HealthFit → intervals.icu : VFC en SDNN, sync bien-être | [Forum intervals.icu](https://forum.intervals.icu/t/healthfit-sync-for-activities-and-health-metrics/2718?page=4) | 2021-03 | 2026-11-25 | moyenne |
| [16] | VFC Series 6 vs ECG (n=78), accord modéré | [MDPI Sensors](https://www.mdpi.com/1424-8220/25/8/2380) | 2025-04 | 2026-11-25 | moyenne |
| [17] | RR Apple Watch vs Polar H7 au repos (n=20) | [Sensors (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC6111985) | 2018 | 2026-11-25 | moyenne |
| [18] | HealthFit importe les séances planifiées ; abonnement requis | [Forum intervals.icu (dév. HealthFit)](https://forum.intervals.icu/t/import-planned-workouts-from-intervals-icu-into-apple-health/112136) | 2026-03 | 2026-11-25 | élevée |
| [19] | HealthFit App Store : tarifs, VFC de récupération Series 12+ | [Apple App Store](https://apps.apple.com/us/app/healthfit/id1202650514) | 2026-10 | 2026-11-25 | moyenne |
| [20] | IntervalsWellnessSync : VFC/FC repos du sommeil | [Forum intervals.icu](https://forum.intervals.icu/t/intervalswellnesssync-simple-apple-watch-health-data-sync-to-intervals-icu/125843) | 2026-03 | 2026-11-25 | faible |
| [21] | Sommeil Series 8 vs polysomnographie (n=35) | [Sensors / PubMed](https://pubmed.ncbi.nlm.nih.gov/39460013/) | 2024 | 2026-11-25 | moyenne |
| [22] | VFC Apple = SDNN à moments opaques ; pas d’accès RR | [Marco Altini (praticien, HRV4Training)](https://medium.com/@marco_alt/how-to-make-sense-of-your-apple-watch-heart-rate-variability-hrv-data-89bf4a510438) | 2023 | 2026-11-25 | moyenne |
| [23] | Vitesse critique et D′ : modèle d = CS·t + D′, 70–90 % VO2max | [Sports Medicine (Jones & Vanhatalo)](https://pmc.ncbi.nlm.nih.gov/articles/PMC5371646) | 2017-03 | 2026-11-25 | élevée |
| [24] | La CS (pas la MLSS) indexe l'état stable métabolique maximal | [Physiological Reports (Jones et al.)](https://doi.org/10.14814/phy2.14098) | 2019-05 | 2026-11-25 | moyenne |
| [25] | Domaines d'intensité ; méthodes de seuil non interchangeables ; % fixes peu valides | [Sports Medicine (Jamnick et al.)](https://pubmed.ncbi.nlm.nih.gov/32729096/) | 2020 | 2026-11-25 | moyenne |
| [26] | FC max = 208 − 0,7 × âge (Tanaka) | [J Am Coll Cardiol (Tanaka et al.)](https://pubmed.ncbi.nlm.nih.gov/11153730/) | 2001-01 | 2026-11-25 | élevée |
| [27] | Formules TRIMP Banister/Edwards, sRPE, réserve cardiaque | [Frontiers in Physiology (Shaw et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7240108/) | 2020-05 | 2026-11-25 | élevée |
| [28] | SDNN vs RMSSD : signification, durées d'enregistrement, normes | [Frontiers in Public Health (Shaffer & Ginsberg)](https://www.frontiersin.org/articles/10.3389/fpubh.2017.00258/full) | 2017 | 2026-11-25 | élevée |
| [29] | Glossaire TSS/rTSS/hrTSS, CTL/ATL/TSB, EF, découplage | [TrainingPeaks](https://www.trainingpeaks.com/learn/articles/glossary-of-trainingpeaks-metrics/) | 2026-06 | 2026-11-25 | élevée |
| [30] | Monotonie et contrainte (Foster 1998) | [MSSE (Foster)](https://doi.org/10.1097/00005768-199807000-00023) | 1998-07 | 2026-11-25 | moyenne |
| [31] | Consensus ECSS/ACSM surmenage/surentraînement | [MSSE / EJSS (Meeusen et al.)](https://pubmed.ncbi.nlm.nih.gov/23247672/) | 2013 | 2026-11-25 | élevée |
| [32] | vVO2max et Tlim (Billat) | [Sports Medicine (Billat & Koralsztein)](https://pubmed.ncbi.nlm.nih.gov/8857705/) | 1996 | 2026-11-25 | moyenne |
| [33] | Économie de course : revue | [Sports Medicine Open (Barnes & Kilding)](https://link.springer.com/article/10.1186/s40798-015-0007-y) | 2015 | 2026-11-25 | moyenne |
| [34] | Dérive cardiovasculaire | [Exerc Sport Sci Rev (Coyle & González-Alonso)](https://scilit.com/publications/b2c41c04695672f811b81cfb09bf4f36) | 2001 | 2026-11-25 | moyenne |
| [35] | Protocole LTHR de Friel (20 min − 5 %) | [Joe Friel (site d'auteur)](https://joefrieltraining.com/a-quick-guide-to-setting-zones/) | 2025-09 | 2026-11-25 | moyenne |
| [36] | Indice de polarisation (Treff) | [Frontiers in Physiology (Treff et al.)](https://www.frontiersin.org/articles/10.3389/fphys.2019.00707/full) | 2019-06 | 2026-11-25 | élevée |
| [37] | Modèle à 3 zones de Seiler | [Frontiers in Physiology (revue)](https://www.frontiersin.org/articles/10.3389/fphys.2015.00295/full) | 2015 | 2026-11-25 | moyenne |
| [38] | Équations VDOT (secondaire, calculateur) | [getfitcraft (secondaire)](https://getfitcraft.com/tools/vdot-calculator) | 2025 | 2026-11-25 | faible |
| [39] | Charge allure intervals.icu = h × IF² × 100 (code développeur) | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/running-load-vs-trainingpeaks-rtss/5774) | 2022-01 | 2026-11-25 | moyenne |
| [40] | HRSS = TRIMP normalisé (développeur) | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/hrss-normalized-trimp-training-load/569) | 2020-03 | 2026-11-25 | moyenne |
| [41] | Form = Fitness − Fatigue le jour même ; 42/7 j ; zone rouge −30 | [Forum intervals.icu (guide + développeur)](https://forum.intervals.icu/t/fitness-page-a-guide-to-getting-started/17702) | 2023-02 | 2026-11-25 | élevée |
| [42] | Critique de l'ACWR (Impellizzeri et al.) | [Journal of Athletic Training](https://pmc.ncbi.nlm.nih.gov/articles/PMC7534938) | 2020 | 2026-11-25 | élevée |
| [43] | Indice de polarisation dans intervals.icu | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/polarization-index-added/49877) | 2023-09 | 2026-11-25 | élevée |
| [44] | Affûtage : 2 sem, volume −41–60 % (méta-analyse) | [MSSE (Bosquet et al.)](https://pubmed.ncbi.nlm.nih.gov/17762369/) | 2007 | 2026-11-25 | élevée |
| [45] | Périodisation par blocs (Issurin) et critique | [Sports Medicine (Issurin)](https://pubmed.ncbi.nlm.nih.gov/20199119/) | 2010 | 2026-11-25 | moyenne |
| [46] | Méta-analyse IPD polarisé vs pyramidal (13 études), sous-groupes | [Sports Medicine (Rosenblat et al.)](https://doi.org/10.1007/s40279-024-02149-3) | 2025-01 | 2026-11-25 | élevée |
| [47] | Méta-analyse TID (17 études) : POL > autres sur VO2peak seulement | [Sports Medicine (Oliveira et al.)](https://doi.org/10.1007/s40279-024-02034-z) | 2024 | 2026-11-25 | élevée |
| [48] | Pyramidal/polarisé > seuil-dominant chez coureurs de fond | [IJSPP (Kenneally et al.)](https://pubmed.ncbi.nlm.nih.gov/29182410/) | 2018 | 2026-11-25 | moyenne |
| [49] | ECR coureurs amateurs 10 km : POL > entre-seuils | [IJSPP (Muñoz et al.)](https://pubmed.ncbi.nlm.nih.gov/23752040/) | 2014 | 2026-11-25 | moyenne |
| [50] | ≥ 70 % bas + ≤ 30 % seuil/HIIT pour demi-fond et fond | [Int J Sports Med (Campos et al.)](https://pubmed.ncbi.nlm.nih.gov/34749417/) | 2022 | 2026-11-25 | moyenne |
| [51] | Méthode Daniels (VDOT, plafonds par séance) – secondaire | [Coach Ray (blog, d'après Daniels' Running Formula)](https://www.coachray.nz/2021/10/11/jack-daniels-phd-formulaic-approach-to-periodisation/) | 2021-10 | 2026-11-25 | moyenne |
| [52] | Plans 80/20 : niveaux 0–3, zones ancrées sur seuil | [TrainingPeaks / 80/20 Endurance](https://www.trainingpeaks.com/partners/80-20-endurance/) | 2025 | 2026-11-25 | moyenne |
| [53] | Modèle norvégien : description, absence d'essais | [IJERPH (Casado et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC10000870/) | 2023 | 2026-11-25 | moyenne |
| [54] | MAF : article d'hypothèse, formule non validée | [Frontiers in Physiology (Maffetone & Laursen)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7142223) | 2020 | 2026-11-25 | élevée |
| [55] | ECR course-marche marathon (n=42) | [J Sci Med Sport (Hottenrott et al.)](https://pubmed.ncbi.nlm.nih.gov/25467199/) | 2016 | 2026-11-25 | moyenne |
| [56] | Affûtage endurance : ≤ 21 j, volume −41–60 % | [PLoS One (Wang et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC10171681/) | 2023 | 2026-11-25 | moyenne |
| [57] | Sortie longue ≈ 25–30 % du volume, plafond ~3 h (Daniels/Hansons) – convention | [Luke Humphrey Running (Hansons)](https://lukehumphreyrunning.com/hansons-marathon-method-the-16-miler/) | 2020 | 2026-11-25 | faible |
| [58] | Évaluation experte de plans ChatGPT (10 coachs, 22 critères) | [J Sports Sci Med (Düking et al.)](https://jssm.org/jssm-23-56.xml-Fulltext) | 2024 | 2026-11-25 | élevée |
| [59] | Étude de cas semi-marathon coaché par ChatGPT (prépublication) | [arXiv](https://arxiv.org/html/2509.26593v1) | 2025-09 | 2026-11-25 | faible |
| [60] | Feedback dans l'auto-suivi : d = 0,29 (corrigé) | [IJBNPA (Krukowski et al.)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10765525/) | 2024-01 | 2026-11-25 | moyenne |
| [61] | Bloom : coach GPT-4o vs app sans LLM (N=54, prépublication CHI 2026) | [arXiv](https://arxiv.org/html/2510.05449v2) | 2026-03 | 2026-11-25 | faible |
| [62] | Limites du modèle de Banister (mauvais conditionnement, instabilité) | [J Sports Sci (Hellard et al.)](https://doi.org/10.1080/02640410500244697) | 2006 | 2026-11-25 | moyenne |
| [63] | Validité des modèles fitness-fatigue mal étudiée ; bruit > 4 % rédhibitoire (thèse) | [Robert Gordon University (Stephens Hemingway, thèse)](https://rgu-repository.worktribe.com/output/682069) | 2021 | 2026-11-25 | moyenne |
| [64] | Zones de Form = Friel, « règle empirique » (développeur intervals.icu) | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/zones-of-form-in-fitness-chart/3623) | 2021-04 | 2026-11-25 | moyenne |
| [65] | Garmin-RUNSAFE : pic sur une sortie vs 30 j ; pas d'association semaine/ACWR | [BJSM (Frandsen et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12421110) | 2025 | 2026-11-25 | élevée |
| [66] | Éditorial : littérature ACWR contestée | [Frontiers (éditorial)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8138569/) | 2021 | 2026-11-25 | moyenne |
| [67] | Méta-analyse VFC guidée : ES 0,40 VO2max | [IJERPH (Granero-Gallegos et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7663087) | 2020 | 2026-11-25 | moyenne |
| [68] | Méta-analyse VFC guidée : non significatif, moins de non-répondeurs | [PubMed (Düking et al.)](https://pubmed.ncbi.nlm.nih.gov/34489178/) | 2021 | 2026-11-25 | moyenne |
| [69] | Méta-analyse VFC guidée : petit, non significatif | [IJERPH (Manresa-Rocamora et al.)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8507742/) | 2021 | 2026-11-25 | moyenne |
| [70] | Suivi par FC/VFC : CV individuel, plus petit changement, aucun seuil prouvé | [Frontiers in Physiology (Buchheit)](https://www.frontiersin.org/journals/physiology/articles/10.3389/fphys.2014.00073/full) | 2014 | 2026-11-25 | élevée |
| [71] | Moyenne VFC sur 7 j > valeurs isolées (cas élites) | [EJAP (Plews et al.)](https://pubmed.ncbi.nlm.nih.gov/22367011/) | 2012 | 2026-11-25 | moyenne |
| [72] | Apple Watch S9/Ultra 2 : SDNN −8,3 ms vs Polar H10 ; FC repos −0,08 bpm | [MDPI Sensors (O'Grady et al.)](https://www.mdpi.com/1424-8220/24/19/6220) | 2024 | 2026-11-25 | élevée |
| [73] | Validation VFC nocturne vs ECG (Oura, Whoop, Garmin ; Apple non testée) | [Physiological Reports (Dial et al.)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12367097) | 2025-08 | 2026-11-25 | élevée |
| [74] | Critique VFC Apple, préférer une mesure matinale (praticien) | [Marco Altini (Substack)](https://marcoaltini.substack.com/p/trusting-the-right-tool-for-the-job) | 2024-11 | 2026-11-25 | moyenne |
| [75] | < 8 h de sommeil : risque de blessure ×1,7 (adolescents) | [J Pediatr Orthop (Milewski et al.)](https://doi.org/10.1097/bpo.0000000000000151) | 2014-03 | 2026-11-25 | moyenne |
| [76] | Mesures subjectives > objectives pour suivre la charge (revue de 56 études) | [BJSM (Saw, Main & Gastin)](https://bjsm.bmj.com/content/50/5/281.short) | 2016 | 2026-11-25 | élevée |
| [77] | Incidence : 17,8 (débutants) vs 7,7 (loisir) blessures/1000 h | [Sports Medicine (Videbæk et al.)](https://pubmed.ncbi.nlm.nih.gov/25951917/) | 2015 | 2026-11-25 | élevée |
| [78] | Incidence ~40 % ; blessures les plus fréquentes | [J Sport Health Sci (Kakouris et al.)](https://pubmed.ncbi.nlm.nih.gov/33862272) | 2021 | 2026-11-25 | élevée |
| [79] | Blessure antérieure = principal facteur de risque | [BJSM (Saragiotto et al.)](https://pubmed.ncbi.nlm.nih.gov/24809248/) | 2014 | 2026-11-25 | moyenne |
| [80] | Facteurs de risque/protection en course de fond (blessure antérieure) | [Sports Medicine (Hulme et al.)](https://www.doi.org/10.1007/s40279-016-0636-4) | 2017 | 2026-11-25 | moyenne |
| [81] | Biomécanique non liée aux blessures chez non-élites | [Sports Medicine Open (méta-analyse)](https://pubmed.ncbi.nlm.nih.gov/35254562/) | 2022 | 2026-11-25 | moyenne |
| [82] | Renforcement musculaire : blessures < 1/3 (tous sports) | [BJSM (Lauersen et al.)](https://pubmed.ncbi.nlm.nih.gov/24100287/) | 2014 | 2026-11-25 | élevée |
| [83] | ECR renforcement hanches/tronc chez débutants (efficace) vs pied/cheville | [PubMed (ECR 2024)](https://pubmed.ncbi.nlm.nih.gov/38724071/) | 2024 | 2026-11-25 | moyenne |
| [84] | GRONORUN : règle des 10 % hebdo n'a pas réduit les blessures | [AJSM (Buist et al.) via J Athl Train](https://pmc.ncbi.nlm.nih.gov/articles/PMC9528699) | 2008 | 2026-11-25 | élevée |
| [85] | Débutants : > 30 %/sem vs < 10 % (p = 0,07) | [JOSPT (Nielsen et al.)](https://vbn.aau.dk/da/publications/excessive-progression-in-weekly-running-distance-and-risk-of-runn) | 2014 | 2026-11-25 | moyenne |
| [86] | Retour progressif après maladie (paliers ≥ 48 h, J5/J10) | [BJSM blog (Elliott et al.)](https://blogs.bmj.com/bjsm/?p=10084) | 2022-04 | 2026-11-25 | moyenne |
| [87] | « Neck check » sans base scientifique solide (revue liée au CIO) | [Univ. Pretoria SEMLI (résumé)](https://www.up.ac.za/Sport-Exercise-Medicine-and-Lifestyle-Institute/news/medical-guidance-athletes-returning-sport-after-acute-respiratory-illness) | 2023 | 2026-11-25 | moyenne |
| [88] | Pas de reprise avant fin de la fièvre ; risque de myocardite | [Sports Medicine (revue)](https://link.springer.com/content/pdf/10.1007/s40279-023-01969-z.pdf) | 2023 | 2026-11-25 | moyenne |
| [89] | Modèle de surveillance de la douleur ≤ 5/10 (Achille, ECR) | [AJSM (Silbernagel et al.)](https://doi.org/10.1177/0363546506298279) | 2007 | 2026-11-25 | moyenne |
| [90] | Reprise après fracture de fatigue tibiale (revue exploratoire) | [Sports Medicine](https://link.springer.com/article/10.1007/s40279-024-02051-y) | 2024 | 2026-11-25 | moyenne |
| [91] | Désentraînement : VO2max −7 % à J21, −16 % à J56 | [J Appl Physiol (Coyle et al.)](https://europepmc.org/article/MED/6511559) | 1984 | 2026-11-25 | élevée |
| [92] | Désentraînement > 4 sem : VO2max récemment acquise perdue | [Sports Medicine (Mujika & Padilla)](https://europepmc.org/article/MED/10999420) | 2000 | 2026-11-25 | élevée |
| [93] | Réduire volume/fréquence en gardant l'intensité préserve la VO2max | [Sports Medicine (Neufer)](https://pubmed.ncbi.nlm.nih.gov/2692122/) | 1989 | 2026-11-25 | moyenne |
| [94] | Trous d'entraînement et marathon (Strava, ~44 000 coureurs) | [Outside (sur Feely, Smyth et al. 2022)](https://www.outsideonline.com/health/training-performance/missed-marathon-training-study-2023) | 2023-01 | 2026-11-25 | moyenne |
| [95] | Table d'ajustement du VDOT après arrêt (convention Daniels) | [VDOT O2 (site officiel Daniels)](https://news.vdoto2.com/?p=1285) | 2022-03 | 2026-11-25 | élevée |
| [96] | Consensus CIO REDs 2023 et outil CAT2 | [BJSM (Mountjoy et al.)](https://pubmed.ncbi.nlm.nih.gov/37752011/) | 2023-09 | 2026-11-25 | élevée |
| [97] | Conformité intervals.icu : charge/prévu, vert 80–120 % | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/compliance-activity-field/47805) | 2023-08 | 2026-11-25 | élevée |
| [98] | Appariement : pas d'auto-appariement si charge < 50 % du prévu | [Forum intervals.icu](https://forum.intervals.icu/t/workout-planning-and-matching-activities/1809) | 2021 | 2026-11-25 | moyenne |
| [99] | Conformité TrainingPeaks (TSS > durée > distance) | [TrainingPeaks](https://www.trainingpeaks.com/learn/trainingpeaks-athlete-user-guide/) | 2025 | 2026-11-25 | moyenne |
| [100] | HRR comme marqueur d'entraînement : revue, facteurs confondants | [IJSPP (Daanen et al.)](https://research.vu.nl/ws/files/3163042/292221.pdf) | 2012 | 2026-11-25 | moyenne |
| [101] | HRRc intervals.icu calculée si FC ≥ FC seuil | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/heart-rate-recovery-hrrc/387) | 2021-12 | 2026-11-25 | moyenne |
| [102] | Découplage chez 82 303 marathoniens ; améliore la prédiction | [Sports Medicine (Smyth et al.)](https://link.springer.com/article/10.1007/s40279-022-01680-5) | 2022 | 2026-11-25 | moyenne |
| [103] | Découplage et EF (Friel) : définitions | [TrainingPeaks blog](https://trainingpeaks.com/blog/efficiency-factor-and-decoupling) | 2024 | 2026-11-25 | moyenne |
| [104] | GAP : polynôme de Minetti, modèle Strava 2017 (rétro-ingénierie) | [Aaron Schroeder (analyse secondaire)](https://aaron-schroeder.github.io/reverse-engineering/grade-adjusted-pace.html) | 2023-11 | 2026-11-25 | moyenne |
| [105] | CS depuis données Strava d'entraînement (400/800/5000 m, 16 sem.) | [MSSE (Smyth & Muniz-Pumares)](https://uhra.herts.ac.uk/handle/2299/22878) | 2020 | 2026-11-25 | élevée |
| [106] | Météo et performance marathon (WBGT 5–25 °C) | [MSSE (Ely et al.)](https://experts.umn.edu/en/publications/impact-of-weather-on-marathon-running-performance/) | 2007 | 2026-11-25 | moyenne |
| [107] | CS/D′ sur la courbe d'allures intervals.icu (efforts 1–5 km) | [Forum intervals.icu (développeur)](https://forum.intervals.icu/t/towards-grade-adjusted-best-paces-and-cs-d-computation/12197) | 2022-05 | 2026-11-25 | moyenne |
| [108] | Pas de mise à jour auto de l'allure seuil en course (intervals.icu) | [Forum intervals.icu (utilisateurs)](https://forum.intervals.icu/t/automatic-threshold-pace-calculation/112254) | 2025-09 | 2026-11-25 | moyenne |
| [109] | Test 30 min FC seuil vs labo : erreur type ~8 bpm | [PubMed (2005)](https://pubmed.ncbi.nlm.nih.gov/16095403/) | 2005 | 2026-11-25 | moyenne |
| [110] | TrainingPeaks : seuil FC auto uniquement à la hausse | [TrainingPeaks](https://www.trainingpeaks.com/learn/articles/thresholds-411/) | 2024 | 2026-11-25 | moyenne |
| [111] | Riegel sous-estime le temps marathon des amateurs ; + kilométrage | [BMC Sports Sci Med Rehabil (Vickers & Vertosick)](https://link.springer.com/article/10.1186/s13102-016-0052-y) | 2016 | 2026-11-25 | moyenne |
| [112] | Retard de la FC optique vs ceinture sur fractionnés (presse) | [Coach (presse)](https://coachweb.com/gear/fitness-trackers/are-apple-and-garmin-heart-rate-monitors-accurate-this-study-says-yes) | 2023 | 2026-11-25 | faible |
| [113] | Échantillonnage VFC Apple ~toutes les 2 h la nuit (blog éditeur) | [Terra (blog éditeur)](https://tryterra.co/research/how-hrv-actually-works) | 2025 | 2026-11-25 | faible |
| [114] | Ligne de base de 4 semaines avant entraînement guidé par la VFC | [IJSPP (Javaloyes et al.)](https://pubmed.ncbi.nlm.nih.gov/29809080/) | 2019 | 2026-11-25 | moyenne |
| [115] | Définition divergente du rouge dans la conformité TrainingPeaks | [TrainingPeaks Help Center](https://help.trainingpeaks.com/hc/en-us/articles/204861204) | 2025 | 2026-11-25 | moyenne |

## 12. Carte de fraîcheur

Calculée avec `recon_kit.py staleness`, avec ces fenêtres : fonctionnalité d'appareil et comportement d'outil 18 mois, guide de pratique 24 mois, étude empirique ou de validation 36 mois, évaluation d'IA 6 mois. **Les articles de référence** (Bosquet 2007, Buist 2008, Saw 2016, Buchheit 2014, Smyth 2020) dépassent leur fenêtre, mais la science de référence n'a pas de date de péremption. J'ai vérifié qu'aucun travail plus récent ne les contredit : Bosquet est confirmé par une méta-analyse de 2023, et Buist est renforcé par Frandsen 2025.

| Affirmation | Classe | Publiée | À revérifier | État |
|---|---|---|---|---|
| [39] Charge allure intervals.icu = h × IF² × 100 | comportement d'outil | 2022-01 | 2023-07 | **périmée : à vérifier dans l'app** |
| [64] Zones de Form = règle empirique | comportement d'outil | 2021-04 | 2022-10 | **périmée** |
| [41] Form calculée le jour même, 42/7 j | comportement d'outil | 2023-02 | 2024-08 | **périmée** |
| [97] Conformité vert 80–120 % | comportement d'outil | 2023-08 | 2025-02 | **périmée** |
| [15] VFC Apple = SDNN | fonctionnalité | 2021-03 | 2022-09 | périmée, mais reconfirmée par le vérificateur (docs Apple) |
| [58] Erreurs des plans ChatGPT | évaluation d'IA | 2024-03 | 2024-09 | **périmée** (modèle de 2023) |
| [86] Retour après maladie (BJSM) | guide | 2022-04 | 2024-04 | périmée : chercher une mise à jour du consensus |
| [95] Table VDOT après arrêt | guide | 2022-03 | 2024-03 | convention stable |
| [108] Pas de mise à jour auto du seuil course | comportement d'outil | 2025-09 | **2027-03** | à jour |
| [6] Stades de sommeil kappa 0,68 | fonctionnalité | 2025-10 | 2027-04 | à jour |
| [18] Abonnement HealthFit requis | comportement d'outil | 2026-03 | 2027-09 | à jour |
| [11] Vitals, VFC Series 12 seulement | fonctionnalité | 2026-09 | 2028-03 | à jour |
| [65] Règle de pic par sortie | empirique | 2025-09 | 2028-09 | à jour |
| [10] VO2max Series 10 | validation | 2026-04 | 2029-04 | à jour |

**La première revérification utile, c'est maintenant.** Les quatre comportements d'intervals.icu tirés de vieux posts du forum ([39][41][64][97]) se vérifient en quelques minutes dans l'app, avant de coder. La prochaine échéance datée est **mars 2027** (seuil de course intervals.icu [108]). Un **Refresh** (mise à jour) ou un **Deepen** (approfondissement) de Deep Recon s'en charge.
