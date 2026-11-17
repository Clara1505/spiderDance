# ADR-004 — Méthode de mesure de la synchronisation (sans vision par ordinateur)

**Date** : 2026-11-17
**Statut** : Proposé (à valider par l'équipe)
**Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
**Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé · `[À DÉFINIR]` valeur à fixer par l'équipe

---

## Contexte

Il faut pouvoir **prouver** que les robots sont synchronisés (« Est-ce que ça marche ? À quel point ? »
[@S01, slide 10]) et montrer l'apport du recalage de l'ADR-002 par une comparaison **avant / après**.

Contraintes :

- [FAIT] **Pas de vision par ordinateur** (décision d'équipe du 2026-11-17) : la mesure ne repose pas
  sur un traitement d'image automatique.
- [FAIT] Les robots n'ont aucun retour de position [@S04, §2.1 p. 10] : seule une mesure **externe**
  peut dire quand un mouvement démarre vraiment.
- [FAIT] Chaque robot a sa propre horloge, et les horloges de nœuds distincts dérivent
  [@S16, introduction] : des journaux horodatés **par les robots** ne peuvent pas être comparés entre
  eux.
- [FAIT] Le robot répond à la requête HTTP **avant** d'exécuter l'ordre [@S06, wifi.cpp l. 115-123] :
  l'heure de la réponse ou de l'envoi côté ordinateur ne dit pas quand le mouvement démarre.
- [FAIT] Une vidéo ne permet pas de mesurer plus finement que la durée d'une image : environ 33 ms à
  30 images par seconde, environ 17 ms à 60. `[À VÉRIFIER : cadence de la caméra disponible]`
- [FAIT] Critère d'acceptation donné par les tuteurs (échange du 2026-11-17, relayé par l'équipe) :
  synchronisé **à l'œil**, avec un recalage sur le mouvement suivant si un décalage apparaît.

## Options envisagées

| Option | Principe | Avantages | Inconvénients | Source |
|---|---|---|---|---|
| **A. Vidéo analysée image par image, à la main** | Filmer tous les robots ensemble ; relever, pour chaque mouvement, la première image où chaque robot bouge | Seule mesure indépendante des horloges des robots ; sans matériel en plus qu'une caméra | Précision limitée à une image ; travail manuel ; définition de « début de mouvement » à fixer | [@S04, §2.1 p. 10] |
| **B. Journaux de l'ordinateur seuls** | Horodater les envois côté poste de contrôle | Automatique | Ne dit pas quand le robot bouge (réponse avant exécution) | [@S06, wifi.cpp l. 115-123] |
| **C. Journaux des robots** | Chaque robot note l'heure de début | Mesure côté robot | Horloges non synchronisées entre elles : non comparables | [@S16, introduction] |
| **D. A + journaux de l'ordinateur en complément** | La vidéo mesure ; les journaux situent chaque mouvement dans la séquence | Retrouver facilement quel mouvement correspond à quelle image | Ne mesure rien de plus que A | — |
| **E. Traitement d'image automatique** | Détecter les mouvements par logiciel | Mesure automatique | **Exclu** : pas de vision par ordinateur (décision d'équipe) | — |

## Décision

**Option D** : mesure par **vidéo analysée image par image à la main**, complétée par les journaux de
l'ordinateur pour repérer les mouvements.

Protocole proposé (à affiner dans la fiche EXP-002) :

1. **Prise de vue** : caméra fixe, plan large où **tous les robots sont visibles ensemble** dans la même
   image, à la cadence la plus élevée disponible `[À VÉRIFIER : 30 ou 60 images par seconde]`.
2. **Repère de calage** : `[HYPOTHÈSE]` allumer / éteindre les LED des robots avec les commandes `LUMOS` et
   `NOX` [@S06, wifi.cpp l. 130-214] pour repérer le début de chaque essai sur la vidéo. À tester. Repli :
   un repère visible à l'œil (par exemple un geste de la personne qui lance l'essai).
