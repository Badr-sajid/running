# Roadmap : application de course à pied personnelle

> Source : session de brainstorming du 2026-11-17 (MoSCoW validé + ordre des fonctionnalités donné par l'utilisateur).
> Principe directeur : **pilote automatique transparent**. L'appli décide (méthode, plan, ajustements), dit toujours pourquoi, et le coureur garde un droit de veto (confirmation).
> La confiance est le produit : synchro fiable, feedback appuyé sur des données, un « pourquoi » en une ligne, des prédictions en fourchette.

---

## Vue d'ensemble

```
  v1 (MUST)            v1.1 (SHOULD)          v1.2 (SHOULD)          v2 (COULD)             v3+ (WON'T pour l'instant)
  Boucle fermée        Motivation &           Continuité &           Corps & contexte       Hors course pure
                       lisibilité             dialogue
 ─────────────────┬────────────────────┬────────────────────┬────────────────────┬──────────────────────────────►
  profil vivant    │ jauge faisabilité  │ retour de blessure │ renfo              │ nutrition
  import données   │ prédiction fourch. │ conversation IA    │ moins de temps /   │ sommeil
  objectif         │ rappel vrai pourquoi│ mémoire profil    │   voyage           │ calendrier
  plan             │ échauffements /    │ saison de plans    │ carnet blessures   │ météo
  synchro A/R      │   étirements       │   enchaînés        │ rapport kiné       │ objectifs maintien /
  feedback         │ vue escalier       │                    │ chaussures, parcours│  trail / régularité
  réajustement     │ 3 notifications    │                    │ journal, simulation│
  empêchement      │                    │                    │ tableau de bord    │
  accueil          │                    │                    │                    │
        ▲                   ▲                     ▲                    ▲                     ▲
        └── colonne vertébrale : profil vivant (places réservées pour nutrition et sommeil dès la v1)
```

Ordre de l'utilisateur pour les fonctionnalités « en plus » : **1) échauffements/étirements (v1.1) → 2) renfo (v2) → 3) nutrition, sommeil, calendrier, météo (v3+)**.

---

## v1 : la boucle fermée (MUST)

**Thème :** générer → synchroniser → courir → remonter → analyser → réajuster, sans aucune saisie manuelle dans intervals.icu ni passage par Strava.

### Étapes de construction (dans l'ordre)

1. **Profil vivant + import intervals.icu / HealthFit** (la colonne vertébrale)
   - Historique, records personnels, charge actuelle, CTL/forme, blessures (actuelles et passées), disponibilités, jour de sortie longue, vacances.
   - Pré-remplissage depuis intervals.icu/HealthFit : le coureur valide ou corrige.
   - Modèle de données avec **des places réservées pour la nutrition et le sommeil** (vides en v1), à brancher plus tard sans refonte.
   - Les données ne servent qu'à la course à pied.
2. **Création d'objectif**
   - 6 types : distance, durée, chrono sur une distance, récupérer après une course, condition physique générale, reprise après blessure.
   - Objectif principal + objectifs secondaires (ex. 10 km en 45 min + ne pas se blesser).
3. **Génération du plan**
   - À partir des données réelles, pas d'une conversation.
   - Progressif d'une semaine à l'autre, chrono cible réaliste.
   - Méthode choisie par l'appli (80/20, Daniels, polarisé…), affichée en une ligne avec sa raison.
4. **Synchro automatique aller/retour**
   - Aller : écriture du plan dans le calendrier intervals.icu via l'API → montre (intervals.icu sert de tuyau invisible).
   - Retour : courses remontées depuis la montre (HealthFit / intervals.icu).
   - Écran « état de la synchro » : dernière séance envoyée, dernière course reçue, statut ✅/⚠️, bouton « resynchroniser ».
5. **Feedback après chaque course**
   - Popup ressenti **pendant le chargement**, **avant** les chiffres : une seule question, 3 emojis (😌 facile / 😐 normal / 🥵 dur).
   - Respect prévu vs réalisé bloc par bloc (échauffement, chaque fractionné, récup) + note de conformité globale.
   - 3 niveaux : la séance, l'objectif de la séance, l'objectif global ; rôle de la séance dans le plan.
   - Structure : 1 point fort factuel, 1 point à améliorer, 1 conséquence pour la suite.
   - Chaque affirmation cite une donnée mesurée ou ressentie ; pas de compliment générique. Ton motivant **et** réaliste.
   - Deux couches visuellement séparées : « Mon avis (IA) » et « Tes données » (3 à 5 indicateurs) ; le détail complet reste dans intervals.icu.
   - Séance ratée : ce qui n'a pas été fait et comment faire mieux la prochaine fois.
6. **Réajustement (matrice respect × ressenti)**
   - « Respectée + 🥵 » et « pas respectée + 😌 » donnent deux lectures différentes de la forme.
   - Cohérent avec les séances précédentes et l'objectif final ; chaque séance part de l'état réel du jour.
   - Aucune modification silencieuse : proposition, « pourquoi » en une ligne, puis confirmation.
7. **Bouton Empêchement**
   - Dernière minute : annuler ou déplacer la séance.
   - Malade : règle au-dessus/en dessous du cou (rhume léger = footing possible, fièvre = repos).
   - Blessé : suspendre ou annuler le plan.
   - L'appli évalue s'il faut refaire le plan ou seulement annuler une ou plusieurs séances.
8. **Accueil minimaliste**
   - Le plan (séance du jour) s'affiche dès l'ouverture ; le feedback de chaque course est à quelques clics.
   - Zéro notification.
   - Inspiré de la clarté de Runna (accueil, présentation des séances).

### Definition of done (côté coureur)
- Je configure mon profil et mon objectif une seule fois, pré-remplis depuis mes données ; je ne fais que valider.
- Mon plan apparaît sur l'Apple Watch sans que je touche à intervals.icu.
- Après ma course, je réponds en un emoji et je reçois un feedback clair, chiffré et crédible, sans ouvrir Strava.
- Quand le plan change, je vois pourquoi et je confirme.
- Un imprévu, une maladie ou une blessure se gère en deux taps.
- Au quotidien, je n'ai presque rien à faire.

### Dépendances
- Aucune version antérieure. Le profil vivant passe avant tout le reste ; tout s'appuie dessus.

---

## v1.1 : motivation et lisibilité (SHOULD, 1re vague)

**Thème :** rendre visible la progression vers l'objectif, qui est le vrai levier de motivation du coureur.

### Fonctionnalités
- **Jauge de faisabilité de l'objectif**, mise à jour à chaque séance (ex. « sub-45 sur 10 km : 72 % → en hausse »). Elle sert aussi d'outil de motivation, d'où sa place en tête.
- **Alerte objectif** : trop difficile à atteindre ou pas assez ambitieux.
- **Prédiction de chrono en fourchette** avec tendance (ex. « 44:30 – 46:10, en amélioration »).
- **Rappel du « vrai pourquoi »** : une phrase écrite à la création de l'objectif, réaffichée quand le coureur hésite ou manque de motivation (la version « minimum vital » a été rejetée).
- **Échauffements / étirements adaptés à chaque séance** (n° 1 dans l'ordre de l'utilisateur).
- **Vue « escalier »** du plan : volume et intensité semaine par semaine.
- **3 notifications importantes** (les seules autorisées) :
  - séance du jour pas envoyée sur la montre ;
  - modification du plan en attente de confirmation ;
  - signal de surmenage (ex. 3 séances 🥵 d'affilée + FC au repos en hausse).

### Definition of done (côté coureur)
- En un coup d'œil, je sais si mon objectif est en bonne voie et quel chrono je peux raisonnablement viser.
- Quand je manque de motivation, l'appli me rappelle pourquoi je cours, pas une séance au rabais.
- Chaque séance arrive avec son échauffement et ses étirements.
- Je ne suis notifié que lorsque c'est vraiment nécessaire.

### Dépendances
- v1 : historique des feedbacks et emojis (jauge, surmenage), écran état de la synchro (notif d'échec), flux de confirmation (notif en attente), structure du plan (vue escalier, échauffements).

---

## v1.2 : continuité et dialogue (SHOULD, 2e vague)

**Thème :** les plans deviennent les chapitres d'une même histoire, et le coureur peut discuter avec son coach.

### Fonctionnalités
- **Retour de blessure progressif** : après une suspension, plan de reprise progressive et recalcul de la faisabilité (proposer de décaler la date ou d'ajuster le chrono).
- **Conversation IA rattachée au feedback** :
  - ouverte avec le contexte de la séance déjà chargé (données, ressenti, rôle) ;
  - points importants mémorisés dans le profil (ex. « le mollet gauche tire après 15 km ») ;
  - peut proposer une mise à jour du plan, présentée en **avant/après** avant confirmation.
- **Saison de plans enchaînés** : préparation → course → récupération → préparation suivante. Le plan suivant part de la forme réelle de fin de plan, avec un bilan du précédent (différenciateur par rapport à Runna).

### Definition of done (côté coureur)
- Après une blessure, je reprends sans risque avec un objectif recalé honnêtement.
- Je peux poser une question sur une séance sans tout réexpliquer, et l'appli s'en souvient pour la suite.
- Je peux enchaîner mes objectifs sans attendre la fin d'un plan ni repartir de zéro.

### Dépendances
- v1 : profil vivant (mémoire des échanges, continuité), empêchement « blessé » (suspension), flux de confirmation.
- v1.1 : jauge de faisabilité (recalcul après blessure, bilan de fin de plan).

---

## v2 : le corps et le contexte (COULD)

**Thème :** enrichir le plan autour de la course : renforcement, matériel, terrain, prévention.

### Fonctionnalités (dans l'ordre suggéré)
- **Renforcement musculaire spécifique à la course** (n° 2 dans l'ordre de l'utilisateur).
- **Empêchements supplémentaires** :
  - « moins de temps » : séance raccourcie qui garde l'essentiel (ex. 2 fractionnés au lieu de 6) ;
  - « voyage » : séance adaptée (tapis, côtes à l'hôtel, renfo sans matériel).
- **Carnet de blessures vivant** : zone, date, déclencheur connu ; le plan évite ces déclencheurs.
- **Rapport kiné exportable** en un clic + **suggestions santé** (médecin, kiné, compléments) seulement quand c'est nécessaire : douleur répétée au même endroit, reprise qui stagne, fatigue persistante malgré l'allègement.
- **Chaussures** : kilométrage par paire, rotation selon le type de séance, alerte d'usure.
- **Parcours** : suggestion d'un parcours connu adapté (plat pour le fractionné, vallonné pour la sortie longue).
- **Journal de bord** : note libre optionnelle (vocale ou texte) que l'IA relit pour repérer des tendances.
- **Simulation de course** : stratégie d'allure km par km + plan de ravitaillement avant l'objectif.
- **Tableau de bord personnalisable** : ajouter des graphiques au fil du temps (manque relevé chez Runna).

### Definition of done (côté coureur)
- Mon plan intègre le renfo et s'adapte quand j'ai peu de temps ou que je voyage.
- L'appli connaît mes points faibles, évite ce qui me blesse et me dit quand consulter, avec un rapport prêt pour le kiné.
- J'arrive le jour de la course avec une stratégie d'allure claire.

### Dépendances
- v1 : profil vivant (carnet de blessures, chaussures, parcours), bouton Empêchement (nouveaux cas), données de course (simulation, tableau de bord).
- v1.1 : échauffements/étirements (le renfo réutilise le même format de contenu associé aux séances).
- v1.2 : mémoire des conversations (alimente le carnet de blessures et les déclencheurs santé).

---

## v3+ : hors de la course pure (WON'T pour l'instant)

**Thème :** brancher le reste de la vie du coureur sur le plan.

### Fonctionnalités
- **Nutrition** (vérité de base pour l'utilisateur, d'où la place réservée dans le modèle dès la v1 ; manque relevé chez Runna).
- **Sommeil** / forme du matin.
- **Calendrier** / emploi du temps.
- **Météo** (peut-être).
- **Types d'objectifs supplémentaires** : maintien / plateau volontaire, trail / dénivelé, régularité (ex. 3 fois par semaine pendant 3 mois).

Rejetés, à ne pas réintroduire : version « minimum vital » pour manque de motivation, choix manuel de la méthode d'entraînement.

### Definition of done (côté coureur)
- Mon plan tient compte de ce que je mange, de mon sommeil, de mon agenda et du temps qu'il fait, sans saisie quotidienne supplémentaire.
- Je peux viser un trail, une période de maintien ou un objectif de régularité.

### Dépendances
- v1 : places nutrition et sommeil réservées dans le profil vivant (pas de refonte).
- v1.1 : notifications et signaux de surmenage (le sommeil les enrichit).
- v2 : parcours (trail / dénivelé), simulation de course (ravitaillement → nutrition).

---

## Risques et points à valider

- **Fiabilité de la synchro via l'API intervals.icu** : écriture des séances structurées dans le calendrier, gestion des erreurs, limites et quotas. À prototyper en premier.
- **Arrivée des séances sur l'Apple Watch S10** : la chaîne intervals.icu → HealthFit → montre doit livrer des séances structurées exploitables. Valider le format et le délai de bout en bout.
- **Remontée des courses** : données complètes (blocs, FC, allure) et sans doublons entre HealthFit et intervals.icu.
- **Crédibilité du feedback IA** : chiffres exacts, pas d'hallucination ni de flatterie, ton réaliste. Prévoir des garde-fous (chaque affirmation liée à une donnée).
- **Cohérence du plan et des réajustements** : progression logique, chrono cible réaliste, prédictions justifiées. Tester sur l'historique réel avant d'y croire.
- **Calibrage de la matrice respect × ressenti** et des seuils de surmenage : éviter les sur-réactions et les alertes bruyantes.
- **Modèle de données du profil vivant** : vérifier tôt que nutrition, sommeil et saison s'y brancheront sans refonte.
- **Charge de configuration initiale** : elle doit rester acceptable puisque l'usage quotidien est quasi nul.
