# 03 — État de l'art

> **Statut** : brouillon v0.2 — à valider par l'équipe
> **Dernière mise à jour** : 2026-09-29
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
>
> ⚠️ **Niveau de lecture** : les skill books (S04, S05), le rapport (S03) et le code (S06, S07) de
> l'an dernier ont été lus en entier. Pour les sources externes (S09 et suivantes), seuls les
> résumés, introductions ou extraits indiqués dans `sources.bib` ont été lus. Elles doivent être
> lues en entier par un membre de l'équipe avant la version finale du rapport.

---

## 1. Méthode

- **Période** : recherches du 2026-09-23 au 2026-09-29.
- **Sources** : documents fournis par les tuteurs (S04, S05), travaux de l'an dernier (S03, S06,
  S07), publications scientifiques et documentations officielles (S09 et suivantes).
- **Hiérarchie** retenue pour tout ce qui concerne le robot : skill books (S04, S05) > code
  officiel `spider_sealk` > code de l'an dernier (S06–S08) > rapport de l'an dernier (S03). Seul un
  test sur nos robots dit ce qu'ils font réellement.
- **Axes**, dérivés des sous-questions de `02-probleme-objectifs.md` :

| Section | Sujet | Question liée |
|---|---|---|
| §2 | Plateforme SEALK | toutes |
| §3 | Travaux antérieurs sur ce robot | Q1, Q3 |
| §4 | Chorégraphie multi-robots et techniques transposables | Q2, Q4 |
| §5 | Architectures de coordination | Q1 |
| §6 | Synchronisation temporelle | Q2 |
| §7 | Moyens de communication | Q3 |
| §8 | Mesure par vision | Q5 |

---

## 2. La plateforme : l'araignée SEALK

### 2.1 Matériel
- [FAIT] 4 pattes, 3 servomoteurs par patte (hanche, fémur, tibia), alimentation par 2 piles
  Li-Ion (2 × 3,7 V) distribuée par la carte mère aux 12 servomoteurs et à l'Arduino Nano
  [@S04, ch. 1 p. 7].
- [FAIT] Segments de patte : a = 53 mm, b = 79,5 mm, c = 30,5 mm [@S05, §2.1.4 p. 13].
- [FAIT] Chaque servomoteur a une plage min/max propre, différente d'un moteur à l'autre et d'une
  araignée à l'autre ; les valeurs sont inscrites sur le couvercle de la boîte [@S05, ex. 3 p. 22].
- [FAIT] La carte mère d'origine n'est plus produite. L'an dernier, elle a été remplacée par une
  Arduino Nano, un shield DFR0012 et des modules d'alimentation séparés [@S03, §2.1 et §2.4].
  `[À VÉRIFIER : version de nos robots]`

### 2.2 Architecture du code officiel `spider_sealk`
[FAIT] Le code est organisé en couches, de la plus haute à la plus basse [@S04, §2.1 p. 9-11] :

| Couche | Rôle |
|---|---|
| Trajectory | Liste des tâches de haut niveau, écrite dans `Trajectory::setup()` |
| Body / Motion | Décompose chaque mouvement en positions successives des bouts de pattes |
| ArmController | File d'ordres (FIFO) ; fixe la vitesse de chaque patte et synchronise les pattes entre elles, par estimation en boucle ouverte |
| Arm / Kinematics | Points intermédiaires à vitesse constante, cinématique inverse, corrections de calibration |
| Motors | Manipule les 12 servomoteurs comme une liste |

[FAIT] Le code complet gère la vitesse (du robot et des servomoteurs), l'enchaînement temporel des
ordres et la stabilité du robot pendant la marche [@S05, ch. 7 p. 39].

