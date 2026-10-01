# 01 — Contexte et besoins

> **Statut** : brouillon v0.3 — à valider par l'équipe
> **Dernière mise à jour** : 2026-09-29
> **Rédaction** : Clara SION SAUNIER (avec assistance IA)
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`

---

## 1. Cadre du projet

[FAIT] Le module PROJEX (« Exploring Project ») de l'EPITA demande d'appliquer des compétences
techniques et de gestion de projet à un thème de robotique et de vision par ordinateur. L'objectif
est de partir d'une idée, de la conceptualiser et de construire un démonstrateur documenté (PoC,
*Proof of Concept*) [@S01, slide 2].

[FAIT] Selon les encadrants, un bon PoC n'est pas seulement un prototype qui fonctionne : c'est une
démarche d'ingénierie raisonnée et documentée, qui enchaîne contexte & besoins, problème &
objectifs, état de l'art, choix techniques, prototypage & expérimentation, validation & résultats
[@S01, slide 3].

Ce document est la première étape de cette démarche.

## 2. Le sujet : SpiderDance

[FAIT] Énoncé du sujet [@S02] :

> *Make a pack of robots dance. Use a communication API to coordinate synchronized movements
> across multiple spider robots and demonstrate collective behaviors through a choreography.*
> *⇒ Application: distributed robotics and multi-robot coordination.*
> *⇒ Domain: Embedded systems*

Autrement dit : faire danser un groupe de robots araignées, coordonner leurs mouvements
synchronisés par une API de communication, et démontrer des comportements collectifs au travers
d'une chorégraphie. Le sujet relève de la **robotique distribuée**, de la **coordination
multi-robots** et des **systèmes embarqués**.

## 3. Le robot : l'araignée pédagogique SEALK

[FAIT] L'araignée SEALK possède 4 pattes articulées chacune par 3 servomoteurs (hanche, fémur,
tibia). Elle est alimentée par 2 piles Li-Ion (2 × 3,7 V) et pilotée par une carte **Arduino
Nano** [@S04, ch. 1 p. 7].

[FAIT] Les skill books prévoient explicitement l'usage visé par notre sujet : après calibration et
tests, « vous pourrez programmer votre trajectoire ou votre chorégraphie » [@S04, ch. 1 p. 7].

[FAIT] Mécaniquement, le robot est un **quadrupède** à 12 servomoteurs, même s'il est appelé
« araignée » [@S03, §1.1].

Trois caractéristiques du robot structurent directement le sujet :

1. [FAIT] **Chaque robot a sa propre carte et sa propre horloge.** Le programme du robot et sa
   séquence de mouvements tournent sur l'Arduino embarquée [@S04, ch. 2 p. 9-10].
2. [FAIT] **La séquence de mouvements est écrite dans le code**, dans `Trajectory::setup()`, et
   n'est pas reçue de l'extérieur [@S04, §2.1 p. 10].
3. [FAIT] **Dans la version de référence, le robot n'a pas de communication à distance** : pas de
   Wi-Fi ni d'autre moyen de communication distante, le robot étant piloté par des séquences
   locales ou par une connexion directe [@S03, §3.4].

[FAIT] Le contrôleur de pattes synchronise les pattes **d'un même robot**, par une estimation en
boucle ouverte [@S04, §2.1 p. 10]. Les skill books ne décrivent aucun mécanisme de coordination
**entre** robots.

## 4. Travaux antérieurs sur ce robot

[FAIT] L'an dernier, une équipe a fait évoluer le robot selon deux axes : l'ajout d'un module de
calcul (Compute Module) relié à l'Arduino par liaison série, et le remplacement de l'Arduino Nano
par une Arduino Nano RP2040 Connect pour ajouter le Wi-Fi et un pilotage à distance [@S03, §4.1].

[FAIT] Selon leurs propres conclusions, le fonctionnement avec plusieurs robots physiques connectés
en même temps reste à préciser et à tester [@S03, §6.8], et leur rapport ne contient aucune mesure
de synchronisation entre robots.

[FAIT] Dans leur code, un même ordre peut être envoyé à plusieurs robots, mais chaque robot le
reçoit indépendamment, par sa propre file et son propre fil d'exécution [@S07, app.py l. 35-141 et
181-203] : il n'existe pas de départ synchronisé.

Ces travaux servent uniquement de **guide** : tout sera refait et mesuré par notre équipe
(détails dans `03-etat-de-l-art.md`, §3).

## 5. Domaine : de la coordination multi-robots à la robotique en essaim

### 5.1 La robotique en essaim
[FAIT] La robotique en essaim est une approche de la robotique collective inspirée des
comportements auto-organisés des animaux sociaux. Par des règles simples et des interactions
locales, elle vise des comportements collectifs robustes, capables de passer à l'échelle et
flexibles [@S09, résumé].

[FAIT] Elle traite de la conception, de la construction et du déploiement de grands groupes de
robots qui se coordonnent pour résoudre un problème ou accomplir une tâche, avec l'objectif de
systèmes plus robustes, tolérants aux pannes et flexibles qu'un robot seul [@S10, introduction].

### 5.2 Positionnement du projet
[HYPOTHÈSE] Avec un nombre de robots probablement réduit, le projet relève davantage de la
**coordination multi-robots** que de la robotique en essaim au sens strict, qui vise de grands
nombres de robots. Le code de l'an dernier référence 17 araignées [@S06, corrections.h l. 15-33],
mais le nombre qui nous sera prêté n'est pas connu `[À VÉRIFIER]`.

## 6. Enjeux du domaine

[FAIT] Le passage de la robotique en essaim aux applications industrielles n'a pas encore été
réussi, et la littérature compte peu d'applications réelles mettant en œuvre de véritables
algorithmes d'essaim [@S11, résumé].

**Conséquence pour le projet** : démontrer une coordination effective sur du **matériel physique**,
et la **mesurer**, est une contribution cohérente avec les enjeux du domaine.

## 7. Pourquoi la chorégraphie comme cas d'étude

[FAIT] La chorégraphie robotique sur musique a été explorée sur des humanoïdes, des quadrupèdes,
des bras robotisés et des essaims de drones. Concevoir des mouvements synchronisés y demande un
important travail de réglage pour garantir faisabilité, sécurité et alignement avec la musique
[@S12, section II.A].

[FAIT] Quand le nombre de robots augmente, la complexité de la conception chorégraphique et de
l'analyse de sécurité peut rapidement devenir ingérable [@S13, introduction].

[FAIT] Sur un essaim de drones, des retards d'exécution doivent être compensés par un algorithme
de correction pour synchroniser les robots [@S15, résumé].

**Intérêt pour le projet** : une chorégraphie rend immédiatement **visible** toute erreur de
synchronisation entre robots. C'est un cas d'étude exigeant, facile à juger à l'œil et adapté à
une démonstration orale.

## 8. Besoins identifiés

### 8.1 Besoins fonctionnels exprimés par l'énoncé [@S02]
| ID | Besoin |
|---|---|
| B1 | Faire évoluer **plusieurs** robots araignées ensemble. |
| B2 | Coordonner les robots au moyen d'une **API de communication**. |
| B3 | Obtenir des **mouvements synchronisés** entre les robots. |
| B4 | Démontrer des **comportements collectifs** au travers d'une **chorégraphie**. |

### 8.2 Besoins techniques qui en découlent
| ID | Besoin | Justification |
|---|---|---|
| B5 | Doter chaque robot d'un **moyen de communication**. | [FAIT] Absent de la version de référence [@S03, §3.4]. |
| B6 | Disposer d'une **référence de temps commune** ou d'un signal de départ commun. | [FAIT] Les horloges de nœuds distincts dérivent les unes par rapport aux autres [@S16, introduction] ; [FAIT] chaque robot a sa propre carte [@S04, ch. 1 p. 7]. |
| B7 | Pouvoir **déclencher** une chorégraphie de l'extérieur au lieu de la lancer à l'allumage. | [FAIT] La séquence est écrite dans `Trajectory::setup()` [@S04, §2.1 p. 10]. |
| B8 | Tenir compte de l'**exécution séquentielle** de l'Arduino. | [FAIT] La simultanéité des pattes est une illusion obtenue en découpant et alternant les mouvements [@S05, ch. 1 p. 7]. |

### 8.3 Besoins issus du cadre PROJEX [@S01, slide 10]
| ID | Besoin |
|---|---|
| B9 | Fournir un **démonstrateur** fonctionnel et ses codes sources documentés. |
| B10 | Pouvoir **prouver** le fonctionnement (« Est-ce que ça marche ? À quel point ? »), donc **mesurer** la synchronisation. |
| B11 | Rendre le travail **reproductible** par un tiers. |
| B12 | Disposer d'une **vidéo de démonstration de secours**. |

## 9. Parties prenantes
| Partie prenante | Rôle / attente |
|---|---|
| Équipe (Clara SION SAUNIER, Rachel DIGNA, Eva DOUMBE NGANGUE, Aida GOUMBLE) | Conception, réalisation, documentation |
| Tuteurs (Loïca AVANTHEY, Laurent BEAUDOIN, SEAL) | Encadrement, fourniture du matériel et des skill books, évaluation [@S01, slide 10] |
| Jury / public de la soutenance | Doit percevoir la synchronisation et le caractère collectif de la chorégraphie |

## 10. Contraintes

### 10.1 Contraintes connues
- [FAIT] Tout choix technique s'accompagne de contraintes : puissance, taille, poids, bande
  passante, latence, fiabilité, coût [@S01, slide 5].
- [FAIT] Équipe de 2 à 4 personnes (ici 4) [@S01, slide 9].
- [FAIT] **Sécurité du matériel** : ne jamais programmer l'araignée allumée [@S04, ch. 1 p. 7] ;
  tester dans un espace dégagé [@S04, ch. 4 p. 15] ; respecter la plage min/max propre à chaque
  servomoteur [@S05, ex. 3 p. 22].
- [FAIT] **Calibration obligatoire** de chaque araignée avant tout test, avec un jeu de corrections
  par identifiant d'araignée (`SPIDER_ID`) [@S04, ch. 3 p. 13-14].
- [FAIT] Robot et poste de contrôle doivent être sur le même réseau local dans la solution Wi-Fi de
  l'an dernier, et certains réseaux isolent les appareils entre eux [@S03, §9.2].

### 10.2 Éléments encore inconnus (à obtenir auprès des tuteurs)
- Nombre d'araignées prêtées et **version électronique** de chacune (carte mère d'origine, Nano +
  shield DFR0012, Nano RP2040 Connect, Compute Module) [@S03, §2.4, §5, §6]. `[À VÉRIFIER]`
- Emplacement du croquis officiel `spider_sealk` : S04 indique « une fois téléchargé » sans donner
  de lien [@S04, ch. 2 p. 9]. `[À VÉRIFIER]`
- API de communication imposée ou libre ; l'API HTTP de l'an dernier est-elle attendue ? `[À VÉRIFIER]`
- Place attendue de la vision par ordinateur dans ce sujet. `[À VÉRIFIER]`
- Date de la soutenance et jalons intermédiaires. `[À VÉRIFIER]`
- Le *Skillbook Arduino – Prise en main*, prérequis des deux skill books [@S04, p. 3 ; @S05, p. 3]. `[À VÉRIFIER : à récupérer]`
