# Intention produit : application de course à pied personnelle

## 1. Vision
Une application à usage strictement personnel qui génère, synchronise, analyse et réajuste un plan d'entraînement à partir de données réelles. Elle remplace la routine actuelle : conversation avec une IA, prompt collé à la main dans intervals.icu, puis consultation des résultats sur Strava.

## 2. Utilisateur & contexte
- Un seul coureur, qui est aussi le développeur.
- Matériel et outils : Apple Watch Series 10, HealthFit, intervals.icu.
- Référence : Runna. Il en apprécie la clarté de l'accueil, le parcours de génération du plan, la présentation des séances et les commentaires générés. Mais l'abonnement est payant, il y a trop peu de graphiques par course, pas de nutrition, on ne peut pas enchaîner les plans, et un nouveau plan ignore l'historique.
- Différenciateur : la continuité. Un seul profil vivant alimente chaque plan. Les plans sont les chapitres d'une même histoire, pas des silos.

## 3. Principes directeurs
- **Pilote automatique transparent** : l'application décide (méthode, plan, ajustements) et explique toujours pourquoi en une ligne. L'utilisateur valide ou refuse.
- **La confiance et la fiabilité avant les fonctionnalités** : synchronisation fiable, cohérence, recommandations crédibles.
- **Configuration une fois, usage quotidien quasi nul** : effort accepté au lancement du plan, quasiment aucune manipulation ensuite.
- **Aucune modification sans confirmation** : jamais de changement silencieux du plan.
- **Données utilisées uniquement pour la course.**
- **Ton motivant et réaliste** : ni démotivant ni flatteur. Aucun compliment sans chiffre qui le justifie.
- **L'avis de l'IA séparé des données brutes**, pour que l'utilisateur garde son propre jugement.
- Bases du plan : progresser, c'est doser stress et récupération. La prévention des blessures est une contrainte de base. Chaque séance part de l'état réel du jour.

## 4. Boucle cœur v1
Générer le plan → synchroniser vers la montre → courir → remonter la séance → analyser (ressenti + données) → réajuster (avec confirmation) → recommencer.
Critère de périmètre : tout ce qui ne sert ni à générer le plan ni à suivre la progression est secondaire.

## 5. Périmètre MUST (v1)
- **Profil vivant + import** : historique, records, forme actuelle et charge (CTL), blessures, disponibilités, importés depuis intervals.icu et HealthFit. Le profil est pré-rempli et l'utilisateur ne fait que valider ou corriger.
- **Création d'objectif** : 6 types (distance, durée, chrono sur une distance, récupération après une course, condition physique générale, reprise après blessure). Un objectif principal + des objectifs secondaires.
  - Entrées : historique, records, forme actuelle, blessures actuelles et passées, jours disponibles, jour de la sortie longue, périodes d'indisponibilité.
- **Génération du plan** à partir des données réelles :
  - progression visible et logique d'une semaine à l'autre ;
  - chrono cible réaliste ;
  - méthode choisie par l'application (ex. 80/20, Daniels, polarisé), affichée en une ligne avec sa raison ;
  - réajustement cohérent avec les séances passées et l'objectif final.
- **Synchronisation automatique dans les deux sens** :
  - plan → intervals.icu → montre ;
  - montre → application.
  - Écran « état de la synchro » : dernière séance envoyée, dernière course reçue, statut ✅/⚠️, bouton « resynchroniser ».
- **Feedback après chaque course** :
  - Une seule question de ressenti, 3 emojis (😌 facile / 😐 normal / 🥵 dur), posée **pendant le chargement du feedback et avant l'affichage des chiffres**.
  - Respect de la séance (prévu vs réalisé) **bloc par bloc** : échauffement, chaque fractionné, récupération. Un verdict d'une ligne par bloc + une note de conformité globale.
  - **3 niveaux** : la séance, l'objectif de la séance (son rôle dans le plan), l'objectif global.
  - Concret : chaque affirmation cite un élément mesuré ou ressenti. Structure : 1 point fort, 1 point à améliorer, 1 conséquence pour la suite. Si la séance n'est pas respectée : ce qui a été mal fait + comment s'améliorer la prochaine fois.
  - Deux couches visuellement séparées : « Mon avis (IA) » et « Tes données » (3 à 5 indicateurs clés). Le détail complet reste dans intervals.icu.
