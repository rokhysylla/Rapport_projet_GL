## Présentation et utilité du projet GSON
### Présentation
Gson est une bibliothèque Java développée par Google qui permet de convertir des objets Java en JSON et inversement.
Plus explicitement ,elle sert à:
**transformer des objets Java en texte JSON (sérialisation)**

**reconstruire des objets Java à partir du JSON (désérialisation)**

### Les fonctionnalités principales
Les principales fonctionnalités de Gson sont :
- Conversion automatique d’objets Java en JSON

- Conversion d’une chaîne JSON en objet Java 

- Support des types génériques Java (List, Map, classes génériques)

- Possibilité de personnaliser la représentation JSON des objets

- Support d’objets complexes avec héritage et structures imbriquées

### Comment lancer / utiliser le projet
Gson n'est pas une application éxécutable mais une bibliothèque Java.Du coup pour l'utiliser ,il faut l'ajouter comme dépendance dans un projet Java.
Ensuite on pourra utiliser l'API dans le code java 
### Résultat du projet
Le projet ne produit pas un programme  ou un fichier éxécutable.Le résultat sera une chaine JSON ou un objet Java . (à revoir)

## Description du projet
Lors de l’audit du projet Gson, nous avons cherché un fichier README décrivant le projet, son installation et son utilisation.  
Cependant, aucun README complet et détaillé n’a été trouvé dans le dépôt analysé, ou celui-ci est très limité et non suffisant pour comprendre le projet en profondeur.

La principale source d’information disponible est une documentation partielle décrivant l’objectif général de la bibliothèque, mais sans fournir de détails sur l’architecture interne ou la structure du code.

## Historique du logiciel 
### Analyse du git 
Lors de l’analyse du dépôt GitHub,nous avons vu que ce projet dispose de trois contributeurs.
Le projet est actuellement en mode maintenance,il est d'ailleurs toujours actif .
Le dépôt contient environ dix branches. La branche principale main est active et contient les versions officielles du projet.  
Les autres branches sont anciennes (plusieurs années) et ne semblent plus utilisée
Environ 97 Pull Requests ont été identifiées.  
Certaines Pull Requests sont acceptées (merge), ce qui est indiqué par un statut vert, tandis que d’autres sont refusées ou fermées sans intégration, indiquées par un statut rouge ou sans statut.


## Analyse approfondie

### Tests
On va s'interesser aux tests