### 2.3 Mouvements disponibles : une bibliothèque de primitives
[FAIT] Mouvements de base : `stand`, `sit`, `step_forward(n)`, `step_back(n)`, `turn_left(n)`,
`turn_right(n)`. Mouvements avancés : `crab_left`, `crab_right`, `rave`, `flex`, `say_hello`,
`applaud_sym`, `applaud_wave`, `tap_leg`, `sputnik`, `scratch`, `pls`, `bolting`, `wings`
[@S04, ch. 4 p. 15-16].

[FAIT] À la fin de chaque séquence de marche ou de rotation, le robot reprend sa pose de départ :
sa position initiale est donc toujours connue [@S05, §5.3 p. 26].

### 2.4 Calibration
[FAIT] Chaque araignée doit être calibrée : pose (100, 70, 42), mesure de la position de chaque
bout de patte sur la mire, report dans `corrections.h`, vérification à ± 5 mm [@S04, ch. 3
p. 13-14]. [FAIT] Un même code peut servir à plusieurs araignées grâce à un jeu de corrections par
identifiant `SPIDER_ID` [@S04, ch. 3 p. 14].

### 2.5 Limites de la plateforme pour notre sujet
| Limite | Source | Conséquence pour SpiderDance |
|---|---|---|
| La séquence est écrite dans `Trajectory::setup()` et jouée au démarrage | [@S04, §2.1 p. 10] | Pas de déclenchement extérieur : le départ commun est à construire. |
| L'Arduino est séquentielle ; la simultanéité des pattes est une illusion | [@S05, ch. 1 p. 7] | Un robot occupé à bouger ne peut pas faire autre chose en même temps. `[HYPOTHÈSE : à confirmer sur la réception de messages]` |
| Arrivée des pattes estimée en boucle ouverte, sans capteur | [@S04, §2.1 p. 10] | Aucun robot ne sait où il en est réellement : pas de retour pour corriger un retard. |
| Aucun moyen de communication à distance dans la version de référence | [@S03, §3.4] | Un changement de carte ou un module est nécessaire (voir §7). |
| Synchronisation des pattes d'**un** robot seulement | [@S04, §2.1 p. 10] | La coordination **entre** robots est entièrement à construire. |

---

## 3. Travaux antérieurs sur ce robot (2025–2026)

> Statut : **guide uniquement**. Ces travaux orientent nos choix ; rien n'est repris dans notre
> dépôt, et toute observation utilisée sera d'abord reproduite dans une fiche EXP.

### 3.1 Ce qui a été tenté
**Axe 1 — Compute Module** [@S03, §5]
- [FAIT] Un module de calcul sous Raspberry Pi OS Lite est ajouté au-dessus du shield ; il envoie
  des caractères à l'Arduino par USB ou par liaison série (UART) pour déclencher des mouvements
  [@S03, §5.5 et §5.6].
- [FAIT] Limites déclarées : alimentation partagée avec les servomoteurs, niveaux logiques 3,3 V /
  5 V sans convertisseur, pas de protocole structuré ; solution qualifiée d'expérimentale
  [@S03, §5.8].

**Axe 2 — Wi-Fi avec Arduino Nano RP2040 Connect** [@S03, §6]
- [FAIT] L'Arduino Nano est remplacée par une Nano RP2040 Connect, ce qui impose de refaire la
  correspondance des broches [@S03, §6.2].
- [FAIT] Le robot expose un petit serveur HTTP ; un backend Python peut déclarer plusieurs robots
  et leur envoyer des ordres, avec une file de commandes par robot [@S03, §6.3 et §6.4].

**Mode radio** (présent dans le code, non décrit dans le rapport)
- [FAIT] Le firmware prévoit un mode radio nRF24L01 qui ne traite que 6 ordres : FORWARD,
  BACKWARD, TURN_LEFT, TURN_RIGHT, STAND, DONE [@S06, radio.h l. 48-49 ; radio.cpp l. 30-49].
- [HYPOTHÈSE] Ce mode ne compilerait pas en l'état : `radio.cpp` est protégé par une constante
  définie seulement dans le fichier principal, et `stand()` y est appelée sans argument
  [@S06, radio.cpp l. 5 et 44 ; trajectory.h l. 52]. À vérifier en compilant.

