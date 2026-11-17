# ADR-001 — Reprise du code de l'année précédente comme base de départ

**Date** : 2026-11-17
**Statut** : Proposé (décision d'équipe du 2026-11-17, à valider par écrit par l'équipe)
**Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
**Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé

---

## Contexte

Le sujet demande de coordonner plusieurs robots araignées par une API de communication [@S02]. Un
code existe déjà, produit l'an dernier :

- [FAIT] **S06** : firmware Arduino de l'araignée, avec modes Wi-Fi et radio [@S06].
- [FAIT] **S07** : superviseur multi-robots (backend Flask, frontend React/Vite) [@S07].
- [FAIT] **S08** : client en ligne de commande Python [@S08].

Ce code a été lu en entier, mais **jamais exécuté sur nos robots**. Le rapport de l'an dernier [@S03] est
un guide : ses affirmations ne sont pas des mesures sur nos robots.

Contraintes qui pèsent sur la décision :

- **Temps** : 27 jours entre le 17/11 et le 14/12, équipe de 4 en entreprise, semaine du 30/11 au 06/12
  à garder légère [@S28 ; `docs/gestion-projet.md` §1].
- **Matériel** : carte Arduino Nano RP2040 Connect déclarée par l'équipe, à confirmer sur chaque robot
  `[À VÉRIFIER]`. Le Wi-Fi du firmware S06 n'est compilé que pour cette carte [@S06, wifi.h l. 3-6].
- **Livrable** : le rapport doit permettre de reproduire le projet **sans notre code** [@S28]. Tout
  élément repris doit donc être **décrit** dans le rapport, pas seulement cité.
- **Évaluation** : les choix doivent être raisonnés et justifiés [@S01, slide 10].

## Options envisagées

| Option | Avantages | Inconvénients | Source |
|---|---|---|---|
| **A. Tout réécrire** à partir du seul croquis officiel `spider_sealk` | Maîtrise complète du code ; pas de dette héritée | Le lien de téléchargement du croquis officiel n'est pas donné `[À VÉRIFIER]` ; il faudrait réécrire aussi le Wi-Fi, l'API et le superviseur ; coût en temps élevé pour 27 jours | [@S04, ch. 2 p. 9] |
| **B. Repartir de S06 à S08** (tracé, testé, crédité) | Wi-Fi, serveur HTTP, file de commandes par robot et superviseur multi-robots déjà écrits ; calibration par `SPIDER_ID` déjà en place pour 17 araignées ; temps gagné pour la partie qui nous est propre (coordination) | Code non testé sur nos robots ; hypothèses de défauts à vérifier ; documentation désynchronisée ; identifiants en dur ; auteur de chaque fichier non indiqué dans le code | [@S06 ; @S07 ; @S08] |
| **C. Repartir de S06 à S08 pour le firmware, réécrire le poste de contrôle** | Firmware éprouvé en partie ; poste de contrôle taillé pour notre départ commun | Réécriture du superviseur sans preuve qu'il faille la faire ; à décider après EXP-002 | [@S07] |

## Décision

**Option B** : on repart de S06, S07 et S08 comme base de développement. Cette décision est prise par
l'équipe le 2026-11-17. L'option C reste ouverte : si une mesure ou un test montre que le superviseur
S07 ne convient pas, une partie pourra être réécrite, avec un nouvel ADR qui remplacera celui-ci
pour ce point.

## Justification

1. [FAIT] Le code couvre déjà ce que le sujet demande en amont de la coordination : Wi-Fi, serveur HTTP
   embarqué, commandes de mouvement et de « show », envoi à plusieurs robots [@S06, wifi.cpp l. 83-214 ;
   @S07, app.py l. 181-203 et 214-261].
2. [FAIT] Ce qui manque est précisément notre contribution : **aucun départ synchronisé** entre robots
   [@S07, app.py l. 181-203] et **aucune mesure** de synchronisation [@S03, §6.8]. Repartir de l'existant
   concentre le temps sur ce manque.
3. [FAIT] Le firmware gère déjà un jeu de corrections et de limites min/max par araignée
   (`SPIDER_ID` 0 à 16) [@S06, corrections.h l. 11-33], ce qui correspond à la méthode des skill books
   pour plusieurs araignées [@S04, ch. 3 p. 14].
