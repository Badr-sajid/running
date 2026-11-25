---
title: "PRD : application de course à pied personnelle"
status: final
created: 2026-11-20
updated: 2026-11-25
---

# PRD : application de course à pied personnelle
*Titre de travail, à confirmer.*

## 0. Objet du document

Ce PRD sert d'entrée à `bmad-ux`, puis à `bmad-architecture` et `bmad-create-epics-and-stories`. L'UX passe avant l'architecture, parce que le rituel après la Course pèse sur le modèle de données. Ce PRD s'appuie sur le [Product Brief du 18/11](../../briefs/brief-running-2026-11-18/brief.md), sur sa revue en party mode du 20/11, sur le brainstorming du 17/11 et, depuis le 25/11, sur la [recherche domaine « données Apple Watch et science de l'entraînement »](../../research/domain-donnees-apple-watch-entrainement-course-2026-11-25/research.md) (références entre crochets, ex. [65]), sans les répéter. Les termes du §2 (Glossaire) sont employés tels quels partout. Les exigences fonctionnelles (FR) sont numérotées globalement. Les arbitrages validés par Badr figurent aux §13 à §15 ; les points encore déduits sans confirmation portent un tag. Le « comment » technique (API, pipelines, hébergement, bandes chiffrées des Méthodes, vérifications en attente) se trouve dans [addendum.md](addendum.md).

## 1. Vision

Une application de course à pied, à usage personnel, qui remplace une routine d'environ 1 h 30 par plan : conversation avec une IA, traduction, puis saisie manuelle dans intervals.icu. Elle garde le **Profil coureur** en mémoire, génère et explique le **Plan** à partir des données réelles, puis le publie dans intervals.icu, d'où HealthFit l'envoie sur l'Apple Watch. Après chaque **Course**, elle donne un **Feedback** concret qui sépare l'avis de l'IA des données. Elle ne touche au Plan qu'en cas d'**Imprévu**, et toujours avec confirmation.

Le problème n'est pas la qualité des plans, qui sont déjà bons, mais le fait que chacun repart de zéro et passe par une saisie manuelle. La v1 ferme la boucle (générer, publier, courir, analyser) avec une barre claire : un plan au moins aussi bon qu'aujourd'hui, pour 10 min de manipulation au lieu d'environ 1 h 30. Chaque Plan est un chapitre d'une même histoire d'entraînement : il part de ce que le précédent a laissé.

C'est aussi un outil qu'on construit soi-même, et bien. Il sert d'abord un utilisateur unique, mais il ne doit pas fermer la porte à un partage avec des amis, voire à une publication si l'usage le justifie (voir NFR-6).

## 2. Glossaire

- **Profil coureur** : mémoire durable de l'utilisateur (historique, records, charge, CTL, FC au repos, durée de sommeil, Seuil d'allure, blessures actuelles et passées, disponibilités, jour de la sortie longue, périodes d'indisponibilité, Bilans de fin de Plan). Il contient un emplacement vide prévu pour la nutrition. Il y en a un par utilisateur.
- **Seuil d'allure** : la vitesse critique (CS), c'est-à-dire l'allure limite tenable sans épuisement rapide, estimée par l'application à partir des meilleurs efforts. Voir FR-23.
- **CTL** : la charge chronique d'entraînement calculée par intervals.icu (la « forme » au sens long terme).
- **Objectif** : ce que vise le Plan. Il y a un Objectif principal (un chrono sur une distance à une date) et zéro ou plusieurs Objectifs secondaires (ex. « ne pas se blesser »). Il peut viser une Épreuve.
- **Épreuve** : la compétition visée (distance, date). Ne pas confondre avec Course.
- **Méthode** : la doctrine d'entraînement choisie par l'application dans une liste fermée (FR-5).
- **Plan** : la suite de Séances générée pour un Objectif selon une Méthode. États : *proposé*, *actif* (accepté et publié), *suspendu*, *terminé*, *annulé*. Un seul Plan est actif ou suspendu à la fois.
- **Mode Reprise** : l'état d'un Plan généré ou repris après au moins 14 jours sans Course (FR-24).
- **Bilan de fin de Plan** : la synthèse déterministe d'un Plan terminé ou annulé, stockée dans le Profil coureur.
- **Consigne** : instruction en langage naturel qui accompagne un refus de Plan et guide sa régénération.
- **Séance** : un entraînement planifié à une date, composé de Blocs. États : *à venir*, *en attente de la course*, *réalisée*, *manquée*, *annulée*.
- **Bloc** : une étape d'une Séance (échauffement, fractionné, tempo, récupération…) avec une durée ou une distance et **une seule** cible (allure ou FC).
- **Publication** : l'écriture ou la mise à jour des Séances du Plan dans le calendrier intervals.icu, sans doublon. C'est la seule écriture vers intervals.icu.
- **Course** : une activité de course à pied réalisée, enregistrée par HealthFit et lue dans intervals.icu. Elle peut être liée à une Séance ou non planifiée.
- **Ressenti** : l'évaluation subjective d'une Course parmi 😌 facile, 😐 normal et 🥵 dur.
- **Conformité** : la comparaison entre le prévu et le réalisé, avec un verdict par Bloc (respecté ou non) et une note globale en pourcentage.
- **Feedback** : l'analyse d'une Course, en deux couches visuellement séparées : « Mon avis (IA) » et « Tes données ».
- **Matrice respect × ressenti** : la lecture combinée de la Conformité et du Ressenti, qui qualifie l'**état du jour**.
- **Ligne de base** : la valeur habituelle d'un signal (FC au repos, durée de sommeil) calculée sur au moins 28 jours, avec sa variabilité propre.
- **Seuil de surmenage** : la combinaison de signaux concordants qui déclenche une proposition d'Ajustement.
- **Imprévu** : un événement qui remet en cause le Plan (dernière minute, maladie, blessure, ou Seuil de surmenage atteint).
- **Ajustement** : une modification du Plan (annuler, déplacer, alléger, suspendre, reprendre, régénérer), proposée avec son « pourquoi » et appliquée après confirmation seulement.

## 3. Principes directeurs

Ces principes arbitrent les choix de l'UX, de l'architecture et des stories quand le PRD ne tranche pas.

1. **La confiance avant les fonctionnalités.** Une fonctionnalité fiable et crédible vaut mieux que deux fonctionnalités approximatives. Un chiffre faux ou une Publication ratée coûte plus cher qu'une fonctionnalité absente.
2. **Simple et efficace d'abord.** L'objectif d'apprentissage technique ne justifie jamais de la complexité.
3. **Pilote automatique transparent.** L'application décide et explique en une ligne ; l'utilisateur garde un droit de veto et ne fait pas le travail ; il peut toutefois reprendre la main sur une Séance (FR-22), sans jamais franchir les garde-fous de sécurité physique.
4. **Le LLM raconte, le code compte.** Tous les chiffres, verdicts et garde-fous sont déterministes ; le LLM génère et commente, sans jamais être la source d'un nombre.
5. **Chaque Plan part de l'état réel.** Le Profil, les Courses et le Bilan de fin de Plan précédent nourrissent toute génération ; on ne réexplique rien.
6. **intervals.icu est le pivot, le transport vers la montre est interchangeable.** HealthFit peut être remplacé sans toucher au reste du produit.

## 4. Utilisateur cible

### 4.1 Jobs To Be Done
- **Fonctionnel** : obtenir un Plan adapté à mon historique réel et le retrouver sur ma montre sans rien saisir.
- **Fonctionnel** : savoir après chaque Course si j'ai respecté la Séance et ce que cela dit de mon état, sans passer par Strava.
- **Contextuel** : quand un Imprévu survient (maladie, blessure, empêchement de dernière minute), adapter le Plan en deux taps au lieu de tout refaire.
- **Émotionnel** : faire confiance aux chiffres (aucun chiffre faux, aucune flatterie), garder mon propre jugement face aux données, et rester motivé par un avis réaliste.
- **Bâtisseur** : posséder et maîtriser mon propre outil plutôt que payer un abonnement de type Runna.

### 4.2 Non-utilisateurs (v1)
- Toute personne autre que Badr : pas de comptes ni d'inscription, pas de multi-utilisateur en v1.
- Les coureurs sans Apple Watch, HealthFit ou intervals.icu.

### 4.3 Parcours utilisateur clés
*Badr : coureur et développeur, 3 à 4 sorties par semaine, Apple Watch Series 10. Son prochain objectif (marathon début avril 2027 en moins de 3 h 30) peut changer et ne fixe pas le périmètre du produit.*

- **UJ-1. Badr configure son Profil coureur une seule fois.** À la première ouverture, l'application importe l'historique depuis intervals.icu et préremplit records, charge et CTL. Badr complète un formulaire (blessures, disponibilités, jour de la sortie longue, périodes d'indisponibilité), puis valide. Le Profil est complet ; la synchronisation initiale peut prendre du temps.
- **UJ-2. Badr crée un Plan pour son Objectif.** Il saisit son Objectif : un chrono sur une distance à une date (« semi en 1 h 40 le 12 avril »). L'application propose un Plan avec la Méthode retenue et sa raison en une ligne. Badr refuse une première fois avec une Consigne (« moins de volume en semaine 3 »), accepte la version régénérée, et les Séances apparaissent dans intervals.icu puis sur la montre. **Climax** : il n'a rien saisi.
- **UJ-3. Badr court, puis lit son Feedback.** Après sa Séance de fractionné, il ouvre l'application. Dès que la Course est récupérée depuis intervals.icu, l'application lui demande son Ressenti (😌 / 😐 / 🥵), avant de montrer le moindre chiffre. Il lit ensuite le verdict bloc par bloc, « Mon avis (IA) » (point fort, point à améliorer, conséquence pour la suite) et « Tes données » (1 chiffre principal, 2 secondaires). **Cas limite** : si la Course n'est pas encore remontée, la Séance du jour s'affiche « en attente de la course ».
- **UJ-4. Badr tombe malade.** Il touche « Empêchement », puis « Malade » (deux taps). Il répond à une question : fièvre ou symptômes sous le cou ? Il a de la fièvre : l'application propose d'annuler les Séances des 5 prochains jours, puis de reprendre par paliers, avec la raison en une ligne et les signaux qui doivent le faire consulter un médecin. Badr confirme d'un tap, et le Plan est republié. Avec un simple rhume, elle aurait proposé d'alléger ou d'annuler la Séance du jour.

## 5. Fonctionnalités

### 5.1 Profil coureur
**Description :** c'est la colonne vertébrale du produit. Le Profil est rempli à la première utilisation par import et par formulaire, puis enrichi par chaque Course et chaque Plan. On vise un Profil complet, pas un démarrage rapide. Il tient aussi à jour le Seuil d'allure (FR-23). Réalise UJ-1.

#### FR-1 : Import initial
Le système importe depuis intervals.icu l'historique des Courses, les records, la charge, la CTL, la FC au repos et la durée de sommeil. HealthFit n'est jamais lu directement : ses données arrivent uniquement via intervals.icu.
- La première synchronisation affiche sa progression ; elle peut durer plusieurs minutes sans être considérée comme une erreur.
- Pour le sommeil, seule la durée est lue ; elle sert de signal au Seuil de surmenage (FR-16) ; les stades et scores de sommeil Apple ne sont pas utilisés (NFR-10).

#### FR-2 : Complément et validation du Profil
L'utilisateur complète par formulaire ce que l'import ne fournit pas (blessures actuelles et passées, disponibilités par jour, jour de la sortie longue, périodes d'indisponibilité) et corrige les valeurs importées.
- Aucun Plan ne peut être généré tant que le Profil n'a pas été validé une fois.
- Le Profil reste modifiable ensuite.

#### FR-3 : Profil vivant
Le Profil intègre automatiquement chaque Course, son Ressenti et chaque Bilan de fin de Plan, pour que tout nouveau Plan parte de l'état réel sans rien réexpliquer.
- Le modèle de données comporte un emplacement nutrition, vide en v1.
- Le Profil est consultable : records, CTL, FC au repos, durée de sommeil, Seuil d'allure (avec sa date de mise à jour) et leurs tendances.

#### FR-23 : Maintien du Seuil d'allure
Le système réestime automatiquement le Seuil d'allure, à la hausse comme à la baisse, pour que les allures, les zones et la Conformité ne dérivent pas [105][108].
- Il est calculé sur les meilleurs efforts des 16 dernières semaines (règle de calcul dans l'addendum §2). Il est réestimé dès qu'une Course améliore l'un de ces meilleurs efforts, et réestimé aussi toutes les 4 à 6 semaines (paramètre) : il baisse quand les efforts qui le fondaient sortent de la fenêtre.
- En Mode Reprise (FR-24), il est réduit par le facteur de désentraînement, puis réestimé sur les Courses de la reprise.
- Le même Seuil d'allure sert aux cibles des Séances et à l'analyse des Courses.
- Un changement du Seuil d'allure n'est jamais appliqué en silence : il produit un Ajustement proposé (« allures de la semaine prochaine plus rapides de 4 s/km : nouveau meilleur 5 km »), appliqué aux Séances à venir après confirmation puis republié (FR-19). Refuser laisse les cibles inchangées ; l'analyse utilise le Seuil d'allure en vigueur à la date de chaque Course.
- Chaque changement est visible dans le Profil avec sa date et sa raison en une ligne.

### 5.2 Objectif et Plan
**Description :** l'utilisateur saisit un Objectif daté ; l'application choisit la Méthode et génère un Plan à partir du Profil. La relecture est binaire : on accepte le Plan entier, ou on le refuse avec une Consigne. Une fois accepté, le Plan reste modifiable (FR-22). Après un arrêt, le Plan repart en Mode Reprise (FR-24). Réalise UJ-2.

#### FR-4 : Saisie de l'Objectif
L'utilisateur saisit un Objectif principal (un chrono sur une distance, avec une date obligatoire : celle de l'Épreuve, ou une date cible) et des Objectifs secondaires en texte libre.
- Le Plan s'étend d'aujourd'hui à la date de l'Objectif ; une date à moins de 4 semaines est refusée.
- Si le chrono cible s'écarte de l'équivalence de performance calculée sur les records des 6 derniers mois, l'application avertit l'utilisateur en une ligne et indique le chrono jugé réaliste. Elle précise que les équivalences de ce type tendent à être optimistes [111] et que les coefficients VDOT eux-mêmes viennent d'une source faible [38]. L'utilisateur peut maintenir son chrono.
- Les Objectifs secondaires sont transmis comme contraintes à la génération et cités dans la raison du Plan quand ils l'influencent.

#### FR-5 : Génération du Plan
Le système génère un Plan à partir du Profil (y compris le dernier Bilan de fin de Plan) et de l'Objectif.
- **Méthode** : choisie par une règle déterministe, pas par le LLM. Par défaut, Daniels (répartition pyramidale) ; au-delà d'un volume hebdomadaire paramétré, polarisé. 80/20 est retiré de la liste en v1, faute de règle qui le distingue de Daniels. La raison du choix de la Méthode, en une ligne, dit honnêtement que les études montrent peu d'écart entre Méthodes [46][47].
- Garde-fous vérifiés de façon déterministe avant affichage, en trois familles (FR-22 en définit le comportement lors d'une modification) :
  - **sécurité physique** :
    - en distance, aucune Séance ne dépasse de plus de 10 % la plus longue des 30 jours précédents, parmi les Courses réalisées et les Séances à venir du Plan ; les Séances manquées ou annulées sont exclues (§15.16). La sortie longue suit le même plafond (paramètre, §15.2) ;
    - une semaine allégée revient toutes les 3 ou 4 semaines ;
    - la répartition des intensités respecte la bande chiffrée de la Méthode annoncée (bandes dans l'addendum §2) ;
    - quand l'Objectif vise une Épreuve, le Plan se termine par un affûtage : 2 semaines (3 pour un marathon), volume réduit dans la fourchette de l'addendum §2, intensité et fréquence maintenues [44][56] ;
    - les blessures du Profil sont respectées ;
  - **secondaire** : le volume hebdomadaire augmente de 10 % au plus d'une semaine à l'autre. La génération respecte toujours cette règle ; ce n'est pas une règle de sécurité (§15.1) ;
  - **préférences du Profil** : les disponibilités, le jour de la sortie longue et les périodes d'indisponibilité sont respectés.
- Les garde-fous qui imposent un minimum (bande d'intensités, intensité maintenue pendant l'affûtage, structure de la semaine allégée) ne sont vérifiés qu'à la génération. Un Ajustement déterministe qui réduit la charge (FR-19) ne déclenche donc jamais de régénération.
- Un Plan qui viole un garde-fou n'est jamais présenté : il est régénéré, 3 tentatives au plus. Après 3 échecs, l'application affiche un échec explicite qui nomme le garde-fou violé, propose de modifier l'Objectif et ne publie rien.
- La progression semaine par semaine est lisible avant l'acceptation.

#### FR-24 : Mode Reprise
Après au moins 14 jours sans Course (maladie, blessure, longue pause), le Plan généré ou repris (bouton « Reprendre », FR-17) passe en Mode Reprise.
- **Allures** : elles sont réduites par un facteur de désentraînement qui dépend de la durée de l'arrêt (facteurs de Daniels, paramètres [95]), via le Seuil d'allure (FR-23). Ce facteur ne s'applique qu'aux allures, jamais au volume.
- **Volume** : la première semaine ne dépasse pas 50 % du volume de la dernière semaine normale avant l'arrêt `[ASSUMPTION]` (paramètre). La première Séance ne dépasse pas la plus longue Séance de cette semaine.
- **Plafond par sortie** : il se calcule sur les seules Courses réalisées depuis la reprise, comme en FR-5 ; une Course d'avant l'arrêt n'ouvre jamais de droit à une sortie plus longue.
- Le Mode Reprise prend fin quand la fenêtre de 30 jours ne contient plus que des Courses de la reprise. La raison en une ligne dit que le Plan démarre en Mode Reprise et pourquoi.

#### FR-6 : Relecture (accepter, ou refuser avec Consigne)
L'utilisateur accepte le Plan entier, ou le refuse avec une Consigne qui déclenche une régénération (soumise aux mêmes garde-fous et à la même limite de tentatives).
- La relecture porte sur le Plan entier ; la modification d'une Séance se fait après acceptation (FR-22).

#### FR-7 : Cycle de vie du Plan
- Accepter un Plan le rend *actif* et le publie (FR-9).
- L'utilisateur peut créer un nouvel Objectif alors qu'un Plan est actif ou suspendu ; l'acceptation du nouveau Plan, après confirmation, *annule* l'ancien et retire ses Séances à venir d'intervals.icu.
- Un Plan passe à l'état *terminé* le lendemain de sa dernière Séance. L'accueil propose alors de définir un nouvel Objectif.

#### FR-8 : Bilan de fin de Plan
À la fin ou à l'annulation d'un Plan, le système calcule un Bilan de fin de Plan de façon déterministe : volume réalisé et volume prévu, Conformité moyenne, Séances manquées, évolution de la CTL et des records, répartition des Ressentis.
- Le Bilan est stocké dans le Profil coureur et lu par la génération du Plan suivant (FR-5).
- Il est consultable par l'utilisateur.
- Le volume prévu est celui du Plan après sa dernière modification ; une Séance déplacée n'est pas comptée comme manquée. Le Bilan ne dépend pas de « Mon avis (IA) ».

### 5.3 Publication et état de la synchro
**Description :** un Plan accepté est publié dans intervals.icu en Séances structurées ; HealthFit les transporte jusqu'à la montre. Les Publications sont rares et volontaires : elles n'ont lieu qu'à l'acceptation d'un Plan, à la confirmation d'un Ajustement, à l'enregistrement d'une modification de l'utilisateur (FR-22) ou quand l'utilisateur touche « resynchroniser ». Les lectures (Courses, FC au repos) ont lieu à chaque ouverture. Réalise UJ-2 et UJ-4.

#### FR-9 : Publication du Plan
Le système publie chaque Séance avec ses Blocs et ses cibles dans le calendrier intervals.icu.
- Chaque Bloc porte une seule cible. Les Blocs à allure spécifique (fractionné, tempo, seuil, allure de course) sont ciblés en allure ; tous les autres (échauffement, récupération, retour au calme, footing, sortie longue en endurance) sont ciblés en FC. Les fractionnés courts sont ciblés et jugés en allure, car, selon des tests de presse, la FC optique réagit en retard [112].
- Republier le même Plan ne crée aucun doublon.
- Une Séance relue depuis intervals.icu après Publication est identique à la Séance du Plan (Blocs, durées, cibles).

#### FR-10 : État de la synchro
L'utilisateur voit la dernière Séance publiée, la dernière Course reçue, un statut ✅ / ⚠️ et un bouton « resynchroniser ».
- Un échec de Publication est signalé dans cet écran avec sa cause, sans perte du Plan.

### 5.4 Feedback après chaque Course
**Description :** le Ressenti est demandé dès que la Course est récupérée, avant tout chiffre, pour que les données ne l'influencent pas. Le Feedback arrive ensuite, en deux couches : l'avis de l'IA, et des données séparées pour que l'utilisateur garde son propre jugement. Réalise UJ-3.

#### FR-11 : Détection et rattachement des Courses
Le système détecte les nouvelles Courses dans intervals.icu à l'ouverture de l'application et sur rafraîchissement manuel, sans notification. Il les rattache aux Séances.
- Une Course est rattachée à la Séance prévue le même jour. S'il y a plusieurs Courses ce jour-là, c'est la plus longue qui est rattachée ; les autres sont traitées comme non planifiées.
- Une Course courue la veille ou le lendemain d'une Séance non réalisée est proposée au rattachement ; l'utilisateur confirme d'un tap. La date de référence est la date actuelle de la Séance, après un éventuel déplacement (FR-22).
- Une même activité intervals.icu n'est jamais comptée deux fois.
- Tant qu'aucune Course n'est rattachée à la Séance du jour, celle-ci s'affiche « en attente de la course ».

#### FR-12 : Saisie du Ressenti
Dès qu'une Course est récupérée, l'utilisateur pose son Ressenti (une seule question, 3 emojis) avant tout affichage de chiffres.
- Le Feedback n'est affiché qu'une fois le Ressenti posé.
- La saisie est obligatoire : elle ne peut pas être ignorée ni reportée, et le Ressenti n'est plus modifiable une fois posé. Elle tient en un tap ; « Empêchement » (FR-17) est accessible juste après. Si plusieurs Courses attendent, les Ressentis sont demandés dans l'ordre chronologique.

#### FR-13 : Conformité bloc par bloc
Le système compare chaque Bloc prévu au réalisé et produit un verdict d'une ligne par Bloc ainsi qu'une note de Conformité globale.
- Un Bloc ciblé en allure est respecté si son allure moyenne ajustée à la pente (GAP) [104] est à ±5 s/km de la cible (tolérance arbitrée, non sourcée). Un Bloc ciblé en FC est respecté si au moins 80 % de sa durée se passe dans la zone cible.
- Pour un Bloc ciblé en FC de plus de 60 min (sortie longue), la Conformité est jugée sur ses deux premiers tiers (paramètre) : la dérive cardiaque de fin de sortie est physiologique [34]. Cette dérive est affichée à titre indicatif, sans pénaliser le verdict.
- La note globale est le pourcentage de Blocs respectés. Une Séance est « respectée » si cette note atteint au moins 80 %.
- La température de la Course est enregistrée et affichée comme contexte du verdict [106].
- Les tolérances sont des paramètres ajustables.
- Si le nombre de laps ne correspond pas au nombre de Blocs, ou si la Course n'était pas planifiée, le Feedback est global, sans verdict par Bloc, et l'application le dit.

#### FR-14 : Feedback en deux couches
Le système affiche « Mon avis (IA) » et « Tes données », visuellement séparés.
- « Mon avis (IA) » couvre trois niveaux (la Séance, son rôle dans le Plan, l'Objectif) et contient exactement 1 point fort, 1 point à améliorer et 1 conséquence pour la suite. Si la Séance n'est pas respectée, il dit ce qui a manqué et comment faire mieux.
- « Tes données » montre 1 chiffre principal et 2 secondaires, choisis par une règle déterministe selon le type de Séance (ex. fractionné : allure moyenne des Blocs rapides par rapport à la cible) ; le détail complet reste dans intervals.icu. Peuvent aussi être retenus, en plus des chiffres de Conformité, deux indicateurs de tendance : la FC à allure comparable [70] et le découplage allure / FC sur les sorties longues [102]. Les données exclues par NFR-10 ne sont jamais retenues.
- Chaque nombre cité dans « Mon avis (IA) » doit figurer dans les données calculées ; sinon, le texte est rejeté et régénéré, 3 tentatives au plus. Après 3 échecs, seuls « Tes données » et la Conformité s'affichent, avec la mention « avis indisponible ».
- En cas d'erreur d'analyse, l'utilisateur peut « Relancer » (nouveau cycle de 3 tentatives au plus) ou « Arrêter l'analyse » (état « avis indisponible »). 2 relances au plus par Feedback, uniquement tant que le Plan est actif. Chaque relance compte dans le coût mesuré (NFR-4).
- Aucun compliment sans chiffre qui le justifie.

### 5.5 Matrice respect × ressenti et surmenage
**Description :** la Matrice lit l'état du jour et alimente le Feedback. Elle ne modifie pas le Plan, sauf si le Seuil de surmenage est atteint : elle déclenche alors un Imprévu. Réalise UJ-3.

#### FR-15 : Lecture de l'état du jour
Le système qualifie chaque Course rattachée selon les 6 cases de la Matrice, et la lecture est transmise au Feedback comme un fait :

| | 😌 | 😐 | 🥵 |
|---|---|---|---|
| **Respectée** | marge, en progrès | dans le Plan | à la limite, à surveiller |
| **Non respectée** | Séance mal calibrée ou trop prudente | exécution à revoir | fatigue probable |

#### FR-16 : Seuil de surmenage
Le système propose un Ajustement quand un ressenti dégradé répété concorde avec au moins un signal objectif [31][70][76] :
- **ressenti** : 3 Courses 🥵 d'affilée (non planifiées comprises) ;
- **signaux objectifs** (un seul suffit) :
  - la FC au repos, en moyenne sur 7 jours, dépasse sa Ligne de base d'un écart exprimé en multiple de sa variabilité propre [70][114] ;
  - la durée de sommeil, en moyenne sur 7 jours, passe sous sa Ligne de base de la même façon.
- Quand les deux signaux objectifs concordent, la proposition est plus ferme (allègement plus marqué). Le ressenti dégradé reste toujours requis (§15.3).
- Sous le Seuil de surmenage, le Plan n'est jamais modifié. Tant que la Ligne de base n'a pas 28 jours de données, le signal concerné n'est pas utilisé, et l'application le dit.
- La proposition affiche son « pourquoi » en une ligne (ex. « allégé car 🥵 ×3 et FC au repos au-dessus de ta normale depuis 7 jours »). Elle cite l'écart mesuré, jamais le seuil comme s'il était un fait établi : les seuils n'ont pas de base numérique validée [64][70].
- Les valeurs du Seuil de surmenage sont des paramètres ajustables, calibrés par rétro-analyse des périodes de maladie et de fatigue de l'historique importé (FR-1) avant la mise en usage. Critère de fin du calibrage : Badr date les épisodes connus, et les paramètres retenus déclenchent sur au moins la majorité d'entre eux sans fausse alerte hors épisode `[ASSUMPTION]`. Faute d'épisodes datés, le squelette vertical tourne avec le détecteur en observation seule (il signale, sans proposer d'Ajustement).

### 5.6 Empêchements, Séances manquées et modifications
**Description :** c'est la gestion des Imprévus déclarés, des Séances non courues et des modifications à l'initiative de l'utilisateur. C'est un arbre de décision déterministe ; l'IA n'intervient que pour régénérer un Plan ou répondre à une demande de l'utilisateur (FR-22). Réalise UJ-4.

#### FR-17 : Déclarer un Imprévu
L'utilisateur déclare un Imprévu en deux taps (« Empêchement », puis le type) : **dernière minute**, **malade** ou **blessé**. Il répond ensuite à une question au plus, puis confirme la proposition d'un tap.
- *Dernière minute* : annuler ou déplacer la Séance.
- *Malade* : la question est « fièvre ou symptômes sous le cou (poitrine, courbatures généralisées, troubles digestifs) ? » [86][88].
  - Oui : les Séances sont annulées pendant au moins 5 jours, puis la reprise se fait par paliers d'au moins 48 h, sans entraînement normal avant environ J10 (paramètres).
  - Non (symptômes au-dessus du cou seulement) : la Séance du jour est allégée ou annulée ; reprise normale vers J5.
  - Dans les deux cas, l'application affiche les signaux d'alarme qui imposent l'arrêt et un avis médical (douleur thoracique, palpitations, essoufflement anormal).
  - Ces règles transposent un guide écrit pour le COVID [86] ; ce sont des paramètres prudents, pas un protocole validé pour toute maladie.
- *Blessé* : une seule question, à trois réponses [89][90] :
  - « Douleur osseuse, ou présente à la marche, ou qui modifie ma foulée, ou au-dessus de 5/10 » (un seul critère suffit) : le Plan est suspendu et l'application conseille de consulter un professionnel de santé ;
  - « Douleur légère (5/10 au plus), qui ne change pas ma foulée » : le Plan est adapté (charge réduite) ;
  - « Autre » : l'utilisateur choisit de suspendre ou d'annuler le Plan.
- Un Plan suspendu est réactivé uniquement par un bouton « Reprendre », que l'utilisateur touche lui-même ; après au moins 14 jours sans Course, la reprise passe en Mode Reprise (FR-24).
- Les messages restent non diagnostiques : l'application oriente, elle ne nomme ni ne soigne une pathologie.

#### FR-18 : Séance manquée
Une Séance dont la date est dépassée, sans Course rattachée ni Imprévu déclaré, passe à l'état *manquée* le lendemain.
- Le Plan ne bouge pas ; le Feedback suivant et le Bilan de fin de Plan en tiennent compte.
- Seuls un Imprévu (FR-17), le Seuil de surmenage (FR-16) ou une modification à l'initiative de l'utilisateur (FR-22) changent le Plan.

#### FR-19 : Ajustement confirmé
Tout Ajustement est proposé avec son « pourquoi » en une ligne et ne s'applique qu'après confirmation ; le Plan est ensuite republié (FR-9).
- Annuler, déplacer, alléger ou transformer en footing est une opération déterministe, sans LLM. Reprendre après une blessure ou régénérer passe par la génération (FR-5) ; « Demander à l'IA » (FR-22) passe par le LLM.
- Une régénération repart du Plan courant et respecte les préférences du Profil : les modifications antérieures de l'utilisateur sur les Séances régénérées ne sont pas conservées, et le « pourquoi » le dit (« ton déplacement du 4 déc. n'est pas conservé »).
- Tout Plan modifié repasse par les garde-fous de FR-5 avant d'être proposé.
- Refuser la proposition laisse le Plan inchangé.

#### FR-22 : Modification du Plan par l'utilisateur
Depuis le Plan actif, l'utilisateur modifie une Séance à venir de trois façons : la **déplacer** à un autre jour, la **modifier à la main** (distance, allure, type), ou **demander à l'IA** en une phrase (« décale ma sortie longue à dimanche », « réduis la semaine 6 »).
- **Séances modifiables** : seulement les Séances à venir sans Course rattachée. Une Séance passée ou rattachée est verrouillée.
- **Déplacer** : le jour cible est postérieur à aujourd'hui, antérieur à la date de l'Objectif et libre ; deux Séances ne tombent jamais le même jour.
- **Modifier à la main** : changer le type reconstruit les Blocs à partir d'un modèle déterministe par type, avec la règle de ciblage allure / FC de FR-9.
- **Demander à l'IA** : une demande porte sur une semaine au plus ; au-delà (plusieurs semaines, ajout ou suppression de Séance, changement d'Objectif, demande ambiguë), l'application répond que ce n'est pas pris en charge et ne change rien. Chaque Séance touchée est vérifiée. La proposition affiche son « pourquoi » en une ligne et respecte la limite de 3 tentatives (FR-5) ; après 3 échecs, l'application nomme le garde-fou, ne change rien et propose la modification à la main. Elle est confirmée ou refusée comme un Ajustement (FR-19). Ce n'est pas une conversation : le dialogue avec l'IA est prévu en v1.2 (§9.2).
- **Vérification** : après toute modification, les garde-fous de FR-5 sont revérifiés de façon déterministe sur toutes les Séances restantes du Plan (semaine de départ et semaine d'arrivée comprises pour un déplacement). Ils forment trois familles (FR-5) :
  - **sécurité physique** : une modification qui augmente le risque, c'est-à-dire qui augmente la distance d'une Séance, l'intensité d'un Bloc ou le volume d'une semaine, et qui franchit un plafond est **refusée**, sans exception. L'application nomme le garde-fou (« Sortie longue à 24 km : +20 % au lieu de 10 % max ») et rien n'est enregistré ni publié. Une modification qui réduit la charge (distance, intensité, volume) n'est jamais refusée, même si elle s'écarte d'un garde-fou à minimum (affûtage, bande d'intensités) ; un déplacement est vérifié contre les plafonds de sa nouvelle date ;
  - **secondaire** : une modification qui dépasse la règle du volume hebdomadaire est enregistrée, avec un avertissement en une ligne qui nomme l'écart (« volume de la semaine 5 : +14 % par rapport à la semaine 4 ») ;
  - **préférences du Profil** : tout changement demandé par l'utilisateur (modification à la main, demande à l'IA, « déplacer » de FR-17) peut s'en écarter ponctuellement, sans changer le Profil. Les Plans générés par l'application les respectent toujours.
- **Après un Ajustement de surmenage** (FR-16) : pendant 7 jours, toute hausse manuelle d'une Séance allégée est refusée.
- **Publication** : toute modification enregistrée est republiée (FR-9) ; un déplacement retire la Séance de son ancienne date dans intervals.icu. Si la republication échoue, la modification est conservée et l'état de la synchro affiche ⚠️ (FR-10).
- **Concurrence** : une proposition en attente (FR-16, FR-17, demande à l'IA) est revérifiée contre le Plan courant au moment de sa confirmation.
- **Comptage** : les modifications de l'utilisateur, y compris les propositions de l'IA qu'il a demandées, sont comptées à part des Ajustements (SM-C1).

### 5.7 Accueil et Séances
**Description :** l'accueil est minimaliste, sans aucune notification. Il montre la Séance du jour dès l'ouverture ; chaque Course et son Feedback sont à quelques taps. La présentation des Séances vise la clarté de Runna. Réalise UJ-2 et UJ-3.

#### FR-20 : Séance du jour
L'accueil affiche une carte unique « Séance du jour » (son but, sa durée, son état) et un accès au Plan, aux Feedbacks récents et à l'état de la synchro.
- Sans Plan actif, l'accueil propose de définir un Objectif.

#### FR-21 : Détail d'une Séance
L'utilisateur consulte le détail de toute Séance : son but dans le Plan (en une ligne), les sensations attendues (ex. « tu dois pouvoir parler en phrases complètes ») [59] et la liste de ses Blocs avec leur durée ou leur distance et leur cible.

## 6. Exigences non fonctionnelles transverses

- **NFR-1 Plateforme** : application web mobile-first, utilisée sur iPhone en PWA. Elle tourne d'abord en local sur le Mac, puis est auto-hébergée. Pas d'app iOS native.
- **NFR-2 Confidentialité** : aucune donnée n'est envoyée à un tiers autre qu'intervals.icu et le fournisseur du LLM. Rien n'est exposé publiquement sur Internet, et les clés (intervals.icu, LLM) ne sont jamais présentes côté client.
- **NFR-3 Exactitude** : tout chiffre affiché est recalculable de façon déterministe à partir des données d'intervals.icu.
- **NFR-4 Coût** : moins de 10 € par mois d'appels IA, sur une base d'environ 1 Plan, 15 Feedbacks et 5 demandes à l'IA (FR-22) par mois. Le coût réel est mesuré et consultable.
- **NFR-5 Explicabilité** : chaque décision de l'application (Méthode, Plan, Ajustement) est accompagnée de sa raison en une ligne.
- **NFR-6 Porte ouverte au partage** : la v1 est mono-utilisateur, sans comptes ni authentification. Rien ne doit pour autant empêcher d'ajouter plus tard plusieurs Profils coureur et leurs identifiants intervals.icu (données rattachées à un utilisateur, aucun identifiant figé dans le code).
- **NFR-7 Dépendances** : intervals.icu et HealthFit sont des tiers non maîtrisés. HealthFit, avec l'achat intégré « Fanatical Supporter », est la seule source des Courses et le transport vers la montre ; il faut ouvrir HealthFit pour que la montre reçoive les Séances, ce qui est accepté en v1. Strava n'est pas connecté à intervals.icu. Les quotas d'API sont respectés, et les erreurs restent visibles dans l'état de la synchro (FR-10).
- **NFR-8 Solidité** : pas d'échéance ; on préfère livrer moins mais stable. Aucune version instable n'est mise en usage. Les règles de qualité de l'addendum §4 (tests, tests de contrat, recalcul des chiffres) sont obligatoires.
- **NFR-9 Temps de réponse** : un Feedback s'affiche en moins de 30 s après la pose du Ressenti, et un Plan en moins de 2 min, avec un état d'attente visible.
- **NFR-10 Hiérarchie des données** : seules les données fiables fondent une décision : allure GPS (fiable à l'usage, sans validation indépendante publiée), FC en effort stable [12], FC au repos [72] et durée de sommeil [6][21]. La puissance, la VO2max, la VFC, la dynamique de foulée, les stades et les scores de sommeil Apple ne sont jamais une cible, une source de charge, un signal du Seuil de surmenage ni la base d'un verdict ; ils peuvent seulement être affichés à titre descriptif [8][10][14][72]. La CTL reste descriptive et ne sert jamais de base à une décision, a fortiori si intervals.icu la calcule à partir de la puissance (§12 Q4).

## 7. Ton et voix

Motivant et réaliste, jamais flatteur. Concret : chaque affirmation cite une donnée mesurée ou ressentie (« les 3 derniers fractionnés à 4'12 au lieu de 4'20 prévus »). Clair et compréhensible immédiatement, sans jargon non expliqué. Les données restent digestes pour que l'utilisateur se forge son propre avis. La référence de clarté est Runna (accueil, présentation des Séances, commentaires). Pas d'intrusion : aucune notification, quasiment aucune manipulation au quotidien.

## 8. Non-objectifs

- Ce n'est pas un éditeur de séances libre : les modifications de l'utilisateur (FR-22) ne franchissent jamais les garde-fous de sécurité physique.
- Ce n'est pas un réajustement continu : le Plan ne bouge pas après chaque Course, seulement sur Imprévu, au Seuil de surmenage ou à la demande de l'utilisateur.
- Ce n'est pas un tableau de bord d'analyse : le détail des données reste dans intervals.icu.
- Pas de Strava ni de lecture directe de HealthKit ou WorkoutKit.
- Pas de choix manuel de la Méthode, ni de version « minimum vital » des Séances.
- Pas de diagnostic médical : face à une maladie ou une douleur, l'application oriente vers un professionnel sans nommer de pathologie (FR-17). Le dépistage du déficit énergétique relatif dans le sport (REDs) est hors périmètre [96].
- Pas un produit commercial en v1 : pas de comptes, de paiement ni de conformité réglementaire au-delà des données de l'utilisateur.

## 9. Périmètre MVP

### 9.1 Inclus
FR-1 à FR-24, soit l'ensemble du §5.

Les préalables du §12 sont levés avant de coder les stories concernées.

Ordre de livraison : un squelette vertical d'abord. Un Plan de 4 semaines, pour un Objectif, est publié, couru et commenté de bout en bout, avec l'expérience après la Course (FR-11 à FR-14) dès ce squelette, avant d'élargir.

### 9.2 Hors MVP
- **v1.1 (motivation et lisibilité)** : jauge de faisabilité continue, alerte « pas assez ambitieux », prédiction de chrono en fourchette, rappel du « vrai pourquoi », échauffements et étirements par Séance, vue « escalier », trois notifications ciblées.
- **v1.2 (continuité et dialogue)** : saison de Plans enchaînés (préparation, Épreuve, récupération, préparation suivante), conversation IA rattachée au Feedback, retour de blessure progressif, signaux de surmenage supplémentaires (douleurs, jambes lourdes, motivation, effort perçu à charge égale) avec un nouveau recalibrage du Seuil.
- **v2** : les cinq autres types d'Objectif (distance, durée, récupération après une Épreuve, condition générale, reprise après blessure) ; un transport vers la montre qui ne demande plus d'ouvrir HealthFit ; renforcement ; empêchements « moins de temps » et « voyage » ; carnet de blessures ; chaussures ; parcours ; graphiques enrichis.
- **v3 et au-delà** : nutrition ; fonctionnalité sommeil dédiée (la durée est lue dès la v1, FR-16) ; calendrier ; météo (la température est enregistrée dès la v1, FR-13) ; partage ou publication à d'autres coureurs.

> `[NOTE FOR PM]` La saison de Plans enchaînés porte le « vrai moteur » du projet : voir §12.

## 10. Critères de succès

La v1 est **finie** quand SM-1 à SM-6 passent en conditions réelles.

- **SM-1 Boucle fermée** : un Plan complet est généré, publié, couru et commenté sans aucune saisie manuelle dans intervals.icu. Valide FR-5 à FR-14.
- **SM-2 Fin de l'ancienne routine** : sur la durée d'un Plan complet, aucun recours à une IA externe ni à Strava pour planifier ou analyser. Valide FR-5, FR-14.
- **SM-3 Temps** : un Plan suivant demande 10 min de manipulation active au plus (contre environ 1 h 30) ; le temps de lecture n'est pas compté. Valide FR-4 à FR-9.
- **SM-4 Coût** : moins de 10 € par mois d'appels IA, mesuré sur la durée d'un Plan. Valide NFR-4.
- **SM-5 Crédibilité** : aucun chiffre faux dans les Feedbacks, ce que garantit le contrôle de FR-14. Test de qualité : pour un même Objectif, un Plan de l'application et un plan de l'ancienne méthode sont comparés à l'aveugle ; celui de l'application doit paraître au moins aussi crédible (protocole : addendum §6).
- **SM-6 Continuité** : le Plan 2 part réellement des données et du Bilan de fin de Plan du Plan 1 ; sa raison en une ligne y fait référence. Valide FR-3, FR-8.

**Critères secondaires (non bloquants)**
- **SM-7 Fiabilité de la synchro** : aucune Séance publiée n'est absente de la montre sans que l'état de la synchro le signale. Valide FR-9, FR-10.
- **SM-8 Usage quotidien quasi nul** : un jour de course, hors lecture, la manipulation se limite à ouvrir HealthFit, ouvrir l'application et poser un Ressenti. Valide FR-11, FR-12, FR-20.

**Contre-métriques (à ne pas optimiser)**
- **SM-C1 Nombre d'Ajustements** : un Plan qui bouge souvent est un échec de la promesse de stabilité, même si chaque Ajustement est justifié. Les modifications de l'utilisateur (FR-22) sont comptées à part : elles mesurent l'écart entre le Plan proposé et la vie réelle, un signal sur la qualité des Plans. Contrebalance FR-16, FR-22.
- **SM-C2 Richesse du Feedback** : plus de chiffres ou plus de texte ne veut pas dire mieux ; le format (1 point fort, 1 point à améliorer, 1 conséquence ; 1 chiffre principal, 2 secondaires) est un plafond. Contrebalance FR-14.
- **SM-C3 Vitesse de configuration** : on ne raccourcit pas UJ-1 au détriment de la complétude du Profil. Contrebalance SM-3.

## 11. Risques

- **Qualité du Plan** : un Plan généré moins crédible que l'ancienne méthode invalide la v1. Parade : les garde-fous de FR-5 et le test à l'aveugle (SM-5).
- **Crédibilité de l'IA** : des chiffres inventés ou de la flatterie détruisent la confiance. Parade : le validateur de FR-14 et le principe « le LLM raconte, le code compte ».
- **Coût réel de l'IA** : il n'est pas encore mesuré. Parade : la mesure de NFR-4 et la limite de tentatives (FR-5, FR-14).
- **Dépendances tierces** : intervals.icu (API, quotas) et HealthFit (bêta, achat intégré) ne sont pas maîtrisés. Parade : l'état de la synchro, le transport interchangeable (principe 6) et le plan B Watchletic (addendum §3).
- **Calibrage des seuils** : les seuils d'alerte (surmenage, FC au repos, sommeil) n'ont aucune base numérique validée [64][70], et la Conformité est fixée sans données d'usage. Parade : des paramètres ajustables, calibrés (FR-16) par rétro-analyse des périodes de maladie et de fatigue de l'historique importé (FR-1) **avant la mise en usage**, puis ajustés à l'usage. Le critère de fin du calibrage et le mode observation seule sont dans FR-16.
- **Seuil d'allure périmé** : un seuil non tenu à jour fausse ensemble les allures, la charge et la Conformité. Parade : FR-23, la vérification du comportement d'intervals.icu (Q4) et la décision Q10.
- **Données Apple trompeuses** : les mesures les plus riches (VO2max, VFC, puissance) sont les moins fiables. Parade : NFR-10.

## 12. Questions ouvertes et points en attente

Seule Q5 demande une mise à jour de l'UX. **Préalables au code** : Q4, Q9, Q10 et Q11 sont levées pendant l'architecture, avant de coder les stories concernées.

- **Q1.** **Vérification sur la montre** : WorkoutKit n'accepte-t-il qu'une cible par Bloc ? (addendum §5)
- **Q2.** **Seuil d'écart du chrono irréaliste** : à fixer en architecture (addendum §2).
- **Q3.** **Bandes de répartition des Méthodes** : à valider en architecture (addendum §2), y compris le volume de bascule de Daniels vers polarisé (FR-5).
- **Q4.** **Mécanismes intervals.icu** : à vérifier en architecture (addendum §1 et §5) : Publication, quotas, mise à jour du seuil d'allure propre à intervals.icu [108] (à revérifier en mars 2027), formule de charge et sa source pour une Course (allure, FC ou puissance), Form, bandes de conformité, mise à disposition de la GAP et de la température.
- **Q5.** **Table type de Séance → chiffres de « Tes données »** : résolue en UX (`ux-designs/ux-running-2026-11-21/EXPERIENCE.md`), mais à mettre à jour pour intégrer la FC à allure comparable et le découplage (FR-14, 25/11), ainsi que les sensations attendues (FR-21). Libellés de la Matrice provisoires, ajustables à l'usage.
- **Q6.** **Saison de Plans enchaînés** : elle porte le « vrai moteur » du projet (la continuité), et la v1 n'en livre que la mémoire et le Bilan de fin de Plan (FR-3, FR-8). À avancer si la v1 se termine vite.
- **Q7.** **Semaine allégée chiffrée** : définir numériquement une semaine allégée (ex. volume ≤ X % de la semaine précédente) pour que FR-5 et FR-22 la vérifient de façon déterministe. À fixer en architecture.
- **Q8.** **Blessures testables** : traduire les blessures du Profil en contraintes vérifiables (types de Séance exclus, distance maximale) pour le garde-fou de FR-5 / FR-22. À fixer en architecture.
- **Q9.** **Contenu du FIT exporté par HealthFit** : fréquence de la FC, tours, puissance. Aucune documentation n'existe [15][19] ; c'est un test d'une soirée sur une vraie Course, avant les stories Feedback (addendum §5).
- **Q10.** **Écriture du Seuil d'allure dans intervals.icu** : si Q4 confirme qu'intervals.icu ne met pas à jour son propre seuil d'allure [108], faut-il une deuxième écriture (réglages de sport), contraire au glossaire « Publication = seule écriture », ou une mise à jour manuelle ? À trancher en architecture.
- **Q11.** **Durée de sommeil disponible** : vérifier qu'elle arrive bien dans intervals.icu via HealthFit (wellness), condition nécessaire au signal sommeil de FR-16.

## 13. Arbitrages validés (20/11)

Les contradictions entre le brainstorming, le brief et la revue party mode ont été tranchées avec Badr :
1. Un seul type d'Objectif en v1 : « chrono sur une distance » ; les cinq autres types passent en v2. → FR-4, §9.2
2. Livraison en squelette vertical (Plan de 4 semaines), avec l'expérience après la Course dès le début. → §9.1
3. Alerte si le chrono cible est irréaliste (avancée depuis la v1.1). → FR-4
4. Pas de multi-utilisateur en v1, mais une porte ouverte au partage. → NFR-6
5. Ressenti demandé **dès la récupération de la Course**, avant les chiffres. Il n'est ni saisi à chaud avant l'arrivée de la Course (recommandation de la revue party mode) ni reporté à une ouverture ultérieure. → FR-12
6. Détection des Courses à l'ouverture et sur rafraîchissement, sans notification. → FR-11
7. Feedback global si les laps ne correspondent pas aux Blocs, ou si la Course n'était pas planifiée. → FR-13
8. Accueil : une carte unique « Séance du jour ». → FR-20
9. Seuil de surmenage, version initiale. *Remplacé par §15.3.*
10. HealthFit n'est lu que via intervals.icu. → FR-1
11. Critère de continuité SM-6 ajouté et bloquant, appuyé sur un Bilan de fin de Plan minimal dès la v1 (FR-8). → FR-8, SM-6
12. Fiabilité de la synchro en critère secondaire, non bloquant (SM-7). → SM-7
13. Une seule cible par Bloc : allure pour les Blocs à allure spécifique, FC pour les autres. → FR-9
14. Garde-fou ajouté : aucune Séance ne dépasse de plus de 10 % la plus longue Course des 30 jours précédents. → FR-5, précisé par l'arbitrage 30
15. Devoir ouvrir HealthFit est accepté ; un remplacement éventuel sera une future fonctionnalité. → NFR-7, §9.2
16. Un Plan suspendu se réactive par un bouton « Reprendre ». → FR-17
17. « Tes données » : 1 chiffre principal et 2 secondaires (et non 3 à 5). → FR-14
18. Conformité : ±5 s/km pour l'allure, 80 % du temps dans la zone pour la FC, note globale en pourcentage de Blocs respectés. → FR-13 *(précisé par §15.5)*
19. Une Séance manquée sans Imprévu ne déclenche aucun Ajustement. → FR-18
20. L'Objectif a une date obligatoire, et un Plan dure au moins 4 semaines. → FR-4
21. 3 tentatives de régénération au plus, puis un échec explicite. → FR-5, FR-14
22. Chrono irréaliste : estimation par équivalence de performance (type VDOT) sur les records des 6 derniers mois ; avertissement non bloquant. → FR-4
23. Objectifs secondaires transmis comme contraintes à la génération et cités dans la raison du Plan. → FR-4
24. Course courue la veille ou le lendemain d'une Séance non réalisée : rattachement proposé, confirmé d'un tap. → FR-11
25. Séance « respectée » si au moins 80 % de ses Blocs le sont. → FR-13
26. Les chiffres de « Tes données » dépendent du type de Séance ; la table est fixée en UX. → FR-14
27. Les 6 lectures de la Matrice sont reprises telles quelles et seront affinées en UX. → FR-15
28. Temps de réponse : Feedback en moins de 30 s, Plan en moins de 2 min. → NFR-9
29. Choix de la Méthode, version initiale. *Remplacé par §15.9.*
30. Garde-fou des 10 % mesuré en distance, sur une fenêtre glissante qui inclut les Séances planifiées. → FR-5 *(précisé par §15.16)*
31. Accepter un nouveau Plan annule le Plan actif, après confirmation, et retire ses Séances à venir. → FR-7

## 14. Arbitrages validés (22/11, après l'UX)

Écarts entre les spécifications UX (`ux-designs/ux-running-2026-11-21/`) et le PRD, tranchés avec Badr :
1. Le Plan accepté est modifiable par l'utilisateur : déplacer, modifier à la main ou demander à l'IA. → FR-22, FR-6, §8, principe 3
2. Une modification manuelle qui franchit un garde-fou de sécurité physique est refusée, sans contournement. Les préférences du Profil (disponibilités, jour de la sortie longue) peuvent être contournées ponctuellement par l'utilisateur. → FR-22 *(précisé par §15.1 et §15.2)*
3. « Demander à l'IA » est un troisième déclencheur de changement du Plan, à l'initiative de l'utilisateur, soumis aux garde-fous et confirmé. → FR-18, FR-22
4. Les modifications de l'utilisateur sont comptées à part des Ajustements. → SM-C1
5. Le Ressenti est obligatoire et définitif. → FR-12
6. Après une erreur d'analyse : « Relancer » ou « Arrêter l'analyse ». → FR-14
7. Le Profil coureur est consultable (records, CTL, FC au repos, tendances). → FR-3
8. Relecture des cas limites de FR-22 (`review-edge-case.md`) : Séances passées ou rattachées verrouillées ; une réduction de charge n'est jamais refusée ; tout changement demandé par l'utilisateur (y compris via l'IA et FR-17) peut s'écarter des préférences ; une demande à l'IA couvre une semaine au plus ; une régénération ne conserve pas les modifications antérieures et le dit ; 2 relances au plus par Feedback ; 7 jours sans hausse manuelle après un allègement de surmenage. → FR-22, FR-19, FR-14, FR-8, FR-11, FR-12, NFR-4, §12

## 15. Arbitrages validés (25/11, après la recherche domaine)

Écarts entre la [recherche du 25/11](../../research/domain-donnees-apple-watch-entrainement-course-2026-11-25/research.md) et le PRD (détail : `reconcile-research-2026-11-25.md`), tranchés avec Badr selon la règle « la prudence d'abord, les preuves ensuite ». Ils remplacent les arbitrages 9 et 29 du §13, et précisent les arbitrages 18 et 30 du §13 ainsi que l'arbitrage 2 du §14.
1. La règle du volume hebdomadaire (+10 %) passe de la sécurité physique à un garde-fou secondaire : respectée à la génération, dépassable avec avertissement lors d'une modification [65][84]. → FR-5, FR-22
2. Le plafond par sortie (+10 %) reste en sécurité physique, sans exception ; il s'applique aussi à la sortie longue, sans plafond plus strict. → FR-5, FR-22
3. Seuil de surmenage : 3 🥵 d'affilée et au moins un signal objectif (FC au repos ou durée de sommeil, moyenne de 7 jours comparée à une Ligne de base de 28 jours au moins). Remplace l'arbitrage 9. → FR-16
4. La durée de sommeil est lue dès la v1. → FR-1, FR-3, §9.2
5. Sortie longue ciblée en FC : Conformité jugée sur les deux premiers tiers ; allure comparée en GAP ; température enregistrée. Précise l'arbitrage 18. → FR-13
6. Imprévus « Malade » et « Blessé » : protocole sur plusieurs jours, renvoi vers un professionnel sur signaux d'alarme, messages non diagnostiques. → FR-17, UJ-4, §8
7. Mode Reprise après au moins 14 jours sans Course : facteur de Daniels sur les allures seulement, volume repris à 50 % au plus, plafond par sortie calculé sur les seules Courses de la reprise. → FR-24
8. Hiérarchie des données : puissance, VO2max, VFC, dynamique de foulée, stades et scores de sommeil Apple ne fondent jamais une décision. → NFR-10
9. Méthode choisie par une règle déterministe (Daniels pyramidale par défaut, polarisé au-delà d'un volume), avec une raison honnête ; 80/20 retiré de la liste v1. Remplace l'arbitrage 29. → FR-5, glossaire
10. Seuil d'allure maintenu automatiquement par l'application ; son écriture dans intervals.icu reste à trancher. → FR-23, §12 Q10
11. Chrono irréaliste : VDOT conservée, avertissement signalant l'optimisme de l'estimation. → FR-4
12. « Tes données » garde le format 1 + 2 ; FC à allure comparable et découplage deviennent éligibles. → FR-14
13. Sensations attendues dans le détail d'une Séance. → FR-21
14. Affûtage obligatoire quand l'Objectif vise une Épreuve. → FR-5
15. Vérifications empiriques avant de coder (FIT HealthFit, comportements d'intervals.icu, sommeil) et calibrage des seuils sur l'historique avant la mise en usage, avec un critère de fin. → §12, §11
16. Relecture finale : la fenêtre des 30 jours exclut les Séances manquées ou annulées ; « augmente le risque » est défini ; les garde-fous à minimum ne sont vérifiés qu'à la génération ; un changement du Seuil d'allure est un Ajustement confirmé ; « Blessé » a une question à trois réponses. Précise l'arbitrage 30 du §13. → FR-5, FR-17, FR-22, FR-23