### 3.2 Ce que révèle le code (S06, S07)
| Constat | Source | Intérêt pour SpiderDance |
|---|---|---|
| [FAIT] La trajectoire est un tableau fixe de 60 entrées {type, répétitions, durées, patte, vitesse} | [@S06, trajectory.h l. 21-30] | Format simple de chorégraphie stockée à bord. |
| [FAIT] Des mouvements supplémentaires existent : `shuriken`, `skater`, `zouk`, `walk_dominance` | [@S06, motion.h l. 123-126] | Primitives de plus, à valider sur nos robots. |
| [FAIT] Chaque entrée attend l'arrivée **estimée** des pattes (interpolation sans capteur), puis 300 ms de pause | [@S06, body.h l. 55-134 ; leg.h l. 45-65] | Durée réelle d'une figure non garantie : à mesurer. |
| [FAIT] Le robot répond à la requête HTTP **avant** d'exécuter l'ordre | [@S06, wifi.cpp l. 115-123 ; spider_SEALK.ino l. 123] | La réponse ne prouve pas que le mouvement a commencé. |
| [FAIT] Le superviseur envoie l'ordre à chaque robot indépendamment, par un fil d'exécution et une file par robot | [@S07, app.py l. 35-141 et 181-203] | **Pas de départ synchronisé** : c'est le cœur de notre sujet. |
| [FAIT] Une « JPO DANCE DEMO » décale deux araignées dans le temps par des pauses `deep_sleep` différentes, jouées depuis l'allumage | [@S06, trajectory.cpp l. 94-158] | Première tentative de chorégraphie à plusieurs, **sans communication**. |
| [HYPOTHÈSE] `deep_sleep` n'a aucun effet : aucun cas ne le traite dans `Body::process` | [@S06, body.h l. 59-129] | Le décalage de la démo JPO ne fonctionnerait pas comme prévu. |
| [HYPOTHÈSE] Après l'ordre `DONE`, toute la trajectoire accumulée serait rejouée : le compteur de lecture est remis à zéro, mais pas la longueur | [@S06, spider_SEALK.ino l. 116-131 ; trajectory.h l. 45-47] | Comportement à éviter dans notre conception. |
| [HYPOTHÈSE] Au-delà de 60 ordres cumulés, les nouveaux seraient ignorés sans erreur | [@S06, trajectory.h l. 81-83] | Idem. |

### 3.3 Limites déclarées par l'équipe précédente
- [FAIT] Le fonctionnement avec plusieurs robots physiques connectés en même temps reste à préciser
  et à tester [@S03, §6.8].
- [FAIT] La carte peut mettre plusieurs secondes à rejoindre le réseau Wi-Fi [@S03, §6.8].
- [FAIT] Robot et backend doivent être sur le même réseau local ; certains réseaux isolent les
  clients [@S03, §9.2].
- [FAIT] Tremblements et mouvements saccadés observés avec la RP2040 [@S03, §6.8].
- [FAIT] Aucune mesure chiffrée de latence ou de synchronisation n'est fournie dans le rapport
  (constat de lecture de [@S03]).

### 3.4 Écarts entre sources, à trancher par un test
| Point | Skill books / rapport | Code de l'an dernier |
|---|---|---|
| Méthode de `/api/command` | `POST` + JSON [@S03, §6.6] | `GET ?cmd=…&n=…` [@S06, wifi.cpp l. 84-124] |
| Port du robot | 2020 (interface) [@S07, frontend/src/App.tsx l. 14] | 2021 [@S06, wifi.h l. 14] |
| Drapeau de vérification | `VERIF` [@S04, ch. 3 p. 14] | `VERIFY` [@S06, corrections.h l. 13] |
| Positions de référence x / y / z levée | 70 / 50 / −20 [@S05, ex. 8-9 p. 26, Q17] | 75 / 55 / −15 [@S06, spider_model_sealk.h l. 9-14] |
| Broches des servomoteurs | [@S05, ch. 1 p. 8] | autre correspondance active [@S06, spider_SEALK.ino l. 27-32] |
| Décalage en z de la calibration | 42 − 27 = 15 [@S06, doc/readme.md l. 117] | commentaire « TODO -12 on Z don't know why » [@S06, kinematics.h l. 32 et 35] |

