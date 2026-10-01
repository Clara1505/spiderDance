# 02 — Problématique et objectifs

> **Statut** : brouillon v0.2 — à valider par l'équipe, **seuils chiffrés à fixer avec les tuteurs**
> **Dernière mise à jour** : 2026-09-29
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé · `[À DÉFINIR]` valeur à fixer par l'équipe
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`

---

## 1. Du besoin au problème

Le contexte (`01-contexte-besoins.md`) fait apparaître quatre constats :

1. L'énoncé demande des **mouvements synchronisés** entre **plusieurs** robots coordonnés par une
   **API de communication** [@S02].
2. Chaque araignée exécute sa propre séquence sur sa propre Arduino [@S04, ch. 1 p. 7 et §2.1
   p. 10], et des horloges distinctes dérivent les unes par rapport aux autres [@S16, introduction].
3. La version de référence du robot n'a pas de communication à distance [@S03, §3.4]. La solution
   Wi-Fi de l'an dernier envoie les ordres à chaque robot indépendamment, sans départ synchronisé
   [@S07, app.py l. 181-203], et aucune mesure de synchronisation n'a été publiée [@S03, §6.8].
4. En chorégraphie multi-robots, les retards d'exécution doivent être compensés pour maintenir la
   synchronisation [@S15, résumé].

La difficulté centrale n'est donc pas de faire bouger un robot, ce que le code officiel sait déjà
faire [@S04, ch. 4 p. 15-16], mais de faire en sorte que **plusieurs robots indépendants exécutent
le même mouvement au même moment**, et que cet accord soit **mesuré**.

## 2. Problématique

> **Comment coordonner, au moyen d'une API de communication, plusieurs robots araignées SEALK
> afin qu'ils exécutent une chorégraphie commune de façon synchronisée et perceptiblement
> collective, alors que chaque robot exécute seul sa séquence, sans retour de position, avec sa
> propre horloge et à travers une liaison de communication imparfaite ?**

### 2.1 Sous-questions
| ID | Sous-question | Traitée dans |
|---|---|---|
| Q1 | Quelle architecture de coordination : un poste qui pilote tous les robots, des robots qui se coordonnent entre eux, ou une solution hybride ? | État de l'art §5, ADR à venir |
| Q2 | Comment établir et maintenir une référence de temps commune, ou un signal de départ commun ? | État de l'art §6, ADR à venir |
| Q3 | Quel moyen de communication ajouter au robot, sachant que la version de référence n'en a pas ? | État de l'art §3 et §7, ADR à venir |
| Q4 | Comment décrire une chorégraphie pour qu'elle soit jouée à l'identique par chaque robot, en s'appuyant sur les mouvements existants ? | État de l'art §2 et §4 |
| Q5 | Comment **mesurer objectivement** la synchronisation obtenue ? | État de l'art §8, EXP à venir |

## 3. Objectifs

Les objectifs sont formulés pour être vérifiables. **Aucun seuil chiffré n'est fixé à ce stade** :
fixer une valeur sans mesure sur nos robots reviendrait à l'inventer. La méthode (§6) consiste à
mesurer d'abord un état de référence, puis à fixer les seuils avec les tuteurs.

### 3.1 Objectif général
Réaliser un démonstrateur dans lequel **N araignées SEALK** `[À DÉFINIR : N ≥ 2]` exécutent une
chorégraphie commune, coordonnée par une API de communication, avec une synchronisation mesurée et
documentée.

### 3.2 Objectifs spécifiques
| ID | Objectif | Critère de réussite | Moyen de preuve |
|---|---|---|---|
| O1 | **Communication** : chaque robot reçoit les messages de coordination. | Taux de messages reçus ≥ `[À DÉFINIR]` % sur une séance de `[À DÉFINIR]` minutes. | Journaux horodatés, fiche EXP |
| O2 | **Départ commun / temps commun** : les robots partagent un signal de départ ou une horloge commune. | Écart entre robots ≤ `[À DÉFINIR]` ms, mesuré au début et à la fin d'une chorégraphie. | Journaux horodatés, fiche EXP |
| O3 | **Synchronisation des mouvements** : un même mouvement démarre au même instant sur tous les robots. | Écart de démarrage ≤ `[À DÉFINIR]` ms, et **meilleur** que l'état de référence mesuré en EXP-002. | Mesure vidéo et/ou journaux, fiche EXP |
| O4 | **Chorégraphie** : enchaînement de figures tirées des mouvements disponibles [@S04, ch. 4 p. 15-16], décrit dans un format commun. | Au moins `[À DÉFINIR]` figures distinctes jouées sans intervention manuelle. | Démonstration, vidéo |
| O5 | **Comportement collectif** : au moins une figure qui n'a de sens qu'à plusieurs (ex. vague, miroir, canon). | Figure identifiable par un observateur extérieur. | Vidéo, retour des tuteurs |
| O6 | **Reproductibilité** : un tiers peut calibrer les robots, réinstaller et relancer la démonstration. | Procédure testée par un membre n'ayant pas écrit le code. | `reproductibilite.md` |

[HYPOTHÈSE] Les exemples de figures collectives de O5 s'inspirent de la famille « organisation
spatiale » des comportements d'essaim [@S11, résumé]. Leur faisabilité dépend des déplacements
réels des robots, à tester.

### 3.3 Priorisation (proposition)
| Priorité | Objectifs | Justification |
|---|---|---|
| Indispensable (MVP) | O1, O2, O3 avec N = 2 | Cœur de la problématique : sans communication ni synchronisation, pas de sujet. |
| Visé | O4, O5, N > 2 | Rend la démonstration convaincante. |
| Bonus | Mesure automatique par vision (état de l'art §8) ; calage sur le rythme de la musique (état de l'art §4) | Lien avec le thème vision du module [@S01, slide 2], si les tuteurs le jugent pertinent. |

## 4. Périmètre

### 4.1 Dans le périmètre
- Ajout ou choix d'un moyen de communication pour chaque robot (à décider par ADR).
- Signal de départ commun et/ou synchronisation temporelle du groupe.
- Format de description d'une chorégraphie et exécution sur chaque robot.
- Mesure et documentation des résultats.

### 4.2 Hors périmètre (proposition, à valider)
- Conception mécanique ou électronique nouvelle des robots (matériel fourni).
- Création de nouveaux mouvements élémentaires : on s'appuie d'abord sur ceux du code officiel
  [@S04, ch. 4 p. 15-16].
- Génération automatique de chorégraphies à partir de la musique ou du langage naturel (voir
  SwarmGPT [@S12], cité à titre de comparaison uniquement).
- Grands essaims (dizaines de robots).

## 5. Hypothèses de travail et contraintes
| ID | Hypothèse / contrainte | Statut |
|---|---|---|
| H1 | Chaque robot possède sa propre unité de calcul (Arduino Nano). | [FAIT] [@S04, ch. 1 p. 7] |
| H2 | La version de référence du robot peut communiquer à distance. | **Faux** : pas de Wi-Fi ni d'autre moyen distant [@S03, §3.4]. Un changement de carte ou un module est nécessaire. |
| H3 | Les robots disposent de mouvements programmables prêts à l'emploi. | [FAIT] Mouvements de base et avancés [@S04, ch. 4 p. 15-16] |
| H4 | Au moins deux robots fonctionnels et calibrés seront disponibles. | `[À VÉRIFIER]` |
| H5 | Des cartes Nano RP2040 Connect (Wi-Fi) sont disponibles. | `[À VÉRIFIER]` — utilisées l'an dernier [@S03, §6.2] |
| C1 | Contraintes génériques : puissance, taille, poids, bande passante, latence, fiabilité, coût. | [FAIT] [@S01, slide 5] |
| C2 | Livrables : PoC + codes, rapport, soutenance + démo, vidéo de secours. | [FAIT] [@S01, slide 10] |
| C3 | L'Arduino exécute les instructions de manière séquentielle. | [FAIT] [@S05, ch. 1 p. 7] |
| C4 | Aucun retour de position : l'arrivée des pattes est estimée en boucle ouverte. | [FAIT] [@S04, §2.1 p. 10] |
| C5 | Le code et le rapport de l'an dernier servent de guide ; tout est refait par l'équipe. | Règle d'équipe (CLAUDE.md §4.6) |

## 6. Démarche proposée pour fixer les seuils
1. **EXP-001 — Prise en main et calibration** : calibrer chaque araignée [@S04, ch. 3 p. 13-14],
   puis passer les tests de base et avancés [@S04, ch. 4 p. 15-16].
2. **EXP-002 — Mesure de référence** : envoyer le même ordre à deux robots sans mécanisme de
   synchronisation particulier, et mesurer l'écart de démarrage (vidéo et/ou journaux).
   `[À VÉRIFIER avec les tuteurs]` Faut-il écrire notre propre outil minimal pour cette mesure, ou
   peut-on utiliser le code de l'an dernier **uniquement comme instrument de mesure**, sans
   l'intégrer à notre dépôt ?
3. Fixer avec les tuteurs les seuils de O1 à O4 à partir de cette référence.
4. Comparer chaque amélioration à cette référence, ce qui prouve l'apport du travail.

## 7. Risques principaux
| Risque | Impact | Parade envisagée |
|---|---|---|
| Robots prêtés sans carte communicante | Élevé | Clarifier tôt avec les tuteurs (H2, H5) ; ADR sur le moyen de communication. |
| Nombre de robots fonctionnels insuffisant | Élevé | Viser le MVP à 2 robots. |
| Connexion Wi-Fi lente au démarrage (plusieurs secondes selon S03) | Moyen | Mesurer ce délai en EXP ; ne pas lancer la chorégraphie avant que tous les robots soient prêts [@S03, §6.8]. |
| Réseau de la salle qui isole les appareils | Moyen | Prévoir notre propre point d'accès ; vidéo de secours [@S03, §9.2 ; @S01, slide 10]. |
| Tremblements des servos avec la RP2040 | Moyen | À reproduire et documenter en EXP avant tout choix de carte [@S03, §6.8]. |
| Écarts entre documentation et code (noms, positions de référence, broches) | Moyen | Suivre la hiérarchie des sources ; trancher par un test (voir état de l'art §3.4). |
| Casse d'un servomoteur ou d'une patte | Élevé | Règles de sécurité : robot éteint pour programmer, garde-fou min/max, calibration faite [@S04, ch. 1 p. 7 ; @S05, ex. 3 p. 22]. |
| Absence de moyen de mesure de la synchronisation | Moyen | Méthode de mesure définie dès EXP-002. |

## 8. Questions à poser aux tuteurs
1. Combien d'araignées nous seront prêtées, et avec quelle version électronique (carte mère
   d'origine, Nano + DFR0012, Nano RP2040 Connect, Compute Module) ?
2. Où récupérer le croquis officiel `spider_sealk` ? Est-ce la même base que le firmware de l'an
   dernier ?
3. A-t-on le droit de partir de la base officielle de mouvement (Motion, contrôleur de pattes) et
   de ne réécrire que la couche communication ?
4. L'API de communication est-elle imposée (API HTTP de l'an dernier, ROS 2 présenté en
   [@S01, slide 6]…) ou à choisir ?
5. Architecture attendue : contrôle centralisé accepté, ou coordination décentralisée exigée ?
6. La vision par ordinateur est-elle attendue sur ce sujet, et sous quelle forme ?
7. Quel niveau de synchronisation serait jugé satisfaisant pour la démonstration ?
8. Peut-on utiliser le code de l'an dernier comme instrument de mesure pour EXP-002 (§6) ?
9. Dates de soutenance et jalons intermédiaires ; accès au *Skillbook Arduino – Prise en main*.
