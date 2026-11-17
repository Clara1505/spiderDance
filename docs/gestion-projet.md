# Gestion de projet

> **Statut** : brouillon v0.2 — à valider par l'équipe
> **Dernière mise à jour** : 2026-11-17
> **Légende** : `[FAIT]` vérifié et sourcé · `[HYPOTHÈSE]` raisonnement non vérifié · `[À VÉRIFIER]` plausible mais non confirmé
> **Citations** : `[@ID, localisation]`, clés définies dans `docs/sources.bib`
>
> ⚠️ Les **poids**, **responsables** et **charges** ci-dessous sont des **propositions** : aucune mesure ni information sur les disponibilités individuelles n'a été fournie. À ajuster par l'équipe.

---

## 1. Cadre et échéances

| Élément | Valeur | Source / statut |
|---|---|---|
| Livrables | Présentation orale (~15 min) + slides ; démonstrateur + vidéo de backup ; archive de code ; rapport de 20 à 50 pages | [FAIT] [@S28] |
| Envoi | Par mail aux **deux** tuteurs, avant la dernière séance | [FAIT] [@S28] |
| Échéance | 15 décembre 2026 (dernière séance) | Déclaré par l'équipe `[À VÉRIFIER]` |
| Avant-dernière séance | Lundi 14 décembre, 10 h à 13 h | Déclaré par l'équipe |
| Objectif interne | Finaliser pendant la séance du 14/12 et **envoyer à 13 h au plus tard**. Visée : envoi à 12 h 30 pour garder une marge | Décision d'équipe |
| Soutenance | Date, heure et format inconnus (probablement la dernière séance) | `[À VÉRIFIER]` |
| Temps restant | 27 jours entre le 17/11 et le 14/12 | Calcul |
| Disponibilité | Les 4 membres sont en entreprise. **La semaine du 30/11 au 06/12 doit être la plus légère possible** | Consigne de l'équipe |

## 2. Équipe et rôles (proposition)

Les compétences et les disponibilités de chacun ne sont pas renseignées : la répartition suit des **fonctions**, à échanger librement. Règle proposée : chaque tâche a un **responsable** et un **relais**, pour qu'aucune tâche critique ne dépende d'une seule personne.

| Membre | Rôle principal | Relais sur | Responsable de |
|---|---|---|---|
| **Clara** | Coordination, poste de contrôle (ordinateur, API, départ commun), journal, intégration du rapport | Slides, mesures | M0, M2, M5, intégration de M10 |
| **Rachel** | Robots et firmware : inventaire, calibration, flash, tests des hypothèses sur le code | Poste de contrôle | M1, EXP-003, M12 |
| **Eva** | Chorégraphie (figures, script, enchaînement) et slides | Calibration | M6, M8, M11 |
| **Aida** | Expériences et mesures : protocole, vidéo, analyse, vidéo de secours | Chorégraphie | M3, M7, M9 |

⚠️ Clara cumule coordination, poste de contrôle et intégration du rapport : c'est un **point de fragilité**. À rééquilibrer si possible (par exemple confier à Rachel ou Aida l'intégration LaTeX).

Tout le monde participe aux calibrations (EXP-001) et écrit les chapitres du rapport qui correspondent à son travail.

## 3. Timeline : macro-tâches, sans parallélisation

Liste **séquentielle** voulue par les encadrants [@S28]. Poids de 1 (léger) à 5 (lourd), relatif. La parallélisation réelle entre les quatre personnes est au §5.