---

## 4. Chorégraphie multi-robots et techniques transposables

### 4.1 Travaux identifiés
**Augugliaro, Schoellig, D'Andrea (2013) — *Dance of the Flying Machines*** [@S17, § à préciser]
- [FAIT] Les trajectoires sont prédéfinies et paramétrées, formant une bibliothèque de
  **primitives de mouvement** que l'utilisateur enchaîne et associe aux sections de la musique.
- [FAIT] Le rythme de la musique est extrait par un logiciel d'extraction de battements, puis les
  mouvements périodiques sont calés automatiquement sur ce rythme.
- [FAIT] Avant le vol, la **faisabilité** des trajectoires est vérifiée numériquement au regard des
  contraintes des moteurs et des capteurs, puis simulée.
- [FAIT] Pendant l'exécution, le décalage de phase est compensé par des **corrections calculées à
  l'avance** à partir d'une identification hors ligne, complétées par une **correction en ligne**
  des erreurs restantes.
- [FAIT] Les transitions entre primitives sont planifiées sans collision par optimisation
  (programmation convexe séquentielle).

**Preiss et al. (2017) — *Crazyswarm*, essaim de 49 petits drones** [@S27]
- [FAIT] L'essentiel du calcul est fait **à bord** (fusion de capteurs, commande, une partie de la
  planification) ; la position est suivie avec une erreur moyenne inférieure à 2 cm [@S27, résumé].
- [FAIT] La station de base envoie la **description complète de la trajectoire** au drone ; comme
  le plan complet est stocké à bord, le système résiste à de fortes pertes de paquets radio
  [@S27, §IV].
- [FAIT] La **diffusion à tous** (*broadcast*) sert à obtenir un comportement synchronisé pour le
  décollage, l'atterrissage ou le **départ d'une trajectoire** ; ces ordres collectifs ne
  demandent pas d'accusé de réception mais sont **répétés plusieurs fois** [@S27, §VIII].
- [FAIT] Latence mesurée : 26 ms pour l'essaim complet de 49 drones [@S27, §XI-A]. Ce chiffre vaut
  pour leur liaison radio et ne dit rien de nos robots.

**Du et al. (2019) — *Fast and in sync*** [@S15, résumé]
- [FAIT] Bibliothèque de primitives de mouvement paramétrables avec peu de paramètres, et
  **algorithme de correction** des retards de mouvement pour synchroniser 25 quadricoptères sur un
  motif périodique.

**Cappo et al. (2018) — performance théâtrale multi-robots** [@S18, résumé]
- [FAIT] Les intentions d'un performeur sont traduites en trajectoires pour des ensembles de robots,
  sans connaître à l'avance l'ordre ni le moment des mouvements, par un **planificateur central**.

**Christiansen (2018) — langage de script pour robots non humanoïdes** [@S14, résumé]
- [FAIT] Le comportement commun des robots est entièrement déterminé par un **script** ; leur seule
  autonomie est la correction des imprécisions pendant la performance. Travail préliminaire,
  première implémentation sur de petits robots Lego Mindstorms.

**SwarmGPT (2025) / Swarm-GPT (2023)** [@S12 ; @S13]
- [FAIT] Génération de chorégraphies de drones à partir du langage naturel, en séparant la
  conception chorégraphique de la planification de mouvement sûre ; validée en simulation jusqu'à
  200 drones et en réel jusqu'à 20 [@S12, résumé]. Hors périmètre pour nous.

