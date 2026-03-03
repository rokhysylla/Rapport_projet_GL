# 3.2 Organisation en paquetages

## 3.2.1 Nombre total de paquetages

On a identifié 25 paquetages Java distincts dans le projet.

Pour obtenir ce résultat, on a parcouru tous les fichiers *.java et extrait les déclarations package ...;. Ensuite, on a compté le nombre de noms distincts. L'objectif était d'avoir une vision globale de la granularité du découpage architectural.

Cela montre que le projet est relativement bien segmenté, sans être éclaté en une multitude excessive de sous-parties.

## 3.2.2 Analyse des liens entre les paquetages

### Méthode utilisée

Plutôt que d'analyser uniquement les import dans le code source, on a utilisé l'outil jdeps sur le JAR compilé. Cette méthode est plus fiable, car elle révèle les dépendances réellement présentes après compilation.

À partir des résultats, on a identifié 83 liaisons entre paquetages, ce qui permet de visualiser le niveau d'interconnexion.

### Qui dépend de qui ?

On distingue deux catégories importantes.

#### Les paquetages les plus dépendants (beaucoup de liens sortants)

- **com.google.gson.internal.bind** : 18 dépendances sortantes.
- **com.google.gson** : 15 dépendances sortantes.

Cela s'explique logiquement :
- internal.bind gère le binding objet ↔ JSON, donc il interagit avec beaucoup d'éléments.
- com.google.gson constitue le cœur de l'API.

#### Les paquetages les plus sollicités (beaucoup de liens entrants)

- com.google.gson
- com.google.gson.internal

Ce sont les fondations du projet. Les autres modules s'appuient largement dessus.

### Organisation en couches

Sur le plan théorique, le projet est structuré en deux grandes zones :

- L'API publique (com.google.gson)
- L'implémentation interne (internal.*)

Cependant, l'analyse montre que l'API publique dépend directement de la partie interne. On n'est donc pas dans une architecture strictement stratifiée.

On peut parler d'architecture semi-stratifiée : la séparation existe, mais les couches ne sont pas totalement indépendantes.

### Dépendances cycliques

On observe des cycles, notamment entre :

- com.google.gson
- internal.bind

Cela signifie que ces deux zones dépendent mutuellement.

Selon le cours, cela traduit :

- un couplage fort,
- une maintenabilité plus fragile,
- une difficulté potentielle à isoler certaines parties.

Cependant, ces cycles restent concentrés au noyau du projet et ne forment pas une structure anarchique.

## 3.2.3 Analyse de la hiérarchie des paquetages

### Profondeur des niveaux

En analysant les déclarations package, on obtient :

- Minimum : 2 niveaux
- Maximum : 6 niveaux
- Moyenne : environ 3,8 niveaux

Exemple typique : `com.google.gson.internal.bind`

La profondeur maximale reste raisonnable. On n'observe pas de sur-segmentation excessive.

### Alignement tests / sources

Comparaison entre src/main/java et src/test/java :

- 16 packages côté main
- 20 packages côté test
- 12 packages communs
- 8 spécifiques aux tests
- 4 du main sans équivalent test

La majorité des tests suivent la hiérarchie des sources, ce qui est une bonne pratique.

Les packages supplémentaires côté test (ex : functional, integration, native_test, jpms_test) correspondent à des tests spécialisés. Ce n'est pas un désordre, mais une extension logique.

### Paquetages conteneurs sans classes

Trois cas détectés :

- com
- com.google
- com.google.gson.extras.examples

Ces packages servent uniquement de conteneurs hiérarchiques. Ce n'est pas un problème architectural.

## 3.2.4 Analyse des noms de paquetages

### Architecture MVC ?

On ne retrouve aucun nom du type :

- controller
- service
- repository
- model
- view

Conclusion : le projet ne suit pas un modèle MVC, ce qui est cohérent. Gson est une bibliothèque technique, pas une application web.

### Lien avec une base de données ?

Aucun paquetage du type :

- dao
- persistence
- jdbc
- database

Le paquetage internal.sql ne correspond pas à une couche d'accès aux données, mais à la gestion de types SQL pour la sérialisation.

Il n'y a donc pas d'architecture orientée base de données.

### Ce que révèlent les noms

- **internal** : Cela montre une séparation claire entre API publique et implémentation interne. C'est conforme au principe d'encapsulation.
- **bind** : Indique un mécanisme de binding objet ↔ JSON. Cela suggère des patterns de type Adapter ou Strategy.
- **reflect** : Indique l'usage intensif de la réflexion Java.
- **stream** : Montre l'existence d'une API bas niveau orientée flux.
- **annotations** : Indique une configuration déclarative via annotations personnalisées.

### Synthèse

Les paquetages montrent :

- Une architecture orientée bibliothèque technique.
- Une séparation API / implémentation interne.
- Une organisation par responsabilités fonctionnelles.
- Une utilisation forte de réflexion, binding et annotations.

Malgré quelques cycles internes, la structure reste globalement cohérente et maîtrisée.