4. [FAIT] Le calendrier est court et l'équipe est en entreprise [`docs/gestion-projet.md` §1].
5. Le rapport S03 propose lui-même les pistes qui prolongent ce code : séquences, délais entre commandes,
   macros [@S03, §10.2].

## Conséquences / risques

**Règles qui accompagnent la décision** (règles de reprise du code, décision d'équipe du 2026-11-17) :

- **Tracer** : chaque élément est classé *repris tel quel / repris et modifié / écrit par nous*, dans
  `docs/reproductibilite.md`, puis repris dans le rapport.
- **Créditer** : les auteurs du rapport de l'an dernier sont Célian Crépin et Maxence Dupont [@S03]. Selon
  ce rapport, le travail sur le Wi-Fi (adaptation du code embarqué, API de commande, interface de
  pilotage à distance) est celui de Maxence Dupont, et le Compute Module celui de Célian Crépin
  [@S03, §1.4]. Les fichiers S06 à S08 ne portent pas de nom d'auteur dans ce que nous avons lu ; on
  ne sait donc pas, fichier par fichier, ce qui vient de Maxence, de Célian ou des tuteurs (le
  firmware reprend l'architecture du croquis officiel [@S04, §2.1]). `[À VÉRIFIER]` : confirmer
  auprès des tuteurs, avec la liste des fichiers.
- **Tester avant de s'y fier** : les constats de lecture sont des hypothèses tant qu'ils ne sont pas
  testés en fiche EXP.
- **Décrire dans le rapport** : rôle, paramètres et procédure de chaque élément repris, car le rapport
  doit se suffire à lui-même [@S28].
- **Contribution propre visible** : séparer « existant repris » et « notre contribution » (coordination,
  départ commun, chorégraphie, mesure).

**Risques identifiés** :

| Risque | Source | Parade |
|---|---|---|
| `[HYPOTHÈSE]` `deep_sleep` n'a aucun effet dans `Body::process` | [@S06, body.h l. 59-129] | EXP-003 |
| `[HYPOTHÈSE]` `traj_length` jamais remis à zéro : ordres ignorés au-delà de 60 | [@S06, trajectory.h l. 45-47, 81-83] | EXP-003 |
| `[HYPOTHÈSE]` Rejeu de la trajectoire accumulée après `DONE` | [@S06, spider_SEALK.ino l. 116-131] | EXP-003 |
| `[HYPOTHÈSE]` La trajectoire de `trajectory.cpp` serait jouée au démarrage en mode Wi-Fi | [@S06, spider_SEALK.ino l. 79] | EXP-003 |
| Documentation (`doc/readme.md`) désynchronisée du code | [@S06, doc/readme.md l. 46-80] | Se fier au code ; vérifier par test |
| Écarts entre sources (GET/POST, port 2020/2021, positions de référence, broches) | `docs/03-etat-de-l-art.md` §3.4 | Trancher par test, jamais par supposition |
| Identifiants Wi-Fi écrits en dur alors que le dépôt est **public** | [@S06, wifi.h l. 9-12] | Fichier local ignoré par git ; valeurs factices dans le dépôt |
| Tremblements des servos avec la RP2040 | [@S03, §6.8] | Vérifier en EXP-001 |
| Mode radio : compilation douteuse | [@S06, radio.cpp l. 5 et 44] | Hors périmètre sauf besoin |

**Ce que cette décision ne tranche pas** : l'architecture de coordination, le moyen de communication
retenu, le mécanisme de départ commun, le format de chorégraphie et la méthode de mesure. Chacun fera
l'objet de son propre ADR.

## Sources

- [@S02] Énoncé du sujet.
- [@S03, §1.4] Répartition du travail entre Célian Crépin et Maxence Dupont.
- [@S03, §6.8 et §10.2] Limites et perspectives de l'équipe précédente.
- [@S04, ch. 2 p. 9 ; ch. 3 p. 14] Croquis officiel et calibration par araignée.
- [@S06] Firmware Arduino de l'an dernier (wifi.cpp, wifi.h, corrections.h, body.h, trajectory.h,
  spider_SEALK.ino, radio.cpp, doc/readme.md).
- [@S07, app.py] Superviseur multi-robots.
- [@S08] Client en ligne de commande.
- [@S28] Consignes PROJEX (livrables, rapport reproductible sans le code).
- `docs/gestion-projet.md` §1 (calendrier) ; `docs/03-etat-de-l-art.md` §3.4 (écarts entre sources).