### 4.2 Techniques transposables aux araignées SEALK
| # | Technique (drones) | Source | Transposition aux araignées | Intérêt |
|---|---|---|---|---|
| T1 | Trajectoire complète stockée à bord, départ donné par un message diffusé à tous et répété | [@S27, §IV et §VIII] | La séquence est déjà stockée à bord [@S04, §2.1 p. 10 ; @S06, trajectory.h l. 21-30]. `[HYPOTHÈSE]` Il suffirait qu'elle attende un signal « départ » au lieu de démarrer à l'allumage. Alternative à l'envoi des ordres un par un [@S07, app.py l. 106-141]. | **Fort** |
| T2 | Bibliothèque de primitives paramétrées, enchaînées et associées à la musique | [@S17 ; @S15, résumé] | Les primitives existent [@S04, ch. 4 p. 15-16]. `[HYPOTHÈSE]` Une chorégraphie devient un enchaînement horodaté de primitives. | **Fort** |
| T3 | Correction du décalage : identification hors ligne, puis correction en ligne | [@S17 ; @S15, résumé] | Aucun retour de position [@S04, §2.1 p. 10]. `[HYPOTHÈSE]` Mesurer la durée réelle de chaque primitive sur chaque robot (fiche EXP), compenser, et prévoir des points de resynchronisation. | **Fort** |
| T4 | Vérification de faisabilité avant exécution, puis simulation | [@S17] | `[HYPOTHÈSE]` Vérifier chaque chorégraphie avant téléversement : plages min/max des servomoteurs [@S05, ex. 3 p. 22], taille maximale de la trajectoire. Pas de simulateur connu. | Moyen |
| T5 | Calage sur le rythme de la musique par extraction des battements | [@S17] | Bonus, hors MVP. | Bonus |
| T6 | Planification sans collision par optimisation | [@S17] | Peu de robots, lents, au sol : un placement prévu à l'avance suffit. | Faible |

**Différence majeure** : les drones cités connaissent leur position mesurée en permanence
[@S27, résumé], alors que nos araignées n'ont aucun retour de position [@S04, §2.1 p. 10]. Les
techniques qui ne demandent pas de capteur (T1, T2, T3 en version « hors ligne ») sont donc les
plus directement transposables, et une mesure externe (§8) prend de la valeur.

### 4.3 Lacune observée
[FAIT] Dans les sources consultées, la chorégraphie synchronisée multi-robots est surtout étudiée
sur des **drones**. Notre recherche, limitée, n'a pas identifié de publication décrivant une
chorégraphie synchronisée d'un groupe de **robots à pattes** ; la seule tentative connue sur nos
robots (démo JPO) ne comporte pas de communication [@S06, trajectory.cpp l. 94-158]. Cela ne
prouve pas qu'il n'en existe aucune.

---

## 5. Architectures de coordination

[FAIT] Une revue de la coordination multi-robots distingue deux architectures de décision :
**centralisée** et **décentralisée** [@S19, résumé].

[FAIT] La robotique en essaim privilégie des règles simples et des **interactions locales** pour
obtenir des comportements collectifs robustes et capables de passer à l'échelle [@S09, résumé].

| Critère | Centralisée (un poste pilote tous les robots) | Décentralisée (les robots se coordonnent entre eux) |
|---|---|---|
| Simplicité | `[HYPOTHÈSE]` Plus simple | `[HYPOTHÈSE]` Plus complexe |
| Point unique de défaillance | `[HYPOTHÈSE]` Oui (le poste) | `[HYPOTHÈSE]` Non |
| Exemples | [FAIT] Planificateur central [@S18] ; superviseur de l'an dernier [@S07, app.py] | [FAIT] Principes de la robotique en essaim [@S09 ; @S10] |
| Adéquation avec l'énoncé (« distributed robotics ») | `[HYPOTHÈSE]` Partielle | `[HYPOTHÈSE]` Forte |

