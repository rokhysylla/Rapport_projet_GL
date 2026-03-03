## Introduction

Cette analyse se concentre sur les conventions de nommage dans trois classes identifiées par SonarQube comme ayant des problèmes significatifs de code smell :

- **TypeAdapters** : Classée comme classe Large/God
- **GsonTypes** : Contient 13 code smells identifiés
- **LinkedTreeMap** : Contient 14 code smells identifiés

Le critère des règles de nommage a été sélectionné pour évaluer ces classes et identifier les opportunités d'amélioration de la lisibilité et de la maintenabilité du code.


### 5.1 Règles de nommage - Analyse de TypeAdapters.java
chemin complet de cette classe : 
gson/src/main/java/com/google/gson/internal/bind/TypeAdapters.java

Dans ce fichier, je constate d’abord que la majorité des noms respectent les principes du Clean Code. Les noms sont globalement descriptifs et en lien direct avec le domaine Gson : BOOLEAN, INTEGER, BIG_DECIMAL, LOCALE, JSON_ELEMENT_FACTORY, etc. On comprend immédiatement à quel type chaque adapter correspond. Les méthodes comme checkValidFloatingPoint, newTypeHierarchyFactory ou atomicLongArrayAdapter sont claires et révèlent bien leur intention.

Je remarque aussi que les noms des variables locales sont cohérents et compréhensibles : tokenType, intValue, requestedType, fields, values. Ils ne créent pas de désinformation sur les structures de données. Par exemple, une variable nommée list est bien une List, et array correspond bien à un tableau. Cela respecte la règle d’éviter les noms trompeurs.

Cependant, je note un point perfectible : certaines constantes portent exactement le même nom que les types Java correspondants, comme URL, URI, UUID. Même si c’est logique dans le contexte, cela peut créer une ambiguïté à la lecture entre le type et l’adapter. Un nom comme URL_ADAPTER serait plus explicite.

De plus, les paramètres génériques comme <TT>, <T1>, <T2> sont courts et standards en Java, mais dans un fichier aussi volumineux, ils peuvent compliquer la compréhension rapide du code.

En conclusion, les règles de nommage sont globalement bien respectées : les noms sont descriptifs, prononçables et cohérents avec le domaine. Les améliorations possibles concernent surtout la réduction d’ambiguïtés et l’augmentation de la clarté dans un fichier très dense.




### 5.1 Règles de nommage - Analyse de LinkedTreeMap.java
Chemin complet de la classe : gson/src/main/java/com/google/gson/internal/LinkedTreeMap.java

Dans ce fichier, je trouve que les noms sont globalement propres et révélateurs d’intention. La classe LinkedTreeMap décrit bien le concept : une structure de type “tree map” avec un chaînage pour l’ordre d’insertion. Les noms du domaine sont bien utilisés (comparator, entrySet, keySet, Iterator, Node, rebalance, rotateLeft, rotateRight) et ils sont prononçables, donc faciles à discuter en équipe.

Les variables importantes sont plutôt bien nommées et non ambiguës : allowNullValues, modCount, expectedModCount, lastReturned, nearest, comparison, unbalanced. Je ne vois pas de désinformation sur les structures de données : quand c’est un Set, ça s’appelle entrySet/keySet, quand c’est un nœud c’est Node, quand c’est un comparateur c’est comparator. Les méthodes suivent une logique verbale claire (findByObject, removeInternalByKey, replaceInParent, nextNode), ce qui aide à lire le code “de haut en bas”.

Les points perfectibles concernent surtout certains noms un peu trop génériques ou ambigus localement. Par exemple left, right, child, node, e sont classiques en structure de données, mais dans un fichier long, ça force parfois à relire pour savoir de quel niveau on parle. De même, EntrySet et KeySet sont corrects, mais le fait que EntrySet ne soit pas final alors que KeySet l’est peut donner une impression de convention pas complètement homogène (même si ce n’est pas un problème fonctionnel). Enfin, NATURAL_ORDER est très clair, mais le type Comparator <Comparable> peut faire bizarre au premier regard ; le nom reste bon, c’est surtout la signature qui surprend.

