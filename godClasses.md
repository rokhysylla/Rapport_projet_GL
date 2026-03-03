# 4.5 – Identification des God Classes

## Nombre de méthodes par classe

Pour analyser la présence éventuelle de god classes, j'ai commencé par étudier le nombre de méthodes par classe pour chaque module (gson,metrics, extras,proto, test-shrinker). Afin d'éviter toute ambiguïté liée aux classes internes, j'ai adopté la convention suivante : un fichier .java correspond à une classe analysée.
### Protocole de recherche
Les chiffres proviennent de la métrique size/functions de sonarqube.
J'ai noté pour toutes les classes de chaque module le nombre de méthodes et donné le minimum, le maximum, la moyenne, la médiane.
### Résultats par module

#### Extras
Min = 1, Max = 12, Moyenne = 5, Médiane = 4

Les classes sont globalement équilibrées. Aucune classe ne se détache fortement.

#### Gson (JSON)
Min = 0, Max = 47, Moyenne ≈ 12,2, Médiane = 3

La médiane est très faible (3), mais la moyenne est élevée (12,2). Cela signifie que la majorité des classes sont petites, mais que certaines classes très volumineuses et donc tirent la moyenne vers le haut. On observe plusieurs classes dépassant 40 méthodes, ce qui constitue un signal fort de possible god class dans ce module.

#### Metrics
Min = 3, Max = 20, Moyenne ≈ 7,2, Médiane = 5

Module relativement homogène. Une classe plus volumineuse (20 méthodes), mais pas de déséquilibre majeur.

#### Proto
Une seule classe avec 16 méthodes. Ce n'est pas problématique, car le module entier repose sur une unique classe.

#### Test Shrinker
Min = 1, Max = 6, Moyenne ≈ 1,8, Médiane = 1

Classes très petites et spécialisées.

### Analyse globale des méthodes

Le module Gson (JSON) est celui qui présente la plus forte dispersion. La différence importante entre la médiane (3) et la moyenne (12,2) montre une concentration de responsabilités dans quelques classes volumineuses.

Ces classes dépassant 40 méthodes constituent des candidates sérieuses à l'identification comme god classes, car elles concentrent un grand nombre de comportements et risquent d'assumer trop de responsabilités.

Les autres modules apparaissent globalement bien structurés, avec des classes de taille modérée et une répartition plus homogène des méthodes.

## Variables d'instance par classe

### Méthodologie

Pour éviter un comptage manuel impossible pour chaque classe, là j'ai utilisé un script PowerShell que j'ai lancé à la racine du projet.

Il parcourt tous les fichiers *.java situés dans `src/main/java` (donc tous les modules, pas les tests), en excluant `package-info.java` et `module-info.java`.

Pour chaque fichier .java, je compte les champs d'instance (variables d'instance) de la classe :
- Déclarations au niveau "classe" qui finissent par `;`
- En excluant tout ce qui est `static` (car ce n'est pas de l'état par instance)
- En évitant de compter les variables locales dans les méthodes.

`Convention assumée : 1 fichier .java = 1 "classe" observée.`

### Résultats globaux

Fichiers analysés : 113

- Min : 0 variable d'instance
- Max : 162 variables d'instance
- Moyenne : 13,38 variables d'instance par classe
- Médiane : 4 variables d'instance par classe

### Interprétation

La médiane à 4 indique que la majorité des classes restent petites/modérées côté état.

La moyenne à 13,38, beaucoup plus haute, montre qu'il y a quelques classes très chargées qui tirent la moyenne vers le haut.

Le max à 162 constitue un signal majeur d'une classe excessivement volumineuse.

### Top des classes les plus chargées

- `com.google.gson.internal.LinkedTreeMap` -> 162 champs
- `com.google.gson.metrics.ParseBenchmark` -> 110 champs
- `com.google.gson.internal.bind.TypeAdapters`-> 81 champs
- `com.google.gson.GsonBuilder` → 79 champs
- `com.google.gson.stream.JsonReader` → 75 champs
- `com.google.gson.stream.JsonWriter` → 72 champs
- `com.google.gson.internal.bind.ReflectiveTypeAdapterFactory` → 65 champs
- `com.google.gson.internal.GsonTypes` → 54 champs

Ces classes sont typiquement des classes "centrales" (binding, streaming, factories, builders, structures internes) où le projet concentre sa complexité.

### Comparaison avec l'analyse des méthodes

Les résultats montrent des classes avec un état gigantesque (LinkedTreeMap, GsonBuilder, JsonReader/Writer, TypeAdapters), qui sont d'excellentes candidates pour l'identification de god classes.

## Nombre de lignes de code par classe et comparaison avec les variables d'instance

Afin d'analyser la taille des classes, je me suis appuyé sur le nombre de lignes de code (LOC) par classe , métrique fournie par sonarQube.
Comme précédemment, j'ai adopté la convention suivante : un fichier .java correspond à une classe analysée, ce qui permet d'éviter les ambiguïtés liées aux classes internes et de garder une méthode cohérente sur l'ensemble du projet.

