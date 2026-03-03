# 3.3 Répartition des classes dans les paquetages

## Approche 1 – Comptage par paquetage

Pour analyser la répartition, j'ai extrait tous les fichiers .java (en excluant package-info.java) et regroupé les classes selon leur déclaration package ...;.

Il faut préciser que ce comptage correspond aux fichiers Java. Or, un fichier peut contenir plusieurs classes (classes internes notamment). Il s'agit donc d'une approximation structurelle.

### Résultats obtenus

Total fichiers .java comptés : 27

Nombre de paquetages concernés : 5

Minimum : 1 classe dans un paquetage

Maximum : 18 classes dans un paquetage

Moyenne : 5,40 classes par paquetage

On observe une forte concentration dans un paquetage principal (18 classes), alors que les autres restent beaucoup plus légers.

Cela montre une organisation assez centralisée.

## Approche 2 – Vision globale du projet

Pour élargir l'analyse, j'ai compté toutes les classes du projet multi-modules (sources + tests).

### Résultats

Total classes détectées (main + tests) : 255

SonarQube indique : 185 classes

La différence s'explique par :

- Des règles d'inclusion/exclusion propres à SonarQube
- Une analyse basée sur l'AST côté Sonar
- Un périmètre différent (modules activés, exclusions éventuelles)

Les chiffres ne sont donc pas contradictoires, mais correspondent à deux périmètres d'analyse distincts.

## Analyse structurelle

### 1) Toutes les classes sont-elles dans le même paquetage ?

Non. Même si un paquetage concentre 18 classes sur 27 dans le périmètre mesuré (soit environ deux tiers), les classes restent réparties sur 5 paquetages distincts.

La structure est donc centralisée, mais pas monolithique.

### 2) Les paquetages non-feuilles contiennent-ils des classes ?

Oui, dans certains cas. J'ai identifié des paquetages purement structurels (comme com, com.google) qui ne contiennent aucune classe directement. Ils servent uniquement à structurer l'arborescence.

Cependant, certains paquetages qui possèdent des sous-paquetages contiennent également des classes. La présence de classes ne dépend donc pas uniquement du fait qu'un paquetage soit feuille ou non.

### 3) Hiérarchies parallèles (main / test)

Comparaison :

- 16 paquetages côté main
- 20 côté test
- 12 communs
- 8 spécifiques aux tests
- 4 spécifiques au main

Les tests suivent majoritairement la hiérarchie des sources, ce qui est une bonne pratique.

Les paquetages supplémentaires côté test (functional, integration, etc.) correspondent à des besoins spécifiques et ne traduisent pas un désordre architectural.

## 3.3.1 Analyse du couplage et de la cohésion

Pour analyser le couplage, je me suis appuyé sur les résultats de jdeps, qui ont permis d'identifier 83 dépendances inter-paquetages.

Je me concentre sur trois paquetages clés :

- com.google.gson
- com.google.gson.internal
- com.google.gson.internal.bind

### Analyse du couplage

**com.google.gson.internal.bind**

18 dépendances sortantes

4 dépendances entrantes

C'est le paquetage le plus dépendant du projet. Il utilise beaucoup d'autres composants internes.
Son couplage sortant est élevé, mais cohérent avec son rôle central de binding Java ↔ JSON.

**com.google.gson.internal**

16 sortantes

5 entrantes

Ce paquetage est fortement connecté, à la fois en entrée et en sortie.
Il constitue clairement un noyau interne du projet.

**com.google.gson**

15 sortantes

5 entrantes

L'API publique dépend logiquement des mécanismes internes.
Son couplage est important mais justifié par son rôle central.

### Analyse de la cohésion

**com.google.gson**

Contient l'API publique. Les classes partagent une responsabilité commune : manipulation et sérialisation JSON.
Cohésion satisfaisante.

**com.google.gson.internal**

Regroupe les mécanismes internes (réflexion, gestion des types, utilitaires).
Même si le paquetage est dense, les classes restent orientées vers une responsabilité technique claire. Cohésion correcte.

**com.google.gson.internal.bind**

Regroupe les adaptateurs et le mécanisme de conversion objet ↔ JSON.
Malgré son fort couplage sortant, la responsabilité reste identifiable. Cohésion satisfaisante.

## Conclusion générale

La répartition des classes est centralisée autour d'un noyau fort.

Le projet comporte un couplage relativement élevé entre certains paquetages centraux.

Toutefois, ce couplage reste cohérent avec l'architecture d'une bibliothèque technique.

La cohésion des paquetages étudiés est bonne : chaque zone possède une responsabilité identifiable.

On observe donc une architecture structurée autour d'un cœur interne solide, sans dispersion anarchique des responsabilités.
