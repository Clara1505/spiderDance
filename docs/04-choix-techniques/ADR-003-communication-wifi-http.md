# ADR-003 — Communication par Wi-Fi et HTTP, et corrections de l'API

**Date** : 2026-11-17
**Statut** : Proposé (à valider par l'équipe)
**Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
**Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé

---

## Contexte

L'énoncé impose une **API de communication** pour coordonner les robots [@S02]. L'architecture est
centralisée : l'ordinateur envoie les ordres à tous les robots (ADR-002). Il faut choisir le moyen de
communication et décider quoi faire des défauts constatés dans l'API de l'an dernier.

Éléments connus :

- [FAIT] Les robots de l'équipe ont, selon elle, une carte **Arduino Nano RP2040 Connect**
  (microcontrôleur RP2040, module Wi-Fi u-blox NINA-W102), à confirmer sur chaque robot `[À VÉRIFIER]`.
- [FAIT] Le firmware S06 contient déjà la connexion Wi-Fi, un serveur HTTP écrit à la main et les
  commandes de mouvement [@S06, wifi.h l. 3-6 ; wifi.cpp l. 83-214]. Le Wi-Fi n'est compilé que pour la
  carte RP2040 Connect [@S06, wifi.h l. 3-6].
- [FAIT] Le poste de contrôle S07 et la CLI S08 pilotent les robots par cette API [@S07, app.py ;
  @S08, spider_cli.py l. 9-34].
- [FAIT] Limites déclarées par l'équipe précédente : connexion de plusieurs secondes, réseau local
  commun, pas d'authentification ni de chiffrement, système multi-robots non consolidé
  [@S03, §6.8 et §9.2].
- Contraintes de conception (puissance, taille, bande passante, latence, fiabilité, coût) [@S01, slide 5].

## Options envisagées

| Option | Avantages | Inconvénients | Source |
|---|---|---|---|
| **Wi-Fi + HTTP** (existant) | Déjà codé sur les trois composants ; aucun matériel à ajouter | Connexion lente ; réseau pouvant isoler les appareils ; API sans sécurité | [@S06 ; @S07 ; @S08 ; @S03, §6.8 et §9.2] |
| Radio nRF24L01 | Prévue dans le firmware | 6 ordres seulement ; compilation douteuse `[HYPOTHÈSE]` ; module à ajouter | [@S06, radio.h l. 48-49 ; radio.cpp l. 5 et 44] |
| Compute Module + liaison série | Testé l'an dernier | Solution expérimentale ; alimentation ; niveaux logiques 3,3 V / 5 V ; sans objet avec la carte RP2040 | [@S03, §5.4 à §5.8] |
| ROS 2 / micro-ROS | Intergiciel robotique, présenté en cours | Dépôt micro-ROS : pas prêt pour la production ; compatibilité avec nos cartes `[À VÉRIFIER]` | [@S01, slide 6 ; @S24] |
| ESP-NOW | Protocole sans connexion, diffusion possible | Puce Espressif ; pour la RP2040 Connect `[À VÉRIFIER]` | [@S23] |

Les colonnes prix, taille et disponibilité demandées par les tuteurs [@S28] sont dans le tableau
comparatif de `docs/03-etat-de-l-art.md` §7, à compléter avec des sources réelles.

## Décision

1. **Moyen de communication : Wi-Fi + HTTP**, tel que déjà codé dans S06, S07 et S08.
2. **Corrections de l'API, faites au passage** (éléments « repris et modifiés », à tracer dans
   `docs/reproductibilite.md`) :

