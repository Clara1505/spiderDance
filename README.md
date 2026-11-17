# SpiderDance — chorégraphier un essaim de robots

Projet PROJEX (EPITA) : faire danser un groupe de robots araignées SEALK en les coordonnant par une API de
communication, et démontrer des comportements collectifs à travers une chorégraphie. Domaine : systèmes
embarqués, robotique distribuée et coordination multi-robots.

**Équipe** : Clara SION SAUNIER, Rachel DIGNA, Eva DOUMBE NGANGUE, Aida GOUMBLE.
**Tuteurs** : Loïca AVANTHEY, Laurent BEAUDOIN (SEAL, EPITA).

## Principe

Un ordinateur envoie, par Wi-Fi et HTTP, les mouvements à 3 ou 4 robots en parallèle. Il attend que tous les
robots soient disponibles avant d'envoyer le mouvement suivant, ce qui recale les robots à chaque mouvement.
La synchronisation est mesurée sur vidéo, sans vision par ordinateur. Détails : `docs/04-choix-techniques/`.

## Où trouver quoi

| Chemin | Contenu |
|---|---|
| `docs/01-contexte-besoins.md` | Contexte, besoins, contraintes |
| `docs/02-probleme-objectifs.md` | Problématique, objectifs, périmètre, risques |
| `docs/03-etat-de-l-art.md` | Robot SEALK, travaux antérieurs, techniques transposables |
| `docs/04-choix-techniques/` | Décisions (ADR-001 à ADR-005), schéma système, tableaux comparatifs |
| `docs/05-experimentations/` | Fiches d'expérience (EXP-xxx) |
| `docs/06-validation-resultats.md` | Résultats mesurés |
| `docs/gestion-projet.md` | Timeline, planning, répartition, risques |
| `docs/reproductibilite.md` | Matériel, versions, procédure, code repris / modifié / écrit par nous |
| `docs/journal/` | Journal de bord daté |
| `docs/sources.bib` | Bibliographie |

## Code

Le code repris de l'an dernier (firmware, superviseur, client en ligne de commande) et notre code seront
ajoutés ici. L'arborescence reste à définir. Voir `docs/reproductibilite.md`.

## Sécurité du dépôt

Ce dépôt est **public**. Ne jamais y mettre d'identifiants Wi-Fi, de mots de passe ni d'adresses privées :
ils vont dans des fichiers locaux ignorés par git (voir `.gitignore`), avec des valeurs factices dans le dépôt.

## Sécurité du matériel

Ne jamais programmer une araignée allumée. Poser le robot sur les plots quand il est allumé et relié au
câble. Tester dans un espace dégagé. Respecter la plage min/max de chaque servomoteur.

## État

Projet en cours : phase de cadrage et de choix techniques (novembre 2026). Rendu avant la dernière séance.
