---
title: "Réconciliation : brainstorming du 17/11 → PRD + addendum du 20/11"
created: 2026-11-20
sources:
  - _bmad-output/brainstorming/brainstorm-app-running-perso-2026-11-17/brainstorm-intent.md
  - _bmad-output/brainstorming/brainstorm-app-running-perso-2026-11-17/roadmap.md
  - _bmad-output/brainstorming/brainstorm-app-running-perso-2026-11-17/.memlog.md
cible:
  - prd.md
  - addendum.md
---

# Réconciliation brainstorming → PRD

**Périmètre.** On cherche les idées du brainstorming, surtout qualitatives, qui manquent, sont affaiblies ou sont contredites dans le PRD et l'addendum.

**Exclus.** Les arbitrages validés au §11 du PRD (un seul type d'Objectif, Ressenti dès la récupération de la Course, seuil de surmenage, SM-7 non bloquant, etc.), les éléments reportés au §8.2 (jauge, fourchette, vrai pourquoi, escalier, notifications, saison, conversation, retour de blessure, v2, v3) et les non-objectifs du §7 (pas de réajustement continu, pas de tableau de bord, pas de minimum vital, pas de choix de Méthode).

Les écarts sont classés par importance.

---

## 1. « La confiance est le produit » n'est pas posé comme principe directeur

- **Source.** roadmap.md, en-tête : « La confiance est le produit : synchro fiable, feedback appuyé sur des données, un "pourquoi" en une ligne, des prédictions en fourchette. » .memlog : « Les 4 sabotages de l'utilisateur portent sur la CONFIANCE […] aucun sur les fonctionnalités : la confiance est le vrai produit. » brainstorm-intent §3 : « La confiance et la fiabilité avant les fonctionnalités. »
- **PRD.** La formule n'apparaît pas. Le §1 ne nomme qu'un seul principe directeur, le pilote automatique transparent. La confiance n'existe qu'en morceaux : le JTBD émotionnel au §2.1 (« faire confiance aux chiffres »), NFR-3, NFR-5 et SM-5. Rien ne dit que la confiance l'emporte sur les fonctionnalités quand il faut arbitrer. Le passage de SM-7 en critère non bloquant est un arbitrage (§11.12) et n'est pas contesté ici, mais sans le principe, un lecteur aval (UX, architecture, stories) n'a pas de règle pour départager.
- **Correction proposée.** Au §1, ajouter un second principe directeur : « La confiance est le produit : en cas de conflit, fiabilité, cohérence et explicabilité passent avant toute fonctionnalité. » Rattacher explicitement NFR-3, NFR-5, FR-8 et FR-12 à ce principe. Dans l'addendum §4, rappeler que les portes de qualité en découlent.

## 2. La « definition of done côté coureur » a disparu au profit de critères côté bâtisseur

- **Source.** roadmap.md, v1 « Definition of done (côté coureur) » :
  - « Je configure mon profil et mon objectif une seule fois, pré-remplis depuis mes données ; je ne fais que valider. »
  - « Après ma course, je réponds en un emoji et je reçois un feedback clair, chiffré et crédible, sans ouvrir Strava. »
  - « Quand le plan change, je vois pourquoi et je confirme. »
  - « Un imprévu, une maladie ou une blessure se gère en deux taps. »
  - « Au quotidien, je n'ai presque rien à faire. »
- **PRD.** Les SM du §9 mesurent le système (boucle, temps de création d'un Plan, coût, chiffres exacts, continuité). Trois exigences du coureur n'ont pas de critère :
  - « au quotidien presque rien à faire » : SM-3 ne compte que la création d'un Plan, pas le rituel quotidien (ouvrir l'app, poser le Ressenti, lire le Feedback) ;
  - « feedback clair » : SM-5 couvre l'exactitude, pas la clarté perçue ;
  - « deux taps » : FR-15 annonce « deux taps au plus », mais UJ-4 enchaîne Empêchement, Malade, la réponse à la règle du cou, le choix de l'option puis la confirmation, soit plus de deux taps. Le PRD se contredit.
- **Correction proposée.**
  - Ajouter au §9 un bloc « Definition of done côté coureur » qui reprend les 6 phrases de la roadmap à la première personne, chacune liée à un SM ou à une FR.
  - Ajouter un SM « rituel quotidien » : de l'ouverture au Feedback lu, en moins de N taps et sans aucune saisie hors Ressenti.
  - Préciser dans FR-15 ce que couvrent les « deux taps » (déclarer l'Imprévu, hors confirmation) ou corriger UJ-4.

## 3. La « clarté Runna » est réduite à une phrase, sans exigence vérifiable

- **Source.** brainstorm-intent §2 : il apprécie « la clarté de l'accueil, le parcours de génération du plan, la présentation des séances et les commentaires générés ». .memlog : 4 idées « Adapter de Runna ». Sabotage « feedback incompréhensible → feedback clair, compréhensible immédiatement ».
- **PRD.** Une seule phrase au §6 (« La référence de clarté est Runna »). Ce qui manque :
  - **parcours de génération du Plan** : UJ-2 et FR-4 à FR-6 décrivent la mécanique (Consigne, accepter ou refuser), pas l'expérience guidée qui plaît chez Runna ;
  - **présentation d'une Séance** : aucune FR ne dit ce qu'on voit en ouvrant une Séance (but, rôle dans le Plan, Blocs et cibles lisibles). FR-17 ne décrit que la carte d'accueil ;
  - **« compréhensible immédiatement »** : aucun critère.
- **Correction proposée.**
  - Ajouter une FR « Détail d'une Séance » : but en une ligne, rôle dans le Plan, Blocs avec leur cible, statut de Publication.
  - Au §6, transformer la référence Runna en consignes pour `bmad-ux` : les 4 points à reprendre, l'onboarding guidé du Plan, et un test « compris sans relecture ».
  - Ajouter une contre-référence : les frustrations Runna à ne pas reproduire, comme l'historique ignoré par un nouveau plan (c'est le rôle de SM-6).

## 4. La Séance manquée sans Imprévu déclaré n'est pas traitée

- **Source.** .memlog, Reverse Brainstorming : « Sabotage : plan qui ignore la séance ratée de la veille → inverse : chaque séance s'appuie toujours sur l'état réel du jour. » .memlog : « Causes réelles des séances ratées : surtout le manque de motivation et les empêchements de dernière minute. »
- **PRD.** FR-9 gère seulement l'attente (« en attente de la course »). Rien ne couvre une Séance passée sans Course et sans Empêchement déclaré, alors que c'est le cas le plus fréquent selon l'utilisateur (manque de motivation). La Séance reste-t-elle indéfiniment « en attente » ? Le Plan l'ignore-t-il ? Le §7 exclut le réajustement continu, pas la gestion de ce statut.
- **Correction proposée.** Ajouter à FR-9 ou FR-15 qu'à J+1, une Séance sans Course passe au statut « manquée ». L'accueil propose alors de la déclarer a posteriori (dernière minute : annuler ou déplacer) via le flux d'Ajustement confirmé (FR-16), sans modification silencieuse. La Matrice compte la Séance manquée dans sa lecture de la forme.

## 5. « Garder son propre jugement » est affaibli : « Tes données » passe de 3-5 indicateurs à 1 + 2, sans arbitrage

- **Source.** brainstorm-intent §3 : « L'avis de l'IA séparé des données brutes, pour que l'utilisateur garde son propre jugement. » .memlog : « L'utilisateur veut garder son propre jugement : l'IA donne un avis, mais les données doivent rester lisibles pour qu'il se forge le sien. » MUST : « Tes données » (3 à 5 indicateurs clés).
- **PRD.** FR-12 et SM-C2 fixent un plafond « 1 chiffre principal et 2 secondaires ». Ce changement ne figure pas au §11. Le *pourquoi* de la séparation (préserver le jugement du coureur) n'apparaît nulle part : la séparation est décrite comme une simple mise en page.
- **Correction proposée.** Soit consigner l'arbitrage au §11 (« 3 indicateurs au lieu de 3 à 5, validé le … »), soit revenir à « 3 à 5 ». Dans tous les cas, ajouter dans la description du §4.4 : « La couche "Tes données" existe pour que le coureur se forge son propre avis avant ou malgré celui de l'IA. »

---

## Écarts secondaires

### 6. Le Ressenti n'occupe plus le temps de chargement
- **Source.** .memlog : « La question du ressenti s'affiche pendant le chargement/génération du feedback (le temps d'attente sert à quelque chose). »
- **PRD.** FR-10 et l'arbitrage §11.5 fixent le *moment* (dès la récupération de la Course), mais l'idée de recouvrir l'attente a disparu. Il y a aussi une ambiguïté : si le Feedback IA dépend du Ressenti (Matrice, FR-13), il ne peut pas être généré pendant la question.
- **Correction proposée.** Préciser dans FR-10 que les calculs déterministes (Conformité, « Tes données ») sont lancés pendant la question, et que la rédaction IA démarre dès que le Ressenti est posé, avec une cible de latence.

