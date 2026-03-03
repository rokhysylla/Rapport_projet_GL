# 4.6 – Analyse des méthodes

Pour analyser la complexité des méthodes, je me suis concentré sur les classes présentant les plus fortes complexités cyclomatiques globales selon SonarQube :

- JsonReader.java (module gson)
- ParseBenchmark.java (module metrics)
- UtcDateTypeAdapter.java (module extras)

Plutôt que de calculer des statistiques globales (min, max, moyenne), j'ai préféré étudier une méthode représentative dans chacune de ces classes afin d'identifier l'origine concrète de la complexité.

## 1) JsonReader.java - Méthode doPeek()

La méthode doPeek() présente une structure très riche en branches :
- nombreux if / else if
- cas non terminés par break
- retours conditionnels multiples
- gestion d'états internes via JsonScope

Cette méthode joue le rôle d'automate de parsing du flux JSON. Elle doit gérer tous les cas possibles : tableaux, objets, noms, valeurs, fin de document, erreurs de syntaxe, 

La complexité cyclomatique élevée est donc en grande partie justifiée par la nature du problème : un parseur implique nécessairement de nombreux chemins d'exécution.

Cependant, SonarQube indique que plusieurs branches de cette méthode ne sont pas couvertes par les tests (chemins en rouge). Cela signifie que certains cas conditionnels ne sont jamais exécutés dans la suite de tests actuelle. Cela pose un risque : une méthode très complexe dont tous les chemins ne sont pas testés augmente la probabilité d'erreurs non détectées.

## 2) UtcDateTypeAdapter.java - Méthode parse(...)

La méthode parse contient :
- plusieurs niveaux d'imbrication conditionnelle
- gestion manuelle du parsing (année, mois, jour, heure, fuseau)
- gestion d'exceptions multiples
- validations intermédiaires

La complexité vient ici du fait que la méthode concentre plusieurs responsabilités : extraction des composants, validation du format, gestion des fuseaux horaires et création de l'objet Date.

Comme pour doPeek(), SonarQube montre que certaines branches ne sont pas couvertes par les tests. Cela signifie que certains cas d'erreur ou formats particuliers ne sont probablement jamais exercés. Dans ce cas, la forte complexité combinée à une couverture partielle constitue un point faible en termes de maintenabilité et de robustesse.

## 3) ParseBenchmark.java - Méthode readToken(...)

La méthode readToken repose sur :
- une boucle while(true)
- un switch multi-cas
- une gestion répétée des différents types de tokens

La structure reste relativement claire, mais le nombre de cas gérés augmente mécaniquement la complexité cyclomatique. Là encore, certaines branches apparaissent comme non couvertes. Même si cette classe est dédiée aux benchmarks et non au cœur métier, cela montre que les scénarios testés ne couvrent pas l'ensemble des comportements possibles. Une refactorisation ou l'application de design patterns serait nécessaire ici.

## Conclusion

L'analyse montre que :
- Les méthodes les plus complexes se trouvent dans les classes liées au parsing JSON() (concentrées plus dans le module gson)
- Leur complexité est en partie fonctionnelle et inhérente au problème traité
- Cependant, ces méthodes présentent de nombreux chemins conditionnels
- Plusieurs de ces chemins ne sont pas couverts par les tests
- La combinaison forte complexité + couverture partielle constitue un risque de défauts non détectés et complique la maintenance


Même si la complexité est parfois justifiée, une meilleure décomposition en sous-méthodes et un renforcement de la couverture de tests permettraient d'améliorer significativement la qualité structurelle du code.


## 4) Analyse des commentaires

Les commentaires ne sont pas tous bien placés. Les méthodes avec une plus grande complexité cyclomatique ne sont pas systématiquement commentées.

### Observations par classe

**ParseBenchmark.java** (chemin : `src/java/main/com/google/gson/metrics`)
- Les méthodes `readToken()` et `parse()` (classe statique `JackSonStreamParser`) ne sont pas commentées du tout, alors que ce sont des méthodes clés.

**UtcDateTypeAdapter.java** (chemin : `src/java/main/com/google/gson/typeadapters/UtcDateTypeAdapter.java`)
- La méthode `parse()` est bien commentée avec les normes Java (`@return`, `@param`, etc.).

**JsonReader.java** (chemin : `src/java/main/com/google/gson/gson/JsonReader.java`)
- La méthode `doPeek()` n'est pas commentée, alors que c'est une fonction pivot appelée par plusieurs autres méthodes.
- La méthode `peekNumber()`, aussi complexe, n'est pas commentée du tout.

## 5) Analyse détaillée des trois méthodes

### Méthodes avec beaucoup d'arguments

Les trois méthodes analysées ne présentent pas un nombre élevé de paramètres :
- `UtcDateTypeAdapter.parse(String date, ParsePosition pos)` : deux paramètres
- `ParseBenchmark.readToken(JsonReader reader)` : un paramètre
- `JsonReader.doPeek()` : aucun paramètre

Aucune ne constitue un problème de conception à ce niveau.

### Méthodes qui modifient l'état et retournent une information

Deux méthodes correspondent à ce critère.

**JsonReader.doPeek()**
- Modifie l'état interne du `JsonReader` : `stack[stackSize - 1] = JsonScope.NONEMPTY_ARRAY;`, `peeked = PEEKED_END_ARRAY;`, etc.
- Retourne une information : `return peeked;`
- Elle combine modification d'état et retour d'information.

**UtcDateTypeAdapter.parse(String date, ParsePosition pos)**
- Modifie un objet paramètre : `pos.setIndex(offset);`
- Retourne une valeur : `return calendar.getTime();`
- Elle correspond pleinement au critère.

**ParseBenchmark.readToken(JsonReader reader)**
- Modifie l'état du `JsonReader` via `reader.beginArray();`, `reader.endArray();`, etc.
- Retourne `void` : ne retourne pas d'information.
- Elle ne correspond pas complètement au critère.

### Méthodes qui retournent un code d'erreur

Aucune des trois méthodes ne retourne explicitement un code d'erreur. La gestion des erreurs repose sur des exceptions :
- `throw syntaxError("Unterminated array");`
- `throw new ParseException("Failed to parse date ...", pos.getIndex());`

Le projet privilégie les exceptions plutôt que les codes d'erreur explicites.

## Conclusion

- Aucune méthode n'a trop d'arguments.
- Deux méthodes (`doPeek` et `parse`) modifient l'état d'un objet tout en retournant une information.
- Aucune ne retourne explicitement un code d'erreur ; les erreurs sont gérées via des exceptions.
-Les commentaires sont inexistants pour  readToken,parse et doPeek().
La complexité provient davantage de la logique de parsing et de gestion des cas particuliers que d'un mauvais usage des paramètres ou des codes d'erreur.
