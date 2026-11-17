# Tableaux comparatifs — options matérielles de communication

> **Statut** : brouillon v0.1 — à valider et à compléter par l'équipe
> **Dernière mise à jour** : 2026-11-17 (date de consultation des sources : 2026-11-17)
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé · `[À RENSEIGNER]` pas de valeur sourcée
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`

Tableaux demandés par les tuteurs : options comparées avec prix, taille, disponibilité, avantages et
inconvénients [@S28].

> ⚠️ **Niveau de lecture** : les valeurs ci-dessous viennent d'**extraits de pages de vendeurs ou de
> fabricants** lus le 2026-11-17, pas de pages relues en entier. Les **prix varient selon le pays, le
> revendeur et le jour** : ce sont des ordres de grandeur, pas des devis. Chaque valeur doit être
> re-vérifiée par un membre de l'équipe avant d'être reprise dans le rapport. Une case `[À RENSEIGNER]`
> signifie qu'aucune valeur sourcée n'a été trouvée : rien n'est inventé.

## 1. Carte de contrôle avec Wi-Fi (option retenue, ADR-003)

| Critère | Arduino Nano RP2040 Connect |
|---|---|
| Rôle | Contrôle des servomoteurs et réception des ordres Wi-Fi [@S03, §6.2] |
| Prix (ordre de grandeur) | 29,99 € sur la boutique Arduino (page pour l'Allemagne) [@S30] ; 31,50 € sur une autre page de la même boutique (Danemark) [@S31] ; 26 £ chez un revendeur britannique [@S32] |
| Taille | 45 mm × 18 mm ; 6 g [@S29] |
| Disponibilité | En stock chez un revendeur britannique le 2026-11-17 [@S32] ; **déjà en possession de l'équipe sur ses robots, selon elle** `[À VÉRIFIER]` |
| Connectivité | Wi-Fi et Bluetooth par le module u-blox NINA-W102 [@S29 ; @S30] |
| Avantages | Format Arduino Nano ; Wi-Fi intégré ; déjà utilisé et codé l'an dernier [@S03, §6.2 ; @S06] |
| Inconvénients | Tremblements des servomoteurs observés ; connexion de plusieurs secondes ; mapping des broches à refaire [@S03, §6.2 et §6.8] |

## 2. Moyens de communication comparés

| Option | Prix (ordre de grandeur) | Taille | Disponibilité | Avantages | Inconvénients |
|---|---|---|---|---|---|
| **Wi-Fi sur Nano RP2040 Connect** (retenu) | Voir §1 | 45 × 18 mm [@S29] | Voir §1 | Déjà codé (firmware S06, superviseur S07, CLI S08) ; aucun matériel à ajouter | Réseau local commun nécessaire, connexion lente [@S03, §6.8 et §9.2] |
| **Radio nRF24L01+** (module) | Environ 1,94 $ en boutique australienne (devise de la boutique non précisée dans l'extrait) [@S33] | Module 29 × 15 mm [@S34] | En stock en boutique le 2026-11-17 [@S33] | Peu coûteux ; prévu dans le firmware [@S06, radio.h l. 48-49] | Module à ajouter à chaque robot ; 6 ordres seulement dans le code ; compilation douteuse `[HYPOTHÈSE]` [@S06, radio.cpp l. 5 et 44] |
| **Raspberry Pi Compute Module 4 + liaison série** | Très variable selon la variante et le vendeur : 85 $ (variante Wi-Fi, vendeur DFRobot, produit retiré) [@S36] ; autres variantes plus chères `[À RENSEIGNER : prix actuel officiel]` | 55 × 40 × 4,7 mm [@S35] | Produit retiré chez DFRobot [@S36] ; production annoncée par le fabricant jusqu'en janvier 2034 au moins [@S37] ; disponibilité réelle `[À VÉRIFIER]` | Puissance de calcul ; Linux | Alimentation, niveaux logiques 3,3 V / 5 V, solution expérimentale [@S03, §5.4 à §5.8] ; sans objet avec la carte RP2040 |
| **micro-ROS** | Logiciel : `[À RENSEIGNER]` | — | — | Porte ROS 2 sur microcontrôleur [@S24] | Dépôt indiquant que ce n'est pas prêt pour la production ; compatibilité `[À VÉRIFIER]` [@S24] |
| **ESP-NOW** | `[À RENSEIGNER]` | — | — | Protocole sans connexion, diffusion possible [@S23] | Puce Espressif ; pour la RP2040 Connect `[À VÉRIFIER]` [@S23] |

## 3. Autres matériels à renseigner

| Matériel | Rôle | Prix | Taille | Disponibilité | État |
|---|---|---|---|---|---|
| Point d'accès Wi-Fi | Réseau dédié aux robots (ADR-003) | `[À RENSEIGNER]` | `[À RENSEIGNER]` | `[À RENSEIGNER]` | Matériel de l'équipe à inventorier |
| Caméra | Mesure par vidéo (ADR-004) | `[À RENSEIGNER]` | — | — | Cadence à vérifier : zone d'ombre 10 |
| Robots SEALK | Plateforme | Prêtés par les tuteurs | `[À RENSEIGNER]` | Nombre inconnu | Inventaire (M1a) |

## 4. Lecture rapide

`[HYPOTHÈSE]` Pour ce projet, le Wi-Fi sur la carte déjà présente est le seul choix qui n'ajoute **aucun
matériel** : le prix, la taille et la disponibilité se réduisent à ce que les robots ont déjà. Les autres
options ajoutent du matériel à chaque robot (3 ou 4 exemplaires) et du code. Cette lecture dépend de la
confirmation que la carte RP2040 Connect est bien présente sur chaque robot.

## Sources

- [@S03, §5, §6] ; [@S06] ; [@S23] ; [@S24] ; [@S28].
- [@S29] à [@S37] : voir `docs/sources.bib`.
