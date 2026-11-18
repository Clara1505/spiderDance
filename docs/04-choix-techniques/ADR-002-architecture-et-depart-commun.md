# ADR-002 — Architecture centralisée et mécanisme de départ commun

**Date** : 2026-11-17
**Statut** : Proposé (choix confirmé avec les tuteurs le 2026-11-17 ; à valider par l'équipe)
**Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
**Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé

---

## Contexte

L'énoncé demande de coordonner plusieurs robots par une API de communication [@S02]. Il faut choisir
**où se prend la décision** (un poste central ou les robots eux-mêmes) et **comment les robots
démarrent un mouvement ensemble**.

Éléments connus :

- [FAIT] Le code de l'an dernier envoie l'ordre à chaque robot indépendamment, par un fil d'exécution et
  une file par robot : il n'y a pas de départ synchronisé [@S07, app.py l. 35-141 et 181-203].
- [FAIT] Le robot répond à la requête HTTP **avant** d'exécuter l'ordre [@S06, wifi.cpp l. 115-123].
- [FAIT] Quand un robot exécute déjà une commande, il peut ne pas répondre tout de suite à une nouvelle
  requête réseau [@S03, §6.4].
- [FAIT] Aucun robot n'a de retour de position : l'arrivée des pattes est estimée en boucle ouverte
  [@S04, §2.1 p. 10].
- [FAIT] La mesure n'est pas plus fine qu'une image vidéo, et les horloges des robots ne sont pas
  synchronisées entre elles [@S16, introduction].

**Réponses des tuteurs (échange du 2026-11-17, relayé par l'équipe)** :

1. L'architecture **centralisée** est acceptée : l'ordinateur envoie les instructions à tous les
   robots via l'API.
2. Niveau de synchronisation attendu : **synchronisé à l'œil**, et si les robots se décalent, ils
   **se recalent sur le mouvement suivant**.
3. Nombre de robots pour la démonstration : **3 ou 4** pour commencer, davantage si l'avancement et la
   disponibilité des robots le permettent. Les robots sont à domicile (déclaré par l'équipe, 2026-11-18) ;
   chez qui et combien : `[À VÉRIFIER]`.

## Options envisagées

| Option | Principe | Avantages | Inconvénients | Source |
|---|---|---|---|---|
| **A. Ordres envoyés un par un, sans recalage** | L'ordinateur envoie chaque mouvement à chaque robot, sans attendre les autres | Existe déjà (superviseur S07) | Les écarts s'accumulent : aucun recalage | [@S07, app.py l. 181-203] |
| **A+. Option A avec recalage à chaque mouvement** | Même architecture, mais le mouvement suivant n'est envoyé qu'une fois **tous** les robots disponibles | Répond exactement au critère des tuteurs (recalage sur le mouvement suivant) ; reste dans l'existant ; ne demande ni stockage à bord ni diffusion | Un robot lent ou en panne retient tous les autres ; temps mort entre deux mouvements ; savoir qu'un robot est disponible reste à tester `[HYPOTHÈSE]` | Réponses des tuteurs ; [@S03, §6.4] |
| **B. Chorégraphie stockée à bord + signal « GO »** | Chaque robot joue seul sa séquence après un signal unique | Un seul instant à synchroniser [@S27, §IV et §VIII] | Aucune correction pendant la danse ; limite de 60 entrées de trajectoire `[HYPOTHÈSE]` ; plus de travail sur le firmware | [@S06, trajectory.h l. 21-30] |
| **C. B avec signal diffusé** | Un seul message pour tous les robots | Écart de départ plus faible en théorie | Diffusion possible avec la bibliothèque Wi-Fi ? `[À VÉRIFIER]` | [@S27, §VIII] |
| **D. Décentralisée** | Les robots se synchronisent entre eux (type lucioles) | Colle à « robotique distribuée » | Complexe ; hors délai ; contraire à la décision centralisée | [@S20] |

## Décision

**Option A+ : architecture centralisée, ordres envoyés par l'ordinateur à tous les robots via l'API
HTTP existante, avec un recalage avant chaque mouvement.**

Principe de fonctionnement :

1. L'ordinateur envoie le mouvement *n* à tous les robots, **en parallèle**.
2. Il attend que **tous** les robots soient de nouveau disponibles (mouvement *n* terminé).
3. Il envoie le mouvement *n+1* à tous les robots en parallèle, et ainsi de suite.

Ainsi un décalage n'est jamais cumulé : chaque mouvement repart d'une situation où tous les robots
sont arrêtés.

`[HYPOTHÈSE]` Un robot est « disponible » quand il répond à `GET /api/health` : s'il ne lit pas les
requêtes pendant un mouvement [@S03, §6.4], la réponse n'arrive qu'à l'arrêt. **À tester** (EXP-003).
**Si l'hypothèse est fausse**, repli : attendre la durée estimée du mouvement plus une marge fixe,
mesurée en EXP-002.

Les options B et C restent des **pistes d'amélioration** si la mesure montre un écart de départ trop
visible ; elles ne sont pas retenues pour le moment.

## Justification

1. [FAIT] Les tuteurs acceptent le centralisé et décrivent un critère (à l'œil, recalage sur le
   mouvement suivant) que A+ remplit directement.
2. [FAIT] A+ s'appuie sur l'existant (HTTP, file et fil par robot) [@S06 ; @S07] : c'est la solution la
   moins coûteuse pour 27 jours avec une équipe en entreprise [`docs/gestion-projet.md` §1].
3. `[HYPOTHÈSE]` Le recalage compense l'absence de retour de position [@S04, §2.1 p. 10] sans capteur
   supplémentaire : le point de rendez-vous remplace la correction continue.
4. La différence entre A (sans recalage) et A+ (avec recalage) donne une **comparaison avant / après**
   mesurable, ce qui prouve l'apport de notre travail [@S01, slide 10].

## Conséquences / risques

| Risque | Source | Parade |
|---|---|---|
| **Au-delà de 60 commandes cumulées, les nouvelles seraient ignorées** : `traj_length` n'est jamais remis à zéro. Avec l'envoi un par un, une chorégraphie longue peut dépasser cette limite | `[HYPOTHÈSE]` [@S06, trajectory.h l. 45-47, 81-83] | **Tester en priorité** (EXP-003). Si confirmé : corriger le firmware (élément « repris et modifié », à tracer), ou découper la chorégraphie |
| « Disponible » = répond à `/api/health` : non vérifié | [@S03, §6.4] | EXP-003 ; repli sur durée estimée + marge |
| Un robot en panne ou perdu du Wi-Fi bloque tous les autres | — | Délai maximal par mouvement, puis la chorégraphie continue sans ce robot ou s'arrête proprement |
| Temps mort entre deux mouvements (le robot attend le plus lent) | `[HYPOTHÈSE]` | À mesurer ; choisir des mouvements de durée voisine |
| Écart de départ résiduel dû au réseau (envoi à 3 ou 4 robots) | [@S07, app.py l. 181-203] | Mesure vidéo (EXP-002 puis EXP-004) ; passer à B ou C si trop visible |
| Wi-Fi à 3 ou 4 robots : isolation des clients, connexion de plusieurs secondes | [@S03, §6.8 et §9.2] | Point d'accès à nous ; tester sur place |
| La réponse HTTP part avant l'exécution : elle ne prouve pas que le mouvement a commencé | [@S06, wifi.cpp l. 115-123] | Ne jamais s'en servir comme preuve ; mesure externe par vidéo |
| Dérive d'horloge entre robots | [@S16, introduction] | Sans effet avec A+, car le rendez-vous est donné par l'ordinateur, pas par les horloges |

**Effets sur le reste du projet** :

- **Objectifs** (`docs/02`) : le critère O3 devient « synchronisé à l'œil, avec recalage sur le mouvement
  suivant », validé par les tuteurs. La mesure image par image reste la **preuve** et la comparaison
  avec la référence ; elle ne fixe plus de seuil imposé.
- **Taille du MVP** : 3 robots visés, 4 si possible, au lieu de 2. Si trop peu de robots sont
  disponibles, on retombe sur 2, à documenter.
- **Communication** : l'API HTTP sur Wi-Fi existante est conservée (ADR de communication à rédiger pour
  la justifier et la comparer).
- **Chorégraphie** : décrite côté ordinateur comme une suite de mouvements ; le format reste à décider
  (ADR suivant).

## Sources

- [@S02] Énoncé du sujet.
- [@S01, slide 10] Critères d'évaluation.
- [@S03, §6.4, §6.8, §9.2] Fonctionnement d'un robot occupé, limites, réseau.
- [@S04, §2.1 p. 10] Boucle ouverte, pas de retour de position.
- [@S06, wifi.cpp l. 115-123 ; trajectory.h l. 21-30, 45-47, 81-83] Réponse avant exécution ; trajectoire.
- [@S07, app.py l. 35-141 et 181-203] Envoi indépendant par robot.
- [@S16, introduction] Dérive des horloges.
- [@S20] Synchronisation inspirée des lucioles.
- [@S27, §IV et §VIII] Trajectoire stockée à bord et signal de départ diffusé.
- Échange avec les tuteurs, 2026-11-17 (réponses relayées par l'équipe ; pas de source écrite).
- `docs/gestion-projet.md` §1.