3. **Définition d'un « début de mouvement »** : la première image où une patte est visiblement en
   mouvement. Même définition pour tous les robots et tous les essais.
4. **Relevé** : pour chaque mouvement *n* et chaque robot, noter le numéro de la première image de
   mouvement. L'écart entre deux robots = différence de numéros d'image × durée d'une image.
5. **Double lecture** : deux membres de l'équipe relèvent indépendamment les mêmes essais, pour estimer la
   fiabilité de la lecture à la main `[HYPOTHÈSE : à confirmer comme utile en EXP-002]`.
6. **Répétitions** : plusieurs essais par scénario `[À DÉFINIR : nombre d'essais]`, avec les mêmes
   mouvements.
7. **Fiche de relevé** : un tableau (une ligne par mouvement et par robot) joint à la fiche EXP, avec la
   vidéo ou son nom de fichier.
8. **Logiciel de lecture** image par image : `[À VÉRIFIER : lequel]`.

Scénarios mesurés :

| Fiche | Scénario | Question |
|---|---|---|
| EXP-002 | Option A de l'ADR-002 : mêmes ordres envoyés en parallèle, **sans recalage** | Quel écart observe-t-on, et s'accumule-t-il ? (mesure de référence) |
| EXP-004 | Option A+ : **avec recalage** avant chaque mouvement | L'écart reste-t-il borné et non cumulatif ? |

## Justification

1. [FAIT] C'est la seule mesure indépendante des horloges des robots, donc la seule qui prouve réellement
   la synchronisation [@S04, §2.1 p. 10 ; @S16, introduction].
2. [FAIT] Elle respecte la décision de ne pas faire de vision par ordinateur.
3. Elle correspond au critère des tuteurs (« à l'œil ») tout en le rendant vérifiable.
4. La comparaison EXP-002 / EXP-004 montre l'apport du recalage, ce qui répond à « à quel point ça
   marche ? » [@S01, slide 10].

## Conséquences / risques

| Risque | Source | Parade |
|---|---|---|
| La précision est limitée à une image (environ 33 ms à 30 images/s) | `[À VÉRIFIER]` caméra | Ne jamais annoncer un écart plus fin que la durée d'une image ; choisir la caméra la plus rapide disponible |
| « Début de mouvement » ambigu selon l'observateur | — | Définition écrite ; double lecture |
| `LUMOS` / `NOX` ne servent peut-être pas de repère visible | [@S06, wifi.cpp l. 130-214] `[HYPOTHÈSE]` | Tester ; repli sur un repère visuel simple |
| Travail manuel important en fin de projet | `docs/gestion-projet.md` §5 | Faire EXP-002 en semaine 2 ; limiter le nombre d'essais |
| Mouvements trop lents ou flous pour repérer un début | — | Choisir des mouvements au départ net pour la mesure |
| Parallaxe : un robot plus loin de la caméra paraît bouger plus tard | — | Caméra face à la ligne de robots, à distance égale |
| Une mesure faite à la main n'est pas totalement objective | — | Le dire dans le rapport ; double lecture ; ne pas surévaluer le résultat |

**Ce que cette décision ne tranche pas** : le nombre exact d'essais, les valeurs de seuil (les tuteurs ne
demandent plus de seuil chiffré : critère « à l'œil + recalage »), et le choix de la caméra.

## Sources

- [@S01, slide 10] Critères d'évaluation.
- [@S04, §2.1 p. 10] Boucle ouverte, pas de retour de position.
- [@S06, wifi.cpp l. 115-123 et 130-214] Réponse avant exécution ; commandes `LUMOS` / `NOX`.
- [@S16, introduction] Dérive des horloges.
- Échange avec les tuteurs, 2026-11-17 (relayé par l'équipe ; pas de source écrite).
- ADR-002 (architecture et départ commun) ; `docs/03-etat-de-l-art.md` §8.