### 7. Le signal « plus dur que ce que disent les chiffres » n'est pas nommé
- **Source.** .memlog : « Écart entre l'effort perçu (RPE) et l'effort mesuré (FC/allure) comme signal clé de fatigue : "plus dur que ce que disent les chiffres" = drapeau rouge. » Autres signaux cités : douleurs, jambes lourdes.
- **PRD.** Le seuil (§11.9) est arbitré, mais FR-13 ne dit pas que la Matrice doit interpréter cet écart, ni ce que valent les 9 cases. Hors blessure déclarée, aucun moyen de signaler une douleur.
- **Correction proposée.** Dans FR-13, définir la lecture des cases clés, au minimum « respectée + 🥵 à FC/allure normales = fatigue probable », et la renvoyer au Feedback. Noter dans les questions ouvertes du §10 qu'il n'existe pas de signal de douleur en v1.

### 8. Le préremplissage « je ne fais que valider » glisse vers un formulaire
- **Source.** brainstorm-intent §5 : « Le profil est pré-rempli et l'utilisateur ne fait que valider ou corriger. »
- **PRD.** UJ-1 et FR-2 : « Badr complète un formulaire. » SM-C3 assume de privilégier la complétude, ce qui est cohérent avec le brief. En revanche, le préremplissage de l'*Objectif* (records, forme, disponibilités, réutilisés depuis le Profil) n'est pas mentionné pour la création d'un Plan suivant.
- **Correction proposée.** Dans FR-4, indiquer que la saisie de l'Objectif reprend par défaut le Profil (disponibilités, sortie longue, indisponibilités) et ne demande que le delta. Cela soutient SM-3.

### 9. Le chrono irréaliste est signalé sans contre-proposition
- **Source.** brainstorm-intent §5 : « chrono cible réaliste ». Sabotage : « prédictions absurdes → prédictions crédibles et justifiées ».
- **PRD.** FR-4 « signale » le problème, sans dire quoi proposer ni sur quoi s'appuyer. La fourchette continue est reportée en v1.1 et n'est pas en cause ici.
- **Correction proposée.** L'alerte de FR-4 affiche sa raison en une ligne (NFR-5), avec la donnée qui la fonde, et propose un chrono cible réaliste que l'utilisateur accepte ou refuse.

### 10. « Chaque course et son feedback » devient « Feedbacks récents »
- **Source.** brainstorm-intent §5 : « quelques clics suffisent pour atteindre chaque course et son feedback ».
- **PRD.** FR-17 ne donne accès qu'aux « Feedbacks récents ». L'historique complet et les Courses non planifiées ne sont pas mentionnés.
- **Correction proposée.** FR-17 : accès en 3 taps au plus à n'importe quelle Course passée et à son Feedback, y compris les Courses non planifiées.
