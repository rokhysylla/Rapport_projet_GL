# Code Smell : Nombres magiques

Ce document analyse les nombres magiques présents dans les fichiers TypeAdapters.java et GsonTypes.java du projet Gson désigné par sonarQube comme ayant les plus de smell code, en identifiant leur intention et en proposant des refactorisations pour améliorer la lisibilité du code en accord avec les principes du Clean Code.


# Vous trouvererez  le chemin complet de chaque classe analysée du projet ici : 
1. `gson/src/main/java/com/google/gson/internal/bind/TypeAdapters.java`
2. `gson/src/main/java/com/google/gson/internal/LinkedTreeMap.java`
3. `gson/src/main/java/com/google/gson/internal/GsonTypes.java`
4. `metrics/src/main/java/com/google/gson/metrics/ParseBenchmark.java`

## 5.2 Nombres magiques - Analyse de TypeAdapters.java

### Problème identifié

En balayant TypeAdapters, je vois plusieurs valeurs numériques "en dur" qui peuvent être considérées comme des nombres magiques, même si elles sont souvent justifiées par le domaine JSON ou par des contraintes de type. Par exemple :
- Dans BIT_SET, les valeurs `0` et `1` servent à représenter l'absence ou la présence d'un bit
- `255` pour le byte et `65535` pour le short, utilisés pour autoriser des valeurs non signées

### Analyse de l'intention

Ces valeurs correspondent à des bornes techniques :
- `0`/`1` pour un bit
- `255` pour un octet non signé
- `65535` pour un "unsigned short"

Le problème : l'intention est surtout portée par un commentaire ("Allow up to …"), pas par le code lui-même. Dans l'esprit Clean Code, ce serait plus lisible si le nom expliquait la valeur.

### Points à améliorer

Remplacer ces valeurs par des constantes privées explicites :
```java
BITSET_FALSE = 0
BITSET_TRUE = 1
UNSIGNED_BYTE_MAX = 255
UNSIGNED_SHORT_MAX = 65535
```

Cela permet aussi d'aligner le code et les tests : au lieu d'assert sur `255`, on assert sur `UNSIGNED_BYTE_MAX`, ce qui rend les tests plus parlants et plus robustes si la règle change.

## 5.3 Nombres magiques - Analyse de GsonTypes.java

### Problème identifié

En parcourant GsonTypes, je repère quelques valeurs numériques "en dur" qui peuvent entrer dans la catégorie nombres magiques. On y trouve `1` utilisé dans des conditions sur la taille des bornes (`bounds.length == 1`, `lowerBounds.length == 1`, `upperBounds.length != 1`), et `30` dans `new StringBuilder(30 * (length + 1))` qui est une optimisation de capacité initiale. On retrouve aussi `31` dans le calcul de hashCode du WildcardTypeImpl, ainsi que `serialVersionUID = 0` dans les classes internes.

### Analyse de l'intention

Deux catégories émergent. D'un côté, les valeurs comme `0` ou `1` sont liées à la logique des APIs Java Reflection : un wildcard "supporté" doit avoir au plus une borne (donc `length == 1`), et la taille `0` sert juste à construire une instance de tableau vide pour récupérer une Class sans allouer un vrai tableau rempli. De l'autre côté, `30` (StringBuilder) et `31` (hashCode) sont des heuristiques/constantes de performance assez typiques, mais restent arbitraires pour quelqu'un qui lit vite le code.

### Points à améliorer

Remplacer seulement les valeurs qui portent une intention "de règle", pas celles qui sont purement structurelles :
```java
private static final int SINGLE_BOUND = 1;
private static final int HASH_PRIME = 31;
private static final int ESTIMATED_CHARS_PER_TYPE_ARGUMENT = 30;
```

Cela rendrait plus explicite pourquoi on compare à `1` ou pourquoi on ajoute `31` dans le hash, et faciliterait la maintenance.

## 5.4 Nombres magiques - Analyse de LinkedTreeMap.java

### Problème identifié

En parcourant LinkedTreeMap, je remarque plusieurs nombres "en dur" qui attirent l'attention. Par exemple, dans l'algorithme d'équilibrage AVL (rebalance), on retrouve des comparaisons comme delta == -2, delta == 2, rightDelta == -1, leftDelta == 1, etc. Ces valeurs sont clairement liées aux règles de l'arbre AVL (déséquilibre de hauteur de 2), mais elles apparaissent directement dans le code sans être nommées. De même, on voit des initialisations comme int leftHeight = 0;, int rightHeight = 0;, ou encore this.height = 1; lors de la création d'un nœud.

### Analyse de l'intention

En essayant de comprendre l'intention, ces nombres ne sont pas arbitraires : 2 représente le seuil de déséquilibre d'un AVL, 1 correspond à la hauteur minimale d'un nœud feuille, et 0 à une hauteur nulle pour un sous-arbre absent. Le problème, c'est que cette signification n'est pas portée par le nom du code mais par la connaissance implicite de l'algorithme. Quelqu'un qui ne connaît pas bien les AVL pourrait se demander "pourquoi 2 et pas 3 ?".

### Points à améliorer

Pour améliorer la clarté, je proposerais d'introduire des constantes privées comme :
```java
private static final int AVL_IMBALANCE_THRESHOLD = 2;
private static final int INITIAL_HEIGHT = 1;
private static final int EMPTY_HEIGHT = 0;
```

Cela rendrait les conditions plus parlantes (if (delta == -AVL_IMBALANCE_THRESHOLD)) et expliciterait l'intention algorithmique. En revanche, je ne remplacerais pas tous les 0 et 1 utilisés dans des contextes évidents (indices de boucle, tailles, comparaisons triviales), car cela risquerait de surcharger le code inutilement. Ici, le vrai enjeu concerne surtout les constantes liées aux propriétés mathématiques de l'AVL, pas les valeurs techniques standard.


### 5.2 Nombres magiques - ParseBenchmark.java

Dans ParseBenchmark, je repère plusieurs nombres magiques assez visibles. Par exemple, dans resourceToString, on a new char[8192] pour le buffer de lecture : 8192 est une taille classique, mais rien dans le code n’explique pourquoi ce choix a été fait. De la même manière, dans JacksonStreamParser, la variable depth est incrémentée et décrémentée avec depth++ / depth--, et la boucle s’arrête avec while (depth > 0) : le 0 représente ici la profondeur racine, mais ce n’est pas nommé. Dans ParameterizedTypeImpl.toString() (dans d’autres fichiers c’était déjà le cas), on voit parfois des constantes numériques pour la capacité initiale d’un StringBuilder, et ici dans Feed.toString() on utilise int i = 1 pour numéroter les items sans constante symbolique.

Selon le cours, ces nombres devraient être remplacés quand ils portent une intention métier ou technique. Ici, je proposerais par exemple private static final int BUFFER_SIZE = 8192; pour expliciter le choix du buffer. Pour la profondeur JSON, on pourrait introduire private static final int ROOT_DEPTH = 0; pour rendre la condition plus claire. En revanche, les 0 ou 1 dans des boucles simples sont acceptables et ne justifient pas forcément une constante. Conclusion : il y a des nombres magiques identifiables, surtout le 8192, et ils peuvent être améliorés pour rendre l’intention plus explicite.

