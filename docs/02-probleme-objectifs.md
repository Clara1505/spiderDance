# 02 — Problématique et objectifs

> **Statut** : brouillon v0.4 — à valider par l'équipe
> **Dernière mise à jour** : 2026-11-17
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
| Q1 | Quelle architecture de coordination : un poste qui pilote tous les robots, des robots qui se coordonnent entre eux, ou une solution hybride ? | État de l'art §5 ; ADR-002 |
| Q2 | Comment établir et maintenir une référence de temps commune, ou un signal de départ commun ? | État de l'art §6 ; ADR-002 |
| Q3 | Quel moyen de communication ajouter au robot, sachant que la version de référence n'en a pas ? | État de l'art §3 et §7 ; ADR-003 |
| Q4 | Comment décrire une chorégraphie pour qu'elle soit jouée à l'identique par chaque robot, en s'appuyant sur les mouvements existants ? | État de l'art §2 et §4 ; ADR-005 |
| Q5 | Comment **mesurer objectivement** la synchronisation obtenue ? | État de l'art §8 ; ADR-004 ; EXP-002 et EXP-004 |

## 3. Objectifs

Les objectifs sont formulés pour être vérifiables. Les tuteurs attendent une synchronisation **à
l'œil**, avec un **recalage sur le mouvement suivant** si un décalage apparaît (échange du 2026-11-17,
relayé par l'équipe) : **aucun seuil chiffré en millisecondes n'est imposé**. La mesure sur vidéo (ADR-004)
sert de **preuve** et de **comparaison avant / après** (§6), sans seuil inventé.

### 3.1 Objectif général
Réaliser un démonstrateur dans lequel **N araignées SEALK** (N = 3 ou 4 visé ; 2 au minimum si les
robots manquent) exécutent une
chorégraphie commune, coordonnée par une API de communication, avec une synchronisation mesurée et
documentée.

### 3.2 Objectifs spécifiques
| ID | Objectif | Critère de réussite | Moyen de preuve |
|---|---|---|---|
| O1 | **Communication** : chaque ordre envoyé par l'ordinateur est reçu **et exécuté** par le robot visé. | 100 % des ordres exécutés sur `[À DÉFINIR]` chorégraphies complètes consécutives. Taux = ordres exécutés / ordres envoyés. | Vidéo et/ou journaux, fiche EXP |
| O2 | **Départ commun et recalage** : l'ordinateur lance chaque mouvement sur tous les robots en parallèle et n'envoie le suivant qu'une fois tous les robots disponibles (ADR-002). | Un décalage constaté sur un mouvement **ne se retrouve pas cumulé** sur le mouvement suivant, du début à la fin d'une chorégraphie. | Analyse vidéo image par image ; journaux pour compléter, fiche EXP |
| O3 | **Synchronisation des mouvements** : un même mouvement démarre au même instant sur tous les robots. | **Synchronisé à l'œil** (critère des tuteurs). Écart mesuré **meilleur** que la référence sans recalage (EXP-002), sans annoncer un écart plus fin qu'une image vidéo (limite de mesure). | Analyse vidéo image par image, fiche EXP |
| O4 | **Chorégraphie** : enchaînement de figures tirées des mouvements disponibles [@S04, ch. 4 p. 15-16], décrit dans un format commun. | Au moins `[À DÉFINIR]` figures distinctes jouées sans intervention manuelle. | Démonstration, vidéo |
| O5 | **Comportement collectif** : au moins une figure qui n'a de sens qu'à plusieurs (ex. vague, miroir, canon). | Figure identifiable par un observateur extérieur. | Vidéo, retour des tuteurs |
| O6 | **Reproductibilité** : un tiers peut calibrer les robots, réinstaller et relancer la démonstration, à partir du seul rapport (20 à 50 pages, sans notre code) [@S28]. | Procédure testée par un membre n'ayant pas écrit le code. | `reproductibilite.md`, rapport |

[FAIT] Un message est « reçu » seulement si l'ordre est **exécuté** : dans le code de l'an dernier, le
robot répond à la requête HTTP avant d'exécuter l'ordre, et répond aussi « ok » à une commande
inconnue [@S06, wifi.cpp l. 115-123, 130-214]. Une réponse du robot ne prouve donc ni l'exécution ni
l'heure de départ. Les journaux de l'ordinateur ne suffisent pas non plus pour O2 et O3 : chaque robot
a sa propre horloge [@S16, introduction].

