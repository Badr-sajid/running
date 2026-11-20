---
title: "Réconciliation : revue party mode du 20/11 → PRD + addendum"
source: "_bmad-output/party-mode/revue-brief-2026-11-20.md"
cible: "prd.md, addendum.md"
created: 2026-11-20
---

# Réconciliation : revue party mode → PRD + addendum

Périmètre : écarts entre la revue (`revue-brief-2026-11-20.md`) et le PRD + l'addendum. Ne sont pas comptés comme écarts : les arbitrages du PRD §11 (en particulier le Ressenti demandé à la récupération de la Course, §11.5), ni les éléments reportés en §8.2 ou exclus en §7.

Points de la revue bien repris, pour mémoire : risques éliminés et plan B Watchletic (addendum §3), « le LLM raconte, le code compte » (FR-12, addendum §2), validateur de Feedback (FR-12), plan en JSON validé puis traduit (FR-5, addendum §2), round-trip et `upsert`/`external_id` (FR-7, addendum §1 et §4), arbre de décision Empêchement (FR-15), Tailscale (NFR-2, addendum §3), état d'attente (FR-9), format 1 + 2 (FR-12), continuité (SM-6), synchro (SM-7), un seul type d'Objectif (FR-4), profil import + formulaire (FR-1, FR-2), test à l'aveugle (SM-5, addendum §7), coût (NFR-4), portes de qualité (addendum §4).

---

## Écart 1 (important) : « pas de date limite, priorité à la solidité » n'apparaît nulle part

- **Source** (revue §2) : « **Pas de date limite, priorité à la solidité.** Il ne veut pas une suite de versions instables. » Et §7 : « Comment "solide" se vérifie sans date limite ».
- **PRD** : absent. Le §1 dit « un outil qu'on construit soi-même, et bien », mais rien ne dit qu'il n'y a pas d'échéance ni qu'on refuse les versions instables. La note du §8.2 (« À revoir si la v1 se termine vite ») suggère même une logique de délai. Les portes de qualité ne figurent que dans l'addendum §4, sans être reliées à une exigence du PRD.
- **Correction proposée** : ajouter une NFR « Solidité » (par ex. NFR-8) : « Pas de date limite. Aucune tranche n'est livrée à l'usage réel tant que ses tests et ses tests de contrat intervals.icu ne passent pas. On préfère un périmètre réduit à une version instable. » Y renvoyer depuis l'addendum §4, et ajouter dans le §9 que la « définition de fini » suppose aussi le respect des portes de qualité.

## Écart 2 (important) : le Seuil de surmenage est figé alors que la revue le présente comme une hypothèse, et « FC au repos en hausse » n'est pas défini

- **Source** (revue §6) : « "Trois séances 🥵 d'affilée et une FC au repos en hausse" représente environ une semaine de signaux […]. C'est une hypothèse sans données derrière. »
- **PRD** : FR-14 et §11.9 fixent le seuil, ce qui est un arbitrage légitime. En revanche, « en hausse » n'est pas quantifié : de combien, par rapport à quelle référence (moyenne sur 7 jours ? sur 28 jours ?), sur quelle fenêtre ? L'exemple « FC au repos +6 » laisse croire qu'un chiffre existe. On ne sait pas non plus si une Course non planifiée compte dans les « 3 d'affilée ». Rien ne prévoit de revoir le seuil après usage réel. Or le principe « le code compte » (NFR-3) impose une règle déterministe.
- **Correction proposée** : dans FR-14, définir la règle exactement, par ex. « FC au repos moyenne des 3 derniers jours ≥ moyenne des 28 jours + N bpm », avec N à fixer. Préciser quelles Courses comptent. Ajouter une question ouverte au §10 : « Seuil à recalibrer après le premier Plan couru (hypothèse sans données, revue §6). » Le relier à la contre-métrique SM-C1.

## Écart 3 (moyen) : la revue demande que l'objectif marathon n'oriente pas le périmètre, le PRD ne le dit pas

