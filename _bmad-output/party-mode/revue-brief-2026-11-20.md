---
title: "Revue du brief en party mode : constats et décisions"
created: 2026-11-20
source: "_bmad-output/planning-artifacts/briefs/brief-running-2026-11-18/"
usage: "Entrée pour /bmad-prd, puis pour l'architecture"
---

# Revue du brief : constats et décisions

Revue du brief et de l'addendum du 2026-11-18, menée en table ronde du 18 au 20 novembre 2026. Ce document sert d'entrée au PRD.

## 1. Risques éliminés (vérifiés par Badr en conditions réelles)

| Risque du brief | Statut | Preuve |
|---|---|---|
| Transport des séances vers l'Apple Watch (import HealthFit en bêta) | **Éliminé** | Séances structurées écrites dans intervals.icu, synchronisées sur la montre via HealthFit, et courues sans problème. |
| Remontée des courses par bloc | **Éliminé** | Les activités qui reviennent dans intervals.icu contiennent un lap par bloc prévu. L'alignement prévu/réalisé peut se faire index par index. |

Plan B si HealthFit casse un jour : Watchletic, qui importe aussi les séances planifiées d'intervals.icu. Garder intervals.icu comme pivot rend le transporteur interchangeable.

## 2. Décisions et intentions de Badr

- **Il construit son propre outil.** Un chat avec un serveur MCP pour intervals.icu peut servir de prototype, mais pas de produit. Le « pourquoi » du PRD doit le dire franchement : il veut son outil, bien construit, en plus du gain de temps.
- **Pas de date limite, priorité à la solidité.** Il ne veut pas une suite de versions instables.
- **Son prochain objectif de course** : un marathon début avril 2027 en moins de 3 h 30. Cet objectif **peut changer et ne doit pas orienter le périmètre du projet**.

## 3. Contradictions du brief à trancher dans le PRD

1. **La synchro.** L'addendum dit « la synchro n'est pas un critère de réussite », mais le critère n°1 (boucle sans saisie manuelle) et le principe « la confiance avant les fonctionnalités » reposent sur elle. Il faut décider si la fiabilité de la synchro devient un critère explicite.
2. **La continuité.** Le brief en fait le vrai moteur du projet, mais aucun critère de la v1 ne la vérifie. Il faut ajouter un critère d'acceptation : **« le plan 2 part réellement des données et du bilan du plan 1 »**.
3. **Les six types d'objectifs en v1.** Chaque type demande son propre générateur et ses propres règles (la reprise après blessure, par exemple). Il faut choisir les types de la v1 ; « chrono sur une distance » est le candidat évident.
4. **Le préremplissage du profil.** Les blessures et les disponibilités ne viennent pas des données importées. Le profil combine un import (records, charge, CTL) et un formulaire.

## 4. Principes de conception retenus par la salle

- **Le LLM raconte, le code compte.** Tous les chiffres (allure par bloc, écart à la cible, note de conformité, dérive cardiaque) sont calculés de façon déterministe, puis passés au modèle comme des faits. Le modèle ne fait aucun calcul.
- **Un validateur de feedback.** Chaque nombre du texte de l'IA doit exister dans les données d'entrée ; sinon, le feedback est rejeté et régénéré. Cela rend mesurable le critère « aucun chiffre faux ».
- **Un plan structuré, validé, puis traduit.** Le LLM produit un JSON (semaines, séances, blocs, cibles). Un validateur applique des garde-fous : volume hebdomadaire en hausse de 10 % au plus, une semaine allégée toutes les 3 ou 4 semaines, répartition des intensités cohérente avec la méthode annoncée. Une fonction déterministe traduit ensuite le plan dans la syntaxe texte `description` d'intervals.icu. Le LLM n'écrit jamais cette syntaxe.
- **Des tests de round-trip** sur intervals.icu : envoyer un plan, relire les événements, comparer au JSON source. Utiliser `events/bulk?upsert=true` avec un `external_id` pour republier sans doublons. Éviter `workout_doc`, qui n'est pas documenté.
- **« Empêchement » est un arbre de décision**, pas un appel à l'IA : dernière minute → déplacer ; symptômes au-dessus du cou → footing possible ; fièvre → repos ; blessure → suspendre. Le LLM n'intervient que s'il faut régénérer le plan.
- **Hébergement** : une PWA servie derrière Tailscale. Rien n'est exposé sur Internet, la clé API reste côté serveur, et il n'y a pas d'authentification à écrire.

## 5. Expérience utilisateur à couvrir

- **L'état d'attente après une course.** Badr ouvre souvent l'app avant que HealthFit ait exporté sa course. Cet état doit être prévu, pas laissé vide.
- **L'emoji posé tout de suite.** Le ressenti peut être saisi avant l'arrivée de la course, puis rattaché à elle. Il est aussi plus honnête à chaud. Le modèle de données doit permettre un ressenti sans course associée.
- **Les indicateurs de « Tes données »** : passer de « 3 à 5 » à un chiffre principal et deux chiffres secondaires.

## 6. Points à calibrer dans le PRD

- **Le seuil de la matrice.** « Trois séances 🥵 d'affilée et une FC au repos en hausse » représente environ une semaine de signaux à 3 ou 4 sorties par semaine. C'est une hypothèse sans données derrière. Il faut aussi vérifier que HealthFit envoie bien la FC au repos à intervals.icu.
- **Le test de qualité du plan, à l'aveugle.** Anonymiser et mélanger le plan de l'app et celui de l'ancienne méthode avant de les comparer. La prépa marathon, faite avec l'ancienne méthode faute d'app prête, servira de plan de référence.
- **Le coût.** Moins de 10 € par mois semble largement tenable (environ une génération de plan par mois et une quinzaine de feedbacks), mais il reste à mesurer.

## 7. Comment « solide » se vérifie sans date limite

- **Des portes de qualité** : aucune story n'est terminée sans ses tests ; aucun appel à intervals.icu sans test de contrat ; aucun chiffre du feedback sans test qui le recalcule.
- **Un squelette vertical d'abord** : un objectif, un plan de 4 semaines, publié, couru, commenté. Ensuite, chaque tranche est épaissie. L'expérience post-course fait partie du squelette dès le début.
- **La définition de « fini »** : la v1 est terminée quand les cinq critères du brief et le critère de continuité passent.

## 8. Séquence recommandée

| # | Agent | Livrable | Commande |
|---|---|---|---|
| 1 | John (PM) | PRD | `/bmad-prd` |
| 2 | Sally (UX) | DESIGN.md et EXPERIENCE.md | `/bmad-ux` |
| 3 | Winston (architecte) | Architecture | `/bmad-architecture` |
| 4 | John (PM) | Epics et stories | `/bmad-create-epics-and-stories` |
| 5 | Murat (test) | Plan de test système | `/bmad-testarch-test-design` |
| 6 | — | Vérification de la préparation, puis sprint | `/bmad-sprint-planning` |
| 7 | Amelia (dev) | Construction, story par story | `/bmad-build` |

L'UX passe avant l'architecture parce que le rituel post-course (le ressenti saisi avant la course, l'état d'attente) a un effet sur le modèle de données.

**À préparer pour le PRD** : le nombre de types d'objectifs en v1, et le seuil de la matrice que Badr juge juste.