[HYPOTHÈSE] Une **architecture hybride**, inspirée de T1, est envisageable : un poste diffuse la
chorégraphie et le signal de départ, puis chaque robot l'exécute seul avec sa propre horloge. Ce
choix fera l'objet d'un ADR.

---

## 6. Synchronisation temporelle

### 6.1 Le problème
[FAIT] Les horloges matérielles sont imparfaites : les horloges locales de nœuds distincts dérivent
les unes par rapport aux autres [@S16, introduction].

### 6.2 Familles de solutions
**a) Signal de départ commun diffusé à tous** [@S27, §VIII]
- [FAIT] Utilisé pour synchroniser le départ d'une trajectoire sur un essaim de drones ; ordres
  répétés plusieurs fois, sans accusé de réception.
- [HYPOTHÈSE] Ne corrige pas la dérive pendant une longue chorégraphie.

**b) Synchronisation d'horloge par échange de messages (type NTP)** [@S22, résumé]
- [FAIT] NTP synchronise les horloges d'ordinateurs sur Internet ; la version 4 peut atteindre
  quelques dizaines de microsecondes sur un réseau local rapide avec des postes modernes, et
  quelques dizaines de millisecondes sur Internet.
- `[À VÉRIFIER]` Précision atteignable sur nos cartes en Wi-Fi : à mesurer, ces chiffres ne
  s'appliquent pas directement.

**c) Protocoles pour réseaux de capteurs sans fil** [@S16, résumé]
- [FAIT] Les ressources limitées (énergie, stockage, calcul, bande passante) rendent les méthodes
  traditionnelles inadaptées à ces réseaux, d'où des algorithmes dédiés.

**d) Synchronisation émergente inspirée des lucioles**
- [FAIT] Une population d'oscillateurs couplés par impulsions évolue, pour presque toutes les
  conditions initiales, vers un état où tous « tirent » ensemble [@S20, résumé].
- [FAIT] Ce principe a été transposé aux réseaux sans fil ad hoc : chaque nœud diffuse
  périodiquement un signal et ajuste son horloge à la réception de celui des autres
  [@S21, section 1, résumant Tyrrell et al. 2006]. `[À VÉRIFIER : lire la source originale]`
- Intérêt : approche **décentralisée**, cohérente avec l'esprit de l'énoncé.

### 6.3 Comparaison
| Approche | Centralisée ? | Complexité | Intérêt pour le projet |
|---|---|---|---|
| Signal de départ diffusé | Oui | `[HYPOTHÈSE]` Faible | MVP ; à compléter pour les longues chorégraphies |
| Type NTP | Oui | `[HYPOTHÈSE]` Moyenne | Horloge commune, si le réseau le permet |
| Inspirée des lucioles | Non | `[HYPOTHÈSE]` Élevée | Vrai comportement collectif décentralisé |

---

## 7. Moyens de communication

⚠️ Le choix dépend de la carte de chaque robot, encore inconnue. Aucune option n'est retenue ici.

| Option | Condition matérielle | Ce qu'on sait | Limite connue |
|---|---|---|---|
| Wi-Fi sur Arduino Nano RP2040 Connect | Remplacer la Nano [@S03, §6.2] | [FAIT] Utilisé l'an dernier avec la bibliothèque WiFiNINA et un serveur HTTP [@S06, wifi.h l. 3-6 ; wifi.cpp l. 83-128] | Connexion de plusieurs secondes, tremblements des servos [@S03, §6.8] |
| Module de calcul + liaison série | Ajouter un Compute Module [@S03, §5] | [FAIT] Commandes simples validées par l'équipe précédente [@S03, §5.7] | Alimentation, niveaux logiques 3,3 V / 5 V [@S03, §5.8] |
| Radio nRF24L01 | Ajouter un module radio | [FAIT] Prévu dans le firmware de l'an dernier [@S06, radio.h l. 48-49] | 6 ordres seulement ; compilation douteuse (§3.1) |
| ROS 2 | Ordinateur capable de l'exécuter | [FAIT] Intergiciel robotique : nœuds, topics, services, actions [@S01, slide 6] | Présenté en cours, non imposé |
| micro-ROS | Microcontrôleur + ordinateur hôte « agent » | [FAIT] Porte ROS 2 sur microcontrôleur via Micro XRCE-DDS ; transports UDP, TCP ou série [@S24] | Le dépôt indique que le logiciel n'est pas prêt pour la production [@S24] ; compatibilité avec nos cartes `[À VÉRIFIER]` |
| ESP-NOW | Puce Espressif | [FAIT] Protocole Wi-Fi sans connexion, diffusion possible, 20 appareils appairés au maximum [@S23] | **Non applicable** à l'Arduino Nano classique ; pour la RP2040 Connect, `[À VÉRIFIER]` |