- **Réajustement par la matrice respect × ressenti** : par exemple « respectée + 🥵 » et « non respectée + 😌 » donnent deux lectures différentes de la forme du jour. Chaque proposition affiche son « pourquoi » en une ligne et attend une confirmation.
- **Bouton « Empêchement »** :
  - dernière minute : annuler ou déplacer la séance ;
  - malade : règle au-dessus/en-dessous du cou (rhume léger = footing possible, fièvre = repos) ;
  - blessé : annuler le plan ou le suspendre jusqu'au rétablissement.
  - L'application évalue s'il faut refaire le plan ou seulement annuler une ou plusieurs séances.
- **Accueil minimaliste** : le plan s'affiche dès l'ouverture, quelques clics suffisent pour atteindre chaque course et son feedback. Zéro notification.

## 6. SHOULD / COULD / WON'T
**SHOULD**
- Retour de blessure : plan de reprise progressive + recalcul de la faisabilité de l'objectif (décaler la date ou ajuster le chrono).
- Jauge de faisabilité mise à jour à chaque séance + alerte si l'objectif est trop dur ou pas assez ambitieux + prédiction de chrono en fourchette avec tendance. C'est le levier de motivation, candidate à remonter tôt.
- Conversation IA ouverte depuis un feedback, avec le contexte préchargé. Les points importants sont mémorisés dans le profil. Une proposition de mise à jour du plan affiche un avant/après avant confirmation.
- Saison de plans enchaînés : le plan suivant part de la forme réelle de fin de plan, avec un bilan du précédent.
- 3 notifications importantes possibles (à arbitrer).
- Vue « escalier » du plan : volume et intensité semaine par semaine.
- Échauffements et étirements adaptés à chaque séance.
- Rappel du « vrai pourquoi », écrit en une phrase à la création de l'objectif et réaffiché en cas d'hésitation.

**COULD**
- Renforcement musculaire spécifique à la course.
- Chaussures : kilométrage par paire, rotation, alerte d'usure.
- Parcours connu suggéré selon la séance.
- Journal de bord : une note libre par séance, relue par l'IA pour repérer des tendances.
- Simulation de course : stratégie d'allure km par km + ravitaillement.
- Carnet de blessures vivant (zone, date, déclencheur), que le plan contourne.
- Rapport kiné exportable + suggestions santé seulement si nécessaire (médecin, kiné, compléments). Déclencheurs : douleur répétée au même endroit, reprise qui stagne, fatigue persistante.
- Empêchements « moins de temps » (séance raccourcie qui garde l'essentiel) et « voyage » (tapis, côtes, renfo sans matériel).
- Tableau de bord personnalisable.

**WON'T (cette fois)**
- Nutrition, sommeil, calendrier, météo.
- Objectifs maintien, trail/dénivelé, régularité.

## 7. Contraintes & décisions d'architecture
- **Le profil vivant est la colonne vertébrale.** Il est construit en premier et rend possibles la continuité entre plans, la saison, la mémoire des conversations et le carnet de blessures.
- **Un emplacement est réservé dès le départ pour la nutrition et le sommeil** dans le profil : ils font partie des bases de la progression et pourront être branchés plus tard sans refonte.
- **L'API d'intervals.icu sert de relais vers la montre** : le plan est écrit directement dans son calendrier (séances structurées), sans saisie manuelle.
- **Import depuis HealthFit et intervals.icu** pour l'historique et les courses réalisées.
- L'emoji de ressenti est la donnée la plus rentable : il alimente la matrice, la détection du surmenage, les notifications et les suggestions santé. Il est critique dès la v1.
- Signaux de surmenage à capter : douleurs, récupération plus longue, effort perçu en hausse à charge égale (écart entre effort perçu et FC/allure), jambes lourdes, baisse de motivation. La plupart sont subjectifs, donc le ressenti compte autant que les données de la montre.

## 8. Idées rejetées
- **Version « minimum vital » quand la motivation manque** : l'utilisateur veut s'obliger à faire la vraie séance. La motivation est un levier mental (le « vrai objectif »), pas une question de dosage.
- **Choix manuel de la méthode d'entraînement** : l'application décide pour rester cohérente avec les données, et explique son choix.

## 9. Questions ouvertes
- Quelles notifications importantes garder ? Candidates : échec d'envoi de la séance du jour vers la montre, proposition de modification en attente de confirmation, surmenage détecté (ex. 3 séances 🥵 d'affilée + FC au repos en hausse).
- Faisabilité technique de la synchronisation : écriture de séances structurées via l'API d'intervals.icu jusqu'à l'Apple Watch, et remontée fiable des courses (HealthFit / intervals.icu) vers l'application.
