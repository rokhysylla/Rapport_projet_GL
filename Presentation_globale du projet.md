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
Le projet ne produit pas un programme  ou un fichier éxécutable.
Le projet produit :
soit une chaîne JSON
soit un objet Java

Il ne génère pas de programme autonome.

## Description du projet
Lors de l’audit du projet Gson, nous avons cherché un fichier README décrivant le projet, son installation et son utilisation.  
Cependant celui-ci ne décrit pas en détail l’architecture interne ni les choix de conception pour comprendre le projet en profondeur.

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
Le projet contient bien un dossier dédié aux tests (src/test).
Plusieurs classes de tests sont présentes, ce qui montre une volonté de validation du code.
Dans ce projet, on retrouve principalement des tests unitaires.
**Données quantitatives** 
Au total, le projet comporte :

4619 tests unitaires
0 erreur
0 échec
19 tests ignorés (skipped)
L’absence d’erreurs et d’échecs indique une bonne stabilité des tests existants.
**Repartition des tests par module**
- Gson (JSON)	4536 test unitaires soit un ratio de 4536/113​=40.14 environ 40 test par classe

- Extra	40 test unitaires soit un ratio de 40/14 = 2.85 environ 3test par classe

- Proto	12 test unitaires soit un ratio de 12/3 = 4 environ 4 test par classe

Les modules Test-Graal-Native et Test-JPMS contiennent respectivement 10 et 12 test unitaires Cependant, ils ne contiennent pas de classes Java dans src/main, ce qui suggère qu’il s’agit de modules techniques ou de tests d’intégration spécifiques.

- le module test-shrinker contient 33 classes, mais aucun test associé n’a été identifié, ce qui constitue un point faible en termes de couverture et de validation.

**Répartition des classes**
- Le projet contient 185 classes principales, réparties comme suit :
113 dans le module Gson
14 dans extras
22 dans metric
3 dans proto
33 dans test-shrinker

Le ratio moyen global est :
    4619/185 =25 tests par classe

Cependant On constate que près de 98 % des tests sont concentrés dans le module Gson (JSON).
Les autres modules présentent un nombre très faible de tests, ce qui suggère un déséquilibre dans la stratégie de validation.

**Structuration des tests**
`Organisation`
Bien que le projet comporte un nombre très important de tests, leur organisation structurelle présente certaines limites.

Dans le module JSON, le dossier src/main est structuré en plusieurs sous-packages (bin, util, reflect, sql, stream, etc.), contenant des classes concrètes, des interfaces, des classes abstraites et des annotations.

Cependant, l’organisation du dossier src/test ne reflète pas fidèlement cette architecture.

Les sous-dossiers de test ne correspondent pas clairement aux sous-packages du code source.
Il est donc difficile d’identifier précisément :

quelle classe est testée,
dans quel module logique elle appartient,
et si toutes les classes possèdent un test associé.
`Classes non testées`
Certaines classes présentes dans src/main ne disposent pas de classes de tests correspondantes.C'est le cas des classes se trouvant dans le  modules metric

`Nombre d’assertions dans certaines méthodes`
On observe que certaines méthodes de test contiennent plusieurs assertions.
En théorie, un test devrait vérifier un comportement précis.
Un test = un comportement vérifié

`Présence de classes non destinées aux tests dans src/test`
On remarque que dans le dossier src/test, certaines classes ont été créées et ne correspondent pas à des classes de test.

Cela peut poser plusieurs problèmes :

Confusion dans la structure du projet
Difficulté à distinguer code de production et code de test
En principe, le dossier src/test devrait contenir uniquement :
des classes de test
éventuellement des classes utilitaires dédiées aux tests (mocks...)
### Les commentaires
Le projet comporte 13 986 lignes de code pour 5 119 lignes de commentaires, soit un taux global de 36,6 %.

Dans le module principal (gson), on observe 11 198 lignes de code et 4 545 lignes de commentaires, soit un taux d’environ 40,5 %.

Ce taux est relativement élevé. Cependant, une analyse qualitative montre que ces commentaires ne sont pas exclusivement des commentaires explicatifs.

On distingue :
- Des commentaires Javadoc documentant les méthodes et paramètres.

- Des commentaires de licence et de copyright en en-tête des fichiers.

- Des lignes de code commentées (code désactivé).