[HYPOTHÈSE] Pour T1 (§4.2), une liaison capable de **diffuser** un message à tous les robots en
une fois serait un avantage ; la solution HTTP de l'an dernier s'adresse aux robots un par un
[@S07, app.py l. 123-127].

---

## 8. Mesure de la synchronisation par vision

[FAIT] Les marqueurs fiduciaires carrés ArUco facilitent l'estimation de pose : un seul marqueur
fournit, par ses quatre coins, assez de correspondances pour obtenir la pose de la caméra, et son
codage binaire le rend robuste grâce à la détection et la correction d'erreurs [@S26, introduction].

[HYPOTHÈSE] Un marqueur ArUco sur chaque robot, filmé par une caméra fixe, permettrait de mesurer
l'instant de début de chaque mouvement et d'en déduire l'écart de synchronisation (objectif O3).
C'est d'autant plus utile que les robots n'ont aucun retour de position (§4.2). Cela relierait le
projet au thème vision du module [@S01, slide 7]. À valider avec les tuteurs.

[FAIT] L'an dernier, une caméra a été montée sur le module de calcul d'un robot [@S03, Figure 2].

### Note sur la génération du mouvement
[FAIT] Les générateurs centraux de rythme (CPG) produisent des motifs rythmiques coordonnés à partir
d'oscillateurs couplés et servent à commander la locomotion de robots articulés [@S25, résumé].
Le code SEALK repose, lui, sur la cinématique inverse et une file d'ordres [@S04, §2.1 p. 9-11 ;
@S05, §3.1 p. 15] : cette piste n'est retenue qu'à titre d'alternative.

---

## 9. Synthèse et positionnement

| Question | Ce que dit l'état de l'art | Piste pour le projet (à décider en ADR) |
|---|---|---|
| Q1 Architecture | Centralisé et décentralisé coexistent [@S19] ; le superviseur de l'an dernier est centralisé, sans départ commun [@S07, app.py l. 181-203] | Hybride : diffusion de la chorégraphie et du départ, exécution autonome (T1) |
| Q2 Temps commun | Dérive inévitable [@S16] ; départ diffusé [@S27], NTP [@S22], lucioles [@S20] | Signal de départ d'abord, puis correction (T3) |
| Q3 Communication | Pas de communication sur la version de référence [@S03, §3.4] ; plusieurs options (§7) | Dépend des cartes prêtées |
| Q4 Chorégraphie | Séquences de primitives [@S17 ; @S15] ; primitives SEALK existantes [@S04, ch. 4] | Script horodaté de primitives (T2), vérifié avant exécution (T4) |
| Q5 Mesure | Aucune mesure l'an dernier ; marqueurs ArUco [@S26] | Mesure vidéo dès EXP-002 |

**Positionnement** : le projet ne cherche pas à dépasser l'état de l'art des drones. Son apport
attendu est de **transposer** des techniques éprouvées (trajectoire stockée à bord et départ
diffusé, primitives, compensation des retards) à un groupe de **robots à pattes embarqués sans
retour de position**, de combler le manque identifié l'an dernier (pas de départ synchronisé,
aucune mesure), et de **mesurer** le résultat.
