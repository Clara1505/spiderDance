# ADR-005 — Format de description et d'exécution d'une chorégraphie

**Date** : 2026-11-17
**Statut** : Proposé (à valider par l'équipe)
**Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
**Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé · `[À DÉFINIR]` valeur à fixer par l'équipe

---

## Contexte

Il faut décrire la chorégraphie et la faire jouer par 3 ou 4 robots. L'ADR-002 impose le cadre : l'ordinateur
envoie le mouvement *n* à tous les robots, attend que tous soient disponibles, puis envoie le mouvement
*n+1* (recalage à chaque mouvement). La chorégraphie est donc une **suite d'étapes** décrite côté
ordinateur.

Éléments connus :

- [FAIT] Mouvements de base : `stand`, `sit`, `step_forward(n)`, `step_back(n)`, `turn_left(n)`,
  `turn_right(n)`. Mouvements avancés : `crab_left`, `crab_right`, `rave`, `flex`, `say_hello`,
  `applaud_sym`, `applaud_wave`, `tap_leg`, `sputnik`, `scratch`, `pls`, `bolting`, `wings`
  [@S04, ch. 4 p. 15-16 et ch. 5 p. 17-18].
- [FAIT] Le firmware de l'an dernier ajoute `shuriken`, `skater`, `zouk` et `walk_dominance`
  [@S06, motion.h l. 88-126].
- [FAIT] L'API accepte des noms de commandes en majuscules (`FORWARD`, `SAY_HELLO`, `RAVE`…) avec un
  paramètre `n` ; durées, patte et vitesse sont fixées en dur [@S06, wifi.cpp l. 130-214].
- [FAIT] À la fin de chaque séquence de marche ou de rotation, le robot reprend sa pose de départ
  [@S05, §5.3 p. 26].
- [FAIT] Le superviseur de l'an dernier regroupe les mouvements « Show » (SAY_HELLO, RAVE, FLEX,
  APPLAUD_WAVE, APPLAUD_SYM, TAP_LEG, SPUTNIK, SCRATCH, PLS, WINGS, BOLTING, SHURIKEN, SKATER, ZOUK)
  [@S07, app.py l. 214-261].
- [FAIT] La durée réelle d'un mouvement n'est pas garantie : les pattes sont suivies en boucle ouverte
  [@S04, §2.1 p. 10] ; les durées de chaque mouvement sont **inconnues** et à mesurer.
- Critères tuteurs : synchronisé à l'œil, recalage sur le mouvement suivant ; **au moins une figure
  collective** (objectif O5, `docs/02`).

## Options envisagées

| Option | Principe | Avantages | Inconvénients | Source |
|---|---|---|---|---|
| **A. Script Python en dur** | La chorégraphie est écrite directement dans le code | Rapide à faire | Il faut modifier le code à chaque essai ; peu lisible pour les non-développeurs ; difficile à décrire dans le rapport | — |
| **B. Fichier de données texte lu par le poste de contrôle** | Une suite d'étapes dans un fichier (YAML ou JSON), lue et jouée par un programme | Modifiable sans toucher au code ; lisible ; versionnable ; vérifiable avant exécution | Un format à définir et un programme de lecture à écrire | [@S17 ; @S15, résumé] (bibliothèque de primitives enchaînées) |
| **C. Interface graphique** (frontend S07) | Composer la chorégraphie à la souris | Accessible à tous | Plus de développement ; hors délai | [@S07, frontend] |
| **D. Trajectoire stockée à bord** | Le robot joue sa séquence seul après un signal | Un seul instant à synchroniser | Écartée par l'ADR-002 (A+ retenu) ; limite de 60 entrées `[HYPOTHÈSE]` | [@S06, trajectory.h l. 21-30 ; @S27, §IV] |

## Décision

**Option B** : la chorégraphie est un **fichier texte** décrivant une suite d'**étapes**, lu et joué par
le poste de contrôle (basé sur S07 / S08), avec recalage entre deux étapes (ADR-002).

Format proposé `[HYPOTHÈSE : à ajuster en essayant]` :

```yaml
titre: exemple
robots: [R1, R2, R3]          # liste des robots, avec leur adresse dans un fichier de configuration local
etapes:
  - nom: debout
    tous: STAND               # même commande pour tous les robots
  - nom: salut
    tous: SAY_HELLO
  - nom: canon                # figure collective : chaque robot bouge à tour de rôle
    R1: RAVE
  - nom: canon-suite
    R2: RAVE
  - nom: miroir               # robots différents, commandes différentes dans la même étape
    R1: CRAB_LEFT
    R2: CRAB_RIGHT
    R3: STAND
  - nom: fin
    tous: SIT
```