En conclusion, sur les règles de nommage, je dirais que ce fichier est plutôt solide : les noms sont cohérents, orientés domaine, et rarement trompeurs. Les améliorations possibles sont surtout des micro-ajustements pour réduire la charge cognitive dans les méthodes d’algorithmes (renommer e en current par exemple, ou préciser certains node en currentNode quand le contexte devient dense).


### 5.1 Règles de nommage - Analyse de GsonTypes.java
Chemin complet de la classe : gson/src/main/java/com/google/gson/internal/GsonTypes.java

Dans cette classe, je trouve que le nommage est globalement très correct et cohérent avec le domaine “reflection / types Java”. La classe GsonTypes annonce clairement son rôle (outils utilitaires sur Type), et les méthodes principales sont bien nommées avec des verbes explicites : canonicalize, getRawType, typeToString, getCollectionElementType, getMapKeyAndValueTypes, resolve. Les noms utilisés sont prononçables et plutôt “cherchables”, ce qui aide à naviguer dans un fichier long.

Je remarque aussi que les structures internes sont nommées de façon descriptive : ParameterizedTypeImpl, GenericArrayTypeImpl, WildcardTypeImpl. Même si c’est un peu verbeux, ça reste sans ambiguïté et on comprend directement ce que chaque implémentation représente. Les variables importantes sont claires (context, contextRawType, toResolve, visitedTypeVariables, upperBounds, lowerBounds), et je ne vois pas de désinformation sur les structures de données : un Map s’appelle visitedTypeVariables, un tableau s’appelle typeArguments ou EMPTY_TYPE_ARRAY, etc.

Là où je comprends que Sonar puisse relever des smells, c’est plutôt sur des noms locaux très courts dans certaines boucles ou méthodes d’égalité : a, b, pa, pb, ga, gb, wa, wb, va, vb. Ces noms sont classiques, mais dans un code déjà complexe (beaucoup de cas instanceof), ils rendent la lecture plus coûteuse et moins “révélatrice d’intention”. Pareil pour t utilisé comme index de boucle : ce n’est pas faux, mais dans un contexte générique, argIndex ou typeArgIndex serait plus parlant.

En conclusion, sur les règles de nommage, GsonTypes est plutôt solide : les noms de méthodes et de classes sont précis, liés au domaine, et globalement non ambigus. Les améliorations possibles sont surtout des micro-renommages de variables locales très courtes dans les sections critiques (comparaison/resolve) pour réduire l’effort de compréhension, ce qui colle directement à l’idée “un nom doit aider la lecture” du Clean Code.




### 5.1 Règles de nommage - ParseBenchmark.java

Dans ParseBenchmark, les noms principaux sont clairs et dans le bon domaine : ParseBenchmark, Document, Api, Parser, setUp, timeParse, resourceToString. Les enums Document et Api sont bien nommées et donnent tout de suite l’idée des scénarios de benchmark (type de données + API testée). Les classes internes GsonStreamParser, GsonSkipParser, GsonDomParser, GsonBindParser, JacksonStreamParser, JacksonBindParser sont très explicites : je sais immédiatement ce qu’elles mesurent. Là où le nommage est moins “clean code”, c’est dans GsonStreamParser.readToken : des variables comme unusedName, unusedBoolean, unusedLong, unusedString indiquent volontairement qu’on jette les valeurs, mais ça fait quand même du bruit et ça alourdit la lecture. Même chose pour des noms ultra-génériques comme jp, i, count, token, data : dans un fichier déjà long, ça rend certaines parties moins lisibles. Les classes Tweet et User contiennent beaucoup de champs en snake_case (created_at, id_str, etc.) : c’est assumé pour coller au JSON, mais ça casse la convention Java et ça oblige à mettre @SuppressWarnings("MemberName"). Enfin, Parser.parse(char[] data, Document document) est correct, mais data pourrait être nommé jsonChars pour être plus précis. Conclusion : l’intention des noms est globalement bonne, mais il y a des zones “benchmark” et “mapping JSON” où le nommage est volontairement moins propre, ce qui explique qu’un outil comme Sonar réagisse.