| # | Défaut constaté | Correction retenue | Source |
|---|---|---|---|
| 1 | Une commande **inconnue** reçoit quand même la réponse « ok » | Répondre par une **erreur** (code 400) : une réponse « ok » ne doit jamais couvrir un ordre non exécuté (objectif O1) | [@S06, wifi.cpp l. 115-123, 130-214] |
| 2 | **Port** : le firmware écoute sur 2021, la CLI et l'interface proposent 2020 | **Une seule valeur**, définie dans un fichier de configuration, jamais écrite en dur à plusieurs endroits. Valeur à confirmer sur robot | [@S06, wifi.h l. 14 ; @S07, frontend/src/App.tsx l. 14 ; @S08, spider_cli.py l. 9-34] |
| 3 | **Méthode** : le rapport S03 décrit `POST` + JSON, tout le code utilise `GET ?cmd=…&n=…` | **Garder `GET`** : S06, S07 et S08 sont d'accord entre eux ; le rapport S03 est un guide, pas une mesure. Documenter la méthode réelle dans le rapport | [@S03, §6.6 ; @S06, wifi.cpp l. 84-124 ; @S07, app.py l. 123 ; @S08] |
| 4 | **Réponse de `/api/health`** : le rapport donne un format, le code un autre | **Garder le code** (`{"ok":true,"status":"up"}`) et l'utiliser comme référence | [@S03, §6.6 ; @S06, wifi.cpp l. 99] |
| 5 | La **réponse part avant l'exécution** : le PC ne sait pas quand le mouvement est fini | **Test en deux temps** : (a) utiliser `/api/health` comme signal « disponible » (hypothèse d'ADR-002, aucune modification) ; (b) si (a) échoue, faire **répondre après l'exécution** | [@S06, wifi.cpp l. 115-123 ; @S03, §6.4] |
| 6 | **Identifiants Wi-Fi écrits en dur** dans un dépôt public | Fichier local ignoré par git ; valeurs factices dans le dépôt | [@S06, wifi.h l. 9-12] |

`[HYPOTHÈSE]` Les points 2, 3 et 4 sont des **choix par défaut** : si un test sur les robots donne un
résultat différent (par exemple si le firmware flashé n'écoute pas sur 2021), c'est le test qui fait foi
et cet ADR est mis à jour.

## Justification

1. [FAIT] Tout est déjà codé : choisir une autre liaison obligerait à ajouter du matériel ou à
   réécrire le firmware, ce qui n'est pas réaliste en 27 jours [`docs/gestion-projet.md` §1].
2. [FAIT] Les trois composants S06, S07 et S08 utilisent la même méthode `GET` : la garder évite de
   les modifier tous pour suivre une description qui n'est pas celle du code.
3. Les corrections 1 et 5 servent directement les objectifs : une réponse « ok » doit signifier que
   l'ordre est connu, et le PC doit savoir quand chaque robot a fini pour recaler le mouvement suivant
   (ADR-002).
4. Les perspectives de l'équipe précédente (délais, séquences, format enrichi) prolongent cette API
   [@S03, §6.6 et §10.2].

## Conséquences / risques

| Risque | Source | Parade |
|---|---|---|
| Un robot occupé ne lit pas les requêtes ; il peut ne pas répondre à `/api/health` | [@S03, §6.4] | Tester (EXP-003) ; sinon correction 5(b) |
| Répondre après l'exécution : la requête reste ouverte longtemps | [@S07, app.py : délai de 90 s] | Vérifier que le délai couvre les mouvements les plus longs |
| `GET` avec effet de bord : une requête répétée par erreur rejoue le mouvement | — | Acceptable pour un prototype ; documenté ; pas de répétition automatique |
| API sans authentification ni chiffrement | [@S03, §6.8] | Réseau local dédié (point d'accès à nous) ; limite assumée et documentée |
| Connexion Wi-Fi lente, réseau qui isole les appareils | [@S03, §6.8 et §9.2] | Point d'accès à nous ; attendre que tous les robots soient connectés avant de lancer |
| Écarts entre sources (voir tableau) non tous testés | `docs/03-etat-de-l-art.md` §3.4 | EXP-003 : trancher par test avant de modifier |
| Modifier le firmware : risque de régression | — | Une modification à la fois, testée sur un robot posé sur les plots ; règles de sécurité (robot éteint pour programmer, garde-fou, calibration) |

**Ce que cette décision ne tranche pas** : le format de la chorégraphie, la méthode de mesure et la
liste exacte des commandes utilisées dans la démonstration.

## Sources

- [@S01, slides 5 et 6] Contraintes de conception ; ROS 2.
- [@S02] Énoncé du sujet.
- [@S03, §5, §6.4, §6.6, §6.8, §9.2, §10.2] Solutions de l'an dernier, limites, réseau.
- [@S06] Firmware : wifi.h, wifi.cpp, radio.h, radio.cpp.
- [@S07] Superviseur : app.py, frontend/src/App.tsx.
- [@S08] CLI : spider_cli.py.
- [@S23] ESP-NOW. [@S24] micro-ROS. [@S28] Consignes PROJEX.
- `docs/gestion-projet.md` §1 ; `docs/03-etat-de-l-art.md` §3.4 et §7.
