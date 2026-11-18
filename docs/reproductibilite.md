# Reproductibilité

> **Statut** : squelette v0.2 — à compléter au fil des essais, à valider par l'équipe
> **Dernière mise à jour** : 2026-11-18
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé · `[À COMPLÉTER]` information à fournir après les essais
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`

Objectif : permettre à un tiers de **refaire le projet de zéro, sans connaissances préalables et sans notre
code** [@S28]. Ce fichier rassemble ce qu'il faut pour cela ; le rapport final le reprend en le
**décrivant**, pas seulement en le citant.

Rien dans ce fichier ne doit contenir de vrais identifiants (Wi-Fi, mots de passe) : le dépôt est public.

## 1. Matériel

| Élément | Quantité | Détail | Statut |
|---|---|---|---|
| Araignées SEALK | 3 ou 4 visées | 4 pattes × 3 servomoteurs [@S04, ch. 1 p. 7] ; robots à domicile (déclaré par l'équipe, 2026-11-18) | Nombre réel `[À COMPLÉTER]` (inventaire, issue M1a) |
| Carte de contrôle de chaque robot | une par robot | Arduino Nano RP2040 Connect déclarée par l'équipe (module Wi-Fi NINA-W102) | `[À VÉRIFIER sur chaque robot]` |
| Alimentation | une par robot | Batterie ; version selon le robot [@S04, ch. 1 p. 7 ; @S03, §3.2] | `[À COMPLÉTER]` |
| Mire de calibration, règle, plots | — | Pour la calibration et les essais robot allumé [@S04, ch. 3 ; @S05, ch. 1 p. 8] | Matériel des tuteurs |
| Ordinateur (poste de contrôle) | 1 | Système, versions de Python, Docker, Arduino IDE | `[À COMPLÉTER]` |
| Point d'accès Wi-Fi | 1 | Réseau local dédié, sans isolation des appareils [@S03, §9.2] | `[À COMPLÉTER]` |
| Caméra | 1 | Cadence en images par seconde | `[À COMPLÉTER]` |

## 2. Inventaire et calibration par robot

Un tableau par robot, rempli pendant EXP-001 (calibration [@S04, ch. 3 p. 13-14]).

| `SPIDER_ID` | Nom / couleur | Carte | Valeurs min/max des servomoteurs (couvercle de la boîte) | Corrections (R2, R1, L1, L2) | Test de base | Test avancé |
|---|---|---|---|---|---|---|
| `[À COMPLÉTER]` | | | | | | |

## 3. Logiciel : ce qui est repris, modifié ou écrit par nous

Classement exigé par la décision de reprise du code (ADR-001). Les lignes « à déterminer » se remplissent au
fur et à mesure.

| Élément | Source | Statut | Modification prévue | Référence |
|---|---|---|---|---|
| Firmware du robot (mouvements, calibration, trajectoire) | [@S06] | Repris ; modifications `[À DÉTERMINER]` | Éventuelle remise à zéro de `traj_length` si l'hypothèse se confirme | ADR-003, EXP-003 |
| Serveur Wi-Fi / HTTP du robot | [@S06, wifi.cpp, wifi.h] | Repris et **modifié** | Erreur 400 pour une commande inconnue ; port défini à un seul endroit ; identifiants sortis du code ; réponse après exécution si le premier test échoue | ADR-003 |
| Superviseur multi-robots | [@S07] | Repris | `[À DÉTERMINER]` | ADR-002 |
| Client en ligne de commande | [@S08] | Repris | `[À DÉTERMINER]` | — |
| Envoi parallèle et recalage avant chaque mouvement | — | **Écrit par nous** | — | ADR-002 |
| Lecteur de fichier de chorégraphie, vérifications, mode à blanc | — | **Écrit par nous** | — | ADR-005 |
| Fichier de chorégraphie | — | **Écrit par nous** | — | ADR-005 |
| Protocole de mesure sur vidéo, fiches de relevé | — | **Écrit par nous** | — | ADR-004 |

Crédits : rapport de l'an dernier par Célian Crépin et Maxence Dupont [@S03] ; tout le code fourni (S06 à S08)
est de Célian Crépin et Maxence Dupont (déclaré par l'équipe, 2026-11-18). Selon le rapport, le Wi-Fi et
l'interface de pilotage sont de Maxence Dupont [@S03, §1.4]. L'auteur de chaque fichier n'est pas indiqué
dans le code : la répartition fichier par fichier reste inconnue (non bloquant).

## 4. Procédure de mise en route (à compléter après les essais)

> Chaque étape doit être **testée par un membre de l'équipe qui n'a pas écrit le code** avant d'être
> déclarée reproductible (objectif O6).

**Règles de sécurité** (à rappeler à chaque manipulation) :
- ne jamais programmer l'araignée allumée [@S04, ch. 1 p. 7 ; @S05, ch. 1 p. 8] ;
- poser l'araignée sur les plots quand elle est allumée et reliée au câble [@S05, ch. 1 p. 8] ;
- tester dans un espace dégagé [@S04, ch. 4 p. 15] ;
- respecter la plage min/max de chaque servomoteur [@S05, ex. 3 p. 22].

| # | Étape | Commande / action | Résultat attendu | Statut |
|---|---|---|---|---|
| 1 | Installer l'environnement de programmation | `[À COMPLÉTER]` | | |
| 2 | Calibrer un robot (par `SPIDER_ID`) | Voir [@S04, ch. 3 p. 13-14] | Pointes de pattes à ± 5 mm | |
| 3 | Créer le fichier local d'identifiants Wi-Fi (non versionné) | `[À COMPLÉTER]` | Robot connecté au réseau | |
| 4 | Téléverser le firmware (robot éteint) | `[À COMPLÉTER]` | | |
| 5 | Lancer le poste de contrôle | `[À COMPLÉTER]` | `GET /api/health` répond | |
| 6 | Jouer une chorégraphie en mode à blanc | `[À COMPLÉTER]` | Étapes affichées sans envoi | |
| 7 | Jouer la chorégraphie sur les robots | `[À COMPLÉTER]` | Robots synchronisés à l'œil | |
| 8 | Filmer et relever les écarts | Protocole de l'ADR-004 | Tableau de relevé | |

## 5. Versions

| Composant | Version utilisée | Statut |
|---|---|---|
| Arduino IDE / cœur de carte | `[À COMPLÉTER]` | |
| Bibliothèques (WiFiNINA, ArduinoQueue…) | `[À COMPLÉTER]` | |
| Python | `[À COMPLÉTER]` (S07 : Python 3.11 [@S07, backend/Dockerfile]) | |
| Docker / Docker Compose | `[À COMPLÉTER]` | |
| Navigateur, outil de lecture vidéo image par image | `[À COMPLÉTER]` | |

## 6. Fichiers de configuration locaux (non versionnés)

| Fichier | Contenu | Dans le dépôt |
|---|---|---|
| Identifiants Wi-Fi du firmware | SSID, mot de passe, nom d'hôte | Valeurs **factices** seulement |
| Adresses et ports des robots | Une adresse par `SPIDER_ID` | Modèle avec valeurs factices seulement |
| Fichier `.env` du superviseur | Réglages du backend | Modèle `.env.example` seulement |

## 7. Résultats de référence

Les mesures (EXP-002, EXP-004) et leurs données sont dans `docs/05-experimentations/` et
`docs/06-validation-resultats.md`. `[À COMPLÉTER]`

## Sources

- [@S03, §1.4, §3.2, §9.2] ; [@S04, ch. 1, 3 et 4] ; [@S05, ch. 1 p. 8 et ex. 3 p. 22] ; [@S06] ; [@S07] ;
  [@S08] ; [@S28].
- ADR-001 à ADR-005.