### Résultats par module

#### JSON (gson)
65 classes, 10 690 LOC au total
- Min = 4
- Max = 1201
- Moyenne ≈ 164,5 LOC par classe

On observe ici une très forte dispersion du nombre de lignes car  certaines classes sont extrêmement volumineuses (plus de 1000 lignes), tandis que d'autres restent très petites. Cela confirme déjà une hétérogénéité structurelle importante dans ce module.

#### extras
8 classes, 746 LOC au total
- Min = 5
- Max = 242
- Moyenne ≈ 93,3 LOC

Le module présente quelques classes relativement longues, mais reste globalement plus modéré que Gson.

#### Metrics
6 classes, 759 LOC
- Min = 18
- Max = 401
- Moyenne ≈ 126,5 LOC

On observe une classe significativement plus longue que les autres, mais le module reste de taille raisonnable.

#### Proto
1 classe, 269 LOC
- Min = Max = 269

La totalité du module repose sur une seule classe. La taille reste importante mais plutôt cohérente avec la structure du module.

#### Test Shrinker
18 classes, 779 LOC
- Max = 256
- Moyenne ≈ 43,3 LOC

Les classes sont globalement petites et spécialisées.

### Comparaison avec les variables d'instance

Lors de la question précédente, l'analyse des variables d'instance avait déjà mis en évidence une forte concentration d'état dans certaines classes (par exemple plus de 160 variables dans certaines classes du module Gson).

En comparant avec les LOC :

- Les classes ayant un nombre très élevé de lignes de code sont souvent celles qui présentent également un nombre important de variables d'instance.
- Le module Gson se distingue nettement : il combine une moyenne élevée de LOC par classe, un maximum très important (1201 LOC) et une forte concentration de variables d'instance.
- Cette combinaison (taille élevée + état important) est caractéristique des classes susceptibles de concentrer trop de responsabilités.
- Cela renforce l'hypothèse que certaines classes du module JSON constituent des candidates sérieuses à l'identification comme God Classes.
- À l'inverse, les modules comme Test Shrinker ou Xtra présentent des classes de taille plus modérée et moins d'état, ce qui suggère une structure plus distribuée et moins centralisée.


# Identification des God Classes

## Analyse du couplage entre classes

Afin d'identifier d'éventuelles God Classes, je complète l'analyse précédente (nombre de méthodes, variables d'instance et lignes de code) par une étude du couplage entre classes à l'aide de l'outil jdeps.

### Résultats globaux

L'analyse porte sur 216 classes. Les résultats globaux sont les suivants :

- Nombre minimal de dépendances uniques : 1
- Nombre maximal : 80
- Moyenne : 11,12
- Médiane : 9

La majorité des classes dépend donc d'environ 9 à 11 autres classes. Toute classe dépassant largement cette valeur peut être considérée comme fortement couplée et potentiellement problématique.

## Classe TypeAdapters

La classe `com.google.gson.internal.bind.TypeAdapters` présente 80 dépendances uniques, soit près de sept fois la médiane. Parmi celles-ci :

- 45 dépendances internes au projet
- 35 dépendances vers la bibliothèque standard Java
- 0 dépendance externe

Cette classe est donc fortement centrale dans l'architecture. En croisant avec les analyses précédentes (taille importante en lignes de code, nombre élevé de méthodes et de variables d'instance), elle apparaît comme la candidate la plus crédible à une God Class.

Elle concentre de nombreuses responsabilités liées à l'adaptation de types et connaît une grande partie du système, ce qui augmente le couplage et peut compliquer la maintenance.

## Classes GsonBuilder et Gson

Les classes `GsonBuilder` (56 dépendances) et `Gson` (55 dépendances) présentent également un couplage élevé :

- Environ 30 dépendances internes
- Environ 25 dépendances Java
- Aucune dépendance externe

Ces classes jouent cependant un rôle central d'orchestration dans le framework. Leur forte connectivité semble cohérente avec leur responsabilité de point d'entrée et de configuration. Elles sont volumineuses, mais leur position centrale est structurellement justifiée.

Elles peuvent donc être considérées comme des classes centrales architecturales plutôt que comme de véritables God Classes problématiques.

## Observation sur les dépendances externes

Aucune classe ne présente de dépendances externes significatives (OtherDeps = 0 pour les principales classes). Cela indique que l'architecture est relativement maîtrisée et peu dispersée vers des bibliothèques tierces.

## Conclusion

En croisant :

- Le nombre de méthodes par classe
- Le nombre de variables d'instance
- Le nombre de lignes de code
- Le nombre de dépendances uniques

La classe `TypeAdapters` apparaît comme la candidate la plus pertinente à une God Class. Elle concentre un volume important de responsabilités et présente un fort couplage interne.

En revanche, les classes `Gson` et `GsonBuilder`, bien que volumineuses et fortement connectées, semblent justifiées par leur rôle d'orchestration au sein du framework.

L'analyse met ainsi en évidence une architecture globalement cohérente, avec quelques classes centrales très chargées mais majoritairement explicables par leur fonction dans le système.


