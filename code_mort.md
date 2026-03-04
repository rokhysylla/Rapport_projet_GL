## 5.4 Introduction - Code mort

L'objectif est d'identifier les portions de code non exécutées ou rarement utilisées, d'évaluer leur nécessité et de proposer des améliorations en matière de couverture de test. Bien que certaines branches conditionnelles restent peu testées, la plupart du code identifié n'est pas véritablement "mort" mais plutôt destiné à gérer des cas limites, des erreurs ou des configurations spécifiques.



### 5.4 Code mort - TypeAdapters.java
L'une des classes avec le plus de code smells selon sonarQube(10), il m'a paru légitime de regarder s'il n'y avait pas de code smells dedans.
Cette classe se trouve exactement :  
1. `gson/src/main/java/com/google/gson/internal/bind/TypeAdapters.java`



Dans TypeAdapters, je ne vois pas de code mort évident du type "méthode jamais appelée", parce que c'est surtout une classe utilitaire avec des constantes publiques. Comme c'est une bibliothèque, c'est normal que certains adapters ne soient pas utilisés par le module Gson lui-même mais par des modules externes.

Par contre, je peux identifier des morceaux de code qui ne servent qu'à gérer des cas rares et sont non couverts par des tests. Par exemple, CLASS.read() et CLASS.write() ne font que lever une exception, donc ce code n'est exécuté que si quelqu'un tente de sérialiser/désérialiser Class. De la même manière, toutes les branches qui lancent des JsonSyntaxException (valeurs invalides, token inattendu) sont du "code d'erreur" qui peut rester non appelé dans un usage standard. Vu nos observations précédentes sur la couverture, ce genre de branches est typiquement peu testé. 
Je ne pense pas qu'il faille supprimer ces blocs : ils sont là pour sécuriser et guider l'utilisateur (messages d'erreur + troubleshooting). En revanche, on devrait mettre en oeuvre des tests unitaires ciblés "entrées invalides", justement pour éviter que ces chemins restent en rouge dans la couverture.

Bref , pas du code mort à supprimer, plutôt du code rarement exécuté mais utile et qu'il faudrait mieux tester.



### 5.4 Code mort - GsonTypes.java
Vous trouverez ce fichier dans ce chemin: 
2. `gson/src/main/java/com/google/gson/internal/GsonTypes.java`
Je soupçonnais du code code mort dans cette classe à cause de son score sur sonarQube : 14, le plus élevé du package internal, l'un des plus élevés du paquetage gson.internal

Dans GsonTypes, là non plus, Il n'y a pas de code mort qui saute aux yeux, parce que la classe regroupe des méthodes utilitaires appelées par différentes parties de Gson (notamment sur la réflexion et la résolution de types). Comme c’est une API interne, certaines méthodes peuvent aussi être peu utilisées selon les cas rencontrés, mais elles restent “appelables” et donc pas mortes par nature. Par contre, je vois plusieurs branches qui ne s’exécutent que sur des cas particuliers : les gros if/else avec instanceof dans canonicalize, getRawType et surtout resolve. Les exceptions (IllegalArgumentException, NoSuchElementException) sont aussi des chemins qui ne tournent que si on passe des types incohérents ou des situations limites. Si on constate une couverture partielle, c’est logique que ces branches rares soient peu testées. Je ne supprimerais pas ce code, parce qu’il sert à gérer correctement tous les types possibles en Java (ParameterizedType, WildcardType, GenericArrayType…).



### 5.4 Code mort - LinkedTreeMap.java
Cette classe a un score de 14 en code smells sur sonarQube il m'a paru pertinent de regarder s'il y avait du code mort.
Vous trouverez ce fichier dans ce chemin : 
3. `gson/src/main/java/com/google/gson/internal/LinkedTreeMap.java`

Dans LinkedTreeMap, je ne vois pas de code mort évident, parce que la classe implémente une structure de données complète et ses méthodes s’appellent entre elles (insertion, suppression, rotations, itérateurs). Les méthodes publiques (put, get, remove, clear, entrySet, keySet) sont clairement destinées à être utilisées, donc elles ne sont pas “mortes”. Par contre, certaines branches ne s’exécutent que dans des situations précises : par exemple les cas de rééquilibrage AVL dans rebalance (delta == -2 ou delta == 2) ne se déclenchent que si l’arbre devient déséquilibré. Les assert et certaines exceptions (NoSuchElementException, IllegalArgumentException) sont aussi des chemins conditionnels. SonarQube signale ces zones non couvertes par des tests, il faudrait des tests qui provoquent ces cas. 


### 5.4 Code mort - ParseBenchmark.java(code smells:51)
Vous trouverez ce fichier dans ce chemin : 
4. `metrics/src/main/java/com/google/gson/metrics/ParseBenchmark.java` 
Parce que sonarQube indique 51 comme score de code smells à ce fichier, qui est le score le plus élvé que j'ai trouvé , je l'ai scruté   pour voir s'il y avait du code mort dedans et voici mon analyse :

Dans ParseBenchmark, je ne peux pas affirmer qu’il y a du code mort sans analyser la couverture ou l’exécution réelle du module metrics dans sa globalité. Toutes les classes et méthodes sont appelables : setUp, timeParse, les différentes implémentations de Parser, ainsi que les classes modèles (Tweet, User, Feed, etc.) sont utilisées dans le cadre du benchmark. Comme le fichier est situé dans le module metrics, il a une responsabilité claire et cohérente.

Selon la définition du cours, le code mort est du code jamais appelé. Ici, je ne vois pas de méthode privée isolée ou de branche impossible à atteindre (pas de if(false) ou de méthode non référencée). Les exceptions levées dans certains switch ou méthodes utilitaires sont des chemins conditionnels, mais ils restent atteignables.

Conclusion : je ne peux pas identifier de code mort certain dans ce fichier. Les smells signalés par Sonar ne correspondent pas à du code non appelé, mais probablement à d’autres problèmes de qualité (complexité, longueur, duplication, etc.).