- **Source** (revue §2) : « un marathon début avril 2027 en moins de 3 h 30. Cet objectif **peut changer et ne doit pas orienter le périmètre du projet**. »
- **PRD** : affaibli. UJ-1 à UJ-4 le qualifient d'« indicatif », sans formuler la règle. L'addendum §7 fait de la prépa marathon le plan de référence de SM-5, ce qui la lie de fait au critère de fin de la v1.
- **Correction proposée** : ajouter au §7 (Non-objectifs) ou au §8 : « L'Épreuve visée par Badr (marathon avril 2027) ne fixe ni le périmètre ni le calendrier de la v1. » Dans SM-5 et l'addendum §7, prévoir un plan de référence de rechange (n'importe quel plan fait avec l'ancienne méthode pour un même Objectif) si l'objectif change.

## Écart 4 (moyen) : la place de l'expérience post-course dans le squelette vertical est affaiblie

- **Source** (revue §7) : « Un squelette vertical d'abord […]. Ensuite, chaque tranche est épaissie. **L'expérience post-course fait partie du squelette dès le début.** »
- **PRD** : §8.1 et §11.2 reprennent le squelette (« publiés, courus et commentés »), mais ne disent pas que le rituel post-course en fait partie : détection de la Course, état « en attente de la course », Ressenti avant les chiffres, Feedback en deux couches. « Commentés » pourrait se lire comme un Feedback minimal, ajouté plus tard.
- **Correction proposée** : dans le §8.1, écrire que le squelette comprend FR-9, FR-10 et FR-12 (au minimum le validateur de chiffres), puis que chaque tranche est épaissie ensuite.

## Écart 5 (mineur) : le principe « intervals.icu comme pivot, transporteur interchangeable » n'est pas une contrainte

- **Source** (revue §1) : « Garder intervals.icu comme pivot rend le transporteur interchangeable. »
- **PRD** : NFR-7 fait de HealthFit un prérequis, et l'addendum §3 cite Watchletic en plan B. Mais rien n'interdit à l'architecture de dépendre de HealthFit, par exemple d'une particularité de format.
- **Correction proposée** : compléter NFR-7 : « L'application ne dépend que d'intervals.icu ; aucune logique ne suppose un transporteur précis (HealthFit ou Watchletic), qui doit rester remplaçable sans modification du code. »

## Écart 6 (mineur) : la justification de la séquence UX → architecture n'est plus à jour

- **Source** (revue §8) : « L'UX passe avant l'architecture parce que le rituel post-course (**le ressenti saisi avant la course**, l'état d'attente) a un effet sur le modèle de données. »
- **PRD** : l'addendum §5 reprend cette justification telle quelle, alors que §11.5 a écarté le Ressenti saisi avant la Course (et donc le Ressenti sans Course associée). La raison n'est plus vraie qu'en partie : seul l'état d'attente reste.
- **Correction proposée** : dans l'addendum §5, reformuler la raison (état d'attente, Ressenti obligatoire avant les chiffres) et noter que le modèle de données n'a pas besoin de gérer un Ressenti sans Course (§11.5).

## Écart 7 (mineur) : le « pourquoi » franc et le rôle du prototype MCP sont atténués

- **Source** (revue §2) : « Un chat avec un serveur MCP pour intervals.icu peut servir de prototype, mais pas de produit. Le "pourquoi" du PRD doit le dire franchement : il veut son outil, bien construit, en plus du gain de temps. »
- **PRD** : §1 et le JTBD « Bâtisseur » le disent, mais comme une raison secondaire (« C'est aussi… »). L'alternative MCP n'apparaît que dans l'addendum §6, présentée comme un élément du paysage et non comme une option écartée.
- **Correction proposée** : au §1, ajouter une phrase : « Un chat branché sur le connecteur MCP d'intervals.icu suffirait comme prototype, mais pas comme produit : Badr veut son outil, bien construit. »