[FAIT] La mesure ne peut pas être plus fine que la durée d'une image de la vidéo (environ 33 ms à
30 images par seconde, environ 17 ms à 60). Les écarts mesurés sont donc donnés **à une image près**.
`[À VÉRIFIER : cadence de la caméra]`

[HYPOTHÈSE] Les exemples de figures collectives de O5 s'inspirent de la famille « organisation
spatiale » des comportements d'essaim [@S11, résumé]. Leur faisabilité dépend des déplacements
réels des robots, à tester.

### 3.3 Priorisation (proposition)
| Priorité | Objectifs | Justification |
|---|---|---|
| Indispensable (MVP) | O1, O2, O3 avec N = 3 (2 si les robots manquent) | Cœur de la problématique : sans communication ni synchronisation, pas de sujet. |
| Visé | O4, O5, N > 2 | Rend la démonstration convaincante. |
| Bonus | Calage sur le rythme de la musique (état de l'art §4) | Hors MVP. La vision par ordinateur est écartée (décision d'équipe du 2026-11-17). |

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
- Vision par ordinateur (décision d'équipe du 2026-11-17) : la synchronisation est mesurée sans
  traitement d'image automatique (vidéo analysée à la main et/ou journaux).

## 5. Hypothèses de travail et contraintes
| ID | Hypothèse / contrainte | Statut |
|---|---|---|
| H1 | Chaque robot possède sa propre unité de calcul (Arduino Nano). | [FAIT] [@S04, ch. 1 p. 7] |
| H2 | La version de référence du robot peut communiquer à distance. | **Faux** : pas de Wi-Fi ni d'autre moyen distant [@S03, §3.4]. Un changement de carte ou un module est nécessaire. |
| H3 | Les robots disposent de mouvements programmables prêts à l'emploi. | [FAIT] Mouvements de base et avancés [@S04, ch. 4 p. 15-16] |
| H4 | Au moins trois robots fonctionnels et calibrés seront disponibles (3 ou 4 visés ; nombre exact et dates d'accès à confirmer). |
| H5 | Les robots disposent d'une carte Nano RP2040 Connect (Wi-Fi). | [FAIT — déclaré par l'équipe] module Wi-Fi NINA-W102 ; présence sur chaque robot, nombre de robots et version de chacun `[À VÉRIFIER]`. Solution utilisée l'an dernier [@S03, §6.2] |
| C1 | Contraintes génériques : puissance, taille, poids, bande passante, latence, fiabilité, coût. | [FAIT] [@S01, slide 5] |
| C2 | Livrables : PoC + codes, rapport, soutenance + démo, vidéo de secours. | [FAIT] [@S01, slide 10] |
| C3 | L'Arduino exécute les instructions de manière séquentielle. | [FAIT] [@S05, ch. 1 p. 7] |
| C4 | Aucun retour de position : l'arrivée des pattes est estimée en boucle ouverte. | [FAIT] [@S04, §2.1 p. 10] |
| C5 | On repart du code de l'an dernier (reprise tracée et testée) ; son rapport reste un guide. | Décision d'équipe du 2026-11-17 |
| C6 | Échéance : envoi des livrables avant la dernière séance (15/12 selon l'équipe, séance du 14/12 de 10 h à 13 h) ; semaine du 30/11 au 6/12 à garder légère (équipe en entreprise). | [@S28] et consigne d'équipe |
| C7 | Architecture centralisée : l'ordinateur pilote et envoie les instructions. | Confirmée par les tuteurs le 2026-11-17 (ADR-002) |

## 6. Démarche de mesure et de comparaison
1. **EXP-001 — Prise en main et calibration** : calibrer chaque araignée [@S04, ch. 3 p. 13-14],
   puis passer les tests de base et avancés [@S04, ch. 4 p. 15-16].
2. **EXP-002 — Mesure de référence** : envoyer le même ordre à deux robots sans mécanisme de
   synchronisation particulier, et mesurer l'écart de démarrage (vidéo et/ou journaux).
   Réponse du 2026-11-17 : le code de l'an dernier peut être utilisé, y compris pour cette mesure.
   La mesure se fait sans vision par ordinateur (vidéo analysée image par image et/ou journaux).
3. **EXP-004 — Mesure avec recalage** (option A+ de l'ADR-002) : même protocole, même vidéo, pour comparer
   à la référence.
4. Comparer chaque amélioration à cette référence, ce qui prouve l'apport du travail.

## 7. Risques principaux
| Risque | Impact | Parade envisagée |
|---|---|---|
| Un robot lent ou perdu du Wi-Fi retient tous les autres (recalage à chaque mouvement) | Moyen | Délai maximal par mouvement (ADR-002). |
| Robots prêtés sans carte communicante | Élevé | Clarifier tôt avec les tuteurs (H2, H5) ; ADR sur le moyen de communication. |
| Nombre de robots fonctionnels insuffisant | Élevé | Viser le MVP à 2 robots. |
| Connexion Wi-Fi lente au démarrage (plusieurs secondes selon S03) | Moyen | Mesurer ce délai en EXP ; ne pas lancer la chorégraphie avant que tous les robots soient prêts [@S03, §6.8]. |
| Réseau de la salle qui isole les appareils | Moyen | Prévoir notre propre point d'accès ; vidéo de secours [@S03, §9.2 ; @S01, slide 10]. |
| Tremblements des servos avec la RP2040 | Moyen | À reproduire et documenter en EXP avant tout choix de carte [@S03, §6.8]. |
| Écarts entre documentation et code (noms, positions de référence, broches) | Moyen | Suivre la hiérarchie des sources ; trancher par un test (voir état de l'art §3.4). |
| Casse d'un servomoteur ou d'une patte | Élevé | Règles de sécurité : robot éteint pour programmer, garde-fou min/max, calibration faite [@S04, ch. 1 p. 7 ; @S05, ex. 3 p. 22]. |
| Absence de moyen de mesure de la synchronisation | Moyen | Méthode de mesure définie dès EXP-002. |
| Calendrier serré : environ 4 semaines, équipe en entreprise, semaine du 30/11 à garder légère | Élevé | Concentrer les manipulations sur les deux semaines du 17 au 29/11 ; voir `gestion-projet.md`. |
| Identifiants Wi-Fi écrits en dur dans le firmware de l'an dernier [@S06, wifi.h l. 9-12], alors que le dépôt GitHub est public | Moyen | Ne jamais committer de vrais identifiants : fichier local ignoré par git, valeurs factices dans le dépôt. |

## 8. Questions du 2026-11-17 : réponses de l'équipe
| # | Question | Réponse de l'équipe | Statut |
|---|---|---|---|
| 1 | Nombre d'araignées et version électronique de chacune ? | Carte **Nano RP2040 Connect** déclarée (module Wi-Fi NINA-W102). Nombre de robots et présence de cette carte sur chacun : à confirmer | Partiel |
| 2 | Où récupérer le croquis officiel `spider_sealk` ? | Un croquis est un programme Arduino. On repart du firmware déjà en main (S06) ; sa provenance exacte reste à vérifier (non bloquant) | Réglé |
| 3 | Peut-on partir de la base existante sans tout réécrire ? | Oui, on repart de ce qu'on a | Réglé |
| 4 | API de communication imposée ou libre ? | Choisie par l'équipe (ADR à écrire) | Réglé |
| 5 | Architecture centralisée ou décentralisée ? | Centralisée : l'ordinateur pilote et envoie les instructions. Confirmé par les tuteurs le 2026-11-17 | Réglé |
| 6 | Vision par ordinateur attendue ? | Non | Réglé |
| 7 | Niveau de synchronisation jugé satisfaisant ? | Synchronisé à l'œil, avec recalage sur le mouvement suivant en cas de décalage (tuteurs, 2026-11-17) | Réglé |
| 8 | Code de l'an dernier utilisable comme instrument de mesure ? | Oui | Réglé |
| 9 | Dates et jalons ; *Skillbook Arduino* ? | Tous les skill books sont obtenus. Rendu pour le 15/12 ; séance du 14/12 de 10 h à 13 h. Date de soutenance à confirmer | Partiel |

### Questions restantes pour les tuteurs
1. Date, heure et ordre de passage de la soutenance.
2. À quelles dates peut-on manipuler les robots (3 ou 4 visés) ?
3. Quels fichiers du firmware viennent du croquis officiel des tuteurs, et lesquels ont été écrits par l'équipe précédente ?