Règles :

1. Une **étape** = au plus une commande par robot, avec un paramètre `n` facultatif (1 par défaut)
   [@S06, wifi.cpp l. 98-124].
2. Les robots **non cités** dans une étape ne reçoivent rien : ils restent immobiles. C'est ce qui permet
   les figures **en canon** ou **en vague** (un robot après l'autre).
3. Le programme envoie l'étape à tous les robots concernés **en parallèle**, puis attend leur
   disponibilité avant l'étape suivante (ADR-002).
4. Les noms de commandes sont ceux du firmware [@S06, wifi.cpp l. 130-214].
5. Les adresses des robots et les identifiants Wi-Fi sont dans un **fichier local ignoré par git**, jamais
   dans le fichier de chorégraphie.

**Vérifications avant exécution** (idée T4 de `docs/03` §4.2, inspirée de [@S17]) :

- chaque commande existe dans la liste du firmware ;
- `n` est un entier positif ;
- chaque robot cité est déclaré ;
- le nombre total de commandes envoyées à un même robot ne dépasse pas la limite de 60 si l'hypothèse
  sur S06 se confirme [@S06, trajectory.h l. 45-47, 81-83] ;
- **mode à blanc** : afficher la suite d'étapes sans envoyer aucun ordre, pour tester une chorégraphie
  sans robot (utile, l'accès aux robots étant limité).

## Justification

1. Un fichier de données se modifie, se relit et se décrit dans le rapport sans toucher au code ; il
   correspond à l'idée de bibliothèque de primitives enchaînées utilisée pour les drones [@S17 ; @S15,
   résumé].
2. Il s'appuie sur ce qui existe : les noms de commandes et l'API de S06 / S07 / S08.
3. Il rend directement possible l'objectif O5 (figure collective) par le canon, le miroir ou la vague.
4. Le mode à blanc permet à toute l'équipe de préparer la chorégraphie **sans robot**, ce qui compte
   pour la semaine du 30/11 au 06/12.

## Conséquences / risques

| Risque | Source | Parade |
|---|---|---|
| Durées des mouvements inconnues : le recalage attend le plus lent | [@S04, §2.1 p. 10] | Mesurer chaque mouvement sur chaque robot (EXP) ; éviter de mêler mouvements très longs et très courts dans la même étape |
| Les marches changent la position des robots ; ils ne savent pas où ils sont | [@S04, §2.1 p. 10] ; reprise de la pose de départ [@S05, §5.3 p. 26] | Privilégier les figures sur place ; compenser les marches (avant / arrière) ; tester le décalage cumulé `[HYPOTHÈSE]` |
| Collision entre robots | — | Espacer les robots ; tester la figure avec un seul robot d'abord |
| Limite de 60 commandes cumulées | `[HYPOTHÈSE]` [@S06, trajectory.h l. 45-47, 81-83] | EXP-003 ; correction du firmware si confirmé (ADR-003) |
| Commandes pas toutes testées (certains mouvements avancés) | [@S04, ch. 4 p. 15-16] | Tester chaque mouvement avant de l'inclure |
| Format à ajuster après les premiers essais | — | Le garder simple ; mettre à jour l'ADR si besoin |
| Pas de calage sur la musique | — | Hors MVP (bonus) |

**Ce que cette décision ne tranche pas** : la chorégraphie elle-même (figures retenues, ordre,
durée) : travail d'Eva, à faire après avoir testé les mouvements.

## Sources

- [@S04, ch. 4 p. 15-16 et ch. 5 p. 17-18] Mouvements de base et avancés.
- [@S05, §5.3 p. 26] Retour à la pose de départ.
- [@S06, motion.h l. 88-126 ; wifi.cpp l. 98-124 et 130-214 ; trajectory.h l. 21-30, 45-47, 81-83].
- [@S07, app.py l. 214-261 ; frontend] Mouvements « Show » ; interface.
- [@S15, résumé] ; [@S17] Primitives de mouvement enchaînées (drones).
- [@S27, §IV] Trajectoire stockée à bord (option écartée).
- ADR-002 (architecture et départ commun) ; `docs/02-probleme-objectifs.md` (objectifs O4 et O5).