| ID | Macro-tâche | Poids | POC intermédiaire / livrable partiel | Zone d'ombre |
|---|---|---|---|---|
| M0 | Cadrage : valider les documents du projet, envoyer les questions restantes aux tuteurs, confirmer dates et formations | 1 | Mail envoyé aux tuteurs ; docs validés | Délai de réponse des tuteurs |
| M1 | Inventaire et calibration des robots (EXP-001) | 3 | **POC 1** : chaque robot calibré passe le test de base et le test avancé [@S04, ch. 3-4] | Nombre de robots, version de chacun, état, créneaux d'accès |
| M2 | Prise en main du firmware Wi-Fi et du poste de contrôle (flash, `SPIDER_ID`, réseau, CLI) + test des hypothèses sur S06 (EXP-003) | 3 | **POC 2** : un robot piloté en Wi-Fi depuis l'ordinateur ; hypothèses de S06 testées | Réseau de la salle (clients isolés) [@S03, §9.2] ; connexion lente [@S03, §6.8] ; identifiants en dur [@S06, wifi.h l. 9-12] |
| M3 | Mesure de référence (EXP-002) : 2 robots, même ordre, sans synchro | 3 | Livrable partiel : écart mesuré (référence sans recalage) | Méthode de mesure sans vision (cadence de la caméra) ; seuil jugé satisfaisant inconnu |
| M4 | Choix techniques : ADR (reprise du code, architecture, communication, départ commun, format de chorégraphie), **schéma système**, **tableaux comparatifs** | 3 | Livrable partiel : ADR-001 à ADR-005, schéma système et tableaux comparatifs **rédigés le 17/11, à valider par le groupe** | Valeurs des tableaux à revérifier ; hypothèses des ADR à tester en M2 et M3 |
| M5 | Envoi parallèle et recalage à chaque mouvement sur 3 robots (option A+ de l'ADR-002 ; MVP : O1, O2, O3) | 4 | **POC 3** : 3 robots enchaînent plusieurs mouvements, chacun lancé après la disponibilité de tous | « Disponible » = répond à `/api/health` `[HYPOTHÈSE]` ; limite de 60 commandes cumulées `[HYPOTHÈSE]` ; un robot occupé ne lit pas les requêtes [@S03, §6.4] ; robot perdu du Wi-Fi |
| M6 | Format de chorégraphie et exécution (vérification de faisabilité avant téléversement) | 3 | **POC 4** : courte chorégraphie jouée par 2 robots | Limite de 60 entrées, `deep_sleep` sans effet `[HYPOTHÈSE]` [@S06, trajectory.h l. 45-47 ; body.h l. 59-129] |
| M7 | Mesure avec départ commun et comparaison à la référence (EXP-004) | 3 | Livrable partiel : tableau avant / après | Fiabilité d'une mesure faite à la main sur vidéo |
| M8 | Chorégraphie finale avec une figure collective (O4, O5) et répétitions | 3 | Démonstrateur complet joué sans intervention manuelle | Fiabilité en conditions de soutenance |
| M9 | Vidéo de secours | 2 | Vidéo complète de la démonstration | Taille du fichier pour l'envoi par mail |
| M10 | **Rapport** (20 à 50 pages, reproductible sans notre code) | 5 | Squelette LaTeX ; chapitres 01 à 03 repris ; chapitres ADR / EXP ; relecture croisée | Volume de rédaction ; relecture par quelqu'un qui n'a pas écrit le chapitre |
| M11 | Slides (~15 min) et répétition chronométrée | 3 | Deck complet, passé une fois en entier | Format de la soutenance inconnu |
| M12 | Archive de code et reproductibilité (`reproductibilite.md`, README) | 2 | Archive nettoyée, **sans identifiants** | Arborescence à définir ; secrets du firmware |
| M13 | Envoi des livrables aux deux tuteurs | 1 | Mail envoyé aux deux adresses | Taille des pièces jointes |
| | **Total** | **39** | | |

## 4. Zones d'ombre

| # | Zone d'ombre | Impact | Action | Pour quand |
|---|---|---|---|---|
| 1 | Nombre de robots et version électronique de chacun (seule la carte Nano RP2040 Connect est déclarée) | Taille du MVP, choix de communication | Inventaire (M1) | 22/11 |
| 2 | Accès aux robots : dates, lieu, créneaux | Tout le planning | Demander aux tuteurs | 20/11 |
| 3 | Niveau de synchronisation jugé satisfaisant | Objectifs O1 à O3 | **Réglé le 17/11** : synchronisé à l'œil, recalage sur le mouvement suivant | — |
| 4 | Architecture centralisée acceptée ? | Conception | **Réglé le 17/11** : acceptée par les tuteurs (ADR-002) | — |
| 5 | Date, heure et format de la soutenance | Slides, démonstrateur | Question aux tuteurs | 22/11 |
| 6 | Modalité d'envoi de la vidéo et de l'archive (taille par mail) | Dernier jour | Question aux tuteurs | 29/11 |
| 7 | Diffusion à tous les robots possible avec la bibliothèque Wi-Fi ? | Piste d'amélioration seulement | Plus nécessaire pour l'option A+ ; à regarder si la mesure montre un départ trop décalé | — |
| 8 | Hypothèses sur S06 (`deep_sleep`, 60 ordres, rejeu après `DONE`, trajectoire jouée au démarrage) | Fiabilité de la base de code | EXP-003 | 29/11 |
| 9 | Réseau : la salle isole-t-elle les appareils ? Faut-il un point d'accès à nous ? | Connexion des robots | Tester sur place ; prévoir un point d'accès [@S03, §9.2] | 29/11 |
| 10 | Caméra disponible et sa cadence | Précision de la mesure | Inventaire du matériel de l'équipe | 22/11 |
| 11 | Formations spécifiques proposées par les tuteurs : dates | Charge | Demander [@S28] | 22/11 |
| 12 | Auteur de chaque fichier du code S06 à S08 (le rapport S03 attribue le Wi-Fi et l'API à Maxence Dupont, le Compute Module à Célian Crépin) | Crédits dans le rapport | Demander aux tuteurs quels fichiers viennent du croquis officiel | 06/12 |
| 13 | Hypothèses de l'option A+ : « disponible » = répond à `/api/health` ; limite de 60 commandes cumulées | Recalage et chorégraphies longues | EXP-003 | 29/11 |

## 5. Planning par semaine (parallélisé)

| Semaine | Charge | Objectif | Clara | Rachel | Eva | Aida |
|---|---|---|---|---|---|---|
| **S1** : mar 17 → dim 22/11 | Moyenne | Cadrer, inventorier, calibrer | Faire valider les documents par le groupe, committer, mail de suivi aux tuteurs (zones 2, 5, 6, 11, 12) | Inventaire des robots (nombre, carte, `SPIDER_ID`, min/max des boîtes) ; EXP-001 robot 1 | Appui calibration ; inventaire des figures disponibles [@S04, ch. 4-5] | Protocole de mesure (caméra, cadence, repère) ; lecture des chapitres calibration |
| **S2** : lun 23 → dim 29/11 | **Élevée** (dernière semaine pleine) | Robots calibrés, Wi-Fi, mesure de référence, début du départ commun | M2 (réseau, CLI, superviseur) ; début M5 ; squelette LaTeX par conversion de `docs/01` à `03` | M1 fin (tous les robots) ; flash du firmware Wi-Fi ; EXP-003 | Calibration avec Rachel ; format de script de chorégraphie ; début des tableaux comparatifs | **EXP-002** avec Clara |
| **S3** : lun 30/11 → dim 06/12 | **Très légère : ≤ 2 h par personne**, travail de bureau, aucun robot obligatoire | Mettre au propre ce qui a été fait | Relire et ajuster les ADR et le schéma système après les premiers essais ; fiche EXP-003 avec Rachel | Fiches EXP-001 et EXP-003 ; section calibrations de `reproductibilite.md` | Revérifier les valeurs des tableaux comparatifs ; chorégraphie écrite (format de l'ADR-005, mode à blanc) | Fiche EXP-002 ; protocole de mesure de l'ADR-004 mis au propre |
| **S4** : lun 07 → dim 13/12 | **Très élevée** (bouclage) | Lun-mer : robots. Jeu : gel du code. Jeu-dim : rapport, slides, vidéo, archive | M5 fini avec Rachel (lun-mar) ; intégration du rapport (jeu-sam) | M5 et M6 sur robots (lun-mer) ; archive et reproductibilité (jeu-ven) | M6 et M8 sur robots (lun-mer) ; slides (jeu-sam) | M7 mesures (mar-mer) ; vidéo de secours (jeu) ; chapitre résultats (ven-sam) |
| **lun 14/12** (10 h-13 h) | Séance | Vérifier et envoyer | Envoi du mail à 12 h 30 au plus tard | Test de l'archive sur une machine propre | Dernière répétition | Lecture de la vidéo, relecture |
| **mar 15/12** | Dernière séance | Soutenance `[À VÉRIFIER]` | Démo | Démo | Présentation | Vidéo de secours prête |

`[HYPOTHÈSE]` Les essais sur robots supposent un accès à l'école : à confirmer (zone d'ombre 2). **Option S3** : un créneau robots court (≤ 2 h), uniquement entre volontaires, si les robots sont accessibles. Il ne doit pas devenir obligatoire.

## 6. Jalons et plan B

| Jalon | Date | Critère | Si non atteint |
|---|---|---|---|
| J0 | dim 22/11 | Mail aux tuteurs envoyé ; inventaire fait ; au moins 1 robot calibré | Relancer les tuteurs ; réduire à ce qui est disponible |
| J1 | dim 29/11 | POC 2 (un robot piloté en Wi-Fi) ; EXP-002 faite ; au moins 2 robots calibrés | Pas de N > 2 ; EXP-002 reportée au lundi 07/12 |
| J2 | dim 06/12 | Fiches EXP, ADR, schéma système, tableaux comparatifs, chorégraphie écrite | Rattraper en début de S4, sans toucher au robot |
| J3 | jeu 10/12 (soir) | **Gel du code** ; démonstrateur validé (POC 3 et 4) | Présenter la mesure de référence et les meilleurs résultats partiels |
| J4 | dim 13/12 | Rapport, slides, vidéo, archive complets ; répétition chronométrée | Priorité : rapport et vidéo, puis slides |
| J5 | lun 14/12, 12 h 30 | Mail envoyé aux deux tuteurs | — |

Les encadrants valorisent aussi ce qui n'a **pas** fonctionné, avec ses causes [@S28] : si le départ commun n'est pas au point, le rapport documente la tentative et les mesures.

## 7. Risques propres au planning

Les risques techniques sont dans `02-probleme-objectifs.md` §7. Risques supplémentaires :

| Risque | Impact | Parade |
|---|---|---|
| La semaine légère du 30/11 reporte la charge sur S4 | Élevé | Viser J1 en S2 ; en S3, uniquement de la rédaction ; en S4, geler le code le jeudi |
| Accès aux robots limité | Élevé | Grouper les essais en S1, S2 et début de S4 ; demander les créneaux |
| Une personne est un point de fragilité unique (Clara) | Moyen | Relais pour chaque tâche (§2) |
| La démonstration en direct échoue | Moyen | Vidéo de secours terminée avant le jeudi 10/12 |
| Fichiers trop gros pour un envoi par mail | Moyen | Demander la modalité aux tuteurs (zone 6) |
| Identifiants Wi-Fi publiés par erreur sur le dépôt public | Moyen | Fichier local ignoré par git ; relecture avant chaque push |

## 8. Suivi

- Point d'équipe de 15 minutes par semaine (30 minutes en visio en S3, sans rien d'autre).
- Une entrée de journal dans `docs/journal/` après chaque session de travail.
- **Seule l'équipe pousse sur GitHub** (règle d'équipe).

## 9. Conformité aux premiers jalons demandés [@S28]

| Jalon | Où | État |
|---|---|---|
| Contexte et problématique (1–2 slides, ½ à 2 pages) | `01`, `02` | Rédigé, à valider |
| Timeline (macro-tâches, POC, poids 1–5, zones d'ombre) | Ce document, §3 et §4 | Fait |
| État de l'art : projets similaires | `03` | Rédigé, à valider |
| État de l'art : tableaux hardware / software (prix, taille, disponibilité…) | `04-choix-techniques/tableaux-comparatifs.md` | Rédigés avec sources ; **valeurs à revérifier** ; certaines cases à renseigner |
| Schéma système (éléments + interconnexions) | `04-choix-techniques/schema-systeme.md` | Rédigé (v0.1), à valider |
| Formations spécifiques | À suivre | Dates à demander